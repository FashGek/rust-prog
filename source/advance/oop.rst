*****************************
物件導向程式設計
*****************************

**物件導向程式設計 (OOP, object-oriented programming)** 是一種程式設計的\
模式 (paradigm)。由於物件導向是近代軟體開發的主流方法，許多程式語言從語法機制可直接\
支援，即使像是 C 這種視為非物件導向的語言，也可以用其他語法機制達到類似的效果 \
(可參考\ `這裡 <https://www.cs.rit.edu/~ats/books/ooc.pdf>`_\ )。
本章會先介紹一般性的物件導向的概念，再介紹如何以 Rust 實作。

===================
物件導向概論
===================

很多語言都直接在語法機制中支援物件導向，然而，每個語言支援的物件導向特性略有不同，\
像 C++ 的物件系統相當完整，而 Perl 的原生物件系統則相對原始。物件導向在理論上是和\
語言無關的程式設計模式，但在實務上卻受到語言特性的影響。學習物件導向時，除了學習在某個特定\
語言下的實作方式外，更應該學習其抽象層次的思維，有時候，暫時放下實作細節，從更高的視角看\
物件及物件間訊息的流動，對於學習物件導向有相當的幫助。

物件導向是一種將程式碼以更高的層次組織起來的方法。大部分的物件導向以\ **類別 (class)** \
為基礎，透過類別可產生實際的\ **物件 (object)** 或\ **實體 (instance)** ，\
類別和物件就像是餅乾模子和餅乾的關係，透過同一個模子可以產生很多片餅乾。物件擁有\
**屬性 (field)** 和\ **方法 (method)**，屬性是其內在狀態，而方法是其外在行為。\
透過物件，狀態和方法是連動的，比起傳統的程序式程式設計，更容易組織程式碼。

許多物件導向語言支援\ **封裝 (encapsulation)**，透過封裝，程式設計者可以決定物件的那些\
部分要對外公開，那些部分僅由內部使用，封裝不僅限於靜態的資料，決定物件應該對外公開的行為\
也是封裝。當多個物件間互動時，封裝可使得程式碼容易維護，反之，過度暴露物件的內在\
屬性和細部行為會使得程式碼相互糾結，難以除錯。

物件間可以透過\ **組合 (composition)** 再利用程式碼。物件的屬性不一定要是基本型別，也\
可以是其他物件。組合是透過\ **有... (has-a) 關係**\ 建立物件間的關連。例如，汽車物件有\
引擎物件，而引擎物件本身又有許多的狀態和行為。\ **繼承 (inheritance)** 是另一個再利用\
程式碼的方式，透過繼承，\ **子類別 (child class)** 可以再利用\
**父類別 (parent class)** 的狀態和行為。繼承是透過\ **是... (is-a) 關係**\ 建立\
物件間的關連。例如，研究生物件是學生物件的特例。然而，過度濫用繼承，容易使程式碼間高度\
相依，造成程式難以維護。可參考\ **組合勝過繼承 (composition over inheritance)** \
這個指導原則來設計自己的專案。

透過\ **多型 (polymorphism)** 使用物件，不需要在意物件的實作，只需依照其公開介面使用\
即可。例如，我們想用汽車物件執行開車這項行為，不論使用 Honda 汽車物件或是 Ford 汽車\
物件，都可以執行開車這項行為，而不需在意不同汽車物件間的實作差異。多型有許多種形式，如：

1. **特定多態 (ad hoc polymorphism)**\ ：

   - **函數重載 (functional overloading)**\ ：同名而不同參數型別的方法 (method)
   - **運算子重載 (operator overloading)** \ ： 對不同型別的物件使用相同\
     運算子 (operator)

2. **泛型 (generics)**\ ：對不同型別使用相同實作
3. **子類型 (Subtyping)**\ ：不同子類別共享相同的公開介面，不同語言有不同的繼承機制

以物件導向實作程式，需要從宏觀的角度來思考，不僅要設計單一物件的公開行為，還有物件間如何\
互動，以達到良好且易於維護的程式碼結構。除了閱讀本書或其他程式設計的書籍以學習如何實作\
物件外，可閱讀關於 **物件導向分析及設計 (object-oriented analysis and design)** 或是\
**設計模式 (design pattern)** 的書籍，以增進對物件導向的了解。

==============
類別
==============

在 Rust，以 struct 或 enum 定義可實體化的類別，同時，也定義出一個新的型別。見下例：

