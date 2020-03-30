#Item 24: Distinguish universal references from rvalue references.

大多数人说真相可以让我们感到自由，但是在某些情况下，一个巧妙的谎言也可以让人觉得非常轻松。这个 Item 就是要编制一个“谎言”。因为我们是在和软件打交道。所以我们避开“谎言”这个词：我们是在编制一种“抽象”的意境。

为了声明一个类型 T 的右值引用，你写下了 T&&。下面的假设看起来合理：你在代码中看到了一个"T&&"时，你看到的就是一个右值引用。但是，它可没有想象中那么简单：

```cpp
void f(Widget&& param);			//rvalue reference
Widget&& var1 = Widget();		//rvalue reference
auto&& var2 = var1;				//not rvalue reference

template<typename T>
void f(std::vector<T>&& param)		//rvalue reference

template<typename T>
void f(T&& param);				//not rvalue reference
```

实际上，“T&&”有两个不同的意思。首先,当然是作为 rvalue reference,这样的引用表现起来和你预期一致：只和 rvalue 做绑定，它们存在的意义就是表示出可以从中 move from 的对象。

“T&&”的另外一个含义是：既可以是 rvalue reference 也可以是 lvalue reference。这样的 references 在代码中看起来像是 rvalue reference（即"T&&"）,但是它们*也*可以表现得就像他们是 lvalue refernces（即"T&"）那样.它们的 dual nature 允许他们既可以绑定在 rvalues(like rvalue references)也可以绑定在 lvalues(like lvalue references)上。进一步来说，它们可以绑定到 const 或者 non-const,volatile 或者 non-volatile，甚至是 const + volatile 对象上面。它们几乎可以绑定到任何东西上面。为了对得起它的全能，我决定给它们起个名字：universal reference.(Item25 将会解释 universal references 总是可以将 std::forward 应用在它们之上，本书出版之时，C++委员会的一些人开始将 universal references 称之为 forward references).

两种上下文中会出现 universal references。最普通的一种是 function template parameters，就像上面的代码所描述的例子那样:

```cpp
template<typename T>
void f(T&& param); 			//param is a universal reference
```

第二种 context 就是 auto 的声明方式，如下所示：

```cpp
auto&& var2 = var1;			//var2 is a universal reference
```

这两种 context 的共同点是:都有 type deduction 的存在。在 template fucntion f 中,参数 param 的类型是被 deduce 出来的，在 var2 的声明中，var2 的类型也是被 deduce 出来的。和接下来的例子(也可以和上面的栗子一块儿比)对比我们会发现，下面栗子是不存在 type deduction 的。如果你看到"T&&",却没有看到 type deduction.那么你看到的就是一个 rvalue reference:

```cpp
void f(Widget&& param);			//no type deduction
									//param is an rvalue reference

Widget&& var1 = Widget();		//no type deduction
									//var1 is an rvalue reference
```

因为 universal references 是 references，所以它们必须被初始化。universal reference 的 initializer 决定了它表达的是 rvalue reference 或者 lvalue reference。如果 initializer 是 rvalue，那么 universal reference 对应的是 rvalue reference.如果 initializer 是 lvalue,那么 universal reference 对应的就是 lvalue reference.对于身为函数参数的 universal reference，initializer 在 call site（调用处）被提供：

```cpp
template<typename T>
void f(T&& param);		//param is a universal reference

Widget w;
f(w);					//lvalue passed to f;param's type is Widget&(i.e., an lvalue reference)
f(std::move(w));		//rvalue passed to f;param's type is Widget&&(i.e., an rvalue reference)
```

对 universal 的 reference 来说，type deduction 是必须的，但还是不够，它要求的格式也很严格，必须是"T&&".再看下我们之前写过的栗子：

```cpp
template<typename T>
void f(std::vector<T>&& param);	//param is an rvalue reference
```

当 f 被调用时，类型 T 会被 deduce（除非调用者显式的指明类型，这种边缘情况我们不予考虑）。param 声明的格式不是 T&&，而是 std::vector<T>&&.这就说明它不是 universal reference,而是一个 rvalue reference.如果你传一个 lvalue 给 f，那么编译器肯定就不高兴了。

```cpp
std::vector<int> v;
f(v);				//error! can't bind lvalue to rvalue reference
```

即使一个最简单前缀 const.也可以把一个 reference 成为 universal reference 的可能抹杀：

```cpp
template<typename T>
void f(const T&& param);		//param is an rvalue reference
```

如果你在一个 template 里面，并且看到了 T&&这样的格式，你可能就会假设它就是一个 universal reference.但是并非如此，因为还差一个必要的条件：type deduction.在 template 里面可不保证一定有 type deduction.看个例子，std::vector 里面的 push_back 方法。

