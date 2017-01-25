*****************
變數和型別
*****************

在本章，我們介紹 Rust 程式的基本概念，包括 Rust 程式的組成、變數和型別。前幾章的程式，大部分\
都很簡單，但仍建議讀者實際練習一次；即使只是看著書照著打一次，都會有一些些幫助，因為 (1) 藉由\
這個過程熟悉建立 Rust 程式的過程，(2) 對於一些初階的錯誤，像是忘了加分號或打錯函式名稱，藉由\
實際練習才會改善。透過閱讀得到知識的過程很快，但也很容易遺忘，透過肌肉操作得到知識的過程較慢，\
然而，一旦學會，記憶可維持較久。

=============================
再訪 Hello World
=============================

我們重新檢視 Hello World 範例：

.. code-block:: rust

   fn main() {
       println!("Hello, World");
   }

這個程式，分為兩個部分，一個是 ``main`` \ **函式 (function)**\ ，\
一個是 ``println!("Hello, World");`` \ **敘述 (statement)**\ 。\
函式是一種程式碼再利用的方式，將程式碼組織成函式，可重覆呼叫，避免撰寫重覆的程式碼。\
``main`` 函式是一個特殊的函式，每個 Rust 主程式都會有一個 ``main`` 函式，而且也只能有一個 \
``main`` 函式。這個函式是程式的進入點。``main`` 函式固定的形式如下：

.. code-block:: rust

   fn main() {

   }

現階段，我們不需要在意函式的其他相關細節，只要記得程式碼要寫在 ``main()`` 函式中即可。

``println!`` 是一個\ **巨集 (macro)**\ ，其作用為在終端機 (console) 印出字串及附加\
換行 (newline) 符號。在我們的範例中，``println!("Hello, World")`` 就會印出 \
*Hello, World* 字串。巨集是一種特殊的函式，在 Rust，巨集會用驚嘆號 ``!`` \
和一般的函式區別開來。現階段，我們不需要了解巨集的細節，就當成現有的函式即可。

``println!("Hello, World");`` 是一條敘述。程式由許多敘述組成，每條敘述的最後面會\
加上分號 ``;``。在預設情形下，程式會由上往下，依序執行敘述。見以下範例：

.. code-block:: rust

   fn main() {
       println!("Hello, Rustacean.");
       println!("Welcome to Rust Programming.");
   }

在本程式中，有兩條敘述，而本程式會由上往下依序印出 *Hello, Rustacean.* 和 \
*Welcome to Rust Programming.* 這兩行字串。

===================
註解
===================

在撰寫程式時，\ **註解 (comment)** 是給程式設計者看的，不會影響程式的運行。Rust 有三種註解，\
分別為：

* 單行註解 (line comments)：以 ``//`` 開頭，之後該行文字皆視為註解
* 多行註解 (block comments)：以一對 ``/*`` 和 ``*/`` 包起來，可跨越多行
* 文件註解 (doc comments) 以 ``///`` 開頭，製作程式說明文件時使用

以下為範例：

.. code-block:: rust

   /* main function is program entry point.
      Each binary should contain one and
      only one main function. */
   fn main() {
       // Print out the string Hello World in console.
       println!("Hello, World");
   }

雖然程式碼本身就足以說明程式運作的過程，良好的註解可使得閱讀程式碼更有效率。\
本書會加入少量的註解以說明程式的運作。

======================
關鍵字
======================

每個程式語言都會有自己的關鍵字 (keyword)，關鍵字在程式碼中被賦予特殊的意義，\
不能作為其他用途，像是 ``fn`` 表示定義一個函式。以下是 Rust 的關鍵字：

.. code-block:: text

   abstract alignof  as       become
   box      break    const    continue
   crate    do       else     enum
   extern   false    final    fn
   for      if       impl     in
   let      loop     macro    match
   mod      move     mut      offsetof
   override priv     proc     pub
   pure     ref      return   Self
   self     sizeof   static   struct
   super    trait    true     type
   typeof   unsafe   unsized  use
   virtual  where    while    yield

讀者不需要去背誦關鍵字，筆者在學習程式設計的過程中，也不會刻意去記憶關鍵字。因為 \
(1) 編輯器或 IDE 會用顏色提示使用者那些部分是關鍵字 (2) 持續使用某個語言一段時間後，\
就會自然記住 (3) 忘記時再查閱即可，像是每個程式語言的 ``else if`` \
的寫法都略有不同，過一陣子沒用就忘了，也不需要刻意去記。

==================
變數
==================

----------------------
使用變數
----------------------

在程式語言中，\ **變數 (variable)** 的作用在於連結資料 (data)，在後續的程式碼中可再使用。\
見以下範例：