.. code-block:: rust

   struct Point {
       x: f64,
       y: f64
   }

   fn main() {
       let p = Point{ x: 3.0, y: 4.0 };

       println!("({}, {})", p.x, p.y);
   }

然而，只有屬性而沒有方法的類別不太實用，下文會介紹方法。另外，對於不可實體化的抽象類別，使用 \
trait 來達成；trait 在物件導向及泛型中都相當重要，我們會於後續相關章節中展示其使用方式。

=============
方法
=============

-----------------------------
公開方法和私有方法
-----------------------------

方法是類別或物件可執行的動作，\ **公開方法 (public method)** 可由物件外存取，而\
**私有方法 (private method)** 則僅能由物件內存取。以下為實例：

.. code-block:: rust

   // We use mod to create a non-public block
   mod lib {
       pub struct Car;

       impl Car {
           // Public method
           pub fn run(& self) {
               // Call private method
               self.drive();
           }
       }

       impl Car {
           // Private method
           fn drive(& self) {
               println!("Driving a car...");
           }
       }
   }

   fn main() {
       let car = lib::Car{};
       car.run();
   }

在本程式中，我們用 ``mod`` 建立一個非公開的程式區塊，在其中的程式碼，除非用 ``pub`` 明確\
指定該部分的存取為公開的，否則，一律視為私有的，這是 Rust 在安全性上的設計。

實作方法時，用 ``impl`` 區塊包住要實作的方法，在 ``impl`` 中，若該方法和某個物件\
相關，第一個參數要加上 ``& self`` 或是 ``&mut self``，視可變性而定。對沒有物件導向\
經驗的讀者來說，``self`` 是一個令人混淆的概念；基本上，``self`` 是一個特殊的變數，代稱\
物件，將方法加上 ``self`` 這個參數，代表該方法和某個物件相關。以本程式來說，\
``self.drive()`` 代表這個物件呼叫了 ``drive`` 方法。對應到主程式中，``self`` 代換為 \
``car`` 這個實際的物件實體。簡單的原則在於類別是物件的設計圖，而非物件。

在本程式中，對主程式來說，``drive`` 方法是不存在的，如果把 ``car.run()`` \
改為 ``car.drive()`` 則會引發以下錯誤：

.. code-block:: console

   error: method `drive` is private

由此可知，主程式的確無法存取 ``drive`` 方法。

---------------------------
Getter 和 Setter
---------------------------

在先前 Point 的實作，我們直接存取物件的資料，在物件導向中，這不是好的方法。比較好的方法，\
是將屬性私有化，而用公開方法存取。我們現在建立 Point 類別，這個類別代表 2D 上的點。首先，\
定義此類別及其屬性：

.. code-block:: rust

   pub struct Point {
       x: f64,
       y: f64
   }

實作\ **建構子 (constructor)** 的部分，建構子是一個特別的函式，用來初始化物件：

.. code-block:: rust

   impl Point {
       // Constructor, which is just a regular method
       pub fn new(x: f64, y: f64) -> Point {
           let mut p = Point{ x: 0.0, y: 0.0 };

           // Set the fields of Point through setters
           p.set_x(x);
           p.set_y(y);

           p
       }
   }

``new`` 用來實體化 ``Point`` 類別。在物件導向的術語中，稱此特殊方法為建構子；\
在 Rust，``new`` 只是普通的方法，將 ``new`` 改為別的名稱也行，讀者可以將以上程式的 \
``new`` 改為 ``create`` 或別的名稱看看，程式仍能正常運作。不過，一般還是會把實體化\
物件的方法稱為 ``new``，這是約定俗成的用法。

接著，實作 setters：

.. code-block:: rust

   impl Point {
       // Setter for x, private
       fn set_x(&mut self, x: f64) {
           self.x = x;
       }
   }

   impl Point {
       // Setter for y, private
       fn set_y(&mut self, y: f64) {
           self.y = y;
       }
   }

在我們的建構子中，我們沒有直接將資料指派到物件屬性，而另外用 setter 來指派；雖然在本例中，\
我們的 setter 沒有特別的行為，但日後我們需要對資料進行篩選或轉換時，只要修改 setter 方法\
即可，其他部分的程式碼則不受影響。在撰寫物件導向程式時，我們會儘量將修改的幅度降到最小。

接著，實作 getters：

