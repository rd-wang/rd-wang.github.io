---
title: Dart dynamic，var，object的区别
date: 2025-11-26 19:05:33 +0800
categories:
  - dart
tags:
  - dart
  - flutter
description: var定义的类型是不可变的，dynamic和object类型是可以变的，而dynamic 与object 的最大的区别是在静态类型检查上 
math: true
---
> [Dart语言详解](https://rd-wang.github.io/posts/Dart-基础概念和内部原理/)

var定义的类型是不可变的，dynamic和object类型是可以变的，而dynamic 与object 的最大的区别是在静态类型检查上
```dart
void main() {
  //var
  var str = "hello world";
  print(str.runtimeType);//String
  print(str);//hello world
  //str=1会报错
  str=1;

  //dynamic
  dynamic mic = "hello world";//编译时不会揣测数据类型，但是运行时会推断
  print(mic.runtimeType);//String
  print(mic);//hello world
  //但是这样的坏处就是会让dart的语法检查失效，所以有可能会造成混乱而不报错
  //所以不要直接使用dynamic
  mic.foo();
  //通过它定义的变量会关闭类型检查,这段代码静态类型检查不会报错，但是运行时会crash，因为mic并没有foo（）方法，
  // 所以建议大家在编程时不要直接使用dynamic
  mic=1;
  print(mic.runtimeType);//int 说明类型是可变的
  print(mic);//1

 //Object
  Object object = "hello world";
  print(object.runtimeType);//String
  print(object);//hello world
  object=1;
  print(object.runtimeType);//int 说明类型是可变的
  print(object);//1
  //object.foo();静态类型检查会运行报错会报错
  object.foo();

}
```