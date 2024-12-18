```cpp
#include <iostream>
#include <variant>

struct X{
    void f(int, double)const {}
};

template<typename T>
void f(T& t, int a, double b){
    if constexpr (std::is_same_v<T, X>){
        t.f(a, b);
    }
}
std::variant<X> var;

int main(){
    X x;
    f(x, 1, 2);
}
```

