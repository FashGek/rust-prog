**********************************
不安全的程式碼
**********************************

大部分的 Rust 程式碼是\ *安全的*\ ，然而，我們有時候需要撰寫\ *不安全的* Rust 程式碼，\
例如：

* 和 C 同等自由地運用指標
* 直接使用 C 函式庫
* 撰寫\ **組合語言 (assembly language)**
* 透過 **FFI (Foreign Language Interface)** 和外部語言合作

Rust 可視為兩個語言的集成，一個是我們熟知的安全的 Rust，另一個則是本章中要介紹的不安全的 \
Rust。不安全的 Rust 程式碼儘量在必要時才使用，因為 Rust 對於不安全程式碼的檢查項目較少。

=============================
撰寫不安全的程式碼
=============================

在 Rust 中，不安全的程式碼需放在 ``unsafe`` 區塊，如以下虛擬碼：

.. code-block:: rust

   unsafe {
       // Possibly dangerous code
   }

或者，可宣告函式或 trait 為 ``unsafe``。

在撰寫不安全的程式碼時，Rust 能夠檢查的項目會變少，就像是在撰寫 C/C++ 程式碼般，程式碼\
安全的責任落在程式設計者身上。在撰寫 Rust 程式時，若程式碼中同時有安全及不安全的部分，\
則更要小心地檢查。Rust 內部使用了許多的不安全程式碼，這些程式碼經由 Rust 開發團隊檢查過，\
確保這些程式碼是安全的。

==============================
Raw Pointer
==============================

在不安全的程式碼中，可以使用 raw pointers，基本上，可視為和 C 相同等級的\
指標 (pointer)。這些指標，有可能指向不合法的記憶體位置，若有配置記憶體，也需手動釋放。\
另外，這些指標也不受到 Rust 的所有權的限制。宣告方式可用 ``*const`` 或是 ``*mut``\ 來\
宣告，視該指標的可變性而定。

以下是一個使用 raw pointer 的範例：

.. code-block:: rust

   extern crate libc;

   use std::mem;

   fn main() {
       // Declare a raw pointer to int
       let mut n_ptr: *mut i32;

       unsafe {
           // Allocate memory for the pointer
           n_ptr = libc::malloc(mem::size_of::<i32>() as libc::size_t)
               as *mut i32;

           // Assign value to the memory chunk
           *n_ptr = 3;

           // Get the value by dereferencing
           println!("{}", *n_ptr);

           // Free the memory
           libc::free(n_ptr as *mut libc::c_void);
       }
   }

等效的 C 程式碼如下：

.. code-block:: c

   #include <stdio.h>
   #include <stdlib.h>

   int main() {
	     // Declare a pointer to int
	     int* n_ptr;

	     // Allocate memory for the pointer
	     n_ptr = (int*) malloc(sizeof(int));

	     // Assign value to the memory chunk
	     *n_ptr = 3;

	     // Get the value by dereferencing
	     printf("%d\n", *n_ptr);

	     // Free the memory
	     free(n_ptr);

	     return 0;
   }

由這兩個例子可看出，除了在語法上略有不同外，Rust 也可以用類似 C 的方式操作指標。

=======================================
實例：實作 C 風格的陣列
=======================================

在這個範例中，我們用 C 風格的陣列重新實作 vector。首先，宣告 Vector 類別，包括該類別的\
大小及實際儲存資料的 C 風格陣列：

.. code-block:: rust

   extern crate libc;

   use std::mem;
   use std::ops::{Index, IndexMut};
   use std::default::Default;

   pub struct Vector<T> where T: Default {
       size: usize,
       vec: *mut T,
   }

實作建構子，在內部，使用 ``alloc`` 函式配置記憶體。在這裡用了一個小技巧，就是對於泛型\
型別 ``T`` 使用預設值，通常即為 0：

.. code-block:: rust

   impl<T> Vector<T> where T: Default {
       pub fn new(size: usize) -> Vector<T> {
           let v: *mut T;
           unsafe {
               v = libc::malloc(size * (mem::size_of::<T>() as libc::size_t))
                   as *mut T;

               for i in 0..size {
                   *(v.offset(i as isize)) = Default::default();
               }
           }
           Vector::<T>{ size: size, vec: v }
       }
   }

