C++ 中的操作符（operator）可以进行重载，于是类对象也可以方便的用操作符表达自己的含义。如，string 对象的 [] 下标操作符，迭代器的 * 解引用操作符。本章的内容主要讲解内置类型的操作符。

表达式（expression）是由多个操作数（operand）拼接操作符组成，其求值可得到一个结果。最简单的表达式是字面量或变量，其值就是字面量或变量值。复杂的表达式由一个操作符和一个或多个操作数组成。

表达式是最基本的计算单元。

## 4.1 fundamentals

有一些基本概念将影响表达式如何求值。如：操作符优先级、结合性以及操作数的求值顺序这些关键概念，以及操作数类型转换、操作符重载以及左值右值区分，都是理解表达式需要理解的更为基础的概念。

### 4.1.1 基础概念

操作符有一元操作符（unary operators）和二元操作符（binary operators）以及三元操作符（ternay）以及函数调用（function all），其中函数调用可以有无限制的操作数。有些操作符有不同的含义，如：* 既可以是解引用也可以是乘号，应当将他们当作独立的符号，依据上下文进行判定。

多个操作符构成的表达式需要理解操作符的优先级（precedence）和结合性（associativity）以及在某些情况下表达式依赖于操作数的求值顺序（order of evaluation）。

当表达式中的操作数不一样时，会将操作数转为相同的类型。如：整型提升，将整数转为浮点数的转换，在赋值表达式中将浮点数转为整数。

对于类类型，可以定义类自己的操作符含义，使得它们可以跟内置类型一样使用，称为操作符重载（overloaded operators）。操作符重载可以改变操作符的操作数类型和结果值类型，但是不能改变操作符的操作数个数以及优先级和结合性。

C++ 中左值（lvalue）和右值（rvalue）是很重要的概念。这两个概念是从 C 中继承过来的，C 定义的很简单，在等号左边就是左值，右边就是右。C++ 的左值表达式生成对象或函数，const 左值是不能放在等号左边的，一些表达式返回临时对象只能作为右值。通常来说，右值使用的是对象的值（内容），左值使用的对象本身（内存中的位置）。操符在需要左右值作为操作数以及返回左右值有重大区别。而需要右值的地方可以使用左值，当这样使用左值时，用的是对象的值。以下列举最常见的左右值操作符解释：

- = 等号的左操作数必须是左值，并且将赋值后的左操作数作为左值返回；
- & 取地址符需要左值，并且以右值形式返回指针；
- `*` 解引用和 [] 下标操作符都将产生左值；
- 自增操作符和自减操作符都需要左值作为操作数，前置形式返回左值，后置形式返回右值；

将 decltype 运用与产生左值的表达式时，返回的是结果类型的引用。

### 4.1.2 优先级和结合性

表达式中有两个或多个操作符是一个复合表达式（compound expression），优先级和结合性决定了符合表达式中的操作符与操作数怎样进行组合，决定了哪些部分先进行求值。可以用括号覆盖这些规则。优先级高的操作符比低的先求值，结合性决定了相同优先级的操作符谁先进行求值。

### 4.1.3 求值顺序

大部分的操作符不会要求操作数的求值顺序，可以以任何顺序对操作数进行求值。如：`int i = f1() * f2();` 就能以任何顺序对 f1 和 f2 函数进行调用。对于不强制要求操作数顺序的操作符，如果对一个操作数又改变又取值就是错误，通常导致未定义行为。如：
````cpp
int i = 0;
cout << i << " " << ++i << endl;
````
编译器可能先去 i 的值在进行 ++i，也可能先进行 ++i 再取 i 值，或者编译器可以同时运算。三种结果是完全不同的。程序应当避免这样做。

只有 4 个操作符是保证求值顺序的。&& 逻辑与、|| 逻辑或、?: 条件操作符、, 逗号操作符。

而且操作数的求值顺序与优先级和结合性是完全独立的。

建议：当对操作符的优先级有怀疑时使用括号进行标明，或许别人有跟你一样的疑惑。不要在一个表达式中又改变操作数值，又取值。

## 4.2 算数运算符

包括：`+` 一元加，`-` 一元减，`*` 乘号，`/` 除号，`%` 取模，`+` 加号，`-` 减号。

所有算数运算符（arithmetic operator）都是左结合的，操作数和结果都是右值。一元加号返回操作数的一个值副本。一元负号返回负的值副本。记住：对于绝大部分操作符，bool 值会被提升为 int ，特别注意的是整数与布尔值进行相等比较时，会将布尔值转为整数进行比较。

算数运算符需要注意的是值溢出，分为上溢和下溢。特别是乘法与除法结合时，虽然最后结果在范围内，但是乘法先计算是有可能导致溢出。此时可以先计算除法再计算乘法来避免。现代计算机的整数表示都是补码，补码溢出时会回绕，上溢之后值变成最小值，下溢则变成最大。

整数除法需要注意的是当遇到负数时应当如何舍入，取模运算（%）只能运用于整数类型，并且其舍入方式与除法是一致的。新标准要求舍入方向是向 0 取整，意味着截断商结果的小数点。由于模运算符合 `(m/n)*n + m%n` 因而，模的结果与被除数 m 的符号一致。通常，极少会对负数进行取模。如：
````cpp
-21 % -8 = -5      -21 / -8 = 2
21  % -5 = 1       21  / -5 = -4
````

## 4.3 逻辑和关系操作符

包括：`!`逻辑非，`<` 小于号，`>` 大于号，`<=` 小于等于，`>=` 大于等于，`==` 等于，`!=` 不等于，`&&` 逻辑与，`||` 逻辑或。

逻辑和关系操作符的操作数是右值，结果是右值。所有这些操作符都返回 bool 值，算术值和指针值中的 0 被认为是 false，其它值被认为是 true 。逻辑与和逻辑或执行短路求值（short-circuit evaluation），意味着它们在决出了结果之后其后面的操作数不再计算。&& 只有在左操作数的求值为真时，才会对右操作数求值。|| 只有在左操作数为假的情况下，才会对右操作数求值。因此，可以让左操作数作为右操作数的守卫，使得右操作数的计算是安全的。如：`index != s.size() && !isspace(s[index])` 只有当 index != s.size() 时，对 s 进行下标操作才是安全的。

关系操作符需要注意的是不要将多个关系符号串在一起，结果肯定是不对的。

#### 相等测试与 bool 字面量

需要注意的是测试算术或指针对象的最好方式是直接在条件语句中测试，而不要让它们与 bool 值进行比较，这样会引起意外的结果。如：
````cpp
if (val) {/*...*/}
if (!val) {/*...*/}
if (val == true) { /* ... */ }  //只有当 val 为 1 时才会相等
````
原因在于第三个语句中 true 会被整型提升为 int 类型，从而 true 转为 1，从而变成 `if (val == 1)` 这肯定不是我们想要的。在 C++ 中 bool 其实是一种小整型。bool 字面量须与 bool 类型值比较才是正确的做法。

## 4.4 赋值操作符

赋值操作符的左操作数必须是可修改的左值，结果是其左操作数，并且是一个左值。当左右类型不一样时，右操作数类型转为左操作数类型。在新标准下，可以提供一个大括弧初始值列表，如果提供一个空列表，左操作数将被赋值一个值初始化的值。列表中的值不能执行精度变小的转换。如：
````cpp
int k;
k = {3.14}; //错误!! 精度变小
k = {}; //k 等于 0
vector<int> vi;
vi = {};
vi = {0,1,2,3,4};
````

赋值是右结合的。赋值语句的结果是最右端的操作数的值，整个链上的值都是一样的。多赋值表达式的类型必须与其右边的操作数一致，或者可以互相转换。

赋值操作的优先级通常较低，特别是赋值比关系操作优先级还低，所以经常需要用括号将赋值操作括起来。

注意不要将赋值操作和等于操作混淆，GCC 编译器会对括号中的没有用括号括起来的赋值语句进行报警，混淆赋值和等于操作的 bug 是非常难定位的。

#### 复合赋值运算符

复合赋值运算就是将左边操作数与右边操作数进行运算然后将结果值赋值到左操作数中。如：`+=` `-=` `*=` `/=` `%=` `>>=` `<<=` `&=` `^=` `|=`，复合赋值语句相当于 `a = a op b`，当使用复合赋值时，左操作数只求值一次，而常规赋值将求值两次。仅当求值会影响时，才在考虑的范围内。

## 4.5 自增和自减操作符

自增和自减操作符可用于指针、迭代器和算术类型。这两个运算符都要求操作数是左值。自增和自减有两种形式：前置和后置。通常在 C++ 中使用前置形式，因为前置返回改变后的对象，返回的是左值。而后置形式返回的改变前的对象，因而必须是对象的副本，是一个右值。对于内置类型来说是无关紧要的，但是对于类类型来说则减少了拷贝。相对的，在 C 中两个操作符返回的都是右值。C++ 建议：仅在必要使用后置形式。

在 C 和 C++ 中 `*pbeg++` 是一种非常常见的模式，程序必须掌握这种地道的写法，绝大多数 C++ 程序都使用这种写法。

## 4.6 成员运算符

C++ 跟 C 一样提供了两种形式的成员运算符，类或结构对象可以直接用 `.` 点号访问成员，对象指针可用 `->` 箭头访问成员。`ptr->mem` 相当于 `(*ptr).mem`，箭头操作符要求一个指针操作数，并且返回一个左值方能访问其成员。点号操作符当对象是左值是返回左值，当对象是右值时返回右值。

## 4.7 条件操作符

`cond ? expr1 : expr2` 条件操作符的优先级非常低，并且求值顺序是有要求的，当条件为真时，? 后的 expr1 求值，否则对 expr2 求值。要求 expr1 和 expr2 的结果是相同类型或者可以相互转换。如果两个表达式都是左值，则整个条件操作的结果是左值，否则将是右值。

条件操作可以嵌套，但最好不要嵌套多于两层。条件操作是右结合的，意味着多个条件操作符嵌套时将从右边开始分析。

由于条件操作符的优先级非常低，所以很多时候必须将条件操作符用括弧括起来。

## 4.8 位（bitwise）操作符

位操作符：`~` 位取反，`<<` 位左移，`>>` 位右移，`&` 位与，`|` 位或，`^` 位异或。
位操作符只能作用于整型操作数，并将整数作为位的集合，位操作符可以操作单个位。如果是小整数首先将会执行整形提升。操作数则既可以是有符号也又可以是无符号整数。如果操作数是负数，则如何处理符号位是机器相关的。特别是那些有可能会改变符号位的操作，比如左移操作。由于语言没有保证如何处理符号位，因而强烈建议只使用无符号类型。

#### 位移（bitwise shift）操作符

位移操作符返回的值是左边操作数的一个副本，它们并不会改变左操作数而是返回一个新值。位移操作将左操作数按照右操作数指定的位数进行左移或者右移。左移时将以 0 填充右边的位，而右移如果是无符号整数则用 0 填充左边的位，如果是有符号数则是机器相关的，可能用符号位填充左边的位，也可能用 0 填充。

右边操作数要求必须是非负数的，并且值必须严格小于结果的位数，否则结果是未定义的。

位移操作符是最常见的重载操作符之一，如：输入输出操作符就是重载过的位移操作符。重载后的操作符的优先级以及结合性与内置版本是一样的。由于这两个操作符的优先级属于中等，意味着经常需要使用括号来保证优先级的正确。

#### 位与、或、异或操作符

通常位于、或、异或用来检查和设置掩码（mask）。如：
````cpp
quiz1 |= 1UL << 27; //将 quiz1 的第 27 位设置为 1
quiz1 &= ~(1UL << 27); //将 quiz1 的第 27 位设置位 0
bool status = quiz1 & (1UL << 27); //测试 27 位是否位 1
````

#### sizeof 操作符

sizeof 操作符返回表达式结果类型或类型名所表示类型的内存尺寸，返回的大小是固定尺寸不会包含动态分配的内存。因而，对于 vector 类型不会包含分配的元素的内存大小。sizeof 是右结合的，sizeof 是一个结果为 size_t 类型的一个常量表达式。操作符形式如：
````cpp
sizeof (type)
sizeof expr
````
sizeof 操作符并不会对表达式进行求值，而是推断其结果类型，进而计算所需的内存大小。甚至不理会表达式本身是否求值出来是未定义的，如：解引用一个未初始化的指针。对解引用一个非法指针就行 sizeof 求值是安全的，因为 sizeof 根本不会真正对指针进行解引用，只要它的返回值类型。如：
````cpp
Sales_data *p;
sizeof *p; //获取 p 所指对象类型的内存大小
````
在新标准中，可以使用 :: 作用域操作符来获取类的成员的大小。如：`sizeof Sales_data::revenue` 。需要注意的是当对数组进行 sizeof 操作时返回的是数组占的内存大小，此时数组不会转为指针。对指针进行 sizeof 得到的就是指针本身所占内存大小，在64位机器上是8字节。对 string 或 vector 进行 sizeof 操作只返回对象所占的固定部分，不会返回为元素分配的动态内存。

