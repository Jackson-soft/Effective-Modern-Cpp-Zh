# Item 22:当使用 Pimpl 的时候在实现文件中定义特殊的成员函数

如果你曾经因为程序过多的 build 次数头疼过，你肯定对于 Pimpl(pointer to implementation)做法很熟悉。它的做法是：把对象的成员变量替换为一个指向已经实现类(或者是结构体)的指针.将曾经在主类中的数据成员转移到该实现类中，通过指针来间接的访问这些数据成员。举个列子，假设 Widget 看起来像是这样：

```cpp
class Widget{			//in header "widget.h"
public:
	Widget();
	...
private:
	std::string name;
	std::vector<double> data:
	Gadget g1,g2,g3;
	//Gadget is some user-defined  type
}
```

因为 Widget 的数据成员是 std::string, std::vector 以及 Gadget 类型，为了编译 Widget,这些类型的头文件必须被包含进来,使用 Widget 的客户必须#include <string>,<vector>,以及 gadget.h。这些头文件增加了使用 Widget 的客户的编译时间，并且让客户依赖这些头文件的内容。如果一个头文件里的内容发生了改变，使用 Widget 的客户必须被重新编译。虽然标准的头文件<string>和<vector>不怎么发生变化，但 gadget.h 的头文件可能经常变化。

应用 C++ 98 的 Pimpl 做法，我们将数据成员变量替换为一个原生指针，指向了一个只是声明并没有被定义的结构体

```cpp
class Widget{			//still in header "widget.h"
public:
	Widget();
	~Widget();	//dtor is needed-see below
	...
private:
	struct Impl;		//declare implementation struct
	Impl *pImpl;		//and pointer to it
}
```

Widget 已经不再引用 std::string,std::vector 以及 gadget 类型，使用 Widget 的客户就可以不#include 这些头文件了。这加快了编译速度，并且 Widget 的客户也不受到影响。

一种只声明不定义的类型被称作 incomplete type.Widget::Impl 就是 incomplete type。对于 incomplete type,我们能做的事情很少，但是我们可以声明一个指向它的指针。Pimpl 做法就是利用了这一点。

Pimpl 做法的第一步是：声明一个成员变量，它是一个指向 incomplete type 的指针。第二步是，为包含以前在原始类中的数据成员的对象(本例中的\*pImpl)做动态分配内存和回收内存。分配以及回收的代码在实现文件中。本例中，对于 Widget 而言,这些操作在 widget.cpp 中进行：

```cpp
#include "widget.h"			//in impl,file "widget.cpp"
#include "gadget.h"
#include <string>
#include <vector>

struct Widget::Impl{
	std::string name;			//definition of Widget::Impl with data members formerly in Widget
	std::vector<double> data;
	Gadget g1,g2,g3;
}

Widget::Widget():pImpl(new Impl)	//allocate data members for this Widget object
{}

Widget::~Widget()	//destroy data members for this object
{
	delete pImpl;
}
```

在上面的代码中，我还是使用了#include 指令，表明对于 std::string,std::vector 以及 Gadget 头文件的依赖还继续存在。然后，这些依赖从 widget.h(被使用 Widget 的客户所使用，对它们可见)转移到了 widget.cpp(只对 Widget 的实现者可见)中，我已经高亮了(不好意思，在代码中高亮这种语法俺做不到啊---译者注)分配和回收 Impl 对象的代码。因为需要在 Widget 析构是，对 Impl 对象的内存进行回收，所以 Widget 的析构函数是必须要写的。

但是我给你展示的是 C++ 98 的代码，散发着上个世纪的上古气息.使用了原生指针，原生的 new 和原生的 delete，全都是原生的啊！本章的内容是围绕着“智能指针大法好,退原生指针保平安”的理念。如果我们想要在一个 Widget 的构造函数中动态分配一个 Widget::Impl 对象，并且在 Widget 析构时，也析构 Widget::Impl 对象。那么 std::unique_ptr(请看 Item 18)就是我们想要的最合适的工具。将原生 pImpl 指针替换为 std::unique_ptr。头文件的内容变为这样：

