+++
author = ""
comments = true
date = "2014-06-07T17:50:24+08:00"
draft = false
image = ""
menu = ""
share = true
slug = "C++智能指针"
tags = ["C++"]
title = "C++智能指针"

+++

&emsp;&emsp;关于智能指针，C++11中废弃了原来的auto_ptr，引入了shared\_ptr,unique\_ptr,weak\_ptr。在引入智能指针之前，我们使用原生指针(built-in or raw)来手动控制内存的操作和释放，稍有不慎常会发生各种内存泄露或者undefined行为，这点使用c/c++开发过项目的必然都深有体会。世间万事万物都有其两面性，我们享受直接内存操作的便利快感的同时，有时候难免要忍受调试各种稀奇古怪的内存bug的酸爽。下面将逐个剖析新的智能指针系列引入的这几种类型的背后实现机制以及对于开发工作会有那些帮助。
#### shared_ptr
&emsp;&emsp;首先这三种类型都是STL中实现的，具体不同的编译器附带的STL中的实现会有差异(事实上差异还比较大)，但是我们使用的时候需要引用的同一个头文件名:

	#include<memory>
由于本人的主要开发环境是OS X,对应的主力编译器是clang,clang默认由原来使用GNU的libstdc++到现在自己重新实现了一套c++标准库，叫libc++,所以这里不加特别说明的话，当介绍具体实现的时候，默认指的是libc++的实现。    
&emsp;&emsp;shared_基本用法:

``` C++

#include <iostream>
#include <memory>
using namespace std;

class X{
public:
    X()=default;
    X(int value)
            :data(value)
    {
        cout <<"constructor"<<endl;
    }
    ~X()
    {
        cout <<"destrucctor"<<endl;
    }
    int get_data() const
    {
        return  data;
    }

    friend ostream& operator << (ostream &out, const X& x)
    {
        out <<x.get_data();
        return out;
    }

private:
    int data;
};
void f()
{
    shared_ptr<X> sp(new X(5)); //
    cout<<*sp<<endl; //the same use as raw point
    cout <<"data = "<<sp->get_data()<<endl;
    cout <<"use_count = " <<sp.use_count()<<endl; // should print use_count = 1
    shared_ptr<X> sp1 = sp;
    cout <<"use_count = " <<sp.use_count()<<endl; // should print use_count = 2
    sp.reset();
    cout <<"use_count = " <<sp.use_count()<<endl; // should print use_count = 0 because sp is reseted
    shared_ptr<X> sp2 = sp1;
    cout <<"use_count = " <<sp1.use_count()<<endl; // should print use_count = 2
    sp = sp1;  // now can also assign to sp with a same(or convertible) type of shared_ptr
    cout <<"use_count = " <<sp.use_count()<<endl; // should print use_count = 3
    sp = nullptr; //the same effect as reset
    cout <<"use_count = " <<sp.use_count()<<endl; //shoud print use_cout = 0
    cout <<"use_count = " <<sp1.use_count()<<endl; // should print use_cout = 2
}
int main()
{
    f();
    return 0;
}
```
运行下测试程序验证输出结果正如如我们注释中所料：
![](http://77g3g7.com1.z0.glb.clouddn.com/shared_ptr.png)
仅仅介绍怎么用远不是在我们这篇文章的终极目标，下面让我们一层一层揭开智能指针的神秘面纱。从上面的例子我们已经可以看出，shared_ptr本质是一个模板类，这个模板类实例化的参数类型就是对应我们要“智能”管理对象类型(或者 convertible的类型)，讲起来还是蛮拗口的,举个栗子：

``` C++
class Y:public X
{
private:
    float value;
};
...
shared_ptr<X> sfp(new Y()); //这么写也是合法的
```
说了这么多，shared_ptr的本质就是一个包含raw指针以及关于这个raw指针的全局引用信息的对象，通过在这个对象的destructor中修改raw指针的全局引用信息来控制raw指针的释放时机。

``` C++
template<class _Tp>
class _LIBCPP_TYPE_VIS_ONLY shared_ptr
{
public:
    typedef _Tp element_type;
private:
    element_type*      __ptr_;
    __shared_weak_count* __cntrl_;

    struct __nat {int __for_bool_;};
public:
    _LIBCPP_CONSTEXPR shared_ptr() _NOEXCEPT;
    _LIBCPP_CONSTEXPR shared_ptr(nullptr_t) _NOEXCEPT;
    template<class _Yp>
        explicit shared_ptr(_Yp* __p,
                            typename enable_if<is_convertible<_Yp*, element_type*>::value, __nat>::type = __nat());
     			...
    element_type* get() const _NOEXCEPT {return __ptr_;}
    _LIBCPP_INLINE_VISIBILITY
    typename add_lvalue_reference<element_type>::type operator*() const _NOEXCEPT
        {return *__ptr_;}
    _LIBCPP_INLINE_VISIBILITY
    element_type* operator->() const _NOEXCEPT {return __ptr_;}
    _LIBCPP_INLINE_VISIBILITY
    long use_count() const _NOEXCEPT {return __cntrl_ ? __cntrl_->use_count() : 0;}
    			...
```
如上代码所示，`__ptr_`即是我们说的raw指针，`__cntrl_`就是那个控制raw指针的全局引用信息的对象指针，其中`__share_weak_count`又是继承自一个叫`__shared_count`的类:

``` C++
class _LIBCPP_TYPE_VIS __shared_weak_count
    : private __shared_count
{
    long __shared_weak_owners_;

public:
    _LIBCPP_INLINE_VISIBILITY
    explicit __shared_weak_count(long __refs = 0) _NOEXCEPT
        : __shared_count(__refs),
          __shared_weak_owners_(__refs) {}
protected:
    virtual ~__shared_weak_count();

public:
    void __add_shared() _NOEXCEPT;
    void __add_weak() _NOEXCEPT;
    void __release_shared() _NOEXCEPT;
    void __release_weak() _NOEXCEPT;
    _LIBCPP_INLINE_VISIBILITY
    long use_count() const _NOEXCEPT {return __shared_count::use_count();}
```
我们上个shared_ptr简单实用的例子中输出的use_count信息实际上完全是由这个`shared_count`来控制的,即实际是下面的__shared_owers_这个变量的值

``` C++
class _LIBCPP_TYPE_VIS __shared_count
{
    __shared_count(const __shared_count&);
    __shared_count& operator=(const __shared_count&);

protected:
    long __shared_owners_;
    virtual ~__shared_count();
private:
    virtual void __on_zero_shared() _NOEXCEPT = 0;

public:
    _LIBCPP_INLINE_VISIBILITY
    explicit __shared_count(long __refs = 0) _NOEXCEPT
        : __shared_owners_(__refs) {}

    void __add_shared() _NOEXCEPT;
    bool __release_shared() _NOEXCEPT;
    _LIBCPP_INLINE_VISIBILITY
    long use_count() const _NOEXCEPT {return __shared_owners_ + 1;}
};
```
`shared_ptr<X> sp(new X(5));`搞懂懂这个语句背后发生了什么，可谓费了好大一番周折，学习了C++中一种叫SFINAE的技巧------*匹配失败并不是一种错误(Substitution Failure Is Not An Error)*.
首先我们要搞懂shared_ptr模板类中下面这个声明的意思:

```C++
template<class _Yp>
explicit shared_ptr(_Yp* __p,typename enable_if<is_convertible<_Yp*, element_type*>::value, __nat>::type = __nat());
```
SFINAE名字听上去很高大上，其实知道is_convertible和enable_if表示什么意思后也不是太难理解.从字面上理解is_convertible应该就是判断两个类型是否可以相互转换,其模板类型：

```C++
template< class From, class To >
struct is_convertible;
```
`typename enable_if<is_convertible<_Yp*, element_type*>::value, __nat>::type = __nat()`这一语句实际上表示的意思是当is_convertible的value为true的时候声明一个带默认值的形参，即成功匹配后的构造函数类型应该是这样的
`shared_ptr(X *__p,__nat=__nat())`,为了验证下我们的分析，将我们上面的测试函数f()稍微修改一下：

```C++
void f()
{
    shared_ptr<X> sp(new X(5),{1});  //这里我们指定一个实参取代__nat() 
    cout<<*sp<<endl; //the same use as raw point
    cout <<"data = "<<sp->get_data()<<endl;
    cout <<"use_count = " <<sp.use_count()<<endl; // should print use_count = 1
    shared_ptr<X> sp1 = sp;
    cout <<"use_count = " <<sp.use_count()<<endl; // should print use_count = 2
    sp.reset();
    cout <<"use_count = " <<sp.use_count()<<endl; // should print use_count = 0 because sp is reseted
    shared_ptr<X> sp2 = sp1;
    cout <<"use_count = " <<sp1.use_count()<<endl; // should print use_count = 2
    sp = sp1;  // now can also assign to sp with a same(or convertible) type of shared_ptr
    cout <<"use_count = " <<sp.use_count()<<endl; // should print use_count = 3
    sp = nullptr; //the same effect as reset
    cout <<"use_count = " <<sp.use_count()<<endl; //shoud print use_cout = 0
    cout <<"use_count = " <<sp1.use_count()<<endl; // should print use_cout = 2
}
```
然后执行，输出相同的结果:

```C++
luobin@luobinMini:shared_point$ clang++ -std=c++11 main.cpp 
luobin@luobinMini:shared_point$ ./a.out 
constructor
5
data = 5
use_count = 1
use_count = 2
use_count = 0
use_count = 2
use_count = 3
use_count = 0
use_count = 2
destrucctor
```
下面是shared_ptr这个构造函数的具体模板实现
```C++
template<class _Tp>
template<class _Yp>
shared_ptr<_Tp>::shared_ptr(_Yp* __p,
                            typename enable_if<is_convertible<_Yp*, element_type*>::value, __nat>::type)
    : __ptr_(__p)
{
    unique_ptr<_Yp> __hold(__p);
    typedef __shared_ptr_pointer<_Yp*, default_delete<_Yp>, allocator<_Yp> > _CntrlBlk;
    __cntrl_ = new _CntrlBlk(__p, default_delete<_Yp>(), allocator<_Yp>()); //__contrl_实际上是个指向个__shared_ptr_pointer类型
    __hold.release();
    __enable_weak_this(__p);
}
```

__shared_ptr_pointer:

```C++
class __shared_ptr_pointer
    : public __shared_weak_count
{
    __compressed_pair<__compressed_pair<_Tp, _Dp>, _Alloc> __data_;
public:
    _LIBCPP_INLINE_VISIBILITY
    __shared_ptr_pointer(_Tp __p, _Dp __d, _Alloc __a)
        :  __data_(__compressed_pair<_Tp, _Dp>(__p, _VSTD::move(__d)), _VSTD::move(__a)) {}

#ifndef _LIBCPP_NO_RTTI
    virtual const void* __get_deleter(const type_info&) const _NOEXCEPT;
#endif

private:
    virtual void __on_zero_shared() _NOEXCEPT;  //这个是负责最终delete掉所管理的raw指针
    virtual void __on_zero_shared_weak() _NOEXCEPT; //这个是处理weak引用，暂不细表
};
```

其成员`__data_`是个`__compressd_pair`,其中first依旧是个`__compressed_pair`,second是个_Alloc(//叫对象生成器?)。first.first是_Alloc生成对象的指针类型(我们的raw指针)，first.second是个`default_delete`(叫对象销毁器?),这个才是关键。
形势逐渐明朗了，现在我们可以一窥shared_ptr如何控制raw指针的释放时机了，进入~shared_ptr():

```C++
template<class _Tp>
shared_ptr<_Tp>::~shared_ptr()
{
    if (__cntrl_)
        __cntrl_->__release_shared();
}
```
进入`__shared_weak_count` 的`__release_shared()`:

```C++
void
__shared_weak_count::__release_shared() _NOEXCEPT
{
    if (__shared_count::__release_shared())
        __release_weak();
}
```
这里调用下`__shared_count::__release_shared()`,true 的话继续处理`__release_weak()`,进入`__shared_count::__release_shared()`:

```C++
bool
__shared_count::__release_shared() _NOEXCEPT
{
    if (decrement(__shared_owners_) == -1) //表示这是raw指针的最后一个引用
    {
        __on_zero_shared();
        return true;
    }
    return false;
}
```
由于`__on_zero_shared()`是个虚函数,所以这里实际上调用的是`__shared_ptr_pointer`中的实现:

```C++
__shared_ptr_pointer<_Tp, _Dp, _Alloc>::__on_zero_shared() _NOEXCEPT
{
    __data_.first().second()(__data_.first().first()); //first.first就是我们的raw指针，这里被最终释放啦
    __data_.first().second().~_Dp(); //?这里是销毁器也要释放的意思吗，看着像default_delete的析构函数？
}
```
有前面分析已知fisrt().second()是一个default_delete:

```C++
template <class _Tp>
struct _LIBCPP_TYPE_VIS_ONLY default_delete
{
#ifndef _LIBCPP_HAS_NO_DEFAULTED_FUNCTIONS
    _LIBCPP_INLINE_VISIBILITY _LIBCPP_CONSTEXPR default_delete() _NOEXCEPT = default;
#else
    _LIBCPP_INLINE_VISIBILITY _LIBCPP_CONSTEXPR default_delete() _NOEXCEPT {}
#endif
    template <class _Up>
        _LIBCPP_INLINE_VISIBILITY default_delete(const default_delete<_Up>&,
             typename enable_if<is_convertible<_Up*, _Tp*>::value>::type* = 0) _NOEXCEPT {}
    _LIBCPP_INLINE_VISIBILITY void operator() (_Tp* __ptr) const _NOEXCEPT
        {
            static_assert(sizeof(_Tp) > 0, "default_delete can not delete incomplete type");
            static_assert(!is_void<_Tp>::value, "default_delete can not delete incomplete type");
            delete __ptr;
        }
};
```
内部重载了operator()，接收一个_Tp* __ptr参数，`__data_.first().second()(__data_.first().first());`这句调用时传进的参数正是我们的raw指针。
关于shared_ptr的分析告一段落。

#### weak_ptr 
&emsp;&emsp;经过上面的分析，shared_ptr似乎可以满足我们大部分关于指针的应用场景，然而智能毕竟不是万能,还有一些shared_ptr干不了的活,熟悉ios开发的同学应该清楚，强大的ARC(Automatic Reference Counting)技术依然解决不好如下图所示的循环引用的问题,所以引入了weak指针，同理C++11中引入weak_ptr来解决这一类问题.
![](http://77g3g7.com1.z0.glb.clouddn.com/weak_ptr.png)
我们可以把上面的`class X`,稍微修改验证下循环引用的问题：

```C++
class X{
public:
    X()= default;
    X(int value)
            :data(value),sp_(nullptr)
    {
        cout <<"constructor"<<endl;
    }
    X(int value,shared_ptr<X> &sp)
    :data(value),sp_(sp)
    {
        cout <<"constructor"<<endl;
    }
    ~X()
    {
        cout <<"destructor"<<endl;
    }
    int get_data() const
    {
        return  data;
    }

    friend ostream& operator << (ostream &out, const X& x)
    {
        out <<x.get_data();
        return out;
    }
    void set_sp(shared_ptr<X> &sp)
    {
        sp_ = sp;
    }

private:
    int data;
    shared_ptr<X>  sp_;
};
```
测试代码如下：

```C++
void g()
{
    shared_ptr<X> sp1(new X(1));
    shared_ptr<X> sp2(new X(2,sp1));
    shared_ptr<X> sp3(new X(3,sp2));
    sp1->set_sp(sp3);
}

int main()
{
    //f();
    g();
    return 0;
}
```
执行编译一下：

```shell
luobin@luobinMini:shared_point$ clang++ -std=c++11 main.cpp 
luobin@luobinMini:shared_point$ ./a.out 
constructor
constructor
constructor
```
事实证明我们new出来的3个对象在g()退出后并没有被释放，而我们又失去了对其的控制，所以造成了内存泄露。
我们将成员`sp_`改成weak_ptr,另外由于weak_ptr不能用nullptr构造,所以还要修改下第二个构造函数：

```C++
    X(int value)
            :data(value)
    {
        cout <<"constructor"<<endl;
    }
```
继续执行相同的测试代码：

```shell
luobin@luobinMini:shared_point$ clang++ -std=c++11 main.cpp 
luobin@luobinMini:shared_point$ ./a.out 
constructor
constructor
constructor
destructor
destructor
destructor
```
perfect,完美解决环形引用的问题。
有了前面的分析基础，我们看下weak_ptr的模板类的实现：

```C++
template<class _Tp>
class _LIBCPP_TYPE_VIS_ONLY weak_ptr
{
public:
    typedef _Tp element_type;
private:
    element_type*        __ptr_;
    __shared_weak_count* __cntrl_;

public:
    _LIBCPP_CONSTEXPR weak_ptr() _NOEXCEPT;
    template<class _Yp> weak_ptr(shared_ptr<_Yp> const& __r,
                   typename enable_if<is_convertible<_Yp*, _Tp*>::value, __nat*>::type = 0)
                        _NOEXCEPT;
    weak_ptr(weak_ptr const& __r) _NOEXCEPT;
    ...
```
结构大同小异，甚至`__cntrl_`都是相同的`__shared_weak_cout`类型。

#### unique_ptr

&emsp;&emsp;关于unique_ptr,从名字看，强调的是unique,表示的是全局只能有一个unique_ptr对象对raw指针拥有控制权。在一些适应的场景下比shared_ptr更高效一些，因为，`shared_ptr`要通过一个`__cntrl_`控制引用计数决定raw指针的施放时机，而`unique_ptr`只需要维护一个`__ptr_` :

```C++
template <class _Tp, class _Dp = default_delete<_Tp> >
class _LIBCPP_TYPE_VIS_ONLY unique_ptr
{
public:
    typedef _Tp element_type;
    typedef _Dp deleter_type;
    typedef typename __pointer_type<_Tp, deleter_type>::type pointer;
private:
    __compressed_pair<pointer, deleter_type> __ptr_;

#ifdef _LIBCPP_HAS_NO_RVALUE_REFERENCES
    unique_ptr(unique_ptr&);
    template <class _Up, class _Ep>
        unique_ptr(unique_ptr<_Up, _Ep>&);
    unique_ptr& operator=(unique_ptr&);
    template <class _Up, class _Ep>
        unique_ptr& operator=(unique_ptr<_Up, _Ep>&);
#endif  // _LIBCPP_HAS_NO_RVALUE_REFERENCES
```
`unique_ptr` 通过delete 掉copy constructor和 copy assignment operators来确保一个`unique_ptr`对象对raw指针的唯一控制权。但是通过`std::move`可以进行控制权之间的转移，所以使用`unique_ptr`谨记一点即可：控制权只能转移，不能复制。 

#### 总结

&emsp;&emsp;使用智能指针可以很大程度上将精力从繁琐的控制指针生命周期的工作中释放出来，这是C++作为一个没有GC的语言在开发效率上的一大进步。但是使用过程中要牢记，要么全部使用raw指针，要么交由智能指针全权负责，换句话说就是：要么裸奔，要么全副武装，谨慎混搭。