.. code-block:: rust

   fn main() {
       let name = "Michael";
       println!("Hello, {}", name);
   }

在以上程式中，我們定義了變數 ``name``\ ，其值為字串 *Michael*\ ，接著，我們將\
此變數傳入 ``println!`` 函式中印出。\ ``"Hello, {}"`` 的寫法是\
**字串安插 (string interpolation)**\ ，會在後面接續變數或資料。\
``let`` 是 Rust 的關鍵字，其作用為定義變數。

我們再看另一個範例：

.. code-block:: rust

   fn main() {
       let greeting = "Goodbye";
       let name = "Michael";
       println!("{}, {}", greeting, name);
   }


在這個程式中，我們安插兩個變數進入 ``println!`` 函式中，結果會印出 \
*Goodbye, Michael* 字串。

如果變數沒有值，會發生什麼事呢？見以下範例：

.. code-block:: rust

   fn main() {
       let x;
       println!("x = {}", x);
   }

這個程式會引發下列錯誤：

.. code-block:: console

   error[E0282]: unable to infer enough type information about `_`

這裡隱含著兩個概念，首先，變數要有相對應的值，否則會引發錯誤，再來，就是\ **型別 (type)** 的觀念。\
程式語言的意義在於操作資料 (data)，大部分的程式語言會將資料分類到不同的型別，Rust 也有許多的\
型別，我們後續會提到型別的相關概念。但是，即使我們加上型別的資訊，若沒有指定值，仍然會引發\
錯誤。見以下範例：

.. code-block:: rust

   fn main() {
       let x: i32;
       println!("x = {}", x);
   }

這時候，引發了另一個錯誤：

.. code-block:: console

   error[E0381]: use of possibly uninitialized variable: `x`

這個錯誤告訴我們變數 ``x`` 未初始化，即未指定值。我們再改寫一下程式：

.. code-block:: rust

   fn main() {
       let x = 3;
       println!("x = {}", x);
   }

這次程式可正確地執行。雖然我們沒有在程式中明確指定 ``x`` 的型別，Rust 可由 ``3`` \
得知 ``x`` 為 ``i32`` (32 位元整數) 型別。這是由於 Rust 提供\
**型別推斷 (type inference)** 的功能，使得程式撰寫起在像 Python 等高階語言。

-----------------
變數名稱
-----------------

目前 Rust 的變數名稱採用以下規則：

- 第一個字元為英文或底線 `_`
- 第二個之後的字元為英文、數字或底線
- 只有單一的底線 `_` 不是變數

以下是合 Rust 規範的變數名稱：

- ``x``
- ``x1``
- ``a_long_variable``
- ``aLongVariable``
- ``_var``

對於較長的變數名稱，Rust 建議 snake case (像是 a_long_variable) 而非 \
camel case (像是 aLongVariable)。Rust 會對不符合其撰碼風格的變數或函式名稱\
發出警告訊息，但不會引發錯誤。雖然 Rust 支援 Unicode，但目前只能用英文字母\
來命名變數 \
(見 Rust `issue #28979 <https://github.com/rust-lang/rust/issues/28979>`_)。

-------------------
變數的可變性
-------------------

以下程式看似正常：

.. code-block:: rust

   fn main() {
       let x = 0;
       x = 3;  // Error
       println!("x = {}", x);
   }

但卻引發了下列錯誤：

.. code-block:: console

   error[E0384]: re-assignment of immutable variable `x`

Rust 和許多程式語言不同，在預設情形下，變數一旦賦值後就不能改變。然而，改變變數狀態\
是程式設計常見的功能，要如何處理呢？Rust 要求程式設計者必需明確指出某個變數是\
可變的 (mutable)。使用較安全的規範，是 Rust 的特色，程式設計者要試著去適應 Rust 的思維。

我們將程式改寫如下：

.. code-block:: rust

   fn main() {
       let mut x = 0;
       x = 3;
       println!("x = {}", x);
   }

這次程式即可正確執行。我們在程式中使用 ``mut`` 這個關鍵字使 Rust 知道我們的變數 ``x`` 是\
可變的。

==============
型別
==============

我們在前面的程式中，使用了 Rust 的變數宣告，卻沒有明確指定 Rust 的\ **型別 (type)**\ 。\
Rust 就像大部分的程式語言，有許多的型別。在程式設計中，資料型別規範該資料在程式中\
允許的操作，像是數字可以加、減、乘、除，字串可以相接等。Rust 定義了數個\
**基礎型別 (primitive types)**\ ，程式設計者可以直接對這些型別的資料進行 Rust 所定義的\
操作。除此之外，使用者也可以新增新的型別和其相關的操作。如果 Rust 可正確推斷型別時，\
不需明確給定型別，但有時仍要明確給定型別，故仍然要有型別的概念。