```cpp
class Widget{
public:
	Widget();
	...
private:
	struct Impl;
	std::unique_ptr<Impl> pImpl;//use smart pointer
	//instead of raw pointer
}
```

实现文件的内容则变成了这样：

```cpp
#include "widget.h"			//in "widget.cpp"
#include "gadget.h"
#include <string>
#include <vector>

struct Widget::Impl{
	std::string name;			//as before
	std::vector<double> data;
	Gadget g1,g2,g3;
}

Widget::Widget()
:pImpl(std::make_unique<Impl>())	//per Item 21,create
{}										//std::unique_ptr
										//via std::make_unique
```

你会发现 Widget 的析构函数不复存在。这是因为我们不需要在析构函数里面写任何代码了。std::unique_ptr 在自身销毁时自动析构它指向的区域，所以我们不需要自己回收任何东西。这就是智能指针的一项优点：它们消除了我们需要手动释放资源的麻烦。

但是呢，使用 Widget 的客户的一句很平凡的用法，就编译出错了啊

```cpp
#include "widget.h"
Widget w; //error
```

你所受到的错误信息内容依赖于你所使用的编译器类型，但是产生的内容大致都是：在 incomplete type 上使用了 sizeof 和 delete.这些操作在该类型上是禁止的。

Pimpl 做法结合 std::unique_ptr 竟然会产生错误，这很让人震惊啊。因为(1)std::unique_ptr 自身标榜支持 incomplete type.(2)Pimpl 做法是 std::unique_ptr 众多的使用场景之一。幸运的是，让代码工作起来也是很简单地。我们首先需要理解为啥会出错。

在执行 w 被析构(如当出作用域时)的代码时，报了错误。在此时，Widget 的析构函数被调用。在定义使用 std::unique_ptr 的 Widget 的时候，我们并没有声明析构函数，因为我们不需要在 Widget 的析构函数内写任何代码。依据编译器自动生成特殊成员函数(请看 Item 17)的普通规则，编译器为我们生成了一个析构函数。在那个自动生成的析构函数中，编译器插入代码，调用 Widget 的数据成员 pImpl 的析构函数。pImpl 是一个`std::unique_ptr<Widget::Impl>`,即，一个使用默认 deleter 的 std::unique_ptr.默认 deleter 是一个函数，对 std::unique_ptr 里面的原生指针调用 delete.然而，在调用 delete 之前，编译器通常会让默认 deleter 先使用 C++ 11 的 static_assert 来确保原生指针指向的类型不是 imcomplete type(staticassert 编译时候检查,assert 运行时检查---译者注).当编译器生成 Widget w 的析构函数时，调用的 static_assert 检查就会失败，导致出现了错误信息。在 w 被销毁时，这些错误信息才会出现，但是因为与其他的编译器生成的特殊成员函数相同，Widget 的析构函数也是 inline 的。出错指向 w 被创建的那一行，因为改行创建了 w,导致后来(w 出作用域时)w 被隐性销毁。

为了修复这个问题，你需要确保，在产生销毁`std::unique_ptr<Widget::Impl>`的代码时，Widget::Impl 是完整的类型。当它的定义被编译器看到时，它就是完整类型了。而 Widget::Impl 在 widget.cpp 中被定义。所以编译成功的关键在于，让编译器只在 widget.cpp 内，在 widget::Impl 被定义之后，看到 Widget 的析构函数体(该函数体就是放置编译器自动生成销毁 std::unique_ptr 数据成员的代码的地方)。

像那样安排很简单，在 widget.h 中声明 Widget 的析构函数，但是不要在其中定义：

```cpp
class Widget{			//as before, in "widget.h"
public:
	Widget();
	~Widget();		//declaration only
	...
private:				//as before
	struct Impl;
	std::unique_ptr<Impl> pImpl;
}
```

在 widget.cpp 里面的 Widget::Impl 定义之后再定义析构函数：

