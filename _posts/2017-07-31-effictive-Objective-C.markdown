---
layout:     post
title:      "《Effiective Objective-C 2.0》--笔记"
subtitle:   "Effiective Objective-C 2.0"
date:       2017-07-31 12:00:00
author:     "MIN"
header-img: "img/post-bg-nextgen-web-pwa.jpg"
header-mask: 0.3
catalog:    true
tags:
    - Objecitve-C
---

## 熟悉 Objective-C

### 第一条：了解 objective-C 语言的起源

objective-C 使用“消息结构”而非“函数调用”。

消息结构的语言：

1. 其运行时所应执行的代码由运行环境来决定。
2. 总是在运行时才回去查找所要执行的方法。
3. 重要工作都由“运行期组件”而非编译器完成。如运行期组件含有全部内存管理方法。运行期组件本质就是一种与开发编译代码相链接的“动态库”，其代码能把开发者编写的所有程序粘合起来。

函数调用的语言：

1. 其运行时所应执行的代码由编译器决定。
2. 如果函数是多态的，那么运行时按照“虚函数表”来查出到底应该执行那个函数。

### 第二条：在类的头文件中尽量少引入其他头文件

因为实现属性、实例变量或者遵循协议会需要引入头文件，这样会增加编译时间，增加彼此的依赖程度。过大的依赖会增加维护的麻烦。

* 除非确有必要，否则不要引入头文件。一般来说，应在某个类的头文件中使用向前声明（@class XXXX;）来提及别的类，并在实现文件中引入那些类的头文件。这样做可以降低类之间的耦合（相互引用）。
* 有时无法使用向前声明，比如声明某个类遵循一项协议。这种情况下，尽量把“协议”的声明移到“class-continuation 分类”中。如果不行的话，就把协议单独放到一个头文件中，然后将其引入。

循环引用：在各自头文件中引入对方的头文件，并且A中包含B的一个对象，就会出现错误（Unknown type name 'B'）。

### 第三条：多用字面量语法，少用与之等价的方法

#### 字面数值

```
	NSNumber *intNumber = @1;
	NSNumber *floatNumber = @2.5f;
	NSNumber *douberNumber = @3.1314159;
	NSNumber *boolNumber = @YES;
	NSNumber *charNumber = @'a';
	int x = 5;
	int y = 6.32f;
	NSNumber *expressionNumber = @(x * y);
```

#### 字面量数组

```
	NSArray *animals = @[@"cat",@"dog",@"mouse"];
```

有一个好处，出现异常时可以更快的发现这个错误。

```
	id ob1 = /* */;
	id ob2 = /* */;
	id ob3 = /* */;
	NSArray *array1 = [NSArray arrayWithObjects: ob1, ob2, ob3, nil];
	
	NSArray *array2 = @[ob1, ob2, ob3];
```

如果ob2为nil，array1会被创建出来，但是只有一个对象ob1(arrayWithObjects:方法在发现nil就会停止执行)

而array2会在创建的时候就抛出异常，这样就会减小查找问题的时间。（在字典中出现nil也会抛异常）

#### 字面量字典

```
NSDictionary *dic1 = [NSDictionary dictionaryWithObjectsAndKeys:@"Matt", @"firstName",
                         @"Galloway", @"lastName",
          NSNumber numberWithInt:28], @"age",
                          nil];
    NSDictionary *dic2 = @{@"firstName" : @"Matt",
                            @"lastName" : @"Galloway",
                                 @"age" : @28};
```
注意字面量字典是键值，而方法声明的是值键，顺序相反。

```
	// 访问
	NSString *lastName = [dic1 objectForKey:@"lastName"];
	NSString *firstName = dic2[@"firstName"];
```
#### 局限性

* 必须是Foundation框架中的对象才行
* 字面量创建出来的字符串、数组、字典都是不可变的

需要可变的话加一个mutableCopy


### 第四条：多用类型常量，少用#define预处理指令

* 不要用预处理指令定义常量。这样定义出来的常量不含类型信息，编译器只是会在编译前据此执行查看与替换操作。即使有人重新定义了常量值，编译器也不会产生警告信息（其实会有警告），这将导致应用程序中的常量值不一致。
* 在实现文件中使用static const来定义“只在编译单元内可见的常量”。由于此类常量不会在全局符号表中，所以无须为其名称加前缀或加一个字母k。
* 在头文件中使用extern来声明全局变量，并在相关实现文件中定义其值。这种常量要出现全局符号表中，所以其名称应该加以区分，通常用与之相关的类名做前缀

