*********************
模組和套件
*********************

我們學會函式後，程式碼可以分離，然而，隨著專案規模上升，函式名稱有可能相互衝突。雖然，我們也\
可以修改函式名稱，但是，只靠函數名稱來區分函式，往往會造成函數名稱變得冗長。像 C 語言中，沒有\
額外的機制處理函式名稱的衝突，就會看到很多長名稱的函式，像是 \
``gtk_application_get_windows`` (出自 GTK+ 函式庫)。Rust 提供\ **模組 (module)** 的\
機制，處理函式命名衝突的問題。

*註：在許多程式語言中，以命名空間 (namespace) 提供類似的機制。*

在我們先前的內容中，函式和主程式都寫在同一個檔案。在實務上，我們會將函式或物件獨立出來，製成\
**套件 (package)**\ ，之後可以重覆利用。例如，同一套函式庫，可供終端機或圖形介面等不同使用者\
介面來使用。在 Rust 中，套件和模組相互關連。Rust 的套件又稱為 **crate**\ 。

==========================
使用模組
==========================

雖然我們在先前的內容沒有強調模組，實際上，我們已經在使用模組了。我們回頭看先前的一個例子：

.. code-block:: rust

   // Call f64 module from std library
   use std::f64;

   fn main() {
       // Call sqrt function
       let n = f64::sqrt(4.0);

       println!("{}", n);
   }

在這個例子中，我們呼叫 ``std`` 函式庫之中的 ``f64`` 模組，之後，就可以呼叫該模組內的 \
``sqrt`` 函式。

===========================
撰寫模組
===========================

在 Rust 中，使用 ``mod`` 這個關鍵字來建立模組。如下例：

.. code-block:: rust

   mod english {
       pub fn hello(name: &str) {
           println!("Hello, {}", name);
       }
   }

   mod chinese {
       pub fn hello(name: &str) {
           println!("你好，{}", name);
       }
   }

   fn main() {
       // Call modules
       use english;
       use chinese;

       // Call two functions with the same name
       english::hello("Michael");
       chinese::hello("麥可");
   }

在本例中，我們在兩個模組中定義了同名而不同功能的函式。由於這兩個函式被區隔不同的模組中，不會有\
命名衝突的問題。模組除了區隔函式名稱外，也提供私有區塊，在模組中的函式或物件，需以 ``pub`` \
關鍵字宣告，否則無法在模組外使用。

如同我們先前看到的範例，模組也可以內嵌。如下例：

.. code-block:: rust

   mod english {
       pub mod greeting {
           pub fn hello(name: &str) {
               println!("Hello, {}", name);
           }
       }

       pub mod farewell {
           pub fn goodbye(name: &str) {
               println!("Goodbye, {}", name);
           }
       }
   }

   fn main() {
       use english;

       english::greeting::hello("Michael");
       english::farewell::goodbye("Michael");
   }

由本例可知，透過模組的機制，可以協助我們整理函式。

======================================
建立套件
======================================

在我們先前的範例中，我們建立的是主程式專案，如下：

.. code-block:: console

   $ cargo new --bin myapp

但若想將函式或物件獨立建置，則要用函式庫專案，如下：

.. code-block:: console

   $ cargo new --lib mylib

我們現在實際建立一個函式庫套件。以上述指令建立 *mylib* 函式庫套件。加入以下函式：

.. code-block:: rust

   // mylib/src/lib.rc

   pub fn hello(name: &str) -> String {
       format!("Hello, {}", name)
   }

之後，退回到上一層目錄，建立 *myapp* 主程式套件。加入以下內容：

.. code-block:: rust

   // myapp/src/main.rs

   // Call mylib
   extern crate mylib;

   fn main() {
       assert_eq!(mylib::hello("Michael"), "Hello, Michael");
   }

透過 ``extern crate`` 可以呼叫外部專案。另外，要修改 *Cargo.toml* 紀錄檔，加入以下內容：

.. code-block:: text

   [dependencies]
   mylib = { path = "../mylib" }