.. code-block:: rust

   impl Point {
       // Getter for x, public
       pub fn x(& self) -> f64 {
           self.x
       }
   }

   impl Point {
       // Getter for y, public
       pub fn y(& self) -> f64 {
           self.y
       }
   }

最後，從外部程式使用此類別：

.. code-block:: rust

   fn main() {
       let p = Point::new(3.0, 4.0);

       println!("({}, {})", p.x(), p.y());
   }

由於本程式將物件屬性和 setter 皆設為私有，對外部程式來說，物件建立後不可修改。在撰寫\
物件時，除了必要的方法外，儘量不要公開物件的其他部分，將公開行為變為私有的，可能會導致\
程式無法順利執行，應盡量避免。

----------------------
帶有方法的 Enum
----------------------

使用 enum 也可以建立具有行為的類別。在以下實例中，我們建立 color model 類別。首先，宣告 \
Color 類別：

.. code-block:: rust

   pub enum Color {
       RGB { r: u8, g: u8, b: u8 },
       CMYK { c: f64, m: f64, y: f64, k: f64 },
       HSL { h: u16, s: f64, l: f64 },
   }

在本例中，我們宣告了 RGB (Red-Green-Blue)、CMYK (Cyan-Magenta-Yellow-Black)、\
HSL (Hue-Saturation-Lightness) 三種 color model，若有需要，也可以宣告其他的 color \
model。同一個 Color 物件，在同一時間內只會儲存其中一種 color model，不同 color model \
間有公式可以轉換，使用 enum 會比使用數個獨立的 struct 來得適合。

實作 RGB model 的建構子，由於 RGB model 是三個 8 位元非負整數來表示顏色的值，故用 \
``u8`` 來儲存：

.. code-block:: rust

   impl Color {
       // Constructor for RGB model
       pub fn from_rgb(r: u8, g: u8, b: u8) -> Color {
           Color::RGB{ r: r, g: g, b: b }
       }
   }

實作 CMYK model 的建構子，由於 CMYK 的值為小於等於一百的百分比，我們額外建立一個私有\
類別方法來檢查值的正確性：

.. code-block:: rust

   impl Color {
       // Constructor for CMYK model
       pub fn from_cmyk(c: f64, m: f64, y: f64, k: f64) -> Color {
           if !Self::is_valid_ratio(c) {
               panic!("Invalid Cyan value {} in CMYK model", c);
           }

           if !Self::is_valid_ratio(m) {
               panic!("Invalid Magenta value {} in CMYK model", m);
           }

           if !Self::is_valid_ratio(y) {
               panic!("Invalid Yellow value {} in CMYK model", y);
           }

           if !Self::is_valid_ratio(k) {
               panic!("Invalid Black value {} in CMYK model", k);
           }

           Color::CMYK{ c: c, m: m, y: y, k: k }
       }
   }

   impl Color {
       // Private class method used to validate ratio value
       fn is_valid_ratio(n: f64) -> bool {
           0.0 <= n && n <= 1.0
       }
   }

實作 HSL model 的建構子，在這裡，我們用整數儲存角度，其值為 ``[0, 360)`` 的區間。：

.. code-block:: rust

   impl Color {
       // Constructor for HSL model
       pub fn from_hsl(h: u16, s: f64, l: f64) -> Color {
           if !Self::is_valid_hue(h) {
               panic!("Invalid Hue value {} in HSL model", h);
           }

           if !Self::is_valid_ratio(s) {
               panic!("Invalid Saturation value {} in HSL model", s);
           }

           if !Self::is_valid_ratio(l) {
               panic!("Invalid Lightness value {} in HSL model", l);
           }

           Color::HSL{ h: h, s: s, l: l }
       }
   }

   impl Color {
       fn is_valid_hue(n: u16) -> bool {
           n < 360
       }
   }

實作 RGB color model 的 getter，若物件內部儲存的 color model 是不同類型，則依公式來\
轉換：

