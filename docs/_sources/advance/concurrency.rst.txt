************************
共時性
************************

隨著物理條件的限制，現在的 CPU 不再以衝高時脈為目標，改以多核心 (multi-core) 的模式來\
生産。然而，若我們在實作程式時，沒有修改程式碼，以\ **共時性 (concurrency)** 或\
**平行運算 (parallel computing)** 的方式執行程式，就無法真正享受多核心 CPU 帶來的益處。\
Rust 將安全的理念也放入共時性程式中，讓程式設計者避開共時性程式常見的問題。

==============================
建立執行緒
==============================

Rust 在標準函式庫中提供程式設計者建立\ **執行緒 (thread)** 的功能，實例如下：

.. code-block:: rust

   use std::thread;

   fn main() {
       // Spawn a new thread
       let thread = thread::spawn(move || {
           println!("Hello World from thread");
       });

       // Wait the thread ending
       let _ = thread.join().unwrap();
   }

``thread::spawn`` 接收一個沒有參數的 Closure，故寫成 ``||``\ 。在建立執行緒後，要在\
主程式等其他執行緒結束，否則會在執行緒完成前就提早結束程式。因為所有權的關係，這裡要使用 \
``move`` 關鍵字。

如果要同時啟動多個執行緒，範例如下：

.. code-block:: rust

   use std::thread;

   static NTHREAD: usize = 10;

   fn main() {
       let mut threads = Vec::new();

       // Spawn 10 threads
       for i in 0..NTHREAD {
           threads.push(thread::spawn(move || {
               println!("Hello World from thread {}", i);
           }));
       }

       // Wait all threads ending
       for thread in threads {
           let _ = thread.join().unwrap();
       }
   }

讀者可試著執行此程式，會發現每次的順序都不同，在撰寫共時性程式時需注意。

=====================================
在執行緒間共享資料
=====================================

我們以一個反例來說明在執行緒間共享資料時會發生的問題。以下是一個使用多執行緒的 C++ 程式：

.. code-block:: c++

   #include <thread>
   #include <exception>
   #include <vector>
   #include <iostream>

   using std::thread;
   using std::vector;
   using std::exception;
   using std::cout;
   using std::cerr;
   using std::endl;

   // Function used in thread
   void add_n(int *x, int n) {
        *x += n;
   }

   int main() {
       int n = 0;
       int *n_ptr = &n;

       try {
           vector<thread*> threads{};

           for (int i = 1; i <= 10; i++) {
               // Initial a thread
               thread t(add_n, n_ptr, i);

               // Detach the thread into background
               t.detach();

               // Push (the pointer of) the thread into the vector
               threads.push_back(&t);
           }

           for (auto &t: threads) {
               if (t->joinable()) {
                   // Wait the thread ending
                   t->join();
               }
           }

           cout << *n_ptr << endl;
       } catch (const exception &e) {
          cerr << "EXCEPTION: " << e.what() << endl;
       }

       return 0;
   }

以 Linux 上的 GCC 為例，編譯過程如下：

.. code-block:: console

   $ g++ -o thread_demo thread_demo.cpp -std=c++11 -lpthread

理論上，\ ``n`` 的值應為 55，實際上，每次執行該程式時，會發現值都不同。這是因為有數個\
執行緒在競相使用 ``n``\ ，造成其中某些執行緒未正確執行。

我們將上述程式以 Rust 重寫如下：

.. code-block:: rust

   use std::thread;

   static NTHREAD: usize = 10;

   fn main() {
       let mut threads = Vec::new();

       let mut x = 0;

       for i in 1..(NTHREAD+1) {
           threads.push(thread::spawn(move || {
               x += i;
           }));
       }

       for thread in threads {
           let _ = thread.join().unwrap();
       }

       assert_eq!(x, 55);
   }

實際執行該程式時，引發以下錯誤：

.. code-block:: console

   thread 'main' panicked at 'assertion failed: `(left == right)` (left: `0`, right: `55`)'

不論程式執行幾次，\ ``x`` 的值都是 0，因為 Rust 認為這樣的程式會對 ``x`` 的值造成損害，\
故不更動 ``x`` 的值，這是 Rust 在安全上的考量。