之後，執行該專案，若可正確執行，代表我們成功地建立套件。

如果函式庫存放在遠端站台上，需修改存取位置。在下例中，我們存取以 Git 存放的函式庫：

.. code-block:: text

   [dependencies]
   rand = { git = "https://github.com/rust-lang-nursery/rand.git" }

*Cargo.toml* 是 Rust 套件 (i.e. crate) 使用的設定檔。建議花一些時間熟悉其\
`官方文件 <http://doc.crates.io/guide.html>`_\ 。

============================
在套件中使用模組
============================

在我們先前的例子中，透過 *mylib* 函式庫對函式命名做最基本的區隔。不過，我們也可以在函式庫\
中使用模組來進一步區隔函式。我們先以實例看加入模組後的效果：

.. code-block:: rust

   // Call external library
   extern crate phrase;

   fn main() {
       assert_eq!("Hello, Michael", phrase::english::greeting::hello("Michael"));
       assert_eq!("你好，麥可", phrase::chinese::greeting::hello("麥可"));
   }

同樣地，需於 *Cargo.toml* 加入套件位置：

.. code-block:: text

   [dependencies]
   phrase = { path = "../phrase" }

退回上一層目錄，建立 *phrase* 函式庫：

.. code-block:: console

   $ cargo new --lib phrase

整個 *phrase* 專案結構如下：

.. code-block:: console

   $ tree
   .
   ├── Cargo.lock
   ├── Cargo.toml
   └── src
       ├── chinese
       │   ├── greeting.rs
       │   └── mod.rs
       ├── english
       │   ├── greeting.rs
       │   └── mod.rs
       └── lib.rs

在 *src/lib.rs* 中宣告模組，記得要宣告公開權限：

.. code-block:: rust

   pub mod english;
   pub mod chinese;

在 *src/english/mod.rs* 中宣告子模組：

.. code-block:: rust

   pub mod greeting;

在 *src/english/greeting.rs* 中實作函式：

.. code-block:: rust

   pub fn hello(name: &str) -> String {
       format!("Hello, {}", name)
   }

同樣地，在 *src/chinese/mod.rs* 中宣告子模組：

.. code-block:: rust

   pub mod greeting;

同樣地，在 *src/chinese/greeting.rs* 中實作函式：

.. code-block:: rust

   pub fn hello(name: &str) -> String {
       format!("你好，{}", name)
   }

由於 Rust 的模組及套件和檔案名稱是連動的，若使用錯誤的檔案名稱將無法編譯，需注意。

===============================
進階的模組使用方式
===============================

在先前的例子中，由於函式庫結構較複雜，使得函式呼叫的動作變得繁瑣，Rust 提供語法來簡化這個\
動作。如下例：

.. code-block:: rust

   extern crate phrase;

   use phrase::english::greeting as en_greeting;
   use phrase::chinese::greeting as zh_greeting;

   fn main() {
       assert_eq!("Hello, Michael", en_greeting::hello("Michael"));
       assert_eq!("你好，麥可", zh_greeting::hello("麥可"));
   }

Rust 官方文件中提供了另一個更複雜的模組呼叫範例：

.. code-block:: rust

   // Rename crate
   extern crate phrases as sayings;

   // Rename module
   use sayings::japanese::greetings as ja_greetings;

   // Glob all functions in a module, NOT a good style
   use sayings::japanese::farewells::*;

   // A complex renaming scheme
   use sayings::english::{self, greetings as en_greetings, farewells as en_farewells};

   fn main() {
       println!("Hello in English; {}", en_greetings::hello());
       println!("And in Japanese: {}", ja_greetings::hello());
       println!("Goodbye in English: {}", english::farewells::goodbye());
       println!("Again: {}", en_farewells::goodbye());

       // Use a globbed function, AVOID it when possible.
       println!("And in Japanese: {}", goodbye());
   }

稍微閱讀一下程式碼，大概就知道如何呼叫模組。要注意的是，globbing 的動作，會直接暴露函式\
名稱到主程式中，喪失使用模組區隔函式名稱的用意，應盡量避免。
