# c++primer学习

## 第7章 类

​    类的基本思想是数据抽象和封装。数据抽象是一种依赖于接口和实现分离的编程(以及设计)技术。

#### 7.1 定义抽象数据类型

##### 1.成员函数

​    定义和声明成员函数的方式和普通函数差不多。成员函数的声明必须在类的内部，它的定义则都可以。作为接口组成部分的非成员函数，例如add、read等等，他们的定义和声明都在类的外部。

​    **ps：定义在类内部的函数是隐式的inline函数。**

##### 2.引入this

​    成员函数通过一个名为this的额外的隐式参数来调用他的那个对象。当我们调用一个成员函数时，用该请求函数的对象地址初始化this。

`total.isbn(); 编译器负责把total的地址传递给isbn的隐式形参this`

​    在成员函数内部，我们可以直接使用调用该函数的对象的成员，而无需通过成员访问符来做这一点。

##### 3.引入const成员函数

​    默认情况下，this的类型是指向类型非常量版本的常量指针。如果函数体内不会改变this所指向对象，应该把this设置为指向常量的指针，然而this是隐式的不会出现在形参列表里。所以我们需要将const关键字放在成员函数的参数列表之后，此时，紧跟在参数列表后面的const表示this是一个指向常量的指针。像这样使用const的成员函数被称作常量成员函数。

```c++
string isbn() const { ... }
```

​    **ps：常量对象，以及常量对象的引用或指针都只能调用常量成员函数。**

##### 5.定义一个返回this对象的函数

​    一般来说，当我们定义的函数类似于某个内置运算符时，应该令该函数的行为尽量模仿这个运算符。内置的赋值运算符把它的左侧运算对象当成左值返回，因此为了与它保持一一致，combine 函数必须返回引用类型。因为此时的左侧运算对象是一个 Sales data的对象，所以返回类型应该是 Sales data&。

```c++
Sales_data& Sales_data::combind(const Sales_data& rhs){
    units_sold += rhs.inits_sold;
    revenue += rhs.revenue;
    return *this;  //返回调用该函数的对象，其实就是返回total的引用
}
```

##### 6.定义类相关的非成员函数

​    我们定义非成员函数的方式与定义其他函数一样，通常把函数的声明和定义分离开来。如果函数在概念上属于类但是不定义在类中，则它一般应与类声明（而非定义）在同一个头文件内。在这种方式下，用户使用接口的任何部分都只需要引入一个文件。

##### 7.构造函数

- 每个类都分别定义了它的对象被初始化的方式，类通过一个或几个特殊的成员函数来控制其对象的初始化过程，这些函数叫做构造函数。构造函数的任务是初始化类对象的数据成员，无论何时只要类的对象被创建，就会执行构造函数。 
- 构造函数的名字和类名相同。和其他函数不同，构造函数没有返回值。
- 构造函数不能被声明成const。当我们创建类的一个const对象时，直到构造函数完成初始化过程，对象才能真正取到其“常量”属性。因此，构造函数在const对象的构造过程中可以向其写值。

###### 1.合成的默认构造函数

​    如果我们没有为类有显示的定义构造函数，那么编译器就会为我们隐式地定义一个默认构造函数。又被称为合成的默认构造函数。

​    **ps：如果类包含有内置类型或者复合类型的成员，则只有当这些成员全都被赋予了类内的初始值时。这个类才适用于使用合成的默认构造函数。**

###### 2.=default

​    如果我们需要默认的行为，那么可以通过在参数列表后面上=default来要求编译器生成构造函数。

###### 3.构造函数初始值列表

​    我们把冒号以及冒号和花括号之间的代码，其中花括号定义了函数体，我们把这部分成为构造函数初始值列表。

```c++
Sales_data(const std::string& s) : bookNo(s) {}
Saled_data(const std::string& s, unsigned n, double p) : bookNo(s), units_sold(n), revenue(p*n) {}
```

​    **ps：构造函数不应该轻易覆盖掉类内的初始值，除非新赋的值与原值不同。如果不不能使用类内初始值，则所有构造函数都应该显式地初始化每个内置类型的成员。**

