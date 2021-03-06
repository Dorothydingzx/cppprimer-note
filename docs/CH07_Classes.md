C++ 中最重要的特性之一就是类，通过类可以有效表达代解决的问题中的概念。类讲求数据抽象（data abstraction），即是将类的实现与类所能表达的功能分离开来。类背后的哲学就是数据抽象和封装（encapsulation）。

数据抽象意在将不必要的细节剥离出来以期更加接近问题的本质特性。数据抽象往往是设计系统的第一步，原因在于完整的系统非常复杂，几乎不能在没有一个简单抽象的情况下一步完成。数据抽象使得可以从最本质的特性开始然后一步步丰富到最终的系统。有一些别的术语与此相近，比如：建模（modeling）、泛化（generalization）。抽象是设计一些软件的开端，亦是最重要的武器。

封装则是两个相关但不同概念的组合：将对象状态隐藏使得外部无法直接访问；将数据和方法打包到一起；在一些语言中隐藏组件不是自动发生的，甚至不可能隐藏，因而，在这些语言中只有第二种含义。而第一种含义有另外一个术语表示：信息隐藏（information hiding）。

类使用数据抽象和封装来定义抽象数据类型（abstract data type，ADT），抽象数据类型使得可以以抽象的方式来思考，即仅需要知道类可以做什么，而不必深究类是怎么做到的，这些实现的细节留给设计者去考虑。

## 7.1 定义抽象数据类型

只有具有行为或者说接口的类型才能称为数据类型，如果只有数据而没有行为，所有行为都需要用户自行编写，那便不能称为抽象的数据类型。“行为”的含义了抽象机器的公理语义（axiomatic semantics）和操作语义（operational semantics），有些语境下还会包含计算复杂度（computational complexity）包括时间复杂度和空间复杂度。现实中的很多数据类型不是完美的抽象数据类型，原因在于机器现实，如：算数溢出等。抽象数据类型是计算机科学中的理论概念，不对应于任何特定的语言特性。这个概念被运用于设计和分析算法，数据结构和软件系统。许多概念类似于抽象数据类型：抽象类型（abstract types）、透传数据类型（opaque data types）、协议（protocols）和按约定设计（design by contract）。