**编译单元内可见变量** 由static和const声明的变量，编译器不会为其创建符号，会像#define一样直接替换，把所有遇到的变量替换成常值，但是定义处会有类型信息

``` 
static const NSTimerInterval kAnimationDuration = 0.3;
```

**全局变量** EOCString为类名

```
// In the header file
extern NSString *const EOCStringConstant;
// In the implementation file
NSString *const EOCStringConstant = @"VALUE";
```
此类常量必须要要定义，且只能定义一次。
由实现文件生成目标文件时，编译器会在“数据段”为字符串分配存储空间。

### 第五条：用枚举表示状态、选项、状态码

* 应该用枚举来表示状态机的状态、传递给方法的选项以及状态码等值，给这些值起一个易懂的名字。
* 如果把传递给某个方法的选项表示为枚举类型，而多个选项又可同时使用，那么就将各选项定义为2的幂，以便通过按位或操作将其组合。
* 用NS_ENUM于NS_OPTIONS宏来定义枚举类型，并指明其底层数据类型。这样做可以确保**枚举**是用开发者所选的**底层数据类型**实现出来的，而不会采用编译器所选的类型。
* 在处理枚举类型的switch语句中不要实现default分支。这样的话，加入新枚举之后，编译器会提示开发者：switch语句未处理所有枚举。

```
typedef NS_ENUM(NSUInteger, EOCConnectionState)
{
    EOCConnectionStateDisconnected,
    EOCConnectionStateConnecting,
    EOCConnectionStateConnected
};


typedef NS_OPTIONS(NSUInteger, EOCPermittedDirection){
    EOCPermittedDirectionUp = 1 << 0,
    EOCPermittedDirectionDown = 1 << 1,
    EOCPermittedDirectionLeft = 1 << 2,
    EOCPermittedDirectionRight = 1 << 3
} ;
```


## 对象、消息、运行期

### 第6条：理解“属性”这一概念

#### 存取方法

通过“属性”开发者可以令编译器自动编写与属性相关的存取方法。

```
@interface EOCPerson : NSObject
@property NSString *firstName;
@property NSString *lastName;
@end
// 上面相当于下面
@interface EOCPerson : NSObject
- (NSString *)fitstName;
- (void)setFirstName:(NSString *)firstName;
- (NSString *)lastName;
- (void)setLastName:(NSString *)lastName;
```
#### @synthesize

同时编译器还要自动向勒种添加适当类型的实例变量： _firstName 和 _lastName

可以通过@synthesize语法来指定实例变量的名字（但是现在一般没有人用了）

```
@implementation EOCPerson
@synthesize fisrtName = _myFirstName; // 默认为 _firstName
@synthesize lastName = _myLastName; // 默认为 _lastName
@end
```

#### @dynamic

@dynamic关键字会告诉编译器：不要自动创建实现属性所用的实例变量，也不要为其创建存取方法。

```
@interface EOCPerson : NSObject
@property NSString *firstName;
@property NSString *lastName;
@end

@implementation EOCPerson
@dynamic firstName, lastName;
@end
```
代码访问其中属性，编译器也不会发生警告，它相信能做运行期找到。

#### 属性特质

```
@property (nonatomic, readwrite, copy) NSString *firstName; // 分别是 原子性 读写权限 内存管理语义
```

**原子性**

在默认情况下，由编译器所合成的方法会通过锁定机制确保其原子性，但是会消耗性能，而且在大量读取操作时也不能保证其原子性，所以一般都用nonatomic而不是atomic（即一个线程不停读取属性值，而另一个线程在同时改这个值）

**读写权限**

readwrite 属性拥有getter和setter方法

readonly 属性拥有getter方法

**内存管理语义** 

编译器在合成存取方法时，要根据这个特质来决定所生产的代码

