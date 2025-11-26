---
title: Dart-类的继承(Extends)与混入(Mixin)
date: 2025-11-26 19:29:04 +0800
categories:
  - dart
tags:
  - dart
  - flutter
description: 本文深入探讨Dart语言的高级特性，包括继承、函数重写、操作符重写、抽象类、接口、泛型继承、混入（mixin）及其实现细节。
math: true
---
> [Dart语言详解](https://rd-wang.github.io/posts/Dart-基础概念和内部原理/)

### Dart中的继承
与Java语言类似，Dart语言标榜自己为“单继承”，也就是一个类只能有一个直接的父类。如果一个类没有显式地声明父类，那么它会默认继承Object类。此外Dart语言又提供了混入（Mixin）的语法，允许子类在继承父类时混入其他类。

Dart语言中使用extends作为继承关键字，子类会继承父类的数据和函数。

```dart
class Animal{
  String name;
  void eat(){
    print("${name}:进食");
  }
}

class Cat extends Animal{
  String color;
  void climb(){
    print("${color}的${name}:爬树");
  }
}
```
#### 函数重写

重写在面向对象中体现的现实意义是“子类与父类在同一行为上有不同的表现形式”。同Java语言类似，Dart语言也支持函数的重写，子类重写父类的函数后，对象调用的即为子类的同名函数。
```dart
class Animal{
  String name;
  void eat(){
    print("${name}:进食");
  }
}

class Cat extends Animal{
  String color;
  @override
  void eat(){//子类重写父类的eat方法
    print("${color}的${name}:吃鱼");
  }
  void climb(){
    print("${color}的${name}:爬树");
  }
}
```

#### 操作符重写
同C++语言类似，Dart语言支持操作符的重写，常规的四则运算和比较运算符都可以进行重写，其中只有`!=`不可重写，`a!=b`相当于`!(a==b)`的语法糖。
示例代码：
重写了`==`和`+`的Rectangle类，当两个对象的width和height一致认为它们是“相等”的，两个Rectangle相加则将两者的width和height进行相加后得到新的Rectangle对象
```dart
class Rectangle{
  int width;
  int height;

  Rectangle(this.width,this.height){}

  @override
  bool operator ==(dynamic other) {
    if(other is! Rectangle){
      return false;
    }
    Rectangle temp = other;
    return (temp.width == width && temp.height == height);
  }

  @override
  Rectangle operator +(dynamic other){
    if(other is! Rectangle){
      return this;
    }
    Rectangle temp = other;
    return new Rectangle( this.width + temp.width, this.height + temp.height);
  }
}
```

#### 抽象类
抽象abstract是面向对象中的一个非常重要的概念，通常用于描述父类拥有一种行为但无法给出细节实现，而需要通过子类来实现抽象的细节。这种情况下父类被定义为抽象类，子类继承父类后实现其中的抽象方法。
同Java语言类似，Dart中的抽象类也使用abstract来实现，不过抽象函数无需使用abstract，直接给出定义不给出方法体实现即可
抽象类中可以有数据，可以有常规函数，可以有抽象函数，但抽象类不能实例化。子类继承抽象类后必须实现其中的抽象函数。
```dart
abstract class Animal{
  String name;  //数据
  void display(){  //普通函数
    print("名字是:${name}");
  }
  void eat(); //抽象函数
}

class Dog extends Animal{
  @override
  void eat() { //实现抽象函数
    print("eat");
  }
}
```
####  接口
Dart语言中没有接口（interface）的关键字，但是有实现（implements）关键字，Dart中可以将类（是否为抽象无关）当做隐式接口直接使用，当需要使用接口时，可以声明类来代替。
```dart
abstract class Animal{
  String name;
  void display(){
    print("名字是:${name}");
  }
  void eat(); //抽象方法
}

abstract class swimable{ //抽象类作为接口
  void swim();
}

class walkable{ //普通类作为接口
  void walk(){}
}

class Dog extends Animal implements swimable, walkable{
  @override
  void eat() {
    print("eat");
  }
  @override
  void swim() {
    print("swim");
  }
  @override
  void walk() {
    print("walk");
  }
}
```

#### 泛型继承
在使用泛型编程时也可以使用extends关键字约束输入泛型的类型。
```dart
//被泛型约束的函数，当传入泛型int时，只能计算整数加法，
//且在声明对象x时泛型只能传入num类型及其子类
class DataUtil<T extends num>{
  T addition(T a, T b){
    return a+b;
  }
}
```

### 混入(mixin)
 - Mixins不是一种在经典意义上获得多重继承的方法。
 - Mixins是一种抽象和重用一系列操作和状态的方法。
 - 它类似于扩展类所获得的重用，但它与单继承兼容，因为它是线性的。
#### mixin解决什么问题
我们来看下面这张关于动物（Animal），哺乳动物（Mammal），鸟（Bird）和鱼（Fish）的继承关系图
![mixin-1](https://rd-wang.github.io/assets/img/dart/mixin-1.png)
这里有一个名为Animal的超类，它有三个子类（Mammal，Bird和Fish）。在底部，我们有具体的一些子类。
小方块代表行为。例如，

 - 黄色方块表示具有此行为的类的实例可以步行（walk）。
 - 蓝色方块表示具有此行为的类的实例可以游泳（swim）。
 - 灰色方块表示具有此行为的类的实例可以飞行（fly）。
 
有些动物有共同的行为：猫(Cat)和鸽子(Dove)都可以行走，但是猫不能飞。
这些行为与此分类正交，因此我们无法在超类中实现这些行为。

在Java语言中我们可以借助接口（Interface）来实现相关的设计，Dart中也可以利用隐式接口来完成相应的设计。
如果不同的子类在某种行为上表现的都不相同，那么使用接口来实现设计是一种良好的设计。
但如果不同的子类在实现某种行为上有着同样的表现，那么使用接口来实现设计可能会造成代码的冗余。**（接口实现强制重写函数）**
因此它并不是一个好的解决方案。

我们需要一种**在多个类层次结构中重用类的代码的方法**。
Mixin就能够办到这一点！

#### 怎么使用mixin
对三种行为分别定义三个类描述它们，分别是Walker，Swimmer和Flyer
```dart
class Walker {
  void walk() {
    print("I'm walking");
  }
}
class Swimmer {
  void swim() {
    print("I'm swimming");
  }
}
class Flyer{
  void fly() {
    print("I'm flying");
  }
}
```
如果不想这三个类被实例化，可以使用抽象类+工厂方式定义
```dart
abstract class Walker {
  // This class is intended to be used as a mixin, and should not be
  // extended directly.
  factory Walker._() => null;

  void walk() {
    print("I'm walking");
  }
}

```
使用混入的关键字是with，它的后面可以跟随一个或多个类名
```dart
class Cat extends Mammal with Walker {}
class Dove extends Bird with Walker, Flyer {}
```
在Cat类上定义了Walker mixin，它允许我们调用walk方法而不是fly方法（在Flyer中定义）。
```dart
main(List<String> arguments) {
  Cat cat = Cat();
  Dove dove = Dove();

  // A cat can walk.
  cat.walk();

  // A dove can walk and fly.
  dove.walk();
  dove.fly();

  // A normal cat cannot fly.
  // cat.fly(); // Uncommenting this does not compile.
}
```
#### 如果混入的类之间有相同的方法
如果Mixin的类和继承类，或者混入的类之间有相同的方法，在调用时会产生什么样的情况，看下面的例子。

AB和BA类都使用A和B Mixin继承至P类，但顺序不同。A，B和P类都有一个名为getMessage的方法。
```dart
class A {
  String getMessage() => 'A';
}

class B {
  String getMessage() => 'B';
}

class P {
  String getMessage() => 'P';
}

class AB extends P with A, B {}

class BA extends P with B, A {}

void main() {
  String result = '';
  AB ab = AB();
  result += ab.getMessage();
  BA ba = BA();
  result += ba.getMessage();
  print(result);
}
```
运行结果： BA
为什么会产生这个结果？
#### 声明mixins的顺序代表了从最高级到最高级的继承链

Dart中的Mixins通过创建一个新类来实现，该类将mixin的实现层叠在一个超类之上以创建一个新类 ，它不是“在超类中”，而是在超类的“顶部”，因此如何解决查找问题不会产生歧义。

实际上，这段代码
```dart
class AB extends P with A, B {}
class BA extends P with B, A {}
```
在语义上等同于
```dart
class PA = P with A;
class PAB = PA with B;

class AB extends PAB {}

class PB = P with B;
class PBA = PB with A;

class BA extends PBA {}
```
最终的继承关系如下图所示。

![mixin-2](https://rd-wang.github.io/assets/img/dart/mixin-2.png)
很显然，最后被继承的类重写了上面所有的getMessage方法，可以理解为处于Mixin结尾的类将前面的getMessage方法都覆盖（override）掉了


#### mixin应用程序实例的类型是什么？
> 通常，它是其超类的子类型，也是mixin名称本身表示的类的子类型，即原始类的类型。

 所以这意味着这个程序的运行结果全部为true
```dart
class A {
  String getMessage() => 'A';
}

class B {
  String getMessage() => 'B';
}

class P {
  String getMessage() => 'P';
}

class AB extends P with A, B {}

class BA extends P with B, A {}

void main() {
  AB ab = AB();
  print(ab is P);  //true
  print(ab is A);  //true
  print(ab is B);  //true

  BA ba = BA();
  print(ba is P);  //true
  print(ba is A);  //true
  print(ba is B);  //true
}
```

完整的实例
```dart
abstract class Animal {}
abstract class Mammal extends Animal {}
abstract class Bird extends Animal {}
abstract class Fish extends Animal {}

abstract class Walker {
  factory Walker._() => null;
  void walk() {
    print("I'm walking");
  }
}

abstract class Swimmer {
  factory Swimmer._() => null;
  void swim() {
    print("I'm swimming");
  }
}

abstract class Flyer {
  factory Flyer._() => null;
  void fly() {
    print("I'm flying");
  }
}

class Dolphin extends Mammal with Swimmer {}
class Bat extends Mammal with Walker, Flyer {}
class Cat extends Mammal with Walker {}
class Dove extends Bird with Walker, Flyer {}
class Duck extends Bird with Walker, Swimmer, Flyer {}
class Shark extends Fish with Swimmer {}
class FlyingFish extends Fish with Swimmer, Flyer {}
```
Mixin类图
![mixin-3](https://rd-wang.github.io/assets/img/dart/mixin-3.png)