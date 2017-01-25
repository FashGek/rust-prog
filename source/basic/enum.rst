**********************
Enum
**********************

和結構類似，Enum (列舉) 也是一種複合型別，列舉中的資料為程式設計者所指定的有限的數個可能性。\
列舉是相當實用的概念，Rust 的標準函式庫中也有許多場合使用到列舉。

=======================
建立列舉
=======================

Rust 的 enum 分為兩種，一種是無資料的 enum，一種是有資料的 enum。以下實例建立無資料的 enum：

.. code-block:: rust

   enum Day {
       Sunday,
       Monday,
       Tuesday,
       Wednesday,
       Thursday,
       Friday,
       Saturday,
   }

在我們這個例子中，雖然也可以用常數 (constant) 來達成類似的效果，但用 enum 較佳，因為建立 \
enum 時，也建立了新的型別，Rust 會利用 enum 的資訊幫我們檢查程式碼。

Rust 的 enum 也可以帶資料，如以下實例：

.. code-block:: rust

   pub enum Color {
       RGB { r: u8, g: u8, b: u8 },
       CMYK { c: f64, m: f64, y: f64, k: f64 },
       HSL { h: f64, s: f64, l: f64 },
   }

===========================
在 match 中使用 enum
===========================

在 ``match`` 中使用到 enum 時，同樣需窮舉所有的可能性。如下：

.. code-block:: rust

   enum Day {
       Sunday,
       Monday,
       Tuesday,
       Wednesday,
       Thursday,
       Friday,
       Saturday,
   }

   fn main() {
       let day = Day::Sunday;

       match day {
           Day::Saturday | Day::Sunday => {
               println!("Relax yourself");
           }
           _ => {
               println!("Work hardly")
           }
       }
   }

===========================
結合資料和行為
===========================

如同 struct，enum 也可以將資料和行為結合，利用物件導向的機制建立自己的型別系統。我們將於後續\
章節介紹相關內容。