.. code-block:: rust

   impl Color {
       pub fn rgb(& self) -> (u8, u8, u8) {
           match *self {
               Color::RGB{r, g, b} => (r, g, b),
               Color::CMYK{c, m, y, k} => {
                   ((255.0 * (1.0 - c) * (1.0 - k)) as u8,
                    (255.0 * (1.0 - m) * (1.0 - k)) as u8,
                    (255.0 * (1.0 - y) * (1.0 - k)) as u8)
               }
               Color::HSL{h, s, l} => {
                   // Saturation == 0
                   if f64::abs(s) < 1e-10 {
                       return ((255.0 * l) as u8,
                               (255.0 * l) as u8,
                               (255.0 * l) as u8);
                   }

                   let v2: f64;
                   let v1: f64;

                   if l < 0.5 {
                       v2 = l * (1.0 + s);
                   } else {
                       v2 = (l + s) - s * l
                   }

                   v1 = 2.0 * l - v2;

                   ((255.0 * Self::hue2rgb(
                       v1, v2, ((h as f64)/360.0 + 1.0/3.0))) as u8,
                    (255.0 * Self::hue2rgb(
                       v1, v2, (h as f64)/360.0)) as u8,
                    (255.0 * Self::hue2rgb(
                       v1, v2, ((h as f64)/360.0 - 1.0/3.0))) as u8)
               }
           }
       }
   }

   impl Color {
       fn hue2rgb(v1: f64, v2: f64, h: f64) -> f64 {
           let mut _h = h;
           if _h < 0.0 {
               _h += 1.0;
           }

           if _h > 1.0 {
               _h -= 1.0;
           }

           if 6.0 * _h < 1.0 {
               return v1 + (v2 - v1) * 6.0 * _h;
           }

           if 2.0 * _h < 1.0 {
               return v2;
           }

           if 3.0 * _h < 2.0 {
               return v1 + (v2 - v1) * ((2.0 / 3.0 - _h) * 6.0);
           }

           v1
       }
   }

最後，從外部程式呼叫：

.. code-block:: rust

   fn main() {
       let green = Color::from_hsl(120, 1.0, 0.5);
       let (r, g, b) = green.rgb();
       println!("{} {} {}", r, g, b);  // 0 255 0
   }

在本例中，由 HSL 轉至 RGB 的公式較複雜，這裡不詳細講解，有興趣的讀者可自行查詢電腦圖學\
相關資料。

==============
解構子
==============

若要實作 Rust 類別的\ **解構子 (destructor)**\ ，實作 Drop trait 即可。由於 Rust 會\
自動管理資源，純 Rust 實作的類別通常不需要實作解構子，但有時仍需要實作 Drop trait，像是\
用到 C 語言函式配置記憶體，則需明確於解構子中釋放記憶體。本書將於後續章節展示 \
Drop trait 的實作。

=============
多型
=============

Rust 使用 trait 來實作多型。以下是一個實際的範例：

首先，用 trait 設立共有的公開方法：

.. code-block:: rust

   // Use Drive trait as a common type
   pub trait Drive {
       fn drive(&self);
   }

接著，建立三個不同的汽車類別，這三個類別各自實作 ``Drive`` trait：

.. code-block:: rust

   pub struct Toyota;

   impl Drive for Toyota {
       fn drive(&self) {
           println!("Driving a Toyota car");
       }
   }

   pub struct Honda;

   impl Drive for Honda {
       fn drive(&self) {
           println!("Driving a Honda car");
       }
   }

   pub struct Ford;

   impl Drive for Ford {
       fn drive(&self) {
           println!("Driving a Ford car");
       }
   }

最後，透過多型的機制由外部程式呼叫這三個類別：

.. code-block:: rust

   fn main() {
       let mut cars = Vec::new() as Vec<Box<Drive>>;
       cars.push(Box::new(Toyota));
       cars.push(Box::new(Honda));
       cars.push(Box::new(Ford));
       for c in &cars {
           c.drive();
       }
   }

在本例中，\ ``Toyota``\ 、\ ``Honda`` 和 ``Ford`` 三個類別實質上是各自獨立的，透過 \
``Drive`` trait，達成多型的效果，從外部程式的角度來說，這三個物件視為同一個型別，\
擁有相同的行為。由於 trait 無法直接實體化，必需藉助 ``Box<T>`` 等容器才能將其實體化，\
``Box<T>`` 會從\ **堆積 (heap)** 配置記憶體，並且不需要解參考，相當於 C/C++ 的 \
smart pointer。

========================
組合勝於繼承
========================

Rust 的 struct 和 enum 無法繼承，而 trait 可以繼承，而且 trait 支援\
**多重繼承 (multiple inheritance)** 的機制；trait 可提供介面和實作，但本身無法\
實體化，反之，struct 和 enum 可以實體化。Rust 用這樣的機制避開 C++ 的\
**菱型繼承 (diamond inheritance)** 問題，類似 Java 的 interface 的味道。

組合就是將某個類別內嵌在另一個類別中，變成另一個類別的屬性，然後再透過多型提供相同的\
公開介面，外部程式會覺得好像類別有繼承一般。

