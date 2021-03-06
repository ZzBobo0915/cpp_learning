# c++primer学习

## 第13章 拷贝控制

​    一个类通过定义5中特殊的成员函数来控制显式地或者隐式地指定对象的拷贝、移动、赋值和销毁时做什么：拷贝构造函数、拷贝赋值运算符、移动构造函数、移动赋值运算符、析构函数。

#### 13.1 拷贝、赋值和销毁

##### 1.拷贝构造函数

​     如果一个构造函数的第一个参数是自身类型的引用，且任何额外参数都有默认值，则此构造函数是拷贝构造函数：

```c++
class Foo{
public:
	Foo();            //默认构造函数
	Foo(const Foo&);  //拷贝构造函数
    ...
}
```

###### 1.合成拷贝构造函数

​    一般情况，合成拷贝构造函数会将其参数的成员逐个拷贝到正在创建的对象中。

###### 2.拷贝初始化

```c++
string dots(10, '');           //直接初始化
string s2 = string(100, '9');  //拷贝初始化
```

​    拷贝初始化会发生在下列情况：

-   用=定义变量时
-   将一个对象作为实参传递给一个非引用类型的形参
-   从一个返回类型为非引用类型的函数返回一个对象
-   用花括号列表初始化一个数组中的元素或一个聚合类的成员

###### 3.参数和返回值

​    在函数调用过程中，具有非引用类型的参数要进行拷贝初始化。拷贝构造函数被用来初始化非引用类类型参数。

###### 4.拷贝初始化的限制

​    拷贝构造函数一般不能是explicit类型的。如果要求使用的初始化值用一个explicit的构造函数来进行类型转换，那么要小心使用直接初始化和拷贝初始化。

###### 5.编译器可以绕过拷贝构造函数

​    在拷贝初始化过程中，编译器可以但不是必须跳过拷贝/移动构造函数，直接创建对象。但是拷贝/移动构造函数必须是存在且可访问的（比如不能是private的）。

##### 2.拷贝赋值运算符

​    和类控制其对象如何初始化一样，类也可以控制其对象如何赋值：

```c++
Lei trans, accum;
trans = accum;
```

###### 1.重载赋值运算符

   重载运算符本质上是函数，其名字由operator关键字后接表示要定义的运算符的符号组成。赋值运算符就是一个名为operator=的函数。

```c++
class Foo{
public:
    Foo& operator=(const Foo&){}
    ...
}
```

​    赋值运算符通常返回一个指向其左侧运算对象的引用，另外注意，赋值运算符通常应该返回一个指向其左值运算对象的引用。

###### 2.合成拷贝赋值运算符

​    如果一个类未定义自己的拷贝赋值运算符，编译器会为它生成一个合成拷贝构造运算符。

##### 3.析构函数

​    析构函数执行与构造函数相反的操作：析构函数释放对象使用的资源，并销毁非static数据成员。

###### 1.析构函数完成什么工作

​    析构函数按照成员初始化的逆顺序来销毁。通常，析构函数释放对象在生存期分配的所有资源。

​    **ps：隐式销毁一个内置指针类型的成员不会delete它所指向的对象；与普通指针不同，智能指针是类类型，所以具有析构函数，因此智能指针成员在析构阶段会被自动销毁。**

###### 2.什么时候会调用析构函数

-   变量在离开其作用域时被销毁
-   当一个对象被销毁时，其成员被销毁
-   容器被销毁时，其元素倍被销毁
-   对于动态分配的对象，当对他指向的指针应用delete运算符时被销毁
-   对于临时对象，当创建它的完整表达式结束时被销毁

###### 3.合成析构函数

​    当一个类未定义自己的析构函数时，编译器会为它定义一个合成析构函数

##### 4.三/五法则

​    有三个基本操作可以控制类的拷贝操作：拷贝构造函数，拷贝赋值运算符，析构函数，新标准下一个类还可以定义一个移动构造函数和移动赋值运算符。

​    **ps：如果一个类需要自定义析构函数，几乎可以肯定它也需要自定义拷贝赋值运算符和拷贝构造函数。一般需要拷贝操作的也需要赋值操作，反之亦然。所以一般由三（析构+拷贝+拷贝赋值/析构+移动+移动赋值）/五个函数。**

##### 5.使用=default

​    我们可以通过将拷贝控制成员定义为=default来显式地要求编译器生成合成的版本：

```c++
class Foo{
public:
	Foo() = default;
  	Foo(const Foo&) = default;
  	Foo& operator=(const Foo&);
	...
}
Foo& Foo::operaor=(const Foo&) = default;
```

​    如果我们不希望合成的成员是内联函数，应该只对成员的的类外定义使用=default，就像上面的拷贝赋值运算符一样。

##### 6.阻止拷贝

###### 1.定义删除的函数

