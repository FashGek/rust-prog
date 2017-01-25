************************
Struct
************************

Struct 是複合型別，其中可包含基礎型別或是其他複合型別。透過 struct，程式設計者可以用有\
效率的方式組織資料。假設沒有 struct，而我們要表示平面上的點，就必需要用兩個變數，如以下 \
Rust 虛擬碼：

.. code-block:: rust

   let x1 = 3.0;
   let y1 = 4.0;

如果有第二個點，就要再用兩個變數，如下：

.. code-block:: rust

   let x1 = 3.0;
   let y1 = 4.0;

   let x2 = 5.0;
   let y2 = 5.0;

讀者可以想像，如果我們要用這樣的方式撰寫程式，程式的可維護性將會是一大問題。若現在我們要\
表示在三度空間的點或是高維度的向量，這樣的方式在實務上變得不可行。

或許有讀者想到，可以用容器來儲存資料，但容器有其限制：(1) 有些容器只能儲存同類型的資料，\
(2) 容器無法建立新的型別，(3) 容器無法和函式結合成物件。

----------------------
建立 struct
----------------------

Rust 的 struct 有三種：

* C 風格 struct
* Tuple struct
* Unit struct

其中第一種最常用。以下範例建立一個 C 風格 struct：

.. code-block:: rust

   // For abs function
   use std::f64;

   struct Point {
       x: f64,
       y: f64,
   }

   fn main() {
       let p = Point{ x: 3.0, y: 4.0 };

       assert!(f64::abs(p.x - 3.0) < 1e-10);
       assert!(f64::abs(p.y - 4.0) < 1e-10);
   }

如果希望 struct 的欄位是可變的，同樣需明確指明：

.. code-block:: rust

   use std::f64;

   struct Point {
       x: f64,
       y: f64,
   }

   fn main() {
       let mut p = Point{ x: 0.0, y: 0.0 };

       // Assign value to the field of struct Point
       p.x = 5.0;

       assert!(f64::abs(p.x - 5.0) < 1e-10);
   }

C 風格 struct 的欄位有名稱，而 tuple struct 則無。實例如下：

.. code-block:: rust

   use std::f64;

   struct Point(f64, f64);

   fn main() {
       let p = Point(3.0, 4.0);

       assert!(f64::abs(p.0 - 3.0) < 1e-10);
       assert!(f64::abs(p.1 - 4.0) < 1e-10);
   }

但是，使用 tuple struct 需記憶欄位順序，在實務上不若 C 風格 struct 實用。

而 unit struct 沒有欄位。實例如下：

.. code-block:: rust

   struct Nil {}

   fn main() {
       let nil = Nil{};
   }

Unit struct 本身較少使用。有時會搭配泛型程式設計來使用。

---------------------------
結合資料和行為
---------------------------

Struct 除了用來組織資料外，很大一部分在於可以利用物件導向的機制，建立自己的型別系統。\
物件的其中一個觀點，就是 ``物件 = 資料 + 行為``\ 。物件導向是較進階的主題，本書後續章節\
會再說明。