---------------
基礎型別
---------------

Rust 在內建語法中包括以下基礎型別：

* 布林 ``bool``
* 字元 ``char``
* 字串 ``str``
* 數字

  * 整數

    * 有號固定整數：包括 ``i8``、``i16``、``i32``、``i64`` 等
    * 無號固定整數：包括 ``u8``、``u16``、``u32``、``u64`` 等
    * 變動整數：包括 ``isize`` 和 ``usize`` 等

  * 浮點數：包括 ``f32`` 和 ``f64`` 等

* 陣列
* Slice
* Tuple

-----------
布林
-----------

Rust 內建布林 (boolean) 值，包括 ``true`` 和 ``false`` 兩種值。\
布林主要用於\ **條件句 (conditional)**\ ，後續的章節會說明。

-----------
字元
-----------

字元代表單一的 Unicode scalar value，字元以一對單引號 ``'`` 括起來。

.. code-block:: rust

   fn main() {
       let x = 'x';
   }

-------
字串
-------

Rust 中有兩種字串型別，一種是 String 類別，一種是 ``str``\ 。我們將於後續章節介紹\
字串的使用。

----------
數字
----------

Rust 有數種數字型別，主要可分為：

- 有號 (signed) 及無號 (unsigned)
- 固定 (fixed) 和變動 (variable)
- 整數 (integer) 和浮點數 (floating point number)

有號和無號整數的差別在於是否有帶正負號，這會影響該數字的最小值和最大值。\
例如，``i8`` 的最小值為 -128，最大值為 127，而 ``u8`` 的最小值為 0，最大值為 255。\
固定整數有一定的位元數，而變動整數的位元數會因平台而有所不同。浮點數有兩種，分別對應 \
IEEE-754 單精確度和雙精確度浮點數。

以下程式列出 Rust 的每個數字型別的最小值和最大值，讀者可自行在自己的電腦上嘗試。

.. code-block:: rust

   // Call related modules in standard library
   use std::{i8, i16, i32, i64, isize};
   use std::{u8, u16, u32, u64, usize};
   use std::{f32, f64};

   fn main() {
       // signed, fixed-width integers
       println!("i8 min: {}, max: {}", i8::min_value(), i8::max_value());
       println!("i16 min: {}, max: {}", i16::min_value(), i16::max_value());
       println!("i32 min: {}, max: {}", i32::min_value(), i32::max_value());
       println!("i64 min: {}, max: {}", i64::min_value(), i64::max_value());

       // unsigned, fixed-width integers
       println!("u8 min: {}, max: {}", u8::min_value(), u8::max_value());
       println!("u16 min: {}, max: {}", u16::min_value(), u16::max_value());
       println!("u32 min: {}, max: {}", u32::min_value(), u32::max_value());
       println!("u64 min: {}, max: {}", u64::min_value(), u64::max_value());

       // variable-width integers (platform dependant)
       println!("isize min: {}, max: {}",
                isize::min_value(),
                isize::max_value());
       println!("usize min: {}, max: {}",
                usize::min_value(),
                usize::max_value());

       // floating point numbers
       println!("f32 min: {}, max: {}, min positive: {}",
                f32::MIN,
                f32::MAX,
                f32::MIN_POSITIVE);
       println!("f64 min: {}, max: {}, min positive: {}",
                f64::MIN,
                f64::MAX,
                f64::MIN_POSITIVE);
   }

**溢位 (overflow)** 是程式在運算時，超過該型別的最大值；而\ **下溢 (underflow)** 則是\
程式在運算時，小於該數字型別的最小值。在 Rust 中，溢位或下溢會引發錯誤，這是較安全的設計。\
例如，以下程式引發溢位：

.. code-block:: rust

   use std::i32;

   fn main() {
       let mut n: i32 = i32::max_value();
       n = n + 1;  // Overflow
   }

顯示以下錯誤訊息：

.. code-block:: console

   thread 'main' panicked at 'attempt to add with overflow'

如果要計算的數字較大，需使用大數運算相關套件，如下例：

.. code-block:: rust

   // Call third-party package
   extern crate num;

   use num::bigint::ToBigInt;

   fn main() {
       let x = 2.to_bigint().unwrap();
       println!("{}", num::pow(x, 100));
   }

若讀者想實際執行本程式，需在 *Cargo.toml* 中加入外部套件，如下：

.. code-block:: text

   [dependencies]
   num = "0.1"

----------------------------
陣列、slice 和 tuple
----------------------------

這些為\ **容器 (container)**\ ，將於後續章節中介紹。
