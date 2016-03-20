+++
author = ""
comments = true
date = "2014-03-21T16:20:01+08:00"
draft = false
image = ""
menu = ""
share = true
slug = "C++右值引用"
tags = ["C++", ]
title = "C++右值引用"

+++

### value category   

&emsp;&emsp;左值/右值是沿用c语言中的叫法，直观上理解就是分别出现在赋值表达式左右。c++标准中关于左值和右值的描述如下:

>An lvalue (so called, historically, because lvalues could appear on the left-hand side of an assignment expression)designates a function or an object. [ Example: If E is an expression of pointer type, then *E is an lvalue expression referring to the object or function to which E points. As another example, the result of calling a function whose return type is an lvalue reference is an lvalue. — end example ]

另外还有一种更加直观的描述(判断)：

>An lvalue ("left value") expression is an expression that has identity and cannot be moved from.--[cpprefrence](http://en.cppreference.com/w/cpp/language/value_category)  

以下是几种左值表达式的例子:  

```C++
	a=b
	++a,--a
	std::cin,std::endl
```

>An rvalue (so called, historically, because rvalues could appear on the right-hand side of an assignment expression) is an xvalue, a temporary object (12.2) or subobject thereof, or a value that is not associated with an object.

同理cppreference中关于右值表达式的描述:

>"An rvalue ("right value") expression is an expression that is either a prvalue or an xvalue. It can be moved from. It may or may not have identity."
几种右值例子：
   
	42 //常量
	a++,a--
	a+b,a&b
	&a
	std::move(x)
	[](int x){return x*x} //lambda表达式
这里提到的xvalue是新的标准中(c++11) 新增加的value category,另外还有glvalue(general lvalue),prvalue(pure rvalue)。glvalue只是个混合概念，由rvalue和xvalue构成；pvalue在标准中的描述为:

>A prvalue (“pure” rvalue) is an rvalue that is not an xvalue. [ Example: The result of calling a function whose return type is not a reference is a prvalue. The value of a literal such as 12, 7.3e5, or true is also a prvalue.

可以这样理解，rvalue里不是xvalue的就是pvalue。cppreference中给的描述是："A prvalue ("pure rvalue") expression is an expression that does not have identity and can be moved from."

>An xvalue (an “eXpiring” value) also refers to an object, usually near the end of its lifetime (so that its resources may be moved, for example). Certain kinds of expressions involving rvalue references (8.3.2) yield xvalues. [ Example: The result of calling a function whose return type is an rvalue reference to an object type is an xvalue (5.2.2). — end example ]

可以理解为，xvalue是rvalue的一种特例。关于xvalue最有代表性的例子是： 

```C++
	a.m //其中a是rvalue，m是其非静态非引用成员
	std::move(x)
```
我们可以查看STL中move的实现:

```C++
	template <class _Tp>
	inline _LIBCPP_INLINE_VISIBILITY _LIBCPP_CONSTEXPR_AFTER_CXX11
	typename remove_reference<_Tp>::type&&
	move(_Tp&& __t) _NOEXCEPT
	{
    	typedef typename remove_reference<_Tp>::type _Up;
    	return static_cast<_Up&&>(__t);
	}
```
其返回值类型正式右值引用(rvalue reference)，正式标准中举的Example 类型。
![](http://77g3g7.com1.z0.glb.clouddn.com/valueCategory.png)  
### rvalue reference  
&emsp;&emsp;好了关于c++的value category背景介绍就到这里，现在开始进入主题 rvalue reference。关于引用我们应该都很熟悉了，教材中都说了，引用是变量的别名`C++ references allow you to create a new name for an existing object.`例如：  
 
```C++
	int var=42;	int& ref=var;	ref=99;	cout<<"var="<<var<<endl;
```
我们会的到输出：var=99;但是如果我们像下面这样写，编译器就不干了。因为传统的&引用是针对左值的，`Lvalue references can only be bound to lvalues`.

```C++
	int &ref=42;
```
但是为了解决一些函数调用的传参问题，C++允许这样写：

```C++
	int const&=42；
```
这实际上是标准中的一个例外，在引入rvalue reference之前，标准中允许你给一个接受引用参数的函数传递临时变量：

```C++
	void print(std::string const& s); 	print("hello");// Create temporary std::string object
```
c++11中引入&&符号，用于标示右值引用,并严格区分左右值。

```C++
	int&& i=42;	int j=42; 	int&& k=j; //error
```
这样我们就可以重写上面的printf函数：

```C++
	void print(std::string const& s);
	void print(std::string const&& s);
```
这样我们再调用:`print("hello");`会调用第二个函数的实现。这正是C++11中实现move constructor 和 move assignment operator的基础。

### move semantics  
&emsp;&emsp;考虑这样一种情况，实现一个函数，接受一个std:vector<int>参数，函数内部需要对这个传入vector进行修改和处理，但是要保证不能影响到原vector,以前我们可以这样实现：

```C++
	void process_copy(std::vector<int> const & vec)
	{
		std::vector<int> n_vec(vec);
		n_vec.push_back(40);
		...
	}
```
对于这种实现，我们既可以传入lvalue实参，也可以传入rvalue实参（参见前文），但是不可避免的我们在内部处理的时候都需要对实参进行一份拷贝，尽管有时候实参只是个用完就会结束生命周期的临时变量。很容易设想，如果我们能够区分传入的是个rvalue，我们只是`move`它的内容而不是对它进行拷贝，这对于占用存储空间很大的结构，会有非常理想的节省时间和存储空间的优化效果。

```C++
	void process_copy(std::vector<int>&& vec)
	{
		vec.push_back(40);
		...
	}
```
当然，关于move semantics最终要的应用就是 move constructor 和 move assignment operator

```C++
	class X{
	public:
    	X()
            :data(new char[10000])
    	{
        print_tips("default constructor");
    	}
    ~X()
    {
        delete [] data;
        print_tips("destructors");
    }
    X(X& other)
            :data(new char[10000])
    {
        std::copy(other.data,other.data+10000,data);
        print_tips("copy constructor");
    }
    X(X&& other)
            :data(other.data)
    {
        other.data = nullptr;
        print_tips("move constructor");
    }
    X& operator=(const X& other)
    {
        data = new char[10000];
        std::copy(other.data,other.data+10000,data);
        print_tips("copy assignment constructor");
        return *this;
    }
    X& operator=(X&& other)
    {
        data = other.data;
        other.data = nullptr;
        print_tips("move assignment constructor");
        return *this;
    }
	private:
    	char *data;
    	void  print_tips(std::string const& str)
    	{
        	cout <<"in "<<str<<endl;
    	}
	};
```
可以测试下不同情况下的函数调用情况：

```C++
	X x1;
	X x2(x1),x3(std::move(x1));
	x2 = x3;
	x2 = std::move(x3);
```
可以预见的是的顺序是(忽略default constructor 和 destructor)：copy constructor，move constructor，copy assignment operator，move assignment operator。执行这段代码验证下输出：

![rvalueReference](http://77g3g7.com1.z0.glb.clouddn.com/rvalueReference.png "rvalueReference")

和std::move(x)等效的一种写法是 static_cast<X&&>(x),如果你明确知道一个lvalue x不会再用了，想把它的内容move过来，move和static_caset可以助你将其变成左值然后实现move semantic操作。
&emsp;&emsp;另外有一点要注意的事，对于函数的右值引用参数，可以将其bind to一个右值，但是在函数内部，其是被当做lvalue对待的，例如:

```C++
	void do_stuff(X&& x)
	{
		X a(x);               //call copy constructor
		X b(std::move(x));    // call move constructor
	}
```
&emsp;&emsp;好了，关于rvalue reference的相关总结暂时告一段落，其他相关内容遇到具体问题再做补充。最近在看C++ concurrency，Thread Library才是关于 move reference使用的重灾区，以后再表。 
	
	