## 4.10 逗号操作符

逗号操作符（comma operator）有两个操作数，而且是从左到右顺序求值的。逗号操作的结果是其右边操作数的值，如果右操作数是左值返回的结果就是左值。逗号操作符常用于 for 的条件部分。

## 4.11 类型转换

C++ 中的类型之间存在转换（conversion）时被认为是类型相关，当需要其中一种类型值时，可以用另一种类型值替换。在第二章中讲述了内置类型间的转换规则，这里不再累述。当不需要程序员干预就可以自动发生的转换称为隐式转换（implicit conversion），隐式转换保证不丢失原有类型的精度。如果转换发生在初始化时，被初始化的类型占据主要地位，意味着别的类型必须转换过去，如：`int ival = 3.541;` 将 double 类型值转换为 int 类型值，即使此时精度会丢失。

以下情况将发生隐式转换：

- 在几乎所有的表达式中，小整型将首先进行整型提升；
- 在条件部分，非 bool 值将首先转为 bool 值；
- 在初始化或者赋值表达式中，右边的值类型转为左边值类型；
- 在算数和条件表达式中，操作数的类型转为一个共同的不丢失精度的类型；
- 函数调用时实参类型转为形参类型；

### 4.11.2 其它隐式转换

- 数组转为指针，在某些条件下不转为指针参考 CH4 描述。
- 指针转化：0 和 nullptr 转为任何类型的指针，非 const 指针可以转为 void*，任何类型的指针可以转为 const void*，子类的指针可以转为父类的指针；
- 转为 bool：任何非 0 值都将转为 true，0 转为 false；
- 转为 const：指向非 const 对象的指针可以转为指向 const 对象指针，于引用是一样的，而不能做相反操作；
- 由类定义的转换：类可以通过定义单参数的构造函数来实现由其参数类型到类类型的转换，除非显式禁止，编译器会自动进行如此转换。编译器类的直接初始化（direct initialization）转换，如果需要通过两步以上的转换才能转为需要的类型，编译器会拒绝；

### 4.11.3 显式转换

显式转换（cast）被 C++ 重新定义为更加精细的多种不同强转，当然 C 风格的强转也可以使用。C++ 中定义了多种具名强转，形如：`cast-name<type>(expression)` 。其中 type 是目标类型，expression 是需要被转换的值，如果 type 是一个引用，则结果是一个左值。强转名有：`static_cast`、`dynamic_cast`、`const_cast`、`reinterpret_cast`。

#### static_cast

这种强转主要用于定义良好的转换，如：算数类型之间的转换，将 void* 指针转为实际的类型指针，以及指向基类的指针转为指向子类的指针，或者可以用 expression 直接初始化 type ，将任何类型转为 void 类型，将枚举转为整数或者相反。如果本身不能这样转换，但是使用了的话则行为是未定义的。对于这种转换，C++ 语言本身不做任何运行时安全检查。相对的，dynamic_cast 将会对转换类型进行运行时检查，确保确实可以从基类指针转为子类指针。

#### const_cast

用于改变操作数的底层 const，最常用的是去掉 const 修饰。一旦去掉了 const ，编译器将不做任何检查，如果对象本身就是一个 const 的，试图改写这个对象将是未定义的。只能通过 const_cast 来改变对象或表达式的 const 属性，如果用别的具名强转将产生编译时错误，同样不能用 const_cast 改变表达式类型。

#### reinterpret_cast

reinterpret_cast 将执行位模式的重新解释，这样可以将一个类型转为任何别的类型。主要用于内置类型的转换，运用于非内置类型可以肯定是错误的。如：
````cpp
int *ip;
char *pc = reinterpret_cast<char*>(ip);
````
接下来的操作将会假设 pc 指向一个 char 类型值，编译器将无法对真实类型做检查。好处在于灵活性，特别是处理网络层读写时经常用到。坏错则是这种操作非常之危险，一朝不慎，整个程序崩溃。

#### C 风格强转

C 风格强转形如：
````cpp
type (expr);  //函数风格
(type) expr;  //C 语言风格
````
C 风格强转根据语境可以是以上四种具名强转的任何一种，在 C++ 中不推荐如此使用。

## 4.12 操作符优先级

参考网页：[https://en.cppreference.com/w/cpp/language/operator_precedence](https://en.cppreference.com/w/cpp/language/operator_precedence)
