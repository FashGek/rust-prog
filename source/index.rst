**********************************
Rust 程式設計從頭開始
**********************************

.. toctree::
   :caption: 目錄
   :maxdepth: 3

   about
   copyright
   author

====================
基礎篇
====================

在這個部分，我們介紹程式設計典範 (paradigm) 中的\ **指令式程式設計 (imperative programming)**\ ，\
包括\ **變數 (variable)**\ 、\ **型別 (type)**\ 、\ **運算子 (operator)**\ 、\
**控制流程 (control flow)** 等，也會介紹數種\ **容器 (container)**\ ，接著，介紹 **struct** 和 \
**enum** 等複合型別。我們會將程式碼進一步組織成\ **函式 (function)**\ ，再將其再包裝成\
**模組 (module)** 和\ **套件 (package)**，這也是大部分程式設計初學者所接觸的程式設計典範。在學完\
這個部分後，讀者就可以自行撰寫基本的 Rust 程式，試著用 Rust 解決自己的問題。

.. toctree::

   basic/intro
   basic/prior_work
   basic/variable_type
   basic/operator
   basic/control_flow
   basic/array_vector_slice
   basic/map_set
   basic/struct
   basic/enum
   basic/function
   basic/module_package
   basic/string

====================
進階篇
====================

在這個部分，我們介紹數個相對進階的主題，像是在 Rust 相當重要且具有特色的\
**所有權 (ownership)**\ ，還有\ **物件導向程式設計 (object-oriented programming)** 、\
**泛型程式設計 (generic programming)**\ 、\ **函數式程式設計 (functional programming)** \
等不同的程式設計典範，接著，會介紹\ **巨集 (macro)** 、相對低階的不安全 (unsafe) \
程式碼、\ **共時性 (concurrency)** 及異種語言合作。學習完基礎篇和進階篇兩部分的內容，\
才算是完整學習 Rust 大部分的語法。

.. toctree::

   advance/ownership
   advance/oop
   advance/generics
   advance/fp
   advance/macro
   advance/unsafe
   advance/concurrency
   advance/play_with_others
   advance/further_steps
