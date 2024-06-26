# 一、继承和多态的概念
多态是在继承的基础上实现的，我们认为，继承是类设计层次的代码复用的一种手段，而多态则是在此基础上实现的多种形态，完成某一件事，可以由于对象的不同产生不同的完成结果，我们称这种现象为多态。

# 二、多态调用和普通调用
## 1.虚函数的重写（覆盖）
- 首先，虚函数的定义是比较简单的，只需要在函数的接口部分加上 virtual 关键字就可以，当虚函数所在类被继承时，派生类会隐含一个基类的虚函数，此时如果派生类重新定义这个虚函数，并且和基类的虚函数的参数列表，返回值，函数名都一样，我们称派生类完成了虚函数的重写。

- 重写这件事只针对于虚函数，普通函数并没有重写，所以重写的要求有两点：必须为虚函数，虚函数必须满足三同（函数名，参数列表，返回值）

## 2.虚函数重写特殊情况
- 子类中继承基类的虚函数可以不加virtual关键字

- 协变也是虚函数重写的特殊情况，三同中返回值可以不同，但是要求返回值必须是一个父子类关系的指针或引用，自己的父类或其他的父类都可以。实际并不常见，只要了解一下这个语法就够了。

-  虚函数的返回值除本身的父子类继承关系中的类型外，还可以是其他继承关系中的父子类指针或引用，这也是协变的另一种场景。

- 析构函数的重写卸酸虚函数重写的特殊情况

## 3.多态调用 VS 普通调用
- 满足多态调用，首先调用的函数必须是重写的虚函数（如果父类虚函数只是简单继承到子类，子类并没有显示写出来虚函数，则这样也不能算是重写的虚函数，不符合多态），更为重要的是必须是父类的指针或者引用调用重写的虚函数，只有同时满足这两点才算多态调用，其他情况下均为普通调用。

- 普通调用只和调用对象的类型有关，对象的类型是什么，就调对应类里面的函数，这个函数可能是普通函数，也可能是虚函数，因为虚函数如果不是父类指针或引用进行调用的话，那他和普通函数就没什么区别。

- 多态调用和指针或引用指向的对象类型有关，指向的是父类那就调用父类的虚函数，指向的是子类那就调用子类重写的虚函数，这里调用的函数只能是虚函数。

# 三、抽象类和接口继承
## 1.抽象类（接口类）的作用
-  如果一个虚函数在接口后面加上=0，则这个虚函数为纯虚函数，纯虚函数所在的类为抽象类，抽象类是不可以被实例化出对象的，如果抽象类被继承，派生类里面天然的就会有纯虚函数，那么派生类也就变成了抽象类，一个类如果连对象都实例化不出来，那还有什么用呢？所以派生类必须重写纯虚函数，要不然派生类就没有用。

- **所以抽象类的作用就是强制其派生类重写纯虚函数**，比如Car他不是车的品牌，而Bmw和BenZ这些才是车的真正品牌，那么Car其实就是一个抽象类，他的作用就是强制Bmw和BenZ这样的类去重写纯虚函数。**另外抽象类也可以体现出来接口继承，重写的是虚函数的实现，继承的是虚函数的接口。**

## 2.接口继承和实现继承
- 虚函数的继承是接口继承，目的就是让子类重写虚函数，重写的是虚函数的实现，因为在继承时继承的是虚函数的接口。普通函数的继承是实现继承，将普通函数直接照搬到派生类里面，没有重写这样的情况发生。

# 多态原理
## 1.虚函数表和虚函数的覆盖
- 当一个类里面出现虚函数时，这个类实例化出的对象模型就会发生改变，他的类成员会多一个虚表指针，这个虚表指针指向一个数组，这个数组里面存放的是类里面虚函数的地址。我们将这个数组称为虚函数表，简称虚表，指向虚函数表的指针简称为虚表指针。

- 另外虚函数的重写其实是语法的叫法，覆盖才是底层的叫法。

## 2.动态绑定和静态绑定
- 静态绑定又称编译时决议或前期绑定，早绑定，即在程序编译期间，即可确定程序行为，例如函数重载的调用，这就是静态多态，通过所传参数类型的不同，在编译期间就可确定调用的函数地址。

- 动态绑定又称运行时决议或后期绑定，，晚绑定，即在程序编译期间无法确定程序行为，例如多态调用，这就是动态多态，只能在程序运行期间，去指针或引用指向的虚表里面去找对应的虚函数地址。

## 3.虚表的位置，虚表是共享的 
- 有很多的文章都说g++平台下的虚表指针存在.rodata段(存疑)

- 另外虚表是共享的，一个类无论实例化出多少对象，对象里面存的虚表指针都是一样的。需要说清楚的一个概念是，虚表存的是虚函数指针，不是虚函数，虚函数和普通函数一样的，都是存在代码段的，只是他的指针又存到了虚表中。另外对象中存的不是虚表，存的是虚表指针