​    在新标准下，我们可以通过将拷贝构造函数和拷贝赋值运算符定义为删除的函数来阻止拷贝。删除函数是一种这样的函数：我们虽然声明了它，但它不能以任何方式使用。在函数后面加上delete来指出我们希望将他定义为删除：

```c++
struct NoCopy{
	NoCopy() = default;
	NoCopy(const NoCopy&) = delete;
	NoCopy& operator=(const NoCopy&) = delete;
  	~NoCopy() = default;
}
```

​    与=default不同，=delete必须出现在函数第一次声明的时候；与=default另一个不同是，我们可以对任意函数指定=delete。

###### 2.析构函数不能是删除的成员

​    如果一个类有某个成员的析构函数是删除的，则该成员无法被销毁；而一个成员无法被销毁，整个对象就无法被销毁。虽然对删除了析构函数的类型，我们不能定义这种类型的变量或成员，但可以动态分配这种类型的对象，但是不能释放这些对象。

###### 3.合成的拷贝控制成员可能是删除的

​    对某些类来说，编译器将这些合法的成员定义为删除的函数：

-   如果类的某个成员的析构函数是删除或不可访问的，则类的合成析构函数被定义未删除的
-   如果类的某个成员拷贝构造函数是删除的或不可访问的，则类的合成拷贝构造函数被定义为删除的。如果某个成员的析构函数是删除或不可访问的，则类合成的拷贝构造函数也被定义为删除的
-   如果类的某个成员的拷贝赋值运算符是删除或者不可访问的，或是类有一个const的或引用成员，则类的合成拷贝赋值运算符被定义为删除的
-   如果类的某个成员的析构函数是删除的或是不可访问的，或是类有一个引用常量，他没有类内初始化器。或是类有一个const成员，她没有类内初始化器且其类型未显式地定义默认构造函数，则该类的默认构造函数被定义未删除的。

​    **ps：本质上，当不了能拷贝、赋值或者销毁成员时，类的合成拷贝控制成员就被定义为删除。**

###### 4.private拷贝控制

​    在新标准发布前，类是通过将其拷贝构造函数和拷贝赋值运算符声明为private的来阻止拷贝。

#### 13.2 拷贝控制和资源管理

​    通常，管理类外资源的类必须定义拷贝控制成员。

##### 1.行为像值的类

​    为了提供类值的行为，对于类管理的资源，每个对象都应该拥有一份自己的拷贝。

​    赋值运算符通常组合了析构函数和构造函数的操作。类似析构函数，赋值操作会销毁掉左侧运算对象的资源。

```c++
HasPtr& HasPtr::operator=(const HasPtr& rhs){
    auto newp = new string(*rhs.ps);
    delete ps;
    ps = newp;
    i = rhs.i;
    return *this;
}
```

​    **ps：当你编写赋值运算符的时候，有两点需要记住：**

-   **如果一个对象赋予它自身，赋值运算符必须能够正确工作**
-   **大多数赋值运算符组合了析构函数和拷贝构造函数的工作**

​    当你编写一个赋值运算符时，一个好的模式是先将右侧的对象拷贝到一个局部临时对象中，当拷贝完成后，销毁左侧对象的现有成员就安全了。

##### 2.定义行为像指针的类

​    另一个类展现类似指针的行为的最好方法就是使用shared_ptr来管理类中的资源。但是有时候我们希望直接管理资源，在这种情况下，使用引用计数就很有用了。

###### 1.引用计数

​    引用计数的工作方式如下：

-   除了初始化对象外，每个构造函数（拷贝构造函数除外）还要创建一个引用计数，用来记录有多少对象正在与创建的对象共享状态。当我们创建一个对象时，只有一个对象共享状态，因此将计数器初始化为1。
-   拷贝构造函数不分配新的计数器，而是拷贝给定对象的数据成员，包括计数器。拷贝构造函数递增共享的计数器，指出给定的状态又被一个新用户所共享。
-   析构函数递减计数器，指出共享状态的用户少了一个。如果计数器变为0，则析构函数释放状态。
-   拷贝赋值运算符递增右侧运算对象的计数器，递减左侧运算对象的计数器。如果左侧运算对象的计数器变为0，意味着他的共享状态没有用户了，拷贝赋值运算符就必须销毁状态。

#### 13.3 交换操作

​    与拷贝控制成员不同，swap不是必须的。但是对于分配了资源的类，定义swap可能是一种很重要的优化手段。

​    在赋值运算符中使用swap。即拷贝并交换：

```c++
HasPtr& HasPtr::operator=(HasPtr rhs){
    swap(*this, rhs);
    return *this;
}
```

​    这个版本的赋值运算符中，参数并不是一个引用，我们将右侧运算符对象以值传递方式给了赋值运算符。

#### 13.6 对象移动

​    新标准的最主要的特性是可以移动而并非拷贝对象的能力。IO类和unique_ptr类可以移动但不能拷贝。

