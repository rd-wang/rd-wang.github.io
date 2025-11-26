---
title: Dart Typedefs
date: 2025-11-26 17:31:16 +0800
categories:
  - dart
tags:
  - dart
  - flutter
description: 函数也是对象，就想字符和数字对象一样。 使用 `typedef `，或者 function-type alias 为函数起一个别名
math: true
---
> [Dart语言详解](https://rd-wang.github.io/posts/Dart基础概念和内部原理/)


### Typedefs
在 Dart 中，函数也是对象，就想字符和数字对象一样。 使用 `typedef `，或者 function-type alias 为函数起一个别名， 别名可以用来声明字段及返回值类型。 当函数类型分配给变量时，typedef会保留类型信息。

请考虑以下代码，代码中未使用 typedef ：
```dart
class SortedCollection {
  Function compare;

  SortedCollection(int f(Object a, Object b)) {
    compare = f;
  }
}

// Initial, broken implementation. // broken ？
int sort(Object a, Object b) => 0;

void main() {
  SortedCollection coll = SortedCollection(sort);

  // 虽然知道 compare 是函数，
  // 但是函数是什么类型 ？
  assert(coll.compare is Function);
}
```
当把 f 赋值给 compare 的时候，类型信息丢失了。 f 的类型是 (Object, Object) → int (这里 → 代表返回值类型)， 但是 compare 得到的类型是 Function 。如果我们使用显式的名字并保留类型信息， 这样开发者和工具都可以使用这些信息：
```dart
typedef Compare = int Function(Object a, Object b);

class SortedCollection {
  Compare compare;

  SortedCollection(this.compare);
}

// Initial, broken implementation.
int sort(Object a, Object b) => 0;

void main() {
  SortedCollection coll = SortedCollection(sort);
  assert(coll.compare is Function);
  assert(coll.compare is Compare);
}
```
>提示： 目前，typedefs 只能使用在函数类型上， 我们希望将来这种情况有所改变。

由于 typedefs 只是别名， 他们还提供了一种方式来判断任意函数的类型。例如：
```dart
typedef Compare<T> = int Function(T a, T b);

int sort(int a, int b) => a - b;

void main() {
  assert(sort is Compare<int>); // True!
}
```