|内存管理语义|含义|
|:------:|:------:|
|assign| “设置方法setter” 只会执行对“纯量类型”的简单复制|
|strong| 特质表明属性定义了一种“拥有关系”。为这种属性设置新值时，先保留新值（计数器+1），并释放旧值（计数器-1），然后再将新值设置上去|
|weak| 此特质表明属性定义了一种“非拥有关系”。为这种属性设置新值时，设置方法既不保留新值也不释放旧值，与assign类似，但是这个作用于对象。当指向的对象被释放的时候，会指向nil|
|unsafe_unretained| 与weak的区别在于对象被释放后，指向不会变为nil|
|copy| 与strong类似，但是在设置的时候会将新值进行拷贝，不保留新值。（注意这里虽然是设置的新值的copy返回来的对象（不可变），但是你要知道深浅拷贝。对于不可变对象来说copy是指针拷贝，对于可变对象来说copy才是内容拷贝。同时对于不可变对象和可变对象来说mutablecopy是内容拷贝。然后就是copy返回不可变对象，mutablecopy才会返回可变对象）|

**方法名**

* getter=\<name\> 一般用于Boolean型，如


```
@property (nonatomic, getter=isOn) BOOL on;
```

* setter=\<name\> 不太常见

### 第七条：在对象内部尽量直接访问实例变量



## 接口与API设计
## 协议与分类

### 第二十七条：使用“class-continuation 分类” 隐藏实现细节


"class-continuation 分类" 和普通的分类不同，它必须定义在其所接续的那个类的实现文件里。

特点

* 是唯一能声明实例变量的分类
* 没有特定的实现文件，其中声明的方法都应该定义在类的主实现文件里
* 没有分类名字

"class-continuation 分类" 就是.m 中写的
	
	// 这个部分就是 class-continuation
	@interface MINClass（）
	{// 块，实例变量声明的地方
	
	}
	// 
	@end
	// 上面的部分是 class-continuation,下面是具体的实现
	@implementation MINClass
	{// 括号里面的表示实现块，这里也可以声明实例变量,属性不能在块中声明
		NSString *justString;
	}
	// 
	@end
	
使用的场景

* 保存不用或者不想让外部知晓的变量或属性
* 编写Objective-C++ 代码时
* 将.h 中设置为readonly的属性拓展为readwrite
* 将未在.h 中声明的方法（即私有方法）放在这里声明，增加代码可读性
* 想使类所遵循的协议不为人所知

#### 保存不用或者不想让外部知晓的变量或属性

主要是声明仅供本类使用的变量和属性，不需要暴露或不想暴露的属性或变量。虽然对于运行期总是可以调用方法可以绕过限制，不过从一般意义上他们还是私有的。

#### 编写Objective-C++ 代码时

```
#import <Foundation/Foundation.h>
#include "SomeCppClass.h" // C++类
@interface EOCClass : NSObject{
@private
	SomeCppClass _cppClass;
}
@end
```
Objective-C++ 是Objective-C与C++混合体，其代码可以用两个语言编写

.mm 拓展名表示编译器应该将此文件按Objective-C++来编译

所以对于EOCClass这个类来说，实现文件需为.mm（将此文件按照Objective-C++来编译），否则无法正确引入SomeCppClass.h。
上面这样写会导致的结果就是引入EOCClass这个类的文件也都需要使用Objective-C++来编写。因为编译器需要完全导入SomeCppClass.h这个类才能知道_cppClass实例变量的大小。

所以最好的方式便是使用“class-continuation 分类”

```
// EOCClass.h
#import <Foundation/Foundation.h>
@interface EOCClass : NSObject
@end
//EOCClass.mm
#import "EOCClass.h"
#include "SomeCppClass.h"
@interface EOCClass (){
	SomeCppClass _cppClass;
}
@end
@implementation EOCClass
@end
```
这样对于外部来说，看起来像一个纯OC的代码。如Webkit框架，其大部分代码都以C++编写，但是对外展示出来的却是一套整洁的OC接口。

#### 将.h 中设置为readonly的属性拓展为readwrite

好处是令外界无法修改对象，又能在内部按照需求管理其数据。

#### 在“class-continuation 分类”中声明类的实现代码中用到的私有方法

主要是为了提高代码可读性。

规范可以在私有方法前加上“p_”以做区分，如

```
- (void)p_myPrivateMethod;
```
#### 想使类所遵循的协议不为人所知

内部协议不需要让他人知道，对于写framework有好处吧。

## 内存管理
## 块与大中枢派发
## 系统框架