若要在多執行緒程式中分享數據，要使用 mutex 物件，mutex 可以防止執行緒對資料的誤寫。而\
為了要有 thread-safe 且 sharable 的 mutex 物件，要用 Arc 容器。範例如下：

.. code-block:: rust

   use std::thread;
   use std::sync::{Arc, Mutex};

   static NTHREAD: usize = 10;

   fn main() {
       let mut threads = Vec::new();

       let x = 0;

       // A thread-safe, sharable mutex object
       let data = Arc::new(Mutex::new(x));

       for i in 1..(NTHREAD+1) {
           // Increment the count of the mutex
           let mutex = data.clone();

           threads.push(thread::spawn(move || {
               // Lock the mutex
               let n = mutex.lock();

               match n {
                   Ok(mut n) => *n += i,
                   Err(str) => println!("{}", str)
               }
           }));
       }

       // Wait all threads ending
       for thread in threads {
           let _ = thread.join().unwrap();
       }

       assert_eq!(*data.lock().unwrap(), 55);
   }

在這個程式中，的確可以每次都正確地産生預期的值。

==============================================
在執行緒間傳遞資料
==============================================

若要在不同執行緒間分享資料，要透過 channel，channel 是一個單向的通道，資料會從 sender \
傳到 receiver。以下是範例：

.. code-block:: rust

   use std::thread;

   // mpsc stands for multiple producers, single consumer
   use std::sync::mpsc::channel;

   fn main() {
       // Create a channel between the sender and the receiver
       let (tx, rx) = channel();

       // Send data
       thread::spawn(move || {
           tx.send(10).ok().expect("Unable to send message");
       });

       // Receive data
       let n = rx.recv().unwrap();

       assert_eq!(n, 10);
   }

在 Rust 的 channel 中，同一個 receiver 可以對應多個 sender，見下例：

.. code-block:: rust

   use std::thread;
   use std::sync::mpsc::channel;

   fn main() {
       let (tx, rx) = channel();

       for i in 1..(10+1) {
           // Clone the sender
           let tx = tx.clone();

           thread::spawn(move || {
               // Do some computation in a new thread
               let answer = i * i;

               // Send data
               tx.send(answer).unwrap();
           });
       }

       // Receive all data and sum them
       assert_eq!(rx.iter().take(10).fold(0, |a, b| a + b),
                  (1..).map(|x| x * x).take(10).fold(0, |a, b| a + b));
   }

====================================
Sync 和 Send
====================================

在 Rust 中，若物件想要有共時性的功能，要實作 Sync 和 Send 這兩個 trait，前者確保資料在\
不同執行緒間可共享，後者確保資料可在不同執行緒間傳遞。若我們的函式庫用到不安全的程式碼，\
Rust 無法自動滿足這兩個 trait，則必需由程式設計者自行實作相關功能。

====================================
Thread Pool
====================================

執行緒池 (thread pool) 是一種用於共時性程式的軟體設計模式。透過執行緒池來執行共時性程式，\
可以節約建立和銷毀執行緒的開銷，使得多執行緒程式更有效率。Rust 沒有內建的執行緒池，而需\
依賴第三方套件，實例如下：

.. code-block:: rust

   extern crate num_cpus;
   extern crate threadpool;

   use std::sync::mpsc::channel;
   use threadpool::ThreadPool;

   fn main() {
       // Get the core number of the host CPU(s)
       let n_worker = num_cpus::get();

       // Create the thread pool
       let pool = ThreadPool::new(n_worker);

       let (tx, rx) = channel();

       // Create threads in the pool
       for i in 1..(10+1) {
           let tx = tx.clone();

           pool.execute(move || {
               // Do some computation
               let answer = i * i;

               // Send data
               tx.send(answer).unwrap();
           })
       }

       // Receive all data and sum them
       assert_eq!(rx.iter().take(10).fold(0, |a, b| a + b),
                  (1..).map(|x| x * x).take(10).fold(0, |a, b| a + b));
   }
