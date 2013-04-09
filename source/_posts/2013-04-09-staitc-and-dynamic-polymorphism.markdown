---
layout: post
title: "使用静态多态辅助动态多态"
date: 2013-04-09 23:13
comments: true
categories: [C++]
---

##静态多态 (编译期多态) 与动态多态 (运行期多态)

关键字: 重载/模版和虚函数

类型: 编译期多态 (静态多态, 早绑定) 和运行期多态 (晚绑定)
编译期多态 (重载/模版), 运行期多态 (虚函数)

应用形式上:
静多态是发散式的, 让相同的实现代码应用于不同的场合.
动多态是收敛式的, 让不同的实现代码应用于相同的场合.

思维方式上:
静多态是泛型式编程风格, 它看重的是算法的普适性.
动多态是对象式编程风格, 它看重的是接口和实现的分离度.


##std::shared_ptr 中的 deleter 是如何工作的?

标准库中的引用计数智能指针 shared_ptr 很有趣——你可以向其构造器传递一个函数或者仿函数 (function object, 或 functor), 当引用计数归零的时候, 它将在被引用对象上调用删除器 (deleter). 乍一看, 似乎没啥了不起啊, 但请看代码:

```cpp
template<typename T>
class shared_ptr {
public:
  template<typename U, typename D>
  explicit shared_ptr(U* ptr, D deleter);
  //...
 
};
```
<!-- more -->
注意 `shared_ptr<T>` 必然在析构时调用类型为 D 的删除器, 然而它根本不知道 D 为何物. 这个对象不能包含类型为 D 的数据成员, 也不能指向类型为 D 的对象, 因为声明其数据成员时, D 对它而言还是未知的. 那么, shared_ptr 对象如何跟踪删除器 (它在构造阶段传入: 当 T 对象将被销毁时, 还得使用它) 呢? 更通俗地说, 构造器如何将未知类型的信息传递给它正在构造的对象, 而这个对象本身对信息类型完全无知? 答案很简单: 让此对象包含一个指向已知类型基类的指针 (标准库中叫它 sp_counted_base), 然后让构造器以 D 为参数实例化一个派生于上述基类的模板 (标准库中叫 sp_counted_impl_p 和 sp_counted_impl_pd), 最后用声明于基类, 实现于派生类的虚函数 (标准库中使用 dispose) 去调用删除器. 关于这个问题, Scott Meyers 在 06 年 9 月份的一篇文章中已经有阐述, 详见: [My Most Important C++ Aha! Moments][00].

   [00]: http://www.artima.com/cppsource/top_cpp_aha_moments.html


##结合静态多态和动态多态实现类型无关的容器

使用 std::shared_ptr 中 deleter 的实现方式, 可以实现类型无关的容器. 代码主干如下:

```cpp
class any
{
public:
  any() : content(nullptr) {}
  template<typename ValueType>
  any(const ValueType& value) : content(new holder<ValueType>(value)) {}
  any(const any& other) : content(other.content ? other.content->clone() : 0) {}
  ~any() { delete content; }
  any& swap(any& rhs)
  {
    std::swap(content, rhs.content);
    return *this;
  }
  template<typename ValueType>
  any& operator=(const ValueType& rhs)
  {
    any(rhs).swap(*this);
    return *this;
  }
  any& operator=(any rhs)
  {
    rhs.swap(*this);
    return *this;
  }
  bool empty() const { return !content; }
 
  const std::type_info& type() const { return content ? content->type() : typeid(void); }
 
private:
  class placeholder
  {
  public:
    virtual ~placeholder() {}
    virtual const std::type_info& type() const = 0;
    virtual placeholder* clone() const = 0;
  };
 
  template<typename ValueType>
  class holder : public placeholder
  {
  public:
    holder(const ValueType& value) : held(value) {}
    virtual const std::type_info& type() const { return typeid(ValueType); }
    virtual placeholder* clone() const { return new holder(held); }
    ValueType held;
 
  private:
    holder& operator=(const holder&);
 
  };
 
private: 
  placeholder* content;
};
```

在上面的代码中, any 类持有一个 placeholder 的基类指针, 在构造 any 对象时, 通过 any 的模板构造函数, 根据具体类型创建具体的 placeholder 子类类型, any 类提供 type() 接口, 用于查询 any 类中存储的实际类型.boost::any 中有类似的实现.
 
使用方式如下：

```cpp
int main()
{
  std::array<bitdewy::any, 3> any_array;
  any_array[0] = 1;
  any_array[1] = std::string("hello");
  any_array[2] = .1;
 
  std::for_each(std::begin(any_array),
                std::end(any_array),
                [](const bitdewy::any& a) {
    if (a.type() == typeid(int)) {
      std::cout << bitdewy::any_cast<int>(a) << std::endl;
 
    } else if (a.type() == typeid(std::string)) {
      std::cout << bitdewy::any_cast<std::string>(a) << std::endl;
 
    } else if (a.type() == typeid(double)) {
      std::cout << bitdewy::any_cast<double>(a) << std::endl;
 
    } else {
      //LOG ...
    }
  });
  return 0;
}
```

###参考

- [boost::any](http://svn.boost.org/svn/boost/trunk/boost/any.hpp)
- [My Most Important C++ Aha! Moments][00]