假設我們要設計一個 RPG 遊戲，有兩個類別，分別是 Creature 和 Character，而這兩者都可\
設定行動優先順序。首先，決定公開界面，為了簡化問題，這裡僅實作優先權的 getter/setter：

.. code-block:: rust

   pub trait Priority {
       fn get_priority(&self) -> i32;
       fn set_priority(&mut self, value: i32);
   }

接著，實作 ``Creature`` 類別：

.. code-block:: rust

   pub struct Creature {
       priority: i32,
       // Other fields omitted.
   }

   impl Creature {
       pub fn new() -> Creature {
           Creature{ priority: 0 }
       }
   }

   impl Priority for Creature {
        fn get_priority(&self) -> i32 {
            self.priority
        }

        fn set_priority(&mut self, value: i32) {
            self.priority = value;
        }
   }

接著，實作 ``Character`` 類別，在這裡，我們透過組合的機制將 ``Creature`` 類別變成 \
``Character`` 類別的一部分：

.. code-block:: rust

   pub struct Character {
       // Creature become a member of Character
      creature: Creature,

      // Other field omitted.
   }

   impl Character {
       pub fn new() -> Character {
          let c = Creature::new();
          Character{ creature: c }
       }
   }

   impl Priority for Character {
       fn get_priority(&self) -> i32 {
           self.creature.get_priority()
       }

       fn set_priority(&mut self, value: i32) {
           self.creature.set_priority(value);
       }
   }

最後，從外部程式呼叫這兩個類別：

.. code-block:: rust

   fn main() {
       let mut goblin = Creature::new();
       let mut fighter = Character::new();

       println!("The priority of the goblin is {}", goblin.get_priority());
       println!("The priority of the fighter is {}", fighter.get_priority());

       println!("Set the priority of the fighter");
       fighter.set_priority(2);
       println!("The priority of the fighter is {} now",
           fighter.get_priority());
   }

在本例中，\ ``Creature`` 是 ``Character`` 的屬性，但從外部程式看來，無法區分兩者是\
透過繼承還是組合得到相同的行為。在 Rust 和 Go 這兩個新興語言中，不約而同拿掉繼承的\
機制，改用組合搭配多型達到類似的效果，這樣的設計，是語言的進步還是語言的缺陷，就留給各位\
讀者自行思考。

前文提到 struct 無法繼承，而 trait 可以繼承。以下展示一個繼承 trait 的實例。首先，\
定義 ``Color`` 和 ``Sharp`` trait，接著由 ``Item`` trait 繼承前兩個 trait：

.. code-block:: rust

   // Parent trait
   pub trait Color {
       fn color<'a>(&self) -> &'a str;
   }

   // Parent trait
   pub trait Shape {
       fn shape<'a>(&self) -> &'a str;
   }

   // Child trait
   pub trait Item: Color + Shape {
       fn name<'a>(&self) -> &'a str;
   }

實作 ``Orange`` 類別，該類別實作 ``Color`` trait：

.. code-block:: rust

   pub struct Orange {}

   impl Color for Orange {
        fn color<'a>(&self) -> &'a str {
            "orange"
       }
   }

實作 ``Ball`` 類別，該類別實作 ``Shape`` trait：

.. code-block:: rust

   pub struct Ball {}

   impl Shape for Ball {
       fn shape<'a>(&self) -> &'a str {
           "ball"
       }
   }

實作 ``Basketball`` 類別，該類別透過多型機制置入 ``Color`` 和 ``Shape`` 兩種類別：

.. code-block:: rust

   pub struct Basketball {
        color: Box<Color>,
        shape: Box<Shape>,
   }

   impl Basketball {
       pub fn new() -> Basketball {
           let orange = Box::new(Orange{});
           let ball = Box::new(Ball{});
           Basketball{ color: orange, shape: ball }
       }
   }

   impl Color for Basketball {
       fn color<'a>(&self) -> &'a str {
           self.color.color()
       }
   }

   impl Shape for Basketball {
       fn shape<'a>(&self) -> &'a str {
           self.shape.shape()
       }
   }

   impl Item for Basketball {
       fn name<'a>(&self) -> &'a str {
           "basketball"
       }
   }

最後，透過外部程式呼叫 ``Basketball`` 類別：