```cpp
#include "widget.h"			//as before in "widget.cpp"
#include "gadget.h"
#include <string>
#include <vector>

struct Widget::Impl{
	std::string name;			//as before definition of
	std::vector<double> data;	//Widget::Impl
	Gadget g1,g2,g3;
}

Widget::Widget()
:pImpl(std::make_unique<Impl>())	//as before

Widget::~Widget(){}	//~Widget definition
```

这样的话就没问题了，增加的代码量也很少。但是如果你想要强调，编译器生成的析构函数会做正确的事情，你声明析构函数的唯一原因是，想要在 Widget.cpp 中生成它的定义，你可以用“=default”定义析构函数体：

```cpp
Widget::~Widget()=default;		//same effect as above
```

使用 Pimpl 做法的类是自带支持 move 语义的候选者，因为编译器生成的 move 操作正是我们想要的：对潜在的 std::unique_ptr 上执行 move 操作。就像 Item 17 解释的那样，Widget 声明了析构函数，编译器就不会自动生成 move 操作了。所以如果你想要支持 move,你必须自己去声明这些函数。鉴于编译器生成的版本就可以胜任，你可能会像下面那样实现：

```cpp
class Widget{			//still in "widget.h"
public:
	Widget();
	~Widget();
	...
	Widget(Widget&& rhs) = default;		//right idea,
	Widget& operator=(Widget&& rhs) = default //wrong code!
	...
private:				//as before
	struct Impl;
	std::unique_ptr<Impl> pImpl;
};
```

这样的做法会产生和声明一个没有析构函数的 class 一样，产生同样的问题，产生问题的原因本质也一样。对于编译器生成的 move 赋值操作符，它对 pImpl 再赋值之前，需要先销毁它所指向的对象，然而在 Widget 头文件中，pImpl 指向的仍是一个 incomplete type.对于编译器生成的 move 构造函数。问题在于编译器会在 move 构造函数内抛出异常的事件中，生成析构 pImpl 的代码，对 pImpl 析构(destroying pImpl)需要 Impl 的类型是完整的。

问题一样，解决方法自然也一样：将 move 操作的定义写在实现文件 widget.cpp 中：
widget.h:

```cpp
class Widget{			//still in "widget.h"
public:
	Widget();
	~Widget();
	...
	Widget(Widget&& rhs);				//declarations
	Widget& operator=(Widget&& rhs); //only
	...
private:				//as before
	struct Impl;
	std::unique_ptr<Impl> pImpl;
};
```

widget.cpp

```cpp
#include <string>			//as before,
...								//in "widget.cpp"
sturct:Widget::Impl {...}; //as before

Widget::Widget()				//as before
:pImpl(std::make_unique<Impl>())
{}

Widget::~Widget() = default; //as before

Widget::Widget(Widget&& rhs) = default;
Widget& Widget::operator=(Widget&& rhs) = default:
//definitions
```

Pimpl 做法是一种减少 class 的实现和 class 的使用之间编译依赖的一种方式，但是，从概念上来讲，这种做法并不改变类的表现方式。原来的 Widget 类包含了 std::string,std::vector 以及 Gadget 数据成员，并且，假设 Gadget，像 std::string 和 std::vector 那样，可以被拷贝。所以按理说 Widget 也要支持拷贝操作。我们必须要自己手写这些拷贝函数了，因为(1)对于带有 move-only 类型(像 std::unique_ptr)的类，编译器不会生成拷贝操作的代码.(2)即使生成了，生成的代码只会拷贝 std::unique_ptr(即，执行浅拷贝)，而我们想要的是拷贝指针所指向的资源(即，执行深拷贝)。

现在的做法我们已经熟悉了，在头文件中声明这些函数，然后在实现文件中实现这些函数；

widget.h:

```cpp
class Widget{			//still in "widget.h"
public:
	...					//other funcs, as before
	Widget(const Widget& rhs);				//declarations
	Widget& operator=(const Widget& rhs); //only
private:				//as before
	struct Impl;
	std::unique_ptr<Impl> pImpl;
};
```

widget.cpp

