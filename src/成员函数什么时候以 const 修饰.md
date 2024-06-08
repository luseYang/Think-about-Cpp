# 举例
```cpp
class Date {
public:
    Month month() const;  // 好
    int month();          // 不好
    // ...
};
```
第一个 month() 比第二个有更多的信息，以 const 修饰，代表不会修改当前的日期，返回类型 Month 也非常明确。  

const 成员函数修饰是为什么，能做什么。大多数情况下默认不修改当前类的数据成员的话，就要加 const，明确语义，可以增加可读性。 但是应该不止如此。
```cpp
#include <iostream>

class Date {
    using Month = int;
public:
    Month m;

    int month() { return m; }          
};

void func(const Date& date){
    std::cout << date.month() << '\n';
}

int main(){
    Date d{10};
    func(d);
}
```
上面的代码会得到一个编译错误，很明显 month() 函数没有以 const 修饰，C++ 不允许 const 的对象调用没有以 const 修饰的成员函数。

我们只需要改成：
```cpp
Month month()const { return m; } // const 对象和非 const 对象都能调用
```
如果你阅读过 STL 源码，或者看过[文档](https://zh.cppreference.com/w/cpp/container/array/operator_at)，会知道，大部分成员函数都要提供 const 和非 const 两种版本，我们以 std::array 的 operator[] 为例。
```cpp
constexpr reference operator[]( size_type pos );
constexpr const_reference operator[]( size_type pos ) const;
```
这两个成员函数都不会修改自己的成员，但是为什么要写 const 版本呢？注意返回类型。
- 如果没有以 const 修饰的 std::array 对象，那么它调用 operator[] 自然是可以修改的，行为就像普通数组那样，我们就返回 reference。

- 如果是以 const 修饰的 std::array 对象，那么它调用 operator[] 根据我们的语义，自然不该让它外部能够修改，所以我们返回 const_reference。

**_一个成员函数是否以 const 修饰，不在于这个成员函数到底是否会修改自己的成员，而在于 “不变性”。_**  
const 修饰本身就代表存在不变的暗示。
