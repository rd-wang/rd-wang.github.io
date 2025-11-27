---
title: Dart-取消Future
date: 2025-11-27 11:26:38 +0800
categories:
- Dart
tags:
- Dart
- Flutter
description: "Future无法取消，那么应该如何避免无用的Future回调"
math: true
---
> [Dart语言详解](https://rd-wang.github.io/posts/Dart-基础概念和内部原理/)



# 问题
假设调用了REST服务，当该服务返回数据时，Future将完成，显示也会随之更新。这种情况可能类似于以下情况：
```dart
main() async {
  List data = await getData();
  updateDisplay(data);
}

Future<List> getData() async {
  // make the REST call
  Response restResponse = await HttpRequest.get(.....);
  
  // massage the data
  List data = restResponse.response.map((Map record) {
    // transform/check raw records
    // return good data
  });
  
  return data;
}

void updateDisplay(List data) {
  // update the display
}
```
如果该REST调用花费很长时间,如果您的用户变得不耐烦并且导航离开了首先需要数据的视图怎么办？

# 解决方案
很多人都知道，无法取消Dart中的Future，但是可以取消Stream的订阅。因此，处理这种情况的一种方法是重写getData()以返回Stream。

幸运的是，可以轻松地从调用方将Future转换为Stream：
```dart
void main() {
  // keep a reference to your stream subscription
  StreamSubscription<List> dataSub;
  
  // convert the Future returned by getData() into a Stream
  dataSub = getData().asStream().listen((List data) {
    updateDisplay(data);
  });

  // user navigated away!
  dataSub.cancel();
}
```
使用Future的asStream()方法，基本上可以将Future当作一个Stream来listen。这样，如果发现不再需要尚未到达的数据，则可以保存您的订阅并使用它取消回调。所有这些都无需修改getData()功能。

## 真正发生了什么
实际上不可能阻止Future的完成。
所能做的就是在特定情况下（例如，响应在返回之前变得无关紧要）停止执行回调。
在代码示例中，dataSub.cancel()在Future完成之前调用即可实现此目的。


## 使用此方法的更实际的应用场景：

```dart
// keep a reference to your stream subscription
StreamSubscription<List> dataSub;

void onGetDataClicked() {
  // make sure any still-pending prior request never gets processed
  dataSub?.cancel();
  
  // convert the Future returned by getData() into a Stream
  dataSub = getData().asStream().listen((List data) {
    updateDisplay(data);
  });
}
```
通过此流程，可以确保只有最新的数据请求最终显示。
