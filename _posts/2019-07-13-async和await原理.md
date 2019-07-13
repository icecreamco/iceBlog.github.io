---
layout: post
title: async和await原理
category: 
 - Flutter
 - dart
tags:
 - 异步
---



# async和await原理

## 原理

`Future`是Dart语法异步的核心，通过`async`标记一个方法为异步，返回一个future对象，进而对flutter进行监听，获取异步方法的返回值。一般用法为：

```dart
Future<int> getCount async {
  ~~~~
  return count;
}

Future<int> future = getCount();

getCount().then((value) {
  int res = value;
});
```

单个异步任务很好理解，代码写起来也很易读，但是多个异步任务协同工作时，这种写法会特别冗余且不易理解，例如：

~~~dart
Future<int> _loadFromDisk(){}
Future<String> _fetchNetworkData(){}

Future<ProcessData> createData() {
  return _loadFromDisk().then((id) {
    return _fetchNetworkData(id);
  }).then((data) {
    return ProcessData(data);
  })
}
~~~

这个方法表示首先调用`_loadFromDisk()`方法，获取id后再调用`_fetchNetworkData()`方法，获取data后返回`ProcessData`，这个是两层嵌套的例子，倘若是3层、4层乃至更多层，代码将杂乱无章。

为了针对这个问题，Dart引入了`async`和`await`机制，将异步嵌套的代码转换为类似同步代码的结构，极大的方便了编码效率和可读性，上面的两层嵌套等同于：

```dart
ProcessData createData() async {
  int id = await _loadFromDisk();
  String data = await _fetchNetworkData(id)
  return ProcessData(data)
}
```

## 结论

`async`标识一个方法为异步方法，返回的是一个`Future`对象，可以监听`Future`对象的返回值，也可以用`then`的方式等待异步方法执行完毕后继续执行一个方法，若有多个异步方法需要顺序执行，可以采用`then`嵌套的方式，但是层数较多时可读性极差，`await`可以理解为一个异步的语法糖，通过这种方式可以用类似同步的代码块实现异步方法的顺序执行，可读性很高。

