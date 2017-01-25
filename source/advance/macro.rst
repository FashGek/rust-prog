*****************
巨集
*****************

在 Rust 中，有許多程式抽象化的工具，像是函式、物件、泛型程式、高階函式等。而\ **巨集 (macro)** 是\
另一個更高層次的抽象化工具。Rust 會在編譯的早期，將巨集擴張成一般的 Rust 程式碼後，再和其他的程式碼\
一起編譯成機械碼。巨集較一般的函式難理解，除錯也較為困難。

===========================
使用巨集
===========================

在本書中，我們已經使用數個巨集，像是 ``println!``\ 、\ ``format!``\ 、\ ``vec!``\ 、\
``assert!`` 等。如以下簡單的例子：

.. code-block:: rust

   fn main() {
       let greeting = "Hello";
       let name = "Michael";
       println!("{}, {}", greeting, name);
   }

在這個例子及我們先前的例子中，我們可以看到 ``println!`` 巨集可接受不特定長度的參數，而一般的\
函數無法做到這一點。

===========================
撰寫巨集
===========================

巨集的語法和一般的 Rust 語法略有不同，可視為一套次語言。撰寫巨集會用到 ``macro_rules!`` 巨集，\
用虛擬碼表示如下：

.. code-block:: text

   macro_rules! macro_name {
       ( pattern ) => {
           expension
       };
       ( pattern ) => {
           expension
       };
       ...
   }

以下是一個簡單的實例，在這個例子中，我們沒用到任何 pattern：

.. code-block:: rust

   macro_rules! hello {
       () => {
           println!("Hello")
       };
   }

   fn main() {
       hello!();
   }


實務上，沒有使用 pattern 的巨集不太實用。我們這裡介紹另一個巨集的實例，我們實作可接受任意長度\
參數的加法計算的巨集：

