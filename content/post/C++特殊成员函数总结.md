+++
author = ""
comments = true
date = "2013-08-25T09:48:23+08:00"
draft = false
image = ""
menu = ""
share = true
slug = "C++特殊成员函数总结"
tags = ["C++"]
title = "C++特殊成员函数总结"

+++
&emsp;&emsp;所谓特殊成员函数，按照c++11的标准描述：
>The default constructor, copy constructor and copy assignment operator , move constructor and move assignment operator , and destructor are special member functions. [Note: The implementation will implicitly declare these member functions for some class types when the program does not explicitly declare them. The implementation will implicitly define them if they are odr-used.
总结下就是：
.   Default constructors  //默认构造函数
.   Destructors           //析构函数
.   Copy constructors     //拷贝构造函数
.   Copy assignment operators //拷贝赋值操作 
.   Move constructors     // move 构造函数
.   Move assignment operators  //move 复制操作

根据标准中的描述，对于特殊成员函数，如果程序中没有明确的声明，编译器实现会进行隐式的声明，而且符合odr规则，也会提供一份隐式的实现。（odr--One Definition Rule）。这里我们举个例子验证下：

	//这六个成员函数我们一个都没有显式的声明和定义
	class X{
	private:
		int data;
	};
	int main()
	{
		X x1;  //但是我们却可以通过默认构造函数生成一个对象
		cout <<"x1.data="<<x1.get_data()<<endl;
		X x2 = x1; //通过拷贝构造函数生成新的对象（对的，这种情况并不是调的assignment，后面会有演示）
		cout <<"x2.data="<<x2.get_data()<<endl;
		X x3;
		cout <<"x3.data="<<x3.get_data()<<endl;
		X3 = x1; //这才是assignment 操作
		cout <<"x3.data="<<x3.get_data()<<endl;
		return 0;
	}
	
执行结果：![](http://77g3g7.com1.z0.glb.clouddn.com/special_member_function_1.png)
正如我们所料，第1、2、4行输出的data值相等。说明隐式实现的拷贝构造和赋值操作会对data的值进行拷贝。	
&emsp;&emsp;利用这一特性方便我们编程的同时，有些时候也会带来一些问题，最简单的，比如有时候我们想禁止对X类型的对象进行进行拷贝，赋值。语言层面就没有提供这样的支持，直到c++11引入了delete关键字。我们可以这样修改上面的class X:

	class X{
	public:
	   //这里有个坑待填，好像是构造函数只要显示的写了一个，不论有没有delete，
	   //默认构造函数不能再implicitly,否则会报错（至少我mac下用clang编译报错了）
	   	X()
		:data(random())
		{}
		X(const X&) = delete;
		X& operator = (const X&) = delete;
	private:
		int data;
	};
此时我们再编译上面那段相同的测试代码，有两个编译错误：
![](http://77g3g7.com1.z0.glb.clouddn.com/special_member_function_error.png)

&emsp;&emsp;与delete对应的一个关键字是default,假如上面的class X，我们不想手写default constructor：

	class X{
	public:
		X() = default;
		X(const X&) = default;
		X& operator = (const X&) = default;
	public:
		int get_data() const
		{
			return data;
		}
	private:
		int data;
	};
&emsp;&emsp;另外，这些special member function 同样遵守c++的access rules，标准中的原话是这样说的：
>Special member functions obey the usual access rules (Clause 11). [Example: declaring a constructor protected ensures that only derived classes and friends can create objects using it. — end example ]

如果我们灵活运用public、protect、private 等access rules，可以实现，满足各种特殊需求，比如常用的所谓单例模式，强制把各种可能构造出对象的相关的成员函数声明为private，仅提供一个static 成员函数用于获取/生成这个单例对象：

	class Singleton  
	{
    	public:
        	static Singleton& Instance()
    		{
        	static Singleton instance；
        	return instance；
    		}
	    private:
        	Singleton()；
        	~Singleton()；
        	Singleton(const Singleton&);
        	Singleton& operator=(const Singleton&);
	};