##### 8.拷贝、赋值和析构

​    除了定义类的对象如何初始化之外，类还需要控制拷贝、赋值和销毁对象时发生的行为。对象在几种情况下会被拷贝，如我们初始化变量以及以值的方式传递或返回一个对象等。当我们使用了赋值运算符时会发生对象的赋值操作。当对象不再存在时执行销毁的操作，比如一个局部对象会在创建它的块结束时被销毁，当 vector 对象（或者数组）销毁时存储在其中的对象也会被销毁。 

#### 7.2 访问控制和封装

​    在c++种，我们使用**访问说明符**来加强类的封装性：

- 定义在public说明符之后的成员在整个程序内可以被访问，public成员定义类的接口。
- 定义在private说明符之后的成员可与被类的成员函数访问，但是不能被使用该类的代码访问，private部分封装了(即隐藏了)类的实现细节。

​    **ps：如果使用了struct关键字开始类的定义，默认访问权限是public；如果使用了class关键字开始类的定义，默认访问权限是private。此外二者定义类无任何区别。**

##### 1.友元

​    类可以允许其他类或者函数访问他的非公有成员，方法是令其他类或者函数成为他的友元(friend)。如果类想把一个函数作为他的友元，只需要增加一条以friend关键字开始的函数声明语句即可：

```c++
class Sales_data{
    //为Sales_data的非成员函数所做的友元声明
    friend Sales_data add(...);
    //类classA可以访问Sales_data的私有部分
    friend class classA;
}
```

​    **ps：一般来说，最好在类定义开始或结束前的位置集中声明友元。**

​    友元的声明仅仅指定了访问的权限，而非一个通常意义上的函数声明。如果我们希望类的用户能够调用某个友元函数，那么我们就必须在友元声明之外再专门对函数进行一次声明。 

#### 7.3 类的其他特性

- 一个const成员如果以引用的形式返回*this，那么它的返回类型将是常量引用。
- 每个类定义了唯一的类型，对于两个类来说，即使他们的成员完全一样，这两个类也是两个不同的类型。

#### 7.4 类的作用域

​    每个类都会定义它自己的作用域。在类的作用域之外，普通的数据和函数成员只能由对象、引用或者指针使用成员访问运算符来访问。对于类类型成员则使用作用域运算符访问。不论哪种情况，跟在运算符之后的名字都必须是对应类的成员∶

```c++
Screen::pos ht = 24, wd = 80;
Screen scr(ht, wd, ' ');
Screen *p = &scr;
char c = scr.get();  //访问scr对象的get成员
c = p->get();        //访问p所指对象的get成员
```

##### 1.名字查找与类的作用域

​    名字查找：寻找与所用名字最匹配的声明的过程。

- 首先，在名字所在的块中寻找其声明语句，只考虑在名字的使用之前出现的声明。
- 如果没找到，继续查找外层作用域。
- 如果最终没有找到匹配的声明，则程序报错。 

​    对于定义在类内部的成员函数来说，解析其中名字的方式与上述有所不同。类的定义分两步处理：

- 首先，编译成员的声明。
- 知道类全部可见后才编译函数体。

##### 1.类型名要特殊处理

​    一般来说，内层作用域可以重新定义外层作用域中的名字，即使该名字已经在内层作用域中使用过。然而在类中，如果成员使用了外层作用域中的某个名字，而该名字代表一种类型，则类不能在之后重新定义该名字∶ 

```c++
typedef double Money;
class Account{
public:
    Money balance() {...};  //使用外层作用域的Money
private:
    typedef double Money;   //错误：不能重新定义Money，即使类型一样
    Money bal;
    ...
}
```

##### 2.成员定义中的普通块作用域的名字查找

​    成员函数中使用的名字按照如下的方式解析：

- 首先，在成员函数内查找该名字的声明。和前面一样，只有在函数使用之前出现的声明才被考虑。
- 如果在成员函数内没有找到，则在类内继续查找，这时类的所有成员都可以被考虑。
- 如果类内也没找到该名字的声明，在成员函数定义之前的作用域内继续查找。

