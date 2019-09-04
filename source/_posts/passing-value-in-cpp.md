---
title: c++中的传参
date: 2019-08-21 10:30:36
tags: cpp
---

#### 前言

最近在用c++写基于libuv的websocket engine的时候发现, 设置callback的参数是一个很有挑战性的工作, 原来觉得c++的复杂在于其模板，oo范式概念的复杂, 现在发现c++的每个方面都很复杂，因为有太多可以通过编译的方式了，我想从传参这个方面切入，让大家了解下c++的复杂（强大）。

ps：本文的传参使基于涉及到动态内存分配对象的传参，一般普通对象的传参基本是不需要考虑这么复杂的(至少我目前这么认为)。

以下是本文中需要传递的参数，一个简单的String, 只保留会讲到的构造函数

```c++
class String: {
  public:
    String(const char* src): data_(new char[strlen(src) + 1]), size_(strlen(src)) {
      ::strcpy(data_, src);
      data_[size_] = 0;
    }
    
    String(const String& lhs): data_(new char[lhs.size() + 1]), size_(lhs.size()) {
      ::strcpy(data_, lhs.data());
      data_[size_] = 0;
    }

    // move
    String(String &&rhs) noexcept: data_(rhs.data_), size_(rhs.size_) {
      rhs.data_ = nullptr;
    }    
};
```

我遇到的一个问题是在MessageCallback中，应该使用`const String& message`还是`String&& message`, 这两种形参的区别是什么

##### 理解`std::move` 和`右值引用`
在弄清上述问题之前，还是要从根本上着手，弄清`std::move`和右值引用。
右值引用是c++11中引入的一种新的引用类型，必须要绑定到右值的引用。
而std::move的作用是可以把几乎任意值转化成一个右值引用。

```c++
void test_passing_value(std::string& s1) {
  std::cout << "in the left reference" << std::endl;
}

void test_passing_value(std::string&& s1) {
  std::cout << "in the right reference" << std::endl;
}

std::string s1 = "hello";
std::string &sr = s1;

test_passing_value(s1); // in the left
test_passing_value(std::move(s1)); // in the right
test_passing_value(sr); // in the left
test_passing_value(std::move(sr)); // in the right

// 可以看到无论是左值，还是左值引用，使用std::move之后都可以变成右值引用
// 意外的情况就是const T& 在使用std::move转化时的特殊情况
```

##### `const T&` vs `T&&`

c++11中引入了右值引用和move语义，初学者（比如我）很容易被这种特性吸引（move比copy快）, 两者其实是解决不同场景下的问题，T&& 的确提供了一种更为高效的传参方式, 让我们看下两者的细节和使用场景吧。

- 仅仅使用`const T&`并不会发生copy
```c++
void foo(const String& s) {
  std::cout << s << std::endl;
}

int main() {
  String s0("hello");
  foo(s0); // 不会发生复制
}
```
- `const T&` 发生复制的情况是在函数体内用到T的local variable， 比如`T local = t`, 这时候会发生拷贝控制

- 仅仅使用`T&&`不会发生move
```c++
void f2(String &&s2) {
  std::cout << s2 << std::endl;
}

int main() {
  f2(String("hello"));
}
```
从以上两种情况来看似乎传参的代价都很低，那么应该如何选择呢，主要还是根据语义来做选择，如果你的实参是个左值自然选择第一种，如果是右值那自然是后者，如果你确定需要第二种那么使用`std::move`也是可以的

##### 结论
- 由于我在传给callback的string是从buffer中复制构造来的，而不是仅仅像stringpiece那样使用，所以使用右值引用更合适，使用者会放心大胆的使用这个string，move之类的更不在话下了

##### 其他
- 可以考虑使用StringPiece类似的技术，不过我感觉StringPiece在这个场景下并不好
- 关于std::move的原理其中涉及到了引用折叠这些比较复杂的概念，所以没有深入介绍
- 在模板中使用T&& 和实参中的&&还是不一样的，模板中的T&& 在转发参数时要保证不丢失T的信息(T可能是引用) 所以有涉及到了完美转发的概念，std::forward可以解决这个问题
