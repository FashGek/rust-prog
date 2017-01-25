********************
字串
********************

==========================
Rust 字串
==========================

Rust 的字串有以下兩種：

* String 類別是以 UTF8 編碼、可伸縮的字串
* str 是基礎型別，通常是使用 ``&str``\ ，即指向 str 的參考

另外，Rust 還有字元 (char) 型別，同樣也是以 UTF8 編碼。

建立字串常數時，預設型別是 ``&str``，即指向 str 的參考，但是，若想要使用 String 類別的\
方法時，可以轉為 String 物件。

.. code-block:: rust

   fn main() {
      // Reference to str
      let s: &str = "Hello";

      // String object
      let s_obj: String = String::from("Hello");
   }

==============================
對字串進行索引
==============================

由於 Rust 字串為 UTF8 字串，其內部長度和字元數不同，不能直接對其進行索引。以下的範例是錯誤的：

.. code-block:: rust

   fn main() {
       assert_eq!("Hello"[0], "H");
   }

本例引發以下錯誤訊息：

.. code-block:: console

   error[E0277]: the trait bound `std::string::String: std::ops::Index<{integer}>` is not satisfied

如果想要得到單一的字元，需用 ``chars`` 方法，會得到走訪字元的迭代器。

.. code-block:: rust

   fn main() {
       assert_eq!("你好，麥可".chars().nth(1).unwrap(), '好');
   }

由於 Rust 的字元是以 UTF8 編碼，即使用非英文的字元也可以。

==============================
字串相接
==============================

字串間可以用 ``+`` 相接，相接的方式是以 String 物件接 ``&str``\ ，如下：

.. code-block:: rust

   fn main() {
       assert_eq!("Hello ".to_string() + "World", "Hello World");
   }

如果要將兩個 String 物件相接，則要將 String 物件取參考，轉為 ``&str``\ ，如下：

.. code-block:: rust

   fn main() {
       let s1 = String::from("Hello ");
       let s2 = String::from("World");

       assert_eq!(s1 + &s2, "Hello World");
   }

==================================
格式化字串
==================================

Rust 提供數個格式化字串的巨集，包括：

* ``format!``\ ：將格式化字串轉為 String 物件
* ``println!``\ ：將格式化字串印到終端機，附帶換行字元
* ``print!``\ ：將格式化字串印到終端機，但不附加換行字元
* ``writeln!``\ ：將格式化字串寫入特定的 buffer 物件，附帶換行字元
* ``write!``\ ：將格式化字串寫入特定的 buffer 物件，但不附帶換行字元

以下實例將錯誤訊息輸出到標準錯誤 (standard error)：

.. code-block:: rust

   use std::io::Write;

   fn main() {
       writeln!(&mut std::io::stderr(), "Error message")
           .expect("Failed to print to stderr");
   }

以下實例輸出不同的格式化字串：

.. code-block:: rust

   fn main() {
       // Hexadeicmal number
       assert_eq!(format!("0x{:X}", 255), "0xFF");

       // Octal number
       assert_eq!(format!("0o{:o}", 127), "0o177");

       // Decimal with specific precision
       assert_eq!(format!("{:.2}", 3.14159), "3.14");

       // Formatted string for debug purpose
       assert_eq!(format!("{:?}", vec![1, 2, 3]), "[1, 2, 3]");

       // Formatted string with specific position
       assert_eq!(format!("{0}, this is {1}. {1}, this is {0}", "Alice", "Bob"),
                  "Alice, this is Bob. Bob, this is Alice");

       // Formatted string with specific name
       assert_eq!(format!("{subject} {verb} {object}", object = "the lazy dog",
                          verb = "jumps over", subject = "The swift fox"),
                  "The swift fox jumps over the lazy dog");
   }