​    **ps：尽管类的成员被隐藏了，但我们仍然可以通过加上类的名字或显式地使用this指针来强制访问成员。**

```c++
class classA{
...
    void add(int val){
        this->val = val;
    }
...
    int val;
}
```

#### 7.5 构造函数再探

##### 1.构造函数初始值列表

- 构造函数有时必不可少：有时我们可以忽略数据成员初始化和赋值之间的差异，但如果成员是const或者引用的时候，必须将其初始化。
- 成员初始化的顺序：成员的初始化顺序与它们在类定义中的出现顺序一致;第一个成员先被初始化，然后第二个，以此类推。构造函数初始值列表中初始值的前后位置关系不会影响实际的初始化顺序。

```c++
class X{
    int i;
    int j;
public:
    X(int val) : j(val), i(j) {}  //错误，因为i在j之前被初始化
}
```

​    **ps：最好令构造函数初始值顺序和成员声明的顺序保持一致。而且如果可以，尽量避免使用某些成员初始化其他成员。**

- 如果一个构造函数为所有参数都提供了默认实参，则它实际上也定义了默认构造函数。

##### 2.委托构造函数

​    C++11 新标准扩展了构造函数初始值的功能，使得我们可以定义所谓的**委托构造函数（delegating constructor）**。一个委托构造函数使用它所属类的其他构造函数执行它自己的初始化过程，或者说它把它自已的一些（或者全部）职责委托给了其他构造函数。 

​    和其他构造函数一样，一个委托构造函数也有一个成员初始值的列表和一个函数体。在委托构造函数内，成员初始值列表只有一个唯一的入口，就是类名本身。和其他成员初始值—样，类名后面紧跟圆括号括起来的参数列表，参数列表必须与类中另外一个构造函数匹配。

```c++
class Saler_data{
public:
    Sales_data(std::string s, unsigned cnt, double price) : bookNo(s), units_sold(cnt), revenue(cnt*price) {}
    //其余构造函数全委托给另一个构造函数
    Sales_data(): Sales_data("", 0, 0) {}
    Sales_data(std::string s): Sales_data(s, 0, 0) {}
    Sales_data(std::istream& is): Sales_data() { read(is, *this); }
}
```

##### 3.默认构造函数的作用

​    当对象被默认初始化或值初始化时自动执行默认构造函数。

​    默认初始化在以下情况下发生∶

- 当我们在块作用域内不使用任何初始值定义一个非静态变量或者数组时。
- 当一个类本身含有类类型的成员且使用合成的默认构造函数时。
- 当类类型的成员没有在构造函数初始值列表中显式地初始化时。

​    值初始化在以下情况发生：

- 在数组初始化的过程中如果我们提供的初始值数量少于数组的大小时。
- 当我们不使用初始值定义一个局部静态变量时。
- 当我们通过书写形如 T（）的表达式显式地请求值初始化时，其中 T 是类型名，他就是使用一个这种形式的实参来对他的元素初始化器进行值初始化。

##### 4.隐式的类类型转换

​    能通过一个实参调用的构造函数定义了一条从构造函数的参数类型向类类型隐式转换的规则。

- 只允许一步类类型转换：

```c++
//combine传入Sales_date类型数据
item.combine("9-999-9999-9");             //错误：需要用户定义的两种转换
item.combine(string("9-999-9999-9"));     //显式地转换成string，隐式的转换成Sales_data
item.combine(Sales_data("9-999-9999-9"))  //隐式的转换成string，显式地转换成Sales_data
```

- 抑制构造函数定义的隐式转换：通过将构造函数声明为**explicit**加以阻止。

```c++
class Sales_data{
public:
    Sales_data() = default;
    Sales_data(const std::string& s, unsigned n, double p) : bookNo(s), units_sold(n), revenue(p*n) {}
    explicit Sales_data(const std::string& s) : bookNo(s) {}
    explicit Sales_data(std::ustream&);
}
```

​    **ps：关键字explicit只能对一个实参的构造函数有效，需要多个实参的构造函数不能用于执行隐式转换。且只能在类内声明构造函数时使用explicit关键字，在类外部定义时不应重复。**