##### 1.右值引用

​    右值引用就是必须绑定到右值的引用，我们通过&&而不是&来获得右值引用。而右值引用有一个重要的性质：只能绑定到一个将要销毁的对象。我们也可称常规引用为左值引用：

```c++
int i = 42;
int &r = i;
int &&rr = i;            //错误：不能将一个右值引用绑定到一个左值上
int &r1 = i * 42;        //错误：i*42是一个右值
int &&rr1 = i * 42;      //正确：将rr1绑定到乘法结果上
const int &r2 = i * 42;  //正确，我们可以将一个const引用绑定到一个右值上
```

###### 1.左值持久、右值短暂

​    由于右值引用只能绑定到临时对象，我们得知：

-   该引用的对象将要被销毁
-   该对象没有其他用户

###### 2.变量是左值

​    变量是左值，因此我们不能讲一个右值引用直接绑定到一个变量上，即使这个变量是右值引用类型也不行。

###### 3.标准库函数move

​    我们可以显式地将一个左值转换为对应的右值引用类型。我们还可以调用一个名为move的新标准库来获得绑定到左值上的右值引用，此函数定义在头文件utility中。

```c++
int &&r3 = std::move(rr1);
```

##### 2.移动构造函数和移动赋值运算符 

​    类似拷贝构造函数，移动构造函数的第一个参数是给类型的一个引用。不同于拷贝构造函数的是，这个引用参数在移动构造函数中是一个右值引用。与拷贝构造函数一样，任何额外的参数都必须有默认实参。

​    除了完成资源移动，移动构造函数还必须确保移后源对象处于这样一个状态--销毁他是无害的。特别是，一旦资源完成移动，源对象必须不再指向被移动的资源--这些资源的所有权已归属于新创建的对象。

​    与拷贝构造函数不同，移动构造函数不分配任何新内存；它接管源对象中的内存。在接管内存后，将给定对象中的指针都置为nullptr：

```c++
Foo::Foo(Foo&& s) noexcept : element(s.element), first_free(s.first_free), cap(s.cap)
{
	s.elements = s.first_free = s.cap = nullptr;
}
```

###### 1.移动操作、标准库容器和异常

​    由于移动操作“窃取”资源，它通常不分配任何资源。因此，移动操作通常不会抛出任何异常。一种通知标准库的方式是我们的构造函数中指明noexcept，且我们必须在头文件的声明中和定义中都指定noexcept。

​    **ps：不抛出异常的移动构造函数和移动赋值运算符都必须标记为noexcept。**

###### 2.移动赋值运算符

​    移动赋值运算符执行与析构函数和移动构造函数相同的工作。

**ps：这块忘保存了，就这样略过吧QAQ**

。。。

###### 5.移动右值，拷贝左值。。。

​    编译器使用普通的函数匹配规则规定来确定使用那个构造函数。

###### 6.但如果没有移动构造函数，右值也被拷贝

​    如果有一个类有一个可用的拷贝构造函数而没有移动构造函数，则其对象是通过拷贝构造函数来“移动”的。

###### 7.拷贝并交换赋值运算符和移动操作

###### 8.移动迭代器

​    标准库中定义了一种移动迭代器适配器，一个移动迭代器通过改变给定迭代器的解引用运算符的行为来适配此迭代器。与其他迭代器不同，移动迭代器的解引用运算符生成一个右值引用。

​    我们通过调用标准库的make_move_iterator函数将一个普通迭代器转换为一个移动迭代器。

```c++
void Foo::reallocate(){
    auto newcapacity = size() ? 2 * size() : 1;
    auto first = alloc.allocate(newcapicity);
    auto last = uninitialized_copy(make_move_iterator(begin()),
    							  make_move_iterator(end()), first);
    free();
    element = first;
    first_free = last;
    cap = elements + newcapacity
}
```

##### 3.右值引用和成员函数

​    如果一个成员函数同时提供拷贝和移动版本，她也能从中受益：

###### 1.右值和左值引用成员函数

​    我们可以指出this的左值和右值属性：在参数列表后放置一个引用限定符：

```c++
class Foo{
public:
    Foo& operator=(const Foo& rhs) &{  //&表示限定左值，&&表示限定右值
      return *this;
    }
}
```

###### 2.重载和引用函数

​    我们可以用限定符和const来区分一个成员函数的重载版本。

​    当我们定义两个或两个以上具有相同名字和相同参数的成员函数，就必须都所有函数都加上引用限定符，或者所有都不加：

```c++
class Foo{
public:
    Foo sorted() &&;          
    Foo sorted() const;       //错误，应该是const&
    using Comp = bool(const int&, const int&);
    Foo sorted(Comp*);        //正确，不同的参数列表
    Foo sorted(Comp*) const;  //正确，都不用引用修饰符
}
```

