***************************
泛型程式設計
***************************

有時候，我們希望同一個實作可以套用在不同的型別上，在動態型別的語言中，例如 Python，不需要\
處理這個問題，因為這些語言的機制會自動處理這個問題，然而，在靜態型別的語言，像是 C++、Java \
或本書中的 Rust，無法自動套用不同型別。其中一個方式就是透過\ **泛型 (generics)** 的機制來\
達到這樣的效果。假設我們想實作一個向量運算的類別，如果沒有泛型，可能的 Rust 虛擬碼如下：

.. code-block:: rust

   struct IntVector<'a> {
       vec: &'a [i32],
   }

   // Implement algorithm for IntVector

   struct FloatVector<'a> {
       vec: &'a [f64],
   }

   // Implement algorithm for FloatVector

由以上範例可知，我們要針對不同型別重覆撰寫兩套相似的程式碼，而且，我們的實作缺乏擴充性，\
日後若要進行有理數 (rational number) 或是複數 (complex number) 或其他型別的運算，又得\
重覆撰寫相似的程式碼。泛型提供較佳的機制來解決這個問題，以泛型改寫上述例子的 Rust 虛擬碼如下：

.. code-block:: rust

   struct Vector<'a, T: 'a> {
       vec: &'a [T],
   }

   // Implement algorithm for Vector with generic type T

之後，要使用此泛型類別時，只要指定型別 ``T`` 即可使用這個類別。由以上範例可知，若我們實作出\
泛型程式後，就可以套用在不同型別上，達到程式碼再利用的效果。

==============================
使用泛型的例子
==============================

在 Rust 的標準函式庫中，已有許多泛型程式的例子，像是 vector、map、set 等容器，在本書\
先前的內容中，已有一些使用容器的實例。另外，Rust 有一些特殊類別，內部也用到泛型的機制，像是 \
``Result<T, E>`` 就是一個泛型 enum。以下是使用 ``Result`` 的實例：

.. code-block:: rust

   fn main() {
       let num = match "12345".parse::<u32>() {
           Ok(n) => n,
           Err(_) => panic!("Unable to parse integer"),
       };

       assert_eq!(num, 12345);
   }

在本例中， ``"12345".parse::<u32>()`` 使用泛型的語法指定解析字串的類別。

==============================
撰寫泛型程式
==============================

泛型程式可以用在函式或是物件的撰寫，在本節中，我們分別以泛型函式和泛型物件展示如何撰寫泛型程式。

------------------
泛型函式
------------------

泛型函式的 Rust 虛擬碼如下：

.. code-block:: rust

   fn foo<T>(x: T)

如果有兩個以上同型別參數，則 Rust 虛擬碼如下：

.. code-block:: rust

   fn bar<T>(x: T, y: T)

如果有兩個以上不同型別的參數，則 Rust 虛擬碼如下：

.. code-block:: rust

   fn baz<T, U>(x: T, y: U)

以下是一個泛型函式的例子，為了簡化程式，我們引用 ``Num`` trait，這個 trait 代表該泛型變數\
為數字。

.. code-block:: rust

   // For a trait of general number
   extern crate num;

   fn add<T>(a: T, b: T) -> T where T: num::Num {
       a + b
   }

   fn main() {
       assert_eq!(5, add(3, 2));
   }

-------------------
泛型物件
-------------------

泛型物件的 Rust 虛擬碼如下：

.. code-block:: rust

   struct Foo<T> {
       x: T,
       y: T
   }

如果需要實作某個物件的方法，Rust 虛擬碼如下：

.. code-block:: rust

   impl<T> Foo<T> {
      fn do_something(x: T, ...) -> ... {
          // Implement method here
      }
   }

如果要實作某個 trait 也可以：

.. code-block:: rust

   // Say that Bar is a trait
   impl<T> Bar for Foo<T> {
       fn method_from_bar(x: T, ...) -> ... {
           // Implement method here
       }
   }

以下是一個泛型物件的實例：

.. code-block:: rust

   use std::fmt;

   pub struct Point<T> where T: Copy + fmt::Display {
       x: T,
       y: T
   }

   impl<T> Point<T> where T: Copy + fmt::Display {
       pub fn new(x: T, y: T) -> Point<T> {
           Point::<T>{ x: x, y: y }
       }
   }

   impl<T> Point<T> where T: Copy + fmt::Display {
       pub fn x(&self) -> T {
           self.x
       }
   }

   impl<T> Point<T> where T: Copy + fmt::Display {
       pub fn y(&self) -> T {
           self.y
       }
   }

   impl<T> fmt::Display for Point<T> where T: Copy + fmt::Display {
       fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
           write!(f, "({}, {})", self.x(), self.y())
       }
   }

   fn main() {
       let p1 = Point::<i32>::new(3, 4);
       println!("{}", p1);

       let p2 = Point::<f64>::new(2.4, 3.6);
       println!("{}", p2);
   }