- 当我们使用explicit声明构造函数时，他将只能以直接初始化的形式使用。而且，编译器不会再自动转换过程中使用该构造函数。

```c++
Sales_data item1(numm_book);   //正确，直接初始化
Sales_data item2 = numm_book;  //错误，不能将explicit构造函数用于拷贝形式的初始化过程
```

- 为转换显式地使用构造函数：我们可以使用explicit的构造函数显式地强制进行转换：

```c++
item.combine(Sales_data(null_book));
item.combine(static_cast<Sales_data>(cin));
```

##### 5.聚合类

​    聚合类使得用户可以直接访问其成员，并且具有特殊化语法形式。当一个类满足如下条件时，我们说他是聚合的：

- 所有成员都是public的。
- 没有定义构造函数
- 没有类内初始值
- 没有基类，也没有virtual函数。

##### 6.字面值常量类

​    如果一个类不是聚合类，但是它满足以下要求，则它是一个字面值常量类：

- 数据成员必须都是字面值类型。
- 类必须至少含有一个constexpr构造函数。
- 如果一个数据成员含有类内初始值，则内置类型成员的初始值必须是一条常量表达式；或者如果成员属于某种类型，则初始值必须使用成员自己的constexpr构造函数。
- 类必须使用析构函数的默认定义，该成员负责销毁对象。

​    constexpr构造函数必须初始化所有数据成员，初始值或者使用constexpr的构造函数，或者一条常量表达式。

#### 7.6 类的静态成员

​    我们通过在成员的声明之前加上关键字static使得其与类关联在一起。和其他成员一样，静态成员可以是 public的或private的。静态数据成员的类型可以是常量、引用、指针、类类型等。

```c++
class Account{
public:
    void calculate() { amount += amount * interesrtRate; }
    static double rate() { return interestRate; }
    static void rate(double);
private:
   std::string owner;
   double amount;
   static double interestRate;
   static double initRate();
}
```

​    静态成员不与任何对象绑定到一起，他们不包含this指针。作为结果，静态成员函数不能声明成const的，而且我们也不能在static函数体内使用this指针。这一限制既适用于this的显式使用，也对调用非静态成员的隐式使用有效。

​    **ps：我们可以使用作用域运算符直接访问静态成员、我们可以使用类的对象、引用或者指针来访问静态成员、成员函数不能通过作用域运算符就能直接使用静态成员。**

##### 1.定义静态成员

​    和其他成员函数一样，我们既可以在类内亦可以在类外定义静态成员函数。

```c++
void Account::rate(double newDate){  //类外定义不需要标明static
    interestRate = newRate;
}
```

​    **ps：和类的所有成员一样，当我们指向类外的静态成员时，必须指明类名。static关键字只出现在类内的声明语句中。要确保对象只定义一次，最好的办法是把静态数据成员的定义与其他非内联函数的定义放在同一个文件中。**

##### 2.静态成员的类内初始化

​    通常情况下，类的静态成员不应该在类的内部初始化。然后我们可以为静态成员提供const整数类型的类内初始化，不过要求静态成员必须是字面值常量类型的constexpr。

​    ps：即使一个常量静态数据成员在类内部被初始化了，通常情况下也应该在类的外部定义一下该成员。

##### 3.静态成员能用于某些场景，而普通成员不能

​    如我们所见，静态成员独立于任何对象。因此，在某些非静态数据成员可能非法的场合，静态成员却可以正常地使用。举个例子，静态数据成员可以是不完全类型。特别的，静态数据成员的类型可以就是它所属的类类型。而非静态数据成员则受到限制，只能声明成它所属类的指针或引用∶

```c++
class Bar{
public:
    ...
private:
    static Bar mem1;  //正确：静态成员可以是不完全类型
    Bar *mem2;        //正确：指针成员可以是不完全类型
    Bar mem3;         //错误：数据成员必须是完全类型
}
```

​    静态成员和普通成员的另外一个区别用静态成员作为默认实参：

```c++
class Screen{
public:
    //bakground表示一个在类中稍后定义的静态成员
    Screen& clear(char = bkground);
private:
    static const char bkground;
}
```

