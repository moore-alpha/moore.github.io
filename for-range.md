`c++`中模板类型的推导和`auto`关键字类型的推导使用相同的机制。

`c++`中`auto`会忽略顶层`const`并推导出参数类型，对于`vector<bool>`则会产生`__bit_reference proxy class`而不是实际类型。关于auto在for循环中的应用有几种场景。

```cpp
for (auto x : vec)    // auto是vec中元素类型，产生复制（vecter<bool>例外）

std::vector<bool> vec {false, false};
for(auto x : vec) {
  x = true; // all elements be true.
}
```

`auto&`：可以引用容器中的元素（`vector<bool>`例外，`proxy`是`rvalue`，编译错误），因此也可以修改元素值。一般使用`auto&&`代替。

`auto&& / T&&`：被称为`Universal Reference`，对`proxy class reference`也可以正确处理

- 当被左值初始化时则推导为左值引用`lvalue reference`

- 当被右值初始化时则推导为右值引用`rvalue reference`

`const auto&`：只想读取元素时使用，对所有类型都适用（包含`proxy class reference`），不产生元素拷贝。

## 迷惑的用法

`const auto`：拷贝元素但又不可修改，没有适用场景。

`const auto&&`：只绑定右值且无法修改元素，完全可以使用`const auto&`代替。

## 总结

拷贝`range`中元素：`for (auto item : data)`

需要修改`range`中元素：`for (auto &&item : data)`。这种情况更容易接受的是`auto &`写法，但是他对于`proxy class reference`支持不够友好，因此最好使用`auto&&`。

只读：`for (const auto& item : data)`

## 参考

auto的各类变种：https://zhuanlan.zhihu.com/p/25148592

universal reference：https://isocpp.org/blog/2012/11/universal-references-in-c11-scott-meyers