實際撰寫泛型程式時，設定相關的 trait 相當重要，Rust 需要足夠的資訊來判斷泛型中的變數是否能夠\
執行特定的行為，而這個資訊是透過 trait 來指定。

===========================
實例：實作向量運算
===========================

接下來，我們用一個比較長的例子展示如何實作泛型程式。在我們這個例子中，我們實作向量類別，這個\
類別可以進行向量運算；為了簡化範例，我們僅實作向量加法。首先，建立 ``Vector`` 類別，內部使用 \
Rust 內建的 vector 來儲存資料，在這裡一併呼叫相關的 trait：

.. code-block:: rust

   // For a trait of general number
   extern crate num;

   use std::fmt;
   use std::ops::Add;

   pub struct Vector<T> where T: Copy + fmt::Display + num::Num {
        vec: Vec<T>,
   }

接著，實作 ``Clone`` trait，使得本向量類別可以像基礎型別般，在計算時拷貝向量，由於 \
Rust 的限制，目前不能實作 ``Copy`` trait。

.. code-block:: rust

   // Currently, Copy trait cannot be implemented
   // impl<T> Copy for Vector<T> {}

   impl<T> Clone for Vector<T> where T: Copy + fmt::Display + num::Num {
        fn clone(&self) -> Vector<T> {
            let mut vec: Vec<T> = Vec::new();

            for i in 0..(self.vec.len()) {
                vec.push(self.vec[i]);
            }

            Vector::<T>{ vec: vec }
        }
    }

我們的建構子可接受 slice，簡化建立物件的流程：

.. code-block:: rust

   // Constructor
   impl<T> Vector<T> where T: Copy + fmt::Display + num::Num {
       pub fn from_slice(v: &[T]) -> Vector<T> {
           let mut vec: Vec<T> = Vec::new();

           for i in 0..(v.len()) {
                vec.push(v[i])
           }

           Vector::<T>{ vec: vec }
       }
   }

實作 ``fmt::Debug`` trait，之後可直接從 console 印出本類別的內容。這裡實作的方式參考\
Rust 的 vector 在終端機印出的形式。

.. code-block:: rust

   // Overloaded debug string
   impl<T> fmt::Debug for Vector<T> where T: Copy + fmt::Display + num::Num {
       fn fmt(&self, f:&mut fmt::Formatter) -> fmt::Result {
           let mut s = String::new();

           s += "[";

           for i in 0..(self.vec.len()) {
               s += &format!("{}", self.vec[i]);

               if i < self.vec.len() - 1 {
                   s += ", ";
               }
           }

           s += "]";

           // Write string to formatter
           write!(f, "{}", s)
       }
   }

實作加法運算子，需實作 ``std::ops::Add`` trait。向量加法的方式是兩向量間同位置元素\
相加，相加前應檢查兩向量是否等長。

.. code-block:: rust

   // Overloaded binary '+' operator
   impl<T> Add for Vector<T> where T: Copy + fmt::Display + num::Num {
       type Output = Vector<T>;

       fn add(self: Vector<T>, other: Vector<T>) -> Vector<T> {
           if self.vec.len() != other.vec.len() {
               panic!("The length of the two vectors are unequal");
           }

           let mut v: Vec<T> = Vec::new();

           for i in 0..(self.vec.len()) {
               v.push(self.vec[i] + other.vec[i]);
           }

           Vector{ vec: v }
       }
   }

最後，從外部程式呼叫此類別：

.. code-block:: rust

   fn main() {
       let v1 = vec![1, 2, 3];
       let v2 = vec![2, 3, 4];

       let vec1 = Vector::<i32>::from_slice(&v1);
       let vec2 = Vector::<i32>::from_slice(&v2);

       // We have to explictly clone our Vector object at present.
       let vec3 = vec1.clone() + vec2.clone();

       println!("{:?}", vec1);
       println!("{:?}", vec2);
       println!("{:?}", vec3);
   }