因為我們使用到 raw pointer，必需實作解構子，內部使用 ``free`` 函式釋放記憶體：

.. code-block:: rust

   impl<T> Drop for Vector<T> where T: Default {
       fn drop(&mut self) {
           unsafe {
               libc::free(self.vec as *mut libc::c_void);
           }
       }
   }

由於 Rust 的限制，我們不能實作 Copy trait，但可以實作 Clone trait：

.. code-block:: rust

   impl<T> Clone for Vector<T> where T: Copy + Default {
       fn clone(&self) -> Vector<T> {
           let v: *mut T;
           unsafe {
               v = libc::malloc(self.size * (mem::size_of::<T>() as libc::size_t))
                   as *mut T;

               for i in 0..(self.size) {
                   *(v.offset(i as isize)) =
                       *(self.vec.offset(i as isize)).clone();
               }
           }
           Vector::<T>{ size: self.size, vec: v }
       }
   }

我們的 Vector 可以從尾端加入資料，內部使用 ``realloc`` 函式，實作如下：

.. code-block:: rust

   impl<T> Vector<T> where T: Copy + Default {
       pub fn push(&mut self, data: T) {
           self.size += 1;
           unsafe {
               self.vec = libc::realloc(self.vec as *mut libc::c_void, self.size)
                   as *mut T;
               *(self.vec.offset((self.size - 1) as isize)) = data;
           }
       }
   }

我們實作 Index trait，這樣子，我們的 Vector 類別就可以像內建陣列般使用數字索引資料：

.. code-block:: rust

   impl<T> Index<usize> for Vector<T> where T: Copy + Default {
       type Output = T;

       fn index(&self, index: usize) -> &T {
           if index >= self.size {
               panic!("Index out of range");
           }
           unsafe {
               &*(self.vec.offset(index as isize))
           }
       }
   }

同時，也實作 IndexMut trait，這樣子，我們就可以用類似陣列的方式對特定位置指派值。在這裡，\
Rust 的官方文件說明不清楚，其實是要回傳指向特定位置的參考，方便更動其值：

.. code-block:: rust

   impl<T> IndexMut<usize> for Vector<T> where T: Copy + Default {
       fn index_mut(&mut self, index: usize) -> &mut T {
           if index >= self.size {
               panic!("Index out of range");
           }
           unsafe {
               &mut *(self.vec.offset(index as isize))
           }
       }
   }

我們在先前陣列的章節提過，若要動態配置記憶體，就使用 vector。其實，我們也可以利用 Rust \
包裝的 C 函式建立 C 風格的陣列，如同本節所展示的實例。

=======================================
撰寫組合語言
=======================================

目前 (2017/01/08) 在 Rust 中撰寫組合語言的功能，僅於 nightly 版本實作。如果想在 Rust \
程式碼中撰寫組合語言，可用 ``asm!`` 巨集，實例如下：

.. code-block:: rust

   #![feature(asm)]

   fn add(a: i32, b: i32) -> i32 {
       let add: i32;

       // x86 assembly language
       unsafe {
           asm!(
           "pushq   %rbp
            movq    %rsp, %rbp
            movl    %edi, -4(%rbp)
            movl    %esi, -8(%rbp)
            movl    -4(%rbp), %edx
            movl    -8(%rbp), %eax
            addl    %edx, %eax
            popq    %rbp
            ret"
           : "=r"(add)
           : "r"(a), "r"(b)
           );
       }

       add
   }

   fn main() {
       assert_eq!(add(3, 2), 5);
   }

========================================
異種語言合作
========================================

Rust 提供一套易於使用的 FFI，使得 Rust 和其他語言的合作相當容易。主要包括兩方面：

* 從 Rust 呼叫 C
* 從其他語言呼叫 Rust

Rust 的角色類似於 C++，做為一個較 C 高階的編譯語言，不僅易於和 C 溝通，也可取代 C 提供\
其他高階語言的核心功能。我們會於後續的章節介紹這個主題。