```cpp
template<class T, class Allocator = allocator<T>>
class vector{
public:
	void push_back(T&& x);
	...
}
```

以上便是只有 T&&格式却没有 type deduction 的例子，push_back 的存在依赖于一个被 instantiation 的 vector.用于 instantiation 的 type 就完全决定了 push_back 的函数声明。也就是说

```cpp
std::vector<Widget> v;
```

使得 std::vector 的 template 被 instantiated 成为如下格式:

```cpp
class vector<Widget, allocator<Widget>>{
public:
	void push_back(Widget&& x);	//rvalue reference
	...
};
```

如你所见，push_back 没有用到 type deduction.所以这个 vector<T>的$push_back$（有两个 overload 的$push_back$）所接受的参数类型是 rvalue-reference-to-T.

与之相反，std::vector 中概念上相近的$emplace_back$函数确实用到了 type deduction:

```cpp
template<class T,class Allocator=allocator<T>>

class vector{
public:
	template <class... Args>
	void emplace_back(Args&&... args);
	...
};
```

type parameter`Args`独立于 vector 的 type parameter `T`,所以每次调用`emplace_back`的时候，`Args`就要被 deduce 一次。（实际上，`Args`是一个 parameter pack.并不是 type parameter.但是为了讨论的方便，我们姑且称之为 type parameter）。

我之前说 universal reference 的格式必须是`T&&`, 事实上，emplace_back 的 type parameter 名字命名为 Args，但这不影响 args 是一个 universal reference,管它叫做 T 还是叫做 Args 呢，没啥区别。举个例子，下面的 template 接受的参数就是 universal reference.一是因为格式是"type&&",二是因为 param 的 type 会被 deduce(再一次提一下，除非 caller 显示的指明了 type 这种边角情况).

```cpp
template<typename MyTemplateType>			//param is a
void someFunc(MyTemplateType&& param); //universal reference
```

我之前提到过 auto 变量可以是 universal references.更准确的说，声明为 auto&&的变量就是 universal references.因为类型推导发生并且它们也有正确的格式("T&&").auto 类型的 universal references 并不想上面说的那种用来做 function template parameters 的 universal references 那么常见，在最近的 C++ 11 和 C++ 14 中，它们变得非常活跃。C++ 14 中的 lambda expression 允许声明 auto&&的 parameters.举个栗子，如果你想写一个 C++ 14 的 lambda 来记录任意函数调用花费的时间，你可以这么写:

```cpp
auto timeFuncInvocation =
	[](auto&& func, auto&&... params)		//C++ 14
	{
		start timer;
		std::forward<decltype(func)>(func)(
			std::forward<decltype(params)>(params)... //invoke func on params
		);
		stop timer and record elapsed time
	}
```

如果你对于"std::forward<decltype(blah blah blah)"的代码的反应是“这是什么鬼!？”，这只是意味着你还没看过 Item33，不要为此担心。
func 是一个可以绑定到任何 callable object(lvaue 或者 rvalue)的 universal reference.args 是 0 个或多个 universal references(即 a universal reference parameter pack)，它可以绑定到任意 type，任意数量的 objects.所以，多亏了 auto universal reference,timeFuncInvocation 可以记录 pretty much any 的函数调用(any 和 pretty much any 的区别，请看 Item 30).

本 Item 中 universal reference 的基础，其实是一个谎言，呃，或者说是一种"abstraction".隐藏的事实被称之为*reference collapsing*,Item 28 会讲述。但是该事实并不会降低它的用途。rvalue references 以及 universal references 会帮助你更准确第阅读 source code(“Does that T&& I’m looking at bind to rvalues only or to everything?”),和同事沟通时避免歧义(“I’m using a universal reference here, not an rvalue reference...”),它也会使得你理解 Item 25 和 Item 26,这两条都依赖于此区别。所以，接受理解这个 abstraction 吧。牛顿三大定律(abstraction)比爱因斯坦的相对论(truth)一样有用且更容易应用，universal reference 的概念也可以是你理解 reference collapsing 的细节。

| 要记住的东西                                                                                                                                                      |
| :---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 如果一个函数的 template parameter 有着 T&&的格式，且有一个 deduce type T.或者一个对象被生命为 auto&&,那么这个 parameter 或者 object 就是一个 universal reference. |
| 如果 type 的声明的格式不完全是 type&&,或者 type deduction 没有发生，那么 type&&表示的是一个 rvalue reference.                                                     |
| universal reference 如果被 rvalue 初始化，它就是 rvalue reference.如果被 lvalue 初始化，他就是 lvaue reference.                                                   |
