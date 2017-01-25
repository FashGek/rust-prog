********************
函式
********************

在前面的內容中，我們將大部分的程式碼寫在主函式中。隨著程式規模上升，這種方式漸漸顯得不足：\
(1) 對於相同的步驟撰寫重覆的程式碼，(2) 主程式變得冗長，(3) 無法將程式碼分離再利用。\
\ **函式 (function)** 是程式碼再利用的基礎，透過函式將程式碼分離後，可再將函式整理成\
**函式庫 (library)** 分享給其他程式。\
**物件導向程式設計 (object-oriented programming)** 的\ **方法 (method)**\ ，也是\
建立在函式之上。

=============================
使用函式
=============================

Rust 已經預先寫好許多的函式，供程式設計者使用。如下例：

.. code-block:: rust

   // Call f64 module
   use std::f64;

   fn main() {
       // Call sqrt function
       let n = f64::sqrt(4.0);

       println!("{}", n);
   }

在本例中，我們呼叫標準函式庫的 f64 模組，接著，呼叫 ``sqrt`` 函式。有時候，函式會和物件\
結合，在本書先前介紹容器時，就介紹了一些 Rust 內建的容器物件；和物件相連的函式，也稱為方法。

==============================
撰寫函式
==============================

撰寫函式或方法使用 ``fn`` 這個關鍵字。我們來看一個簡單的實例：

.. code-block:: rust

   fn hello() {
       println!("Hello, World");
   }

   fn main() {
       hello();
       hello();
       hello();
   }

在本例中，\ ``hello`` 函式相當單純，每次呼叫時都會在終端機中印出 *Hello, World* 字串。\
由此可以看出，將程式碼寫入函式後，可以重覆呼叫。

在上例中，\ ``hello`` 函式的行為是固定的。函式可透過\ **參數 (parameter)**\ 來改變\
函式的行為。如下例：

.. code-block:: rust

   fn hello(name: &str) {
       println!("Hello, {}", name);
   }

   fn main() {
       hello("Michael");
       hello("Tom");
       hello("Alice");
   }

在本例中，我們輸入不同的名字，\ ``hello`` 函式會印出不同的字串。雖然 Rust 可自動推斷\
型別，在撰寫 Rust 函式時，必需明確指定參數的型別，這是 Rust 設計上的考量。

除了可以傳入參數，函式也可以有\ **回傳值 (returning value)**\ ，如下例：

.. code-block:: rust

   fn capitalize(s: &str) -> String {
       let mut c = s.chars();
       match c.next() {
           None => String::new(),
           Some(f) => f.to_uppercase().collect::<String>() + c.as_str(),
       }
   }

   fn main() {
       assert_eq!(capitalize("apple"), "Apple");
       assert_eq!(capitalize("banana"), "Banana");
       assert_eq!(capitalize("orange"), "Orange");
   }

在本例中，我們依據所到的字元來回傳不同的值，若字元為空，回傳一個空字串，若字元不為空，則將首\
字母轉為大寫後和其他字元相接後回傳。要注意的是，回傳值後不加上分號 (``;``)，在 Rust 中，沒加\
分號的是表達式 (expression)，而有加分號的是敘述 (statement)。以下是一個錯誤的例子：

.. code-block:: rust

   fn add_one(x: i32) -> i32 {
       x + 1;
   }

若改成表達式，則正確：

.. code-block:: rust

   fn add_one(x: i32) -> i32 {
       x + 1
   }

若不習慣這種語法，也可改為回傳敘述：

.. code-block:: rust

   fn add_one(x: i32) -> i32 {
       return x + 1;
   }

回傳表達式，是比較典型的 Rust 習慣語法。

Rust 的函式只能回傳單一值，而 Rust 發展出一些特殊容器來應對。例如前文提過的 Option 容器，可\
回傳 None 表空值或是以 Some 包裝的參考。另外一個例子是 Result 容器，可回傳值或是表示錯誤的 \
Err。

===================================
改變狀態的函式
===================================

若搭配\ **參考 (reference)**\ ，函式也可改變資料的狀態。如下例：