```cpp
#include <string>			//as before,
...								//in "widget.cpp"
sturct:Widget::Impl {...}; //as before

Widget::~Widget() = default; //other funcs,as before

Widget::Widget(const Widget& rhs); //copy ctor
: pImpl(std::make_unique<Impl>(*rhs.pImpl))
{}

Widget& Widget::operator=(const Widget& rhs)//copy operator=
{
	*pImpl = *rhs.pImpl;
	return *this;
}
```

两个函数的实现都比较常见。每种情况下，我们都是从源对象(rhs)到目的对象(\*this)，简单的拷贝了 Impl 的结构体的内容.我们利用了这样的事实：编译器会为 Impl 生成拷贝操作的代码，这些操作会自动将 stuct 的内容逐项拷贝，就不需要我们手动来做了。我们因此通过调用 Widget::Impl 的编译器生成的拷贝操作符来实现了 Widget 拷贝操作符。在 copy 构造函数中，注意到我们遵循了 Item 21 的建议，不直接使用 new，而是优先使用了 std::make_unique.

在上面的例子中，为了实现 Pimpl 做法，std::unique_ptr 是我们使用的智能指针类型，因为对象内(在 Widget 内)的 pImpl 对对应的实现对象(Widget::Impl 对象)拥有独占所有权。但是，很有意思的是，当我们对 pImpl 使用 std::shared_ptr 来替代 std::unique_ptr,我们发现本 Item 的建议不再适用了。没必要在 Widget.h 中声明析构函数，没有了用户自己声明的析构函数，编译器会很乐意生成 move 操作代码，而且生成的代码表现的行为正合我们意。widget.h 变得如下所示：

```cpp
class Widget{					//in "widget.h"
public:
	Widget();
	...						//no declarations for dtor
							//or move operations
private:
	struct Impl;						//std::shared_ptr
	std::shared_ptr<Impl> pImpl;	//instead of std::unique_ptr
};
```

`#include widget.h`的客户代码：

```cpp
Widget w1;
auto w2(std::move(w1));			//move-consturct w2

w1 = std::move(w2);				//move-assign w1
```

所有代码都会如我们所愿通过编译，w1 会被默认构造，它的值会被 move 到 w2,之后 w2 的值又被 move 回 w1.最后 w1 和 w2 都得到析构(这使得指向的 Widget::Impl 对象被析构)

对 pImpl 应用 std::unique_ptr 和 std::shared_ptr 的表现行为不同的原因是：它们之间支持自定义的 deleter 的方式不同。对于 std::unique_ptr，deleter 的类型是智能指针的一部分，这就使得编译器生成更小的运行时数据结构以及更快的运行时代码成为可能。更好的效率的结果是要求当编译器生成特殊函数(如析构以及 move 操作)被使用时，std::unique_ptr 所指向的类型必须是完整的。对于 std::shared_ptr 来说，deleter 的类型不是智能指针的一部分。虽然会造成比较大的运行时数据结构和慢一些的代码。但是在调用编译器生成的特殊函数时，指向的类型不需要是完整的。

对于 Pimpl 做法，在`std::unique_ptr`和`std::shared_ptr`的特点之间，其实并没有一个真正的权衡。因为 Widget 和 Widget::Impl 之间是独占的拥有关系，`std::unique_ptr`在此项工作中很合适。然而，在一些其他的场景中，共享式拥有关系存在，`std::shared_ptr`才是一个合适的选择，就没必要像依靠`std::unique_ptr`这样的函数定义的做法了。

| 要记住的东西                                                                                                                                                       |
| :----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Pimpl 做法通过减少类的实现和类的使用之间的编译依赖减少了 build 次数                                                                                                |
| 对于`std::unique_ptr` pImpl 指针，在 class 的头文件中声明这些特殊的成员函数，在 class 的实现文件中定义它们。即使默认的实现方式(编译器生成的方式)可以胜任也要这么做 |
| 上述建议适用于`std::unique_ptr`,对`std::shared_ptr`无用                                                                                                            |