.. code-block:: rust

   fn main() {
       let b: Box<Item>= Box::new(Basketball::new());
       println!("The color of the item is {}", b.color());
       println!("The shape of the item is {}", b.shape());
       println!("The name of the item is {}", b.name());
   }

在本程式中，\ ``Item`` 繼承了 ``Color`` 和 ``Shape`` 的方法，再加上自身的方法，共有\
三個方法。在實作 ``Basketball`` 物件時，則需要同時實作三者的方法；在我們的實作中， \
``color`` 和 ``shape`` 這兩個屬性實際上是物件，由這些內部物件提供相關方法，但由外部程式\
看起來像是靜態的屬性，這裡利用到物件導向的封裝及組合，但外部程式不需擔心內部的實作，展現\
物件導向的優點。

在撰寫物件導向程式時，可將 trait 視為沒有內建狀態的抽象類別。然而，trait 類別本身不能\
實體化，要使用 ``Box<T>`` 等容器來實體化。這些容器使用到泛型的觀念，在本章，我們暫時\
不會深入泛型的概念，基本上，透過泛型，同一套程式碼可套用在不同型別上。

==================
方法重載
==================

方法重載指的是相同名稱的方法，但參數不同。Rust 不支援方法重載，如果有需要的話，可以用泛型\
結合抽象類別來模擬，可見泛型章節中的範例。

================
運算子重載
================

Rust 的運算子重載利用了內建 trait 來完成。每個 Rust 的運算子，都有相對應的 trait，在\
實作新類別時，只要實作某個運算子對應的 trait，就可以直接使用該運算子。運算子重載並非\
物件導向必備的功能，像是 Java 和 Go 就沒有提供運算子重載的機制。善用運算子重載，可減少\
使用者記憶公開方法的負擔，但若運算子重載遭到濫用，則會產生令人困惑的程式碼。

在本範例中，我們實作有理數 (rational number) 及其運算，為了簡化範例，本例僅實作加法。\
首先，宣告有理數型別，一併呼叫要實作的 trait：

.. code-block:: rust

   // Trait for binary '+' operator
   use std::ops::Add;

   // Trait for formatted string
   use std::fmt;

   // Automatically implement Copy and Clone trait
   // Therefore, our class acts as a primitive type
   #[derive(Copy, Clone)]
   pub struct Rational {
       num: i32,
       denom: i32,
   }

實作其建構子，在內部，我們會求其最大公約數後將其約分：

.. code-block:: rust

   impl Rational {
       pub fn new(p: i32, q: i32) -> Rational {
           if q == 0 {
               panic!("Denominator should not be zero.");
           }

           let d = Rational::gcd(p, q).abs();

           Rational{ num: p / d, denom: q / d }
       }
   }

   impl Rational {
       fn gcd(a: i32, b: i32) -> i32 {
           if b == 0 {
               a
           } else {
               Rational::gcd(b, a % b)
           }
       }
   }

實作加法運算，在這裡，實作 Add trait，之後就可以直接用 ``+`` 運算子來計算：

.. code-block:: rust

    // Implement binary '+' operation
    impl Add for Rational {
        type Output = Rational;

        fn add(self, other: Rational) -> Rational {
             let p = self.num * other.denom + other.num * self.denom;
             let q = self.denom * other.denom;
             Rational::new(p, q)
         }
     }

我們額外實作 Display trait，方便我們之後可以直接從終端機印出有理數：

.. code-block:: rust

   impl fmt::Display for Rational {
       fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
           let is_positive = (self.num > 0 && self.denom > 0)
               || (self.num < 0 && self.denom < 0);

           let sign = if is_positive { "" } else { "-" };

           write!(f, "{}{}/{}", sign, self.num.abs(), self.denom.abs())
       }
   }

最後，從外部程式呼叫此有理數物件。由於我們實作了相關的 trait，使用起來很類似基礎型別：

.. code-block:: rust

   fn main() {
        let a = Rational::new(-1, 4);
        let b = Rational::new(1, 5);

        // Call operator-overloaded binary '+' operator
        let y = a + b;

        // Call operator-overloaded formated string
        println!("{} + {} = {}", a, b, y);
   }

如果我們繼續將這個範例發展下去，就可以實作有理數的運算系統，不過，這只是展示運算子重載的\
範例，我們就把實作有理數的任務留給有興趣的讀者。在本例中，我們實作了兩個 trait，Rust 自動\
幫我們實作兩個 trait，使得外部程式使用起來相當接近操作內建形別的過程，體現運算子重載的優點。