.. code-block:: rust

   struct Point {
       x: f64,
       y: f64,
   }

   fn point_new(x: f64, y: f64) -> Point {
       Point{ x: x, y: y }
   }

   fn point_get_x(p: & Point) -> f64 {
       (*p).x
   }

   fn point_get_y(p: & Point) -> f64 {
       (*p).y
   }

   fn point_set_x(p: &mut Point, x: f64) {
       (*p).x = x;
   }

   fn point_set_y(p: &mut Point, y: f64) {
       (*p).y = y;
    }

   fn point_to_string(p: & Point) -> String {
       format!("({}, {})", point_get_x(p), point_get_y(p))
   }

   fn main() {
       let mut p = point_new(0.0, 0.0);
       assert_eq!(point_to_string(&p), "(0, 0)");

       point_set_x(&mut p, 3.0);
       point_set_y(&mut p, 4.0);
       assert_eq!(point_to_string(&p), "(3, 4)");
   }

有 C 語言經驗的讀者，應該可以發現本例採用 C 風格的物件導向。由於 Rust 本身即支援物件導向，\
在實務上不建議這種寫法，這個範例僅是展示如何在函式中使用參考。

==============================
函式指標
==============================

函式可以視為一種型別，也可將函式視為值。如下例：

.. code-block:: rust

   fn add_one(x: i32) -> i32 {
       return x + 1;
   }

   fn main() {
       let f: fn(i32) -> i32 = add_one;
       assert_eq!(f(5), 6);
   }

在這裡，也可以用 Rust 的型別推斷功能，將型別省略。

==============================
預設參數
==============================

Rust 的函式不支援預設參數，要用其他的方式來模擬這個特性。其中一個方式是利用 Default trait，\
範例如下：

.. code-block:: rust

   use std::default::Default;

   #[derive(Debug)]
   pub struct Parameter {
       a: u32,
       b: u32,
       c: u32,
   }

   // Set default values for Parameter struct
   impl Default for Parameter {
       fn default() -> Self {
           Parameter { a: 2, b: 4, c: 6}
       }
   }

   fn some_calc(p: Parameter) -> u32 {
       let (a, b, c) = (p.a, p.b, p.c);
       a + b + c
   }

   fn main() {
       // Set default values for p except c
       let p = Parameter { c: 10, .. Parameter::default() };
       println!("{}", some_calc(p));
   }

在這個例子中，我們運用一點點物件導向的功能，將 ``Parameter`` struct 加人 ``default`` 函式，之後\
就可以呼叫此函式以得到預設值。Rust 大量運用 trait 實作物件導向及泛型相關的功能，想了解的讀者可在\
後續的章節看到更多範例。

==============================
遞迴
==============================

\ **遞迴 (recursion)** 是可以呼叫自己的函式，直到回到某個特定的基本條件為止。\
Fibonacci 數是一個典型的遞迴實例，程式碼範例如下：

.. code-block:: rust

   fn fib(n: u32) -> u32 {
       if n == 0 { // Base condition 1
           0
       } else if n == 1 { // Base condition 2
           1
       } else {
           fib(n - 1) + fib(n - 2)
       }
   }

   fn main() {
       for i in 0..11 {
           print!("{} ", fib(i));
       }
       println!("");
   }

初學者對遞迴程式感到困惑，而不知道如何追踪遞迴程式碼。遞迴函式就像是一般的函式般，是\
完成某個特定的行為的藍圖，而不是該行為本身。遞迴函式會在某個遞迴的步驟告訴電腦「在這裡我們要\
再呼叫一次這個方法，幫我完成這個步驟，然後將結果傳回來」。另外一個學習遞迴的方法，是重新將\
**數學歸納法 (mathematical induction)** 的教材看一次，這兩者間有異曲同工之妙。要寫好\
遞迴程式，關鍵在於設置正確的終止條件。以 Fibonacci 數來說，有兩個終止條件，即 \
``F(0) = 1`` 和 ``F(1) = 1``\ ，其他的數字，最後都會呼叫到這兩個數字。

在程式設計中，遞迴是相當重要的概念，許多用控制流程的程式都可用遞迴重新改寫。使用遞迴實作程式，\
代碼往往更為簡潔。許多資料結構和演算法的內部，也大量使用遞迴，像是堆積 (heap)、樹 (tree)、\
圖 (graph)等。