[https://en.wikipedia.org/wiki/Abstract_data_type](https://en.wikipedia.org/wiki/Abstract_data_type)

### 7.1.1 设计类

设计类时需要分清楚两种不同的编程角色：设计者和用户。类的设计者负责设计接口和实现。用户则负责使用这些接口完成别的任务。不论在任何时候都需要清楚分离设计者和用户角色，这样能够很好的解耦从而使得程序演变更加容易。在一些简单的应用中，常常类的用户和设计者时同一个人，作为设计者应该努力思考让类更加容易使用，从而使得用户必须了解类的实现就能够很好使用类。设计良好的类通常接口时直观且易用的，并且实现地足够高效。

### 7.1.2 定义类

类中有一些函数是实现的一部分，这些函数不会作为接口的一部分，这些函数需要被定义为私有的。C++ 中会包含一些作为接口的非成员函数，这些函数将定义在类的外部。定义和声明成员函数（member functions）于普通函数类型，成员函数必须在类中声明，定义在类内或者类外。定义在类内的函数隐式称为内联函数。

示例代码：[use_sales_data.cc](https://github.com/chuenlungwang/cppprimer-note/blob/master/code/use_sales_data.cc)

当调用一个对象的成员函数时，所有操作都作用在此对象上。如：
````cpp
string isbn() const { return bookNo; }
````
当调用 `total.isbn()` 时，返回的是 `total.bookNo`，成员函数通过一个额外的隐式参数 this 来访问调用对象。this 被默认初始化为函数调用对象的地址。此处 this 是 total 对象的地址。就如以下代码：`isbn(&total)`

在成员函数内可以直接使用成员名字而不需要使用成员访问符去访问，这种直接使用成员名字会被编译器隐式转换为 `this->bookNo` 形式。

this 参数由编译器隐式定义，并且不允许程序员定义参数或者变量名为 this ，this 总是指向当前调用的对象，所以，this 是一个 const 指针，我们不能改变 this 使其指向别的对象。

参数列表后的 const 用来说明 this 指针指向 const 对象。通常情况下 this 是指向非 const 对象的 const 指针，尽管 this 是隐式定义的，它也遵循初始化原则，即不能将非 const 的 this 指向 const 对象，从而不能在 const 对象上调用非 const 的函数。为了在 const 对象上调用成员函数，必须使得 this 指向 const 对象，如果 this 指针可以在参数列表中，那么它将形如：`const Sales_data *const` ，然而 this 是隐式定义的并且不可能出现在参数列表中，所以语言通过在参数列表后放置 const 来表明 this 指针指向 const 对象，从而使得此成员函数称为 const 成员函数（const member function）。isbn 函数可以描述为：
````cpp
string Sales_data::isbn(const Sales_data *const this)
{ return this->isbn; }
````
this 指针指向 const 对象，说明 const 成员函数不能改变调用对象的状态。const 对象、指向 const 对象的指针和绑定 const 对象的引用只能调用 const 成员函数。

**类作用域和成员函数**

类本身就是一个作用域，所以，成员函数的定义处于类作用域中，因而，当 isbn 不加任何限定符使用 bookNo 时，将被解析为使用 Sales_data 中的 isbn。值得注意的是这种使用甚至可以在 bookNo 的定义之前。编译器处理类是分两步走的：成员定义先处理，然后才是成员函数体。因此，成员函数可以使用其它成员而不必纠结它们出现的顺序。

**在类外部定义成员函数**

与任何别的函数一样，当在类外部定义成员函数时需要定义与声明完全一致。特别是参数列表后的 const 需要一致。除此之外，在类外定义的成员名字必须用类名加以限定。如：
````cpp
double Sales_data::avg_price() const {
    if (units_sold)
        return revenue/units_sold;
    else
        return 0;
}
````
Sales_data::avg_price 的含义就是说函数名字 avg_price 是声明在作用域 Sales_data 类中的。一旦编译器看到函数名字，剩下的代码将被解释为处于类作用域中。因而，revenue , units_sold 都是隐式解释为 Sales_data 的成员。

**定义函数返回“当前”对象**

成员函数返回当前调用对象的关键在于设置返回类型和书写返回语句。返回当前对象意味着需要返回当前对象的引用，而且需要在返回时对 this 进行解引用。如：
````cpp
Sales_data& Sales_data::combine(const Sales_data &rhs)
{
    units_sold += rhs.units_sold;
    revenue += rhs.revenue;
    return *this;
}
````
调用 `total.combine(trans);` 将返回 total 对象的引用。

### 7.1.3 定义类相关的非成员函数

类作者可以定义一些辅助函数，这些函数定义的操作在概念上属于类接口的一部分，但是本身不在类内。这些函数通常与类定义在同一个头文件中，使得用户只需要包含一个文件就可能够使用与类相关的任何接口部分。这些函数与普通的函数并没有什么区别，它们并不在类的作用域内，因而，没有隐式的 this 指针以及不能直接使用成员，甚至调用时是直接写出函数名，而不是由对象进行调用。

### 7.1.4 构造函数

每个类都定义该类的对象如何进行初始化，类通过定义一个或者多个特殊成员函数来控制初始化，称为构造函数（constructors）。构造函数的工作就是初始化类的数据成员，每当对象创建时构造函数就会调用。

构造函数和类的初始化是很复杂的，有一个总原则就是初始化所有数据成员，而不是依赖于编译器设置默认值。原因在于，编译器在不同的情况下会不会设置默认值是不一样的，而记住所有不同情况下的行为是很累人的。这些复杂的初始化参考：[https://en.cppreference.com/w/cpp/language/initialization](https://en.cppreference.com/w/cpp/language/initialization) 中值初始化（value initialization）、默认初始化（default initialization）和零初始化（zero initialization）。

构造函数与类名相同而且没有返回值，除此之外与常规函数没有别样。构造函数与常规函数一样可以进行重载。构造函数不能被声明为 const 的，原因在于即便是 const 对象亦需要在构造函数完成初始化之后才能变成常量。

**合成默认构造函数**

当定义对象时不给定初始值将执行默认初始化。类控制默认初始化的构造函数称为默认构造函数（default constructo），这个构造函数是没有任何参数的构造函数。其特殊性在于如果类没有显式定义任何构造函数，编译器会隐式定义一个默认构造函数，这个生成的默认构造函数称为合成默认构造函数（synthesized default constructor）。合成构造函数初始化每个数据成员的方式是：

- 如果有类内初始值，就用那个初始值进行初始化；
- 没有，则使用默认初始化，对于类成员将调用其默认构造函数，对于内置类型则不进行任何初始化；

事实上，默认初始化远远比上面提到的两句话要复杂。在 [Initialization in C++ is bonkers](https://blog.tartanllama.xyz/initialization-is-bonkers/) 一文中做出了详尽的解释，是目前阅读过的最好的一篇关于初始化的文章。如果要更加理解初始化过程中的内容，建议阅读 C++ 标准中相关章节。

**一些类不能依赖于合成默认构造函数**

只有十分简单的类可以依赖于合成的默认构造函数。最常见的原因在于编译器只在程序员没有为此类定义任何构造函数时才会生成。如果定义了任何构造函数，将必须手动定义默认构造函数，编译器不会再协助生成了。原因在于编译器认为如果程序员在一种情形下控制了对象的初始化，多半会需要在所有情况下控制对象的初始化。

第二个原因是合成的构造函数不能达到预想的效果。内置类型和复合类型（数组、指针）作为类成员，在默认初始化时不做任何事情。所以，内置类型或复合类型的成员都应该在类内初始化或者定义自己的默认构造函数。只有当内置类型或复合类型有类内初始值时才能依赖于合成默认构造函数。

第三个原因是有些类编译器根本无法合成默认构造函数。原因在于，一个类可以有一个没有默认构造函数的类类型成员，那么编译器将无法初始化此成员。

** `=default` 的含义**
````cpp
struct Sales_data {
    Sales_data() = default;
};
Sales_data::Sales_data() = default;
````
这种形式的默认构造函数执行与编译器合成的默认构造函数一样的功能。通过在参数列表后加上 `= default` 将要求编译器为我们生成默认构造函数。`=default` 可以出现在类内的声明处，或者出现在函数定义处。定义在类内则合成的默认构造函数自动称为内联的，定义在类外则是非内联的。这两种形式的差异还在于定义在类外的函数是用户提供（user-provided）的构造函数。用户提供的构造函数在对象是值初始化时，将执行默认初始化。而编译器生成的默认构造函数，在值初始化时，将先执行零初始化再执行默认初始化。

在 C++11 之后才提供类内初始化，在之前的 C++ 版本是不提供的，此时必须使用构造初始值列表（constructor initializer list）来初始化所有成员。

**构造初始值列表**
````cpp
Sales_data(const string &s, unsigned n, double p):
    bookNo(s), units_sold(n), revenue(p*n) {}
````
在参数列表后的冒号直到函数体的括弧之间的代码称为构造初始值列表（constructor initializer list），它的作用是给对象的成员提供初始值，它是一系列的成员名，在每个成员名后跟随括号或括弧中的初始值，多个成员初始值之间用逗号分隔。

当一个成员从构造初始值列表中省去时，它是默认初始化的，使用合成构造函数的初始化规则。

使用类内初始值的好处在于不必每次都在构造初始值列表中提供值。如果语言版本不支持的话，则需要在每个构造函数中初始化每一个内置类型的成员。

以上构造函数的函数体是空的，这在 C++ 中是很常见的，这个构造函数所做的工作就是给数据成员提供值。

**在类外定义构造函数**
````cpp
Sales_data::Sales_data(std::istream &is)
{
    read(is, *this);
}
````
定义在类外的构造函数没有返回类型，直接从名字开始，亦无返回语句。与其它成员函数一样，函数名字需要用类名进行限定。尽管此处构造初始值列表是空的，类成员依然在构造函数体执行之前先初始化，此时这些没有出现在构造初始值列表中的成员将被默认初始化或被类内初始值进行初始化。

### 7.1.5 拷贝、赋值和析构

除了定义类对象如何初始化，类还可以控制对象赋值（copy）、赋值（assign）和销毁（destroy）时的操作。当初始化变量或者传递参数、返回值时将发生对象复制。当使用赋值操作符时发生赋值操作。当离开定义对象所在的块时将销毁对象，当 vector 或数组被销毁时，其中元素亦被销毁。

如果程序员不定义这些操作的函数，编译器将合成这些函数。通常，编译器生成的版本就是将对象的每个成员分别进行拷贝、赋值和销毁。如编译器生成的 Sales_data 的赋值函数如：
````cpp
total = trans;
//////////////////
total.bookNo = trans.bookNo;
total.units_sold = trans.units_sold;
total.revenue = trans.revenue;
````

**有些类不能依赖于合成版本**

尽管编译器可以合成拷贝、赋值和析构函数，但需要记住的是默认版本的行为不一定是符合要求的。特别是当类在类对象本身外部分配了资源时（动态内存、数据库连接、TCP 连接），合成版本通常无法正确工作。通常如果类有分配动态内存，是不能依赖合成的版本。

值得提醒的时，很多需要动态内存管理的类可以使用 string 或 vector 来管理需要内存。使用这些组件的类避免了分配和回收内存的复杂度，而且合成的函数可以在此之上正确工作。当拷贝带有 vector 成员的对象时，vector 类会正确处理其中元素的拷贝和赋值。当销毁时，vector 将被销毁，其中的元素亦被销毁。

直到你知道如何正确定义以上三种函数时，尽量不要在类所占据内存外分配资源。

## 7.2 访问控制和封装

在 C++ 中使用访问说明符（access specifiers）来强制封装，即信息隐藏：

- 定义在 public 说明符后的成员程序的所有部分都可以访问的，public 成员是类的接口（interface）；
- 定义在 private 说明符后的成员只能由本类中其它部分访问，不能被使用类的代码使用，private 成员封装了类的实现；

一个类可以有零个或多个访问说明符，对于一个访问说明符出现的频率并没有限制。每个访问说明符说明接下来的成员访问级别，这些访问级别持续到下一个访问说明符的出现或者类的结尾。

**使用 class 或 struct 关键字**

使用 class 关键字或者 struct 关键字唯一的区别在于默认的访问级别。对于 struct 关键字而言，定义在第一个访问说明符前的成员是共有的，对于 class 而言则是私有的。即 class 的默认访问说明符是 private，而 struct 的默认访问说明符是 public 。

### 7.2.1 友元

尽管类相关的非类成员函数在概念上是类的一部分，但是依然需要遵循常规函数的规则，即不能访问类的私有成员。一个类可以通过将另一个类或函数定义为 friend 来使得其可以访问此类的非 public 成员。这样做只需要简单地将类或函数声明放在类内，并在之前加上 friend 关键字即可。如：
````cpp
class Sales_data {
    friend Sales_data add(const Sales_data&, const Sales_data&);
    friend istream &read(istream&, Sales_data&);
};
Sales_data add(const Sales_data&, const Sales_data&);
istream &read(istream&, Sales_data&);
````
友元声明需要放在类定义内，它们可以出现在类的任何地方。友元不是类的成员，因而，不受访问控制的影响。将所有的友元声明集中放在一起，或置于类定义头部或尾部，是一个好的处理方式。

**封装的益处**

使用封装可以防止被封装的对象状态被破坏；被封装的类的实现可以随着时间的推移进行修改而不需要涉及到用户代码的修改，api 的改变往往会导致用户的怨声载道。将数据定义为 private 的，类的作者可以按照自己的意愿改变其中数据。实现的概念只会影响到类中使用到此部分代码的行为，测试仅限于这些部分。而改变接口往往会涉及到用户代码的改变，所有依赖于旧接口的代码都会被打破，导致要重写所有这些使用旧接口的代码。

将数据设置为 private 的另一个好处是数据不会被用户无意中改变。如果有一个 bug 破坏了对象的状态，查找 bug 的地方将仅限于实现代码，从而极大简化了维护成本。

尽管封装避免了用户代码的改变，但改变类实现代码依然需要编译所有用到此类的源文件必须重新编译。

**声明友元**

友元声明只说明了访问权限，它并不是一个通用的函数声明。为了调用此友元函数，依然需要在类外部声明函数本身，如上面所做的那样。为了使友元函数为用户所见，通常在同一个头文件中再次声明这些函数。

许多编译器并不强制要求友元函数必须在类外再次声明，但并不是所有编译器都支持。为了最大的兼容性，最好在类外部再次声明函数。

## 7.3 其它类特性

本节将介绍类型成员，类类型成员的类内初始化，mutable 数据成员以及内联成员函数，从成员函数中返回 `*this` ，以及更多关于如何定义和使用类类型，以及友元类的知识。

### 7.3.1 类成员

**定义类型成员（Type Member）**

类可以使用 typedef 或 using 为类型定义自己的本地名字，由类定义的类型名字可以像其它成员一样定义其访问权限。如：
````cpp
class Screen {
public:
    typedef std::string::size_type pos;
};
````
在 public 部分定义 pos 类型，这样用户代码亦可以通过 `Screen::pos` 使用此名字。通过定义这种类型别名将隐藏类是如何实现的。其实，`std::string::size_type` 亦是一种类型别名，其真实类型将依据不同的平台定义，客户代码只需要知道此类型即可。

另外用 using 声明是一样的效果。如：
````cpp
class Screen {
public:
    using pos = std::string::size_type;
};
````
需要注意的一点是，与常规成员不同，类型别名定义需要出现在任何其被使用之前。因而，类型成员通常出现在类定义的顶部。

**将成员函数设为 inline **

类经常有一些小的成员函数可以从内联中获益，定义在类定义内的成员函数自动是内联的，这可以运用于构造函数以及常规成员函数。可以显式将成员函数声明为 inline 的，有两种方式：一种是在类定义内的成员函数声明处声明，另一种是在类外面的成员函数定义处声明。同时在两处都声明内联是合法的。书中认为仅在类外的定义处声明内联可以使得类更加易读。

内联的成员函数应该定义在头文件中，这与常规内联函数遵从一致规则。且应当和与其对应的类定义放在同一个头文件中。

**重载成员函数**

与常规函数一样，成员函数亦是可以重载的。而且函数匹配的规则与常规函数的匹配是一样的。

**mutable 数据成员**

偶尔 C++ 中的类的数据成员需要即便是在 const 成员函数中亦是可以改变的，这种数据成员需要用 mutable 关键字进行声明。mutable 数据成员永远不是 const 的，即便其在 const 对象中，因而，const 成员函数可以改变 mutable 成员。如：
````cpp
class Screen {
public:
    void some_member() const;
private:
    mutable size_t access_ctr;
};
void Screen::some_member() const
{
    ++access_ctr;
}
````

**类类型数据成员的初始值**

在新标准中，除了可以为内置类型数据成员设定类内初始值外，亦可以为类类型数据成员设定初始值。而且，在新标准中最好的设置默认值的方式就是类内初始值。如：
````cpp
class Window_mgr {
private:
    std::vector<Screen> screens{Screen(24, 80, ' ')};
};
````
当初始化类类型成员时，其实是通过提供实参给那个类的构造函数并调用而进行初始化。

类内初始值遵循一种格式规范即：要么用等号（=）号形式，要么用括弧形式进行初始化。除此之外的形式都是不合法的。

### 7.3.2 返回 `*this` 的函数

````cpp
inline Screen &Screen::set(pos r, pos col, char ch)
{
    contents[r*width+col] = ch;
    return *this;
}
````
函数返回引用即其结果为左值，意味着返回的是对象本身而不是对象的拷贝。如果将操作串连起来就是连续对此对象进行操作。如：
````cpp
myScreen.move(4, 0).set('#');
````
如果返回的是对象，而不是引用，那么以上操作中的 set 就是对返回的临时量进行操作，而不是 myScree，其形如：
````cpp
Screen temp = myScreen.move(4, 0);
temp.set('#');
````

**从 const 成员函数中返回 `*this`**

const 成员函数返回的 `*this` 是一个 const 引用，因为，this 是一个指向 const 对象的指针。这将导致其不能与返回 `*this` 的非 const 成员函数串连在一起。

**基于 const 进行重载**

````cpp
class Screen {
public:
    Screen &display(std::ostream &os);
    const Screen &display(std::ostream &os);
private:
    void do_display(std::ostream &os);
};
````

可以按照一个成员是否是 const 来进行重载，原因在于，this 指针指向的对象是否为 const 可以进行正确函数匹配。非 const 成员函数不能被 const 对象调用，const 对象只能调用 const 成员函数。然而，在非 const 对象上可以调用任意版本，但非 const 版本是更合适的，意味着如何同时存在 const 和非 const 成员函数，非 const 对象将调用非 const 版本，而 const 对象将只能调用 const 版本。

当一个成员函数调用另外一个成员函数时，this 指针被隐式传递。当一个非 const 成员函数调用 const 成员函数时，其 this 指针将被隐式转为指向 const 对象的指针。非 const 成员返回 `*this` 是常规引用，const 成员返回 `*this` 则是 const 引用。通过调用对象是否为 const 来决定哪个版本的函数被调用，以及返回什么样的引用。如：
````cpp
Screen myScreen(5, 3);
const Screen blank(5, 3);
myScreen.set('#').display(cout); //调用非 const 版本
blank.display(cout); //调用 const 版本
````

**建议：使用私有 utility 函数来提供公共功能**

在设计良好的 C++ 程序中有许多类似于 `do_display` 的小好函数来做真正的函数工作，接口则将其功能指派给这些工具函数。有几点好处：

- 避免了在超过一个地方书写相同的代码；
- 当类随着时间推移变得愈加复杂时，其功能也愈加复杂，将代码写在的一处的好处将更加明显；
- 将代码写在一处对于添加和删除调试代码更加简单；
- 通过将工具函数定义为 inline 的，消除了调用函数的额外运行时负担；

### 7.3.3 类类型

C++ 中的类按名字进行区分，每个类就是一个独特的类型。两个不同的类即便有完全一致的代码亦是不同类型。两个类中的成员是完全独立的。

使用类类型有两种方式：直接使用类的名字，或者在类名前面加上 class 或 struct，如：
````cpp
Sales_data item1;
class Sales_data item1;
````
以上两种使用类类型的方式是完全一样的。第二种方式是从 C 中继承过来的。

**类声明**

可以声明一个类而先不定义它，如：`class Screen;` 这种声明有时成为前向声明（forward declaration），将名字 Screen 引入到程序中，并指示 Screen 是一个类类型。当发生声明出现在定义之前时，类型 Screen 是未完成类型（incomplete type），它仅仅知道 Screen 是一个类类型，但不知道其成员是什么。

未完成类型的使用有诸多限制：可以定义这种类型的指针或引用，可以声明使用未完成类型作为参数或返回值的函数。

只有当知道类的定义时，才能书写创建这种类型对象的代码，否则，编译器不知应当如何存储对象。同样，在用指针或者引用访问类的成员时，必须知道类的定义。总之，如果类没有被定义，编译器便不知道类的成员是什么。

除了静态成员可以是未完成类型外，所有的数据成员只能被设定为已经定义了的类类型。只有当类的定义完成时，编译器才知道如何存储其数据成员。一个类只有到达类体的末尾时才算是完成了定义，一个类不能定义其本身类型的数据成员。然而，只要看到类名字，就算是声明了类，因而，可以定义本身类型的指针或引用。

### 7.3.4 友元再探

一个类可以使得另一个类作为其友元类，或者将指定特定的成员函数作为其友元。另外，一个友元函数可以定义在类的内部，这个函数则隐式成为内联的。

**类之间的友元关系**

友元类的成员函数可以访问授权类的所有成员，包括所有非公有的成员。需要记住的是友元关系是不可传递的。一个类是另一个类的友元并不意味着这个类自己的友元可以访问那个类，每个类控制着哪些函数或类是自己的友元。如：
````cpp
class Screen {
friend class Window_mgr;
};
````

**使成员函数成为友元**

除了可以使得整个类作为友元，类还可以使得特定的成员函数作为友元。当指定成员函数作为友元时需要指定是哪一个类的成员。如：
````cpp
class Screen {
friend void Window_mgr::clear(ScreenIndex);
};
````
为了使得一个成员函数作为友元，有一些需要注意的地方：小心的组合程序结构使得定义和声明之间的相互依赖得以合法。如：先得定义 Window_mgr 类，并在类中声明 clear 函数，但不定义 clear 函数体。这是由于 clear 会用到 Screen 的成员，所以需要先定义 Screen，而声明成员函数作为友元又需要先完成那个类的定义。此后就是定义 Screen 类并声明 clear 函数作为友元。最后是定义 clear 函数所做的事，其中会使用到 Screen 的成员。

**函数重载与友元**

尽管重载的函数使用的是同一个名字，它们依然是不同的函数。因而，声明友元函数时必须显式指定其中想要的函数原型，单单声明一个并不会将整个重载的函数集加入进来。如：
````cpp
class Screen {
friend std::ostream &storeOn(std::ostream&, Screen&);
friend BitMap& storeOn(BitMap&, Screen&);
};
````

**友元声明和作用域**

类和非成员函数在被用作声明友元之前不需要提前声明。当名字第一次出现在友元声明中时，这个名字就已经是可见的了。然而友元并不是通用的声明形式。即便是在类内定义了友元函数，亦需要在类外提供一个函数声明，使得函数可见。即便是在授权类的成员函数中调用，这种单独声明亦是必不可少的。如：
````cpp
struct X {
friend void f() { /* function body */ }
    X() { f(); }  //错误!!未声明函数 f
    void g();
    void h();
};
void X::g() { return f(); }  //错误!! f 未声明
void f();
void X::h() { return f(); }  //正确，此时 f 已经被声明了
````
需要理解的是友元声明仅仅影响访问权限，其并不是真正的声明形式。而，有一些编译器并不强制要求这种查找规则，友元函数可以不在类外再单独声明一次。

## 7.4 类作用域

每个类定义其自己的新作用域。在类作用域外，必须通过对象、引用或指针以成员访问符来访问数据和函数成员，而类型成员（typedef 、using）则通过作用域运算符访问。

**定义在类外的成员及其与类作用域的关系**

存在类作用域的事实解释了为何在类外定义成员函数时必须同时提供类名和函数名。在类外，其成员的名字将被隐藏。

一旦遇到了类名，剩下的定义部分（包括形参列表和函数体）就在类作用域内，因此，不需要类名进行限定就可以使用其它类成员。如：
````cpp
void Window_mgr::clear(ScreenIndex i)
{ /* 函数定义 */}
````
在遇到类名 Window_mgr 后，引用类的类型成员时不再需要用类名进行限定。

而另一方面，当一个成员函数在类外定义时，由于返回类型出现函数名字前，此时作用域并不在类作用域内，因而，必须加上类的名字进行限定。如：
````cpp
Window_mgr::ScreenIndex
Window_mgr::addScreen(const Screen &s);
````
实质上，仅在类成员函数定义时才能够在类名后进入类作用域，返回值类型并不会使得进入限定其的类名的作用域。

### 7.4.1 名称查找和类作用域

涉及到定义在类体内的类成员函数时进行名称查找（name lookup）将复杂一点。类定义分为两步：其一，成员的声明被编译；其二，当整个类定义完成时，成员函数定义将被编译；请记得，成员函数定义的编译发生在类的所有成员的声明之后。

由于此，成员函数体可以使用类中的任何名字。特别是，可以使用在成员函数体后才声明的名字。否则，处理成员声明的顺序将是脆弱而复杂的。

**类成员声明的名称查找**

类定义的两步走仅仅对于类成员函数使用的类体中的名字有效。对于运用于声明中的名字，包括返回值类型和参数列表中的类型则必须在使用前是可见的。如果成员声明使用了类此刻未见的名字，编译器将在类定义的外部作用域寻找此名字。如：
````cpp
typedef double Money;
string bal;
class Account {
public:
    //Money使用的是外部声明的，bal 则使用的后面声明的。这是两步走的缘故。
    Money balance() { return bal; }
private:
    Money bal;
};
````

**类型名字是特殊的**

通常，内部作用域可以重定义外部作用域中的名字，即便那个名字已经被用于内部作用域。然而，在类中如果一个成员使用了外部作用域中的名字，而此名字是一个类型，那么在类中就不能重定义此名字。如：
````cpp
typedef double Money;
class Account {
public:
    Money balance() { return bal; }
private:
    typedef double Money; //错误!!不能重定义 Money 类型
    Money bal;
};
````
尽管此重定义类型名字是一种错误，编译器并不一定识别此错误。一些编译会静默的接受这样的代码，即便这样的程序是错的。类型定义应该放在类的头部，这样其它成员就可以看到此类型定义。

**成员函数体内的名字查找规则**

成员函数体中使用的名字通过如下方式进行查找：

- 在成员函数内查找声明，只有出现在使用之前的声明才是可见的；
- 如果未找到，在类定义内查找声明，所有的成员的名字都可见；
- 如果未找到，继续从定义成员函数定义所在的作用域内向上查找，如果成员函数定义在类中则从类外部作用域查找，如果成员函数定义在类外，则从类外的位置向上查找；

通常定义参数名字与别的成员名字一样是不好的戏关，如果这样的话，函数内部使用的名字将是参数名字而不是成员名字。可以通过加上类名进行限定而访问成员名字，或者使用 this 指针进行访问。然而最好的方式是给参数取一个不同的名字。如：
````cpp
int height;
class Screen {
public:
    typedef std::string::size_type pos;
    void dummy_fcn(pos height)
    {
        cursor = width * height; //访问参数而非成员
        cursor = width * this->height; // 用 this 访问
        cursor = width * Screen::height; //用类名进行限定
        cursor = width * ::height; //访问全局作用域中的 height
    }
private:
    pos cursor = 0;
    pos height = 0, width = 0;
};
````

**在类作用域之后，于外部作用域中查找**

如果外部作用域中的名字被类作用域中相同名字给隐藏了，如：height ，可以通过作用域操作符（scope operator）显式访问全局中的名字。

**名字在它们出现在文件中的位置进行解析**

当成员定义在类外时，名字查找的第三步会检查函数定义所在的地方，也会继续往上查找穿过类定义的位置。如：
````cpp
class Screen {
public:
    typedef std::string::size_type pos;
    void setHeight(pos);
    pos height = 0;
};
Screen::pos verify(Screen::pos);
void Screen::setHeight(pos var) {
    height = verify(var);
}
````
这里的 verify 是不为类定义 Screen 所见的，因而不能运用于类定义中。但是，类外定义的成员函数 setHeight 却可以看到此名字，函数体内的名字并不是从类体处开始查找的，而是从函数定义处开始查找的。

## 7.5 构造函数再探

构造函数是任何 C++ 中都至关重要的一部分。

### 7.5.1 构造初始值列表

当定义变量时通常是马上进行初始化而不是定义了然后再进行赋值。这种区别同样运用于数据成员的初始化和赋值。如果不在构造初始值列表中显式初始化，成员将会被默认初始化，然后才进入构造函数体中。如：
````cpp
Sales_data::Sales_data(const string &s, unsigned cnt, double price)
{
    bookNo = s;
    units_sold = cnt;
    revenue = cnt * price;
}
````
这个版本的构造函数将先默认初始化数据成员，然后在函数体内进行赋值。如果数据成员是类类型，那么则会先调用其默认构造函数，再调用其重载后的赋值操作符，这可能造成低效或者某些类甚至是错误的。

**构造初始值在某些情况下是必需的**

初始化和赋值之前的区别并不总是可以忽略。const 成员和引用成员必须被初始化。同样的，没有默认构造函数的类类型必须被初始化。省略了这些构造初始值将是错误，当构造函数体开始执行时，初始化已经完成，初始化 const 成员和引用成员的唯一机会就是在构造初始值中。因而，必须在构造初始值列表中给 const 、引用、没有构造函数的类类型数据成员提供初始值。

**数据成员的初始化顺序**

构造初始值列表只给成员提供初始值，并不决定初始化的顺序。成员的初始化顺序由它们出现在类定义中顺序决定，先出现先初始化，后出现的后初始化。在构造初始值列表中的顺序并不会影响初始化顺序。

通常初始化顺序无关紧要。然而，如果一个成员的初始化依赖于另一个成员，此时成员的初始化顺序就很重要。如：
````cpp
class X {
    int i;
    int j;
public:
    //错误!! i将先初始化，然后才是 j
    X(int val):j(val), i(j) {}
};
````
以上代码的结果是 i 将被初始化为未定义值 j ，然后 j 被初始化为值 val 。一些编译器足够聪明可以为这种情况产生一个警告，通过 `-Wall` 选项可以让 g++ 产生这个警告。将构造初始值的顺序书写的与类定义中的声明的顺序一致是一个好的习惯。并且，如果可能，不要用别的成员来初始化另一个成员，这样可以不必思考成员初始化顺序。如：
````cpp
X(int val): i(val), j(val) {}
````

**默认实参和构造函数**

构造函数与类的成员函数可以设置默认实参。如：
````cpp
class Sales_data {
public:
    Sales_data(std::string s = ""):
        bookNo(s) { }
};
````
如果定义了带默认实参的构造函数，就不能再定义默认构造函数了，不然编译器就无法进行函数匹配了。即：一个构造函数如果提供了所有默认实参就相当于隐式定义了一个默认构造函数。

### 7.5.2 代理构造函数

在新标准中，扩展了构造初始值使得可以定义所谓的代理构造函数（delegating constructors），代理构造函数使用本类的另一个构造函数才执行初始化。即将一些或者全部工作“代理”给其它构造函数完成。

代理构造函数的构造初始值列表中只有一项就是另外一个构造函数的调用，实参列表必须匹配另外一个构造函数的签名。如：
````cpp
class Sales_data {
public:
    Sales_data(std::string s, unsigned cnt, double price):
        bookNo(s), units_sold(cnt), revenue(cnt*price) {}
    Sales_data(): Sales_data("", 0, 0) {}
    Sales_data(std::string s): Sales_data(s, 0, 0) {}
};
````
当一个构造函数代理给另外一个构造函数时，那个函数的构造初始值列表和函数体都会先于代理构造函数执行，意味着在代理构造函数体开始执行之前，提供代理那个构造函数体将先执行完。

### 7.5.3 默认构造函数的角色

当执行默认初始化或值初始化时，默认构造函数将被自动调用。

默认初始化发生在：

- 当在块作用域中以未初始化的方式定义非静态变量或非静态数组；
- 且类初始化时使用合成构造函数，而且其包含类类型成员；
- 且此类类型成员没有显式地在构造初始值列表中初始化；

值初始化发生在：

- 当数组初始化时提供的值少于数组的长度，将进行值初始化；
- 定义本地静态变量（local static object）而不提供初始值；
- 当显式用 `T()` 对类 T 的对象进行初始化，想一下 vector 就是用这种方式对元素进行至初始化的；

值初始化所做的事：如果类有用户提供（user-provided）默认构造函数，则直接调用此构造函数。如果默认构造函数是编译器隐式定义的，那么所有的非静态成员将递归值初始化。这里隐含的意思在于用户提供的默认构造函数如果不对某些内置类型的数据成员进行初始化，那么它将是未定义值。而由编译器隐式定义的默认构造函数，在值初始化时会对整个对象进行零初始化。

建议：如果定义了任何构造函数，好的做法是同时提供一个默认构造函数。并且，构造函数应当初始化所有成员。

**使用默认构造函数**

以下使用默认构造函数的形式是错误的，如：`Sales_data obj();` 以上定义了一个返回 Sales_data 的函数 obj ，我之前就用错过。要正确使用应当写做 `Sales_data obj{};` 或 `Sales_data obj = Sales_data();` ，这将会导致 obj 进行值初始化。而 `Sales_data obj` 进行默认初始化。

### 7.5.4 隐式类类型转换

除了语言定义的内置类型之间的自动转换。程序员可以定义类之间的隐式转换。每个只有一个参数的构造函数都定义了一个从参数到类类型之间的隐式转换。这种构造函数被称为转换构造函数（converting constructors），除此之外，还可以在类中直接定义转换函数，而从此类转为其它类型。如：
````cpp
//Sales_data
string null_book = "9-999-99999-9";
item.combine(null_book);
````
combine 成员函数的参数是 `const Sales_data&` ，而 Sales_data 有一个以 string 为参数的构造函数。因而，这种形式的 combine 调用，会以 null_book 创建一个 Sales_data 的临时量，再以此临时量调用 combine 函数。由于，combine 的参数是 const 对象的引用，所以传递一个临时量过去。

**只允许一个类类型转换**

编译器只允许一次类类型的自动转换。如：`item.combine("9-999-99999-9");` 是不能编译通过的。原因在于，从字符指针到 Sales_data 对象需要经过两步转换。为了进行此调用，必须进行显式的转换。如：
````cpp
item.combine(string("9-999-99999-9"));
item.combine(Sales_data("9-999-99999-9"));
````

**类类型自动转换并不总是有用**

有时这种类类型的自动转换根本不是程序员的意图，这样做只会增加理解程序的负担，而且产生的错误还可能是难以发现的。

**抑制由构造函数定义的隐式类类型转换**

通过在构造函数前加上 explicit 关键字，来抑制此构造函数被用于隐式转换。如：
````cpp
class Sales_data {
public:
    explicit Sales_data(std::istream&);
    explicit Sales_data(const std::string&);
};
````
explicit 关键只有对仅有一个参数的构造函数有效，其它形式的构造函数因为并不用于隐式类型转换，所以并不需要指定为 explicit 。explicit 关键只能用于构造函数声明处，不必在类外部函数定义处进行重复。

**explicit 构造函数只能用于直接初始化**

需要隐式转换的场景之一便是拷贝形式的初始化，explicit 构造函数不能用于这种形式，其必须使用直接初始化。如：
````cpp
Sales_data item1(null_book);
Sales_data item2 = null_book; //explicit 构造函数不能用于拷贝形式的初始化
````

**显式使用构造函数进行转换**

尽管不能将 explicit 构造函数进行隐式转换，但可以用于显式转换。如：
````cpp
item.combine(Sales_data(null_book)); //直接调用构造函数
item.combine(static_cast<Sales_data>(cin)); //使用 static_cast 进行显式转换
````
标准库中的 string 类中以 `const char*` 作为参数的构造函数是可以进行隐式转换的。而 vector 类以长度为参数的构造函数是 explicit，不允许隐式转换。

### 7.5.5 聚合类

聚合类（aggregate class）可以直接让用户访问其成员，并且有特殊的初始化语法。聚合类有点类似 POD（Plain Old Data）结构，满足以下条件的类可以被称为是聚合的：

- 所有的数据成员是 public 的；
- 没有定义任何构造函数；
- 没有类内初始值；
- 没有基类或者虚函数（virtual）；

如以下类便是聚合的：
````cpp
struct Data {
    int ival;
    string s;
};
````
可通过括弧列表中的初始值来初始化聚合类的成员，如：`Data val1 = {0, "Anna"};` 要求是括弧中的初始值需要与声明的数据成员的顺序一致。

与初始化数组元素一样，如果初始值列表中的值少于类的成员数，剩余的成员将被值初始化。初始值列表中包含的值一定不能多于类成员数。

聚合类有一些缺陷：

- 将正确初始化所有成员的责任强加在用户身上，这种初始化是冗长而易错的；
- 当类的成员被添加或移除时，所有的初始化的地方都需要更新；

### 7.5.6 字面类

constexpr 函数的参数和返回值都必须时字面类型。除了算数类型、指针和引用外，还可以定义字面类（literal classes）。字面类中的成员函数可以是 constexpr 的，这些成员函数必须满足常量表达式函数的所有要求。这些成员函数隐式是 const 的。

一个聚合类如果其所有数据成员都是字面类型是一个字面类。一个非聚合类，如果满足以下条件亦是一个字面类。

- 所有数据成员都是字面类型；
- 类至少有一个 constexpr 构造函数；
- 如果数据成员有类内初始值，内置类型的初始值必须是常量表达式，如果是类类型，初始值必须是调用 constexpr 构造函数产生的；
- 类必须使用编译器生成的析构函数；

**constexpr 构造函数**

字面类必须至少包含一个 constexpr 构造函数。constexpr 构造函数可以被声明为 `=default` 或者为 deleted 函数。否则，这种构造函数通常是空的，原因在于构造函数没有 return 语句，而 constexpr 函数唯一的语句便是 return 语句。如：
````cpp
class Debug {
public:
    constexpr Debug(bool b = true): hw(b), io(b), other(b) {}
private:
    bool hw;
    bool io;
    bool other;
};
````
constexpr 构造函数必须初始化所有数据成员，初始值要么是调用 constexpr 构造函数，要么是一个常量表达式。constexpr 构造函数常用于给 constexpr 函数构建参数或返回值。

## 7.6 static 成员

类有时需要一些成员是与整个类关联的，而不是与类的单个对象关联。从效率的角度来看，不必为每个对象存储一样的值，而且将值存储为类相关，当值发生改变时所有对象都可以马上使用此新值。

**声明静态成员**

通过在成员前加上 static 关键可以使得其与整个类关联。与其它成员类似，static 成员可以是 public 或 private 的，static 数据成员可以是 const 、引用、数组或类类型。

[static_class_member.cc](https://github.com/chuenlungwang/cppprimer-note/blob/master/code/static_class_member.cc) 包含详细示例代码。

静态成员存在于任何对象之外，对象中不包含静态成员数据。

类似的，静态成员函数不与任何对象关联，它们没有 this 指针。因而，静态成员函数不能被定义为 const 的，并且在函数体内不能使用 this 指针，包括显式使用 this 指针或者通过访问非静态成员数据或调用非静态成员函数隐式使用。

**使用静态成员**

可以通过作用域操作符直接访问，或者通过类对象、指针、引用访问静态成员，虽然静态成员并不是类对象的一部分。如：
````cpp
double r = Account::rate();  //通过作用域操作符
Account ac1;
Account *ac2 = &ac1;
r = ac1.rate(); //对象访问
r = ac2->rate(); //指针访问
````
静态成员可以被任何成员函数直接访问，不管是静态的还是常规的，这些访问是不需要作用域操作符进行限定的。

**定义静态成员**

可以在类的内部或外部定义静态成员函数。当在外部定义时，不需要重复 static 关键字，此关键字只应该放在类内的声明处。静态数据成员不能在类内进行初始化。所有的静态数据成员都必须在类外进行定义和初始化，并且只能被定义一次。如全局对象一样，静态数据成员在任何对象之外定义，因而，一旦它们被定义了，它们将持续到程序完全退出。

定义静态数据成员需要在成员名之前加上类名进行限定。如：
````cpp
double Account::interestRate = initRate();
````
以上需要注意的是一旦定义中看到类名之后，定义的其余部分就处于类作用域中，因而使用 initRate 成员就不必用类名进行限定，即便 initRate 是私有成员依然是可以访问的，原因是任何成员的定义都可以访问类的私有成员。

通常将定义静态数据成员的代码放在与定义类的非内联函数一起的源文件中。

**静态数据成员的类内初始化**

可以在类内为 const 整数类型的静态成员进行初始化，并且必须给 constexpr 字面类型的静态成员进行类内初始化。初始值必须是常量表达式，这些成员本身亦是常量表达式，它们可以被用于需要常量表达式的地方。如可以用于定义数组成员的维度。如：
````cpp
class Account {
    static constexpr int period = 30;
    double daily_tbl[period];
};
````
如果常量字面类型静态成员只运用于编译器可以将成员替换为其值的地方，那么这个静态成员不需要额外定义。如果此静态成员被用于一个不可以进行替换的场景，那么必须为此成员进行定义。例如，将 Account::period 传递给一个接收 `const int&` 的函数，那么 period 必须进行定义。

如果在类内提供了初始值，类外的成员定义不需要指定初始值。如：
````cpp
constexpr int Account::period;  //初始值已经在类内提供了
````
通常在类外部定义静态常量成员是一个好习惯。

以 const 修饰的静态成员，如果在类内初始化则是常量表达式，如果在类外进行初始化则为常规的 const 常量，而非常量表达式。

**静态成员可以被用于常规成员不能运用的场景**

静态数据成员可以使用未完成的类型。特别是，静态数据成员可以使用当前定义的类本身的类型，而非静态数据成员只能被声明为指针或引用。

另外要给区别在于可以将静态数据成员作为默认参数，而常规数据成员是不可以的。如：
````cpp
class Screen {
public:
    Screen& clear(char = bkground);
private:
    static const char bkground;
};
````

## 关键概念

- 抽象数据类型（abstract data type）：封装了实现的数据结构；
- 类声明（class declaration）：形如：`class T;` ，如果一个类只被声明而没有定义，它是一个未完成类型（incomplete type）；
- 类作用域（class scope）：每个类都定义一个作用域。类作用域比其它作用域跟复杂，成员函数甚至可以使用出现在其后的声明的成员；
- 常量成员函数（const member function）：不改变对象状态的成员函数，其函数体内的 this 指针指向 const 对象，成员函数可以通过是否为 const 进行重载；
- 构造函数（constructor）：用于初始化对象的特殊成员函数，每个构造函数都应该给每个数据成员一个良好定义的初始值；
- 构造初始值列表（constructor initializer list）：为每个数据成员指定初始值，成员被初始化为初始值列表中指定的值，这个过程发生在函数体执行之前。未出现在构造初始值列表中的成员将被默认初始化；
- 转换构造函数（converting constructor）：非 explicit 构造函数的单一参数构造函数可以被用于隐式转换其参数类型到类类型；
- 数据抽象（data abstraction）：一种聚焦于类型接口的编程技术。数据抽象使得程序员可以忽略类型的实现细节，只关注于类型可以执行的操作。数据抽象是面向对象和泛型编程的基础；
- 封装（encapsulation）：将实现和接口分开，封装隐藏了实现的细节；
- 前置声明（forward declaration）： 声明一个尚未定义的名字。通常用于在表示在定义一个类之前先声明类，它是一个未完成类型；
- 未完成类型（incomplete type）：只有声明而没有定义的类型，不能用未完成类型定义变量或者成员，但是可以定义此类型的指针或引用；
- 名称查找（name lookup）：将名字的使用与其声明相匹配的过程；
- this 指针（this pointer）：隐式传递给每个非 static 成员函数的实参，this 指针指向当前调用对象；