.. code-block:: rust

   macro_rules! add {
       ( $( $x:expr ),* ) => {
           {
               let mut vec = Vec::new();

               $(
                   vec.push($x);
               )*

               vec.iter().fold(0, |a, b| a + b)
           }
       };
   }

   fn main() {
       let n = add!(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
       assert_eq!(n, 55);
   }

``( $( $x:expr ),* )`` 部分的語法即是 pattern，這部分看起來較為困難，我們將 pattern 拆開來看。\
首先，最外層的 ``( ... )`` 是固定的語法，在括號內填入我們所需的 pattern。內層的 ``$( ... ),*`` \
表示該 pattern 接受出現零次以上的逗號，而最內層的 ``$x:expr`` 是巨集特有的語法，以 \
``$name : designator`` 的格式命名，以本例來說，\ ``$x`` 是一個 ``expr`` (表達式)。Rust 的官方文件\
中有提到可用的 designator \
(可看\ `這裡 <https://doc.rust-lang.org/beta/reference.html#macros>`_)。

接下來，我們看一下巨集右側的代碼。大部分的代碼，其實就是 Rust 程式碼，但有少數特別的部分，像是

.. code-block:: rust

   $(
       vec.push($x);
   )*

這個部分，和左側的 pattern 相對應。這時指的是，將符合左側 pattern 的 ``$x`` 分別放入 vec 中。

另外，注意到在這裡，我們使用 block，如下：

.. code-block:: rust

   ( ... ) => {
       {
           // Put your code here
       }
   };

因為巨集的右側預期得到表達式，若在右側有多行敘述，則需要使用 block 將這些敘述包起來。

========================
Hygiene
========================

有些程式語言的巨集是用字串代換的方式，例如 C 語言，有時候，這種方式會造成問題。以下是一個 C 語言\
常見的巨集陷阱：

.. code-block:: c

   #include <stdio.h>
   #define FIVE_TIMES(x) 5 * x

   int main(void) {
       printf("%d\n", FIVE_TIMES(2 + 3));
       return 0;
   }

這個程式印出來的結果是 13，而非 25。在這個程式中，\ ``FIVE_TIMES(2 + 3)`` 被替換為 \
``5 * 2 + 3`` ，結果就得到 13。

以下是等效的 Rust 巨集：

.. code-block:: rust

   macro_rules! five_times {
       ( $x:expr ) => {
           5 * $x
       };
   }

   fn main() {
       let n = five_times!(2 + 3);
       assert_eq!(n, 25);
   }

在這個程式中，的確可以得到 25。

另外一個 C 語言的反例：

.. code-block:: c

   #include <stdio.h>
   #define INCI(i) do { int a=0; ++i; } while(0)

   int main() {
       int a = 4, b = 8;
       INCI(a);
       INCI(b);
       printf("a is now %d, b is now %d\n", a, b);
       return 0;
   }

在這個例子中，會得到 a 為 4，b 為 9；這是因為在巨集中不慎引入變數 a，覆蓋掉 \
(shadow) 外部的 a。

以下是等效的 Rust 巨集：

.. code-block:: rust

   // Suppress unused_variables warning message
   #![allow(unused_variables)]

   macro_rules! incr {
       ( $x:expr ) => {
           {
               // A hacky do-while loop in Rust
               while {
                   let a = 0;
                   $x += 1;

                   false
               } {}
           }
       };
   }

   fn main() {
       let mut a = 4;
       let mut b = 8;

       incr!(a);
       incr!(b);

       assert_eq!(a, 5);
       assert_eq!(b, 9);
   }

由以上例子可看出，Rust 的巨集藉由引入新的變數，和內部的變數區隔開來，以避開變數覆蓋的問題。\
由以上兩例可知，Rust 的巨集的確具有 *hygienic* 的特性。

==================================
在套件中使用巨集
==================================

若要引入其他套件的巨集，使用 ``#[macro_use( ... )]``\ ，如下 (摘自 Rust 官方文件)：

.. code-block:: rust

   #[macro_use(foo, bar)]
   extern crate baz;

如果只寫 ``#[macro_use]`` 則會引入該套件所有的巨集。

如果要輸出套件中的巨集，在巨集上加上 ``#[macro_export]`` 即可。

=============================================
(案例選讀) 建立矩陣的巨集
=============================================

在 Rust 中，使用 ``vec!`` 巨集建立 vector 相當直覺和方便，然而，Rust 沒有內建的 \
矩陣 (matrix) 類別，也沒有好用的巨集來建立矩陣。因此，在本節中，我們建立一個直覺的 \
``matrix!`` 巨集，以建立矩陣。

首先，先看一下使用效果：

.. code-block:: rust

   // sc stands for scientific computing.
   #[macro_use(matrix, count)]
   extern crate sc;

   use sc::Matrix;

   fn main() {
       let m = matrix![1, 2, 3;
                       4, 5, 6;
                       7, 8, 9];

       assert_eq!(m.get(0, 1), 2);
       assert_eq!(m.get(1, 1), 5);
       assert_eq!(m.get(1, 2), 6);
   }

由本例可看出，透過 ``matrix!`` 巨集可以很直覺地建立矩陣物件。接下來，我們看一下 Matrix \
類別的實作：

.. code-block:: rust

   pub struct Matrix<T> where T: Copy {
       m: Vec<Vec<T>>,
   }

   impl<T> Matrix<T> where T: Copy {
       pub fn from_vec(s: & Vec<Vec<T>>) -> Matrix<T> {
           let mut rows = Vec::new();

           for i in 0..(s.len()) {
               let mut col = Vec::new();

               for j in 0..(s[0].len()) {
                   col.push(s[i][j]);
               }

               rows.push(col);
           }

           Matrix{ m: rows }
       }
   }

   impl<T> Matrix<T> where T: Copy {
       pub fn get(& self, row: usize, col: usize) -> T {
           self.m[row][col]
       }
   }

我們的 Matrix 類別，內部以 vector of vector 儲存資料。為了簡化範例，我們在這個範例沒有\
實作矩陣運算。接下來，我們來看 ``matrix!`` 巨集如何實作：

.. code-block:: rust

   #[macro_export]
   macro_rules! count {
       () => (0usize);
       ( $x:tt $($xs:tt)* ) => (1usize + count!($($xs)*));
   }

   #[macro_export]
   macro_rules! matrix {
       ( $( $x:expr ),* ) => {{
           let mut vec = Vec::new();
           let mut col = Vec::new();

           $( col.push($x); )*
           vec.push(col);

           Matrix::<_>::from_vec(&vec)
       }};
       ( $( $x0:expr ),* ; $( $( $x:expr),* );* ) => {{
           let _width0 = [(); count!($($x0)*)].len();
           let mut rows = Vec::new();
           let mut col = Vec::new();

           $( col.push($x0); )*

           rows.push(col);

           $(
               let _width = [(); count!($($x)*)].len();
               if _width != _width0 {
                   panic!("Unequal col size: {} and {}", _width0, _width);
               }
               let mut col = Vec::new();
               $( col.push($x); )*
               rows.push(col);
           )*

           Matrix::<_>::from_vec(&rows)
       }};
   }

首先，我們實作 ``count!`` 巨集，這個巨集用來計算參數的長度，可用來協助 ``matrix!`` \
巨集。接著，在 ``matrix!`` 巨集中，分為兩個部分，一個是只有單列的矩陣，一個是多列的矩陣。\
根據不同的情境，實作不同的程式碼。單列矩陣較單純，請讀者自行閱讀。在多列矩陣的地方，要於\
每一列檢查列長度是否相等，若不相等，我們就強制結束程式。

本節的靈感源自於 Stackoverflow 上的\
`此討論串 <http://stackoverflow.com/questions/34304593/counting-length-of-repetition-in-macro>`_\ ，\
讀者可相互比較兩者實作上的異同處。