========================================
(案例選讀) 連結串列
========================================

**連結串列 (linked list)** 是一種基礎的線型容器。和陣列不同的是，連結串列插入資料的效率\
較佳，而且不需預先知道串列的大小，然而，在串列中搜尋特定編號的資料的效率較陣列差。以下是\
一個\ **雙向連結串列 (doubly linked list)** 的示意圖：

.. image:: img_unsafe/doubly-linked-list.png
   :alt: 雙向連結串列

實作連結串列是一個練習指標使用的好題目，因為連結串列的觀念不困難，實作也不會太複雜。在\
本節中，我們將實作一個雙向連結串列。呼叫相關的套件及宣告內部結點：

.. code-block:: rust

   extern crate libc;

   use std::mem;
   use std::ptr;

   // Internal node
   struct Node<T> {
       data: T,
       next: *mut Node<T>,
       prev: *mut Node<T>
   }

宣告串列類別：

.. code-block:: rust

   // Struct that holds a doubly-linked list
   pub struct List<T> {
       head: *mut Node<T>,
       tail: *mut Node<T>
   }

實作 push，這個方法會從串列尾端插入資料：

.. code-block:: rust

   impl<T> List<T> {
       pub fn push(&mut self, value: T) {
           unsafe {
               let node = libc::malloc(
                   mem::size_of::<Node<T>>() as libc::size_t)
                   as *mut Node<T>;
               ptr::write(node, Node {
                   data: value,
                   next: ptr::null_mut(),
                   prev: ptr::null_mut()
               });

               if self.head.is_null() {
                   self.head = node;
                   self.tail = node;
               } else {
                   (*self.tail).next = node;
                   (*node).prev = self.tail;
                   self.tail = node;
               }
           }
       }
   }

實作 pop，這個方法會從尾端取出資料：

.. code-block:: rust

   impl<T> List<T> where T: Copy {
       pub fn pop<'a>(&mut self) -> Option<T> {
           unsafe {
               if self.head.is_null() {
                   return None;
               }

               if self.head == self.tail {
                   let data = (*self.tail).data;
                   libc::free(self.tail as *mut libc::c_void);
                   self.head = 0 as *mut Node<T>;
                   self.tail = 0 as *mut Node<T>;
                   Some(data)
               } else {
                   let data = (*self.tail).data;
                   let mut current = self.tail;
                   self.tail = (*current).prev;
                   (*self.tail).next = 0 as *mut Node<T>;
                   (*current).prev = 0 as *mut Node<T>;
                   libc::free(current as *mut libc::c_void);
                   Some(data)
               }
           }
       }
   }

利用類似的方式，也可以實作 unshift 和 shift。

由於這個串列的實作用到 raw pointer，必需實作釋放記憶體的程式碼。在 Rust，需實作 Drop \
trait，實例如下：

.. code-block:: rust

   impl<T> Drop for List<T> {
       fn drop(&mut self) {
           unsafe {
               let mut current = self.head;
               while !current.is_null() {
                   self.head = (*current).next;

                   if !self.head.is_null() {
                       (*self.head).prev = 0 as *mut Node<T>;
                       (*current).next = 0 as *mut Node<T>;
                       libc::free(current as *mut libc::c_void);
                   }

                   current = self.head;
               }
           }
       }
   }

還有其他的方法，這裡就不列出。本節內容的完整範例位於\
`這裡 <https://github.com/cwchentw/linked-list-in-Rust>`_\ ，有興趣的讀者可自行\
前往觀看。

如果不使用 unsafe 程式碼，能否實作連結串列或其他的資料結構？雖然可以，但相當困難。筆者實地\
嘗試過，反而比使用 unsafe 程式碼還難，因為會碰到許多所有權相關的問題。若有興趣的讀者，可到\
`這裡 <https://github.com/cwchentw/safe-linked-list-in-Rust>`_\ 觀看範例。還可到\
`這裡 <http://cglab.ca/~abeinges/blah/too-many-lists/book/>`_\ 看更多相關的說明。