若我們將這個範例繼續發展下去，就可以實作具有泛型機制的向量運算類別，有興趣的讀者可以自行\
嘗試。由於 Rust 為了保持函式庫的相容性，現階段不允許對 non-Copy data 實作 Copy trait，\
像是本例的向量類別內部使用的 vector，所以，我們必需要在外部程式中明確地拷貝向量類別。經\
筆者實測，對於有解構子的類別也不能使用 Copy trait，所以，即使我們用 C 風格的陣列重新實作 \
vector，同樣也不能用 Copy trait。

另外，我們在這裡用了一個外部函式庫提供 ``Num`` trait，這個 trait 代表該型別符合數字，\
透過使用這個 trait，不需要重新實作代表數字的 trait，簡化我們的程式。

剛開始寫 Rust 泛型程式時，會遭到許多錯誤而無法順利編譯，讓初學者感到挫折。解決這個問題的關鍵\
在於 Rust 的 trait 系統。撰寫泛型程式時，若沒有對泛型變數 ``T`` 加上任何的 trait 限制，\
Rust 沒有足夠的資訊是否能對 ``T`` 呼叫相對應的內建 trait，因而引發錯誤訊息。即使是使用\
運算子，Rust 也會呼叫相對應的 trait；因此，熟悉 trait 的運作，對撰寫泛型程式有相當的幫助。

========================================
(案例選讀) 模擬方法重載
========================================

Rust 不支援方法重載，不過，可以利用泛型加上多型達到類似的效果。由於呼叫泛型函式時，不需要明確\
指定參數的型別，使得外部程式在呼叫該函式時，看起來像是方法重載般。接下來，我們以一個範例來展示\
如何模擬方法重載。首先，定義公開的 trait：

.. code-block:: rust

   use std::fmt;

   // An holder for arbitrary type
   pub trait Data: fmt::Display {
        // Omit interface
        // You may declare more methods later.
   }

   pub trait IntoData {
        type OutData: Data;

        fn into_data(&self) -> Self::OutData;
   }

接著，實作 ``Reader`` 類別，在這個類別中，實作了一個泛型函式，搭配先前的 trait 類別來模擬\
方法重載。

.. code-block:: rust

   pub struct Reader {}

   // Use generic method to mimic functional overloading
   impl<'a> Reader {
        pub fn get_data<I>(& self, data: I) -> Box<Data + 'a>
        where I: IntoData<'a> + 'a {
            Box::new(data.into_data())
        }
   }

接著，實作 ``StrData`` 類別，這個類別會實作 ``Data`` 和 ``IntoData`` 這兩個 trait，以\
滿足前述介面所定義的行為。

.. code-block:: rust

   pub struct StrData<'a> {
        str: &'a str
   }

   impl<'a> StrData<'a> {
       pub fn new(s: &'a str) -> StrData<'a> {
            StrData{ str: s }
       }
   }

   impl<'a> fmt::Display for StrData<'a> {
       fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
             write!(f, "{}", self.str)
       }
   }

   impl<'a> Data for StrData<'a> {
       // Omit implementation
   }

   impl<'a> IntoData<'a> for StrData<'a> {
       type OutData = &'a str;

       fn into_data(&self) -> &'a str {
            self.str
       }
   }

   /* Even Data trait is empty, it is necessary to
      explictly implement it. */
   impl<'a> Data for &'a str {
       // Omit implementation
   }

接著，以類似 ``StrData`` 的方式實作 ``IntData``：

.. code-block:: rust

   pub struct IntData{
       int: i32
   }

   impl IntData {
       pub fn new(i: i32) -> IntData {
           IntData{ int: i }
       }
   }

   impl fmt::Display for IntData {
       fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
           write!(f, "{}", self.int)
       }
   }

   impl Data for IntData {
       // Omit implementation
   }

   impl IntoData for IntData {
        type OutData = i32;

        fn into_data(&self) -> i32 {
            self.int
        }
   }

   /* Even Data trait is empty, it is necessary to
      explictly implement it. */
   impl<'a> Data for i32 {
        // Omit implementation
   }

最後，從外部程式呼叫：

.. code-block:: rust

   fn main() {
       let reader = Reader{};
       let str_data = StrData::new("string data");
       let int_data = IntData::new(10);

       // Call hidden generic method to minic functional overloading
       let str = reader.get_data(str_data);
       let int = reader.get_data(int_data);

       println!("Data from StrData: {}", str);
       println!("Data form IntData: {}", int);
   }

在我們這個範例中，除了用泛型的機制模擬出方法重載以外，另外一個重點在於 ``get_data`` 函式\
隱藏了一些內部的操作，對於程式設計者來說，只要實作 ``Data`` 和 ``IntoData`` 後，從外部程式\
呼叫時，不需要在意其中操作的細節，這也是物件導向的優點之一。
