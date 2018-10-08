---
layout:     post
title:      "swift 的学习旅程"
subtitle:   "a journey to learn swift"
date:       2017-08-15 12:00:00
author:     "MIN"
header-img: "img/post-bg-nextgen-web-pwa.jpg"
header-mask: 0.3
catalog:    true
tags:
    - swift
---

### 集合类型

Swift 语言提供 ```Arrays```、```Sets```、```Dictionaries``` 三种基本的集合类型用来存储集合数据。

* 数组（Arrays）是有序数据的集。
* 集合（Sets）是无序无重复数据的集。
* 字典（Dictionaries）是无序的键值对的集。

#### 数组

##### 创建数组

**创建一个空数组**

```
var someInts = [Int]();
print("someInts is of type [Int] with \(someInts.count)");
```

注意这个变量 ```someInts``` 的类型就确定了，如果重写给它赋值，也需要是一个 ```Int``` 类型的数据，否则，编辑器会报错

```
someInts = []; // 这里虽然给它的是空数组，但是也会被解析成 int 类型的空数组
someInts = [3]; 
```

**创建一个带默认值的数组**

```
var threeDoubles = Array(repeating: 3.0, count: 3);
print("threeDoubles is of type [Double] with \(threeDoubles.count)");
```

**字面量创建数组**

```
var shoppingList = ["Eggs", "Milk"]; // 注意这里我们没有声明数组类型，Swift 可以推断出其真正类型，因为这里存放的都是相同的类型
// 如果我们要存储不同类型的对象
var anyObjectArr:[AnyObject] = [MINArray()]
// 不一定是对象
var anyArr : [Any] = ["string", 1, 1.0]
```


Swift 3 中 String in Swift is a struct and does not conform to AnyObject

所以这个要跟 NSString 区分开

**通过两个数组相加创建一个数组**

```
var someArr = [1.0, 3.0];
var anotherArr = [1.0, 2.0];
var newArr = someArr + anotherArr;
```

我们可以使用加法操作符来组合两种已存在的相同类型数组。

##### 访问和修改数组

**count和isEmpty**

isEmpty 判断数组是否为空

```
var array = [1];
print("array isEmpty : \(array.isEmpty), count : \(array.count)
```

**append和+=**

```
var array = [1];
array.append(2);
print("\(array)"); // [1, 2]
array += [3];
print("\(array)"); // [1, 2, 3]
array += [4, 5, 6];
print("\(array)"); // [1, 2, 3, 4, 5, 6]
```

**修改**

修改

```
var array = [1, 2, 3, 4, 5, 6];
array[2...4] = [10, 11];
print("\(array)"); // [1, 2, 10, 11, 6]
```

插入

```
array.insert(0, at: 0);
print("\(array)"); // [0, 1, 2, 10, 11, 6]
```

删除

```
array.remove(at: array.endIndex - 1);
print("\(array)"); // [0, 1, 2, 10, 11]
array.removeLast();
print("\(array)"); // [0, 1, 2, 10]
```

**遍历**

```
var array = ["one", "two", "three", "four"];
for item in array {
    print(item);
}
for (index, value) in array.enumerated() {
    print("item \(index) : \(value)");
}
```

enumerated 返回索引值和数据值组成的元组


#### 集合

一个类型为了存储在集合中，该类型必须是可哈希化的，该类型必须提供一个方法来计算他的哈希值。

Swift 的所有基本类型（比如 ```String```,```Int```,```Double```,```Bool```）默认都是可哈希化的，可以作为集合的值的类型或字典的键的类型。没有关联值的枚举成员值默认也是可哈希化的。

