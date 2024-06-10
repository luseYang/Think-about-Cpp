# new、delete 是线程安全的吗？
如果标准达到 C++11，要求下列函数是线程安全的：  
- new 运算符和 delete 运算符的库版本
- 全局 new 运算符和 delete 运算符的用户替换版本
- [std::calloc](https://zh.cppreference.com/w/cpp/memory/c/calloc)、[std::malloc](https://zh.cppreference.com/w/cpp/memory/c/malloc)、[std::realloc](https://zh.cppreference.com/w/cpp/memory/c/realloc)、[std::aligned_alloc](https://zh.cppreference.com/w/cpp/memory/c/aligned_alloc) (C++17 起)、[std::free](https://zh.cppreference.com/w/cpp/memory/c/free)

举个例子：
```cpp
void f(){
    T* p = new T{};
    delete p;
}
```
内存分配、释放操作是线程安全，构造和析构不涉及共享资源。而局部对象 p 对于每个线程来说是独立的。换句话说，每个线程都有其自己的 p 对象实例，因此它们不会共享同一个对象，自然没有数据竞争。

如果 p 是全局对象（或者外部的，只要可被多个线程读写），多个线程同时对其进行访问和修改时，就可能会导致数据竞争和未定义行为。因此，确保全局对象的线程安全访问通常需要额外的同步措施，比如互斥量或原子操作。
```cpp
T* p = nullptr;
void f(){
    p = new T{}; // 存在数据竞争
    delete p;
}
```
即使 p 是局部对象，如果构造函数（析构同理）涉及读写共享资源，那么一样存在数据竞争，需要进行额外的同步措施进行保护。
```cpp
int n = 1;

struct X{
    X(int v){
        ::n += v;
    }
};

void f(){
    X* p = new X{ 1 }; // 存在数据竞争
    delete p;
}
```
一个直观的展示是，我们可以在构造函数中使用 std::cout，看到无序的输出，[例子](https://godbolt.org/z/eYbePxc6G)。

---
值得注意的是，如果是自己重载 operator new、operator delete 替换了库的全局版本，那么它的线程安全就要我们来保证。
```cpp
// 全局的 new 运算符，替换了库的版本
void* operator new  (std::size_t count){
    return ::operator new(count); 
}
```
以上代码是线程安全的，因为 C++11 保证了 new 运算符的库版本，即 ::operator new 依然是线程安全的，我们直接调用它自然不成问题。如果你需要更多的操作，就得使用互斥量之类的方式保护了。
