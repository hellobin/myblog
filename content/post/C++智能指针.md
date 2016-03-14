+++
author = ""
comments = true
date = "2016-03-07T17:50:24+08:00"
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
SFINAE名字听上去很高大上，其实知道is_convertible和enable_if表示什么意思后也不是太难理解.从字面上理解is_convertible应该就是判断两个类型是否可以相互转换,起末班

```C++
template< class From, class To >
struct is_convertible;
```