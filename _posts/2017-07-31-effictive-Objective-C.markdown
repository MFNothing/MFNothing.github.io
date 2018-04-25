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

### 第1条：了解 objective-C 语言的起源

objective-C 使用“消息结构”而非“函数调用”。

消息结构的语言：

1. 其运行时所应执行的代码由运行环境来决定。
2. 总是在运行时才回去查找所要执行的方法。
3. 重要工作都由“运行期组件”而非编译器完成。如运行期组件含有全部内存管理方法。运行期组件本质就是一种与开发编译代码相链接的“动态库”，其代码能把开发者编写的所有程序粘合起来。

函数调用的语言：

1. 其运行时所应执行的代码由编译器决定。
2. 如果函数是多态的，那么运行时按照“虚函数表”来查出到底应该执行那个函数。

### 第2条：在类的头文件中尽量少引入其他头文件

因为实现属性、实例变量或者遵循协议会需要引入头文件，这样会增加编译时间，增加彼此的依赖程度。过大的依赖会增加维护的麻烦。

* 除非确有必要，否则不要引入头文件。一般来说，应在某个类的头文件中使用向前声明（@class XXXX;）来提及别的类，并在实现文件中引入那些类的头文件。这样做可以降低类之间的耦合（相互引用）。
* 有时无法使用向前声明，比如声明某个类遵循一项协议。这种情况下，尽量把“协议”的声明移到“class-continuation 分类”中。如果不行的话，就把协议单独放到一个头文件中，然后将其引入。

循环引用：在各自头文件中引入对方的头文件，并且A中包含B的一个对象，就会出现错误（Unknown type name 'B'）。

### 第3条：多用字面量语法，少用与之等价的方法

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


### 第4条：多用类型常量，少用#define预处理指令

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

编译器会自动编写访问这些属性所需的方法（get和set方法）

同时编译器还要自动向类中添加适当类型的实例变量： _firstName 和 _lastName

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

**直接访问与通过属性访问的区别**

* 直接访问不经过Objective-C的“方法派发”步骤，所以访问实例变量更快，编译器会直接访问保存对象实例对象的那块内存
* 直接访问不会调用getter或setter方法，从而绕开了相关属性定义的“内存管理语义”。（如果property定义的copy）
* 不会触发键值对KVO

**总结**

* 在对象内部读取数据时，应该直接通过实例变量来读，而写入数据时，则应通过属性来写。（在setter方法中通过属性写入/获取中通过属性读取会导致死循环，因为属性先会调用setter方法/getter方法，然后setter方法/getter方法中有属性又会调用setter方法/getter方法从而导致死循环。）
* 在初始化方法(setter和getter方法)及dealloc方法中，总是应该直接通过实例变量读写数据
* 有时会使用惰性初始化（重写getter方法，在getter方法中初始化实例变量）技术配置某份数据，这种情况下，需要通过属性来读取数据。否则实例变量永远不会初始化。

### 第8条：理解“对象等同性”这一概念

在Objective-C中比较两个对象时使用==操作符，比较的是两个指针本身而非对象。如果需要做对象比较需要使用NSObject协议中声明的“isEqual:”方法来进行比较。

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

### 第37条：理解“块”这一概念

块与函数类似，只不过是直接定义在另一个函数里的，和定义它的那个函数共享同一个范围内的东西。

```
- (void)simpleBlock {
    ^{
        NSLog(@"unsed block");
    }; // 这个只是声明这个block，但是并未使用它，它可以访问simpleBlock这个函数内的和可以访问到的所有变量或方法
}
```

块从上面的例子看出，它就是一个值，而且自有其相关类型。与 int、 float  或 Objective-C 对象一样，也可以把块赋值给变量，然后像使用其他变量那样去使用它。

块类型的语法结构如下：

```
return_type (^block_name)(parameters)
```

几个简单的块及使用

```
- (void)usingBlock {
    void (^printBlock) (void) = ^{
        NSLog(@"没有参数也没有返回值的block");
    };
    printBlock();
    int (^addNum)(int a, int b) = ^(int a, int b){
        return a + b;
    };
    int num = addNum(1, 2);
    NSLog(@"%d", num);
}
```

为块所捕获的变量，是不可以在块里修改的，如果想要在块内修改变量的值，可以加上 __block。

```
- (void)usingBlockChangeValue {
    __block int a = 0;
    __block NSMutableArray *array = [NSMutableArray array];
    void (^changeValueBlock)(void) = ^{
        a = 10;
        [array addObject: [NSNumber numberWithInt: a]];
    };
    changeValueBlock();
    NSLog(@"%d %@", a, array);
}
```
如果块所捕获的变量是对象类型，那么就会自动保留它。

块本身可视为对象，有引用计数。当最后一个指向块的引用移走后，块就回收了。

如果将块定义在Objective-C类的实例方法中，那么除了可以访问类的所有变量之外，还可以使用self变量。块总能修改实例变量，所以在声明无须加__block。

如果通过读取或写入操作捕获了实例变量，那么也会自动把self变量一并捕获了。直接访问实例变量和通过self来访问是等效的。(注意这里的self访问是下面的方式，与属性访问实例变量的方式不一样，不会调用属性的 setter 或 getter 方法)

```
- (void)visitAnInstanceVariable
{
    void (^visitAnInstanceVariableBlock)(void) = ^{
        self->_anInstanceVariable = @"anInstanceVariable"; 
//        _anInstanceVariable = @"anInstanceVariable"; // 与上面等价
    };
    visitAnInstanceVariableBlock();
    NSLog(@"%@", self.anInstanceVariable); // 这里才调用了setter 和 getter 方法
}
```

之所以要捕获self变量，原因正因如此，在块中会通过这个方式去访问实例变量，即使没有加 self->。这里也会产生一个问题，就是如果 self 保留了这个块就会出现“保留环”（一般我们说的循环引用）。

#### 块的内部结构

内部结构

![](/img/in-mpost/Effective-Objective-C/块对象的内存布局.png)

首个变量是指向Class 对象的指针，该指针叫做 isa。

最重要的 invoke 变量，这个函数指针，指向块的实现代码。函数原型至少接受一个void *型的参数，此参数代表块。刚才说过块其实是一种代替函数指针的语法结构。

descriptor 变量是指向结构提的指针，每个块里都包含此结构体，其中声明了块对象的总体大小，还声明了 copy 和 dispose 这两个辅助函数对应的函数指针。辅助函数在拷贝及丢弃块对象时运行，前者要保留捕获的对象，后者则将捕获对象释放。

块会把它捕获的所有变量都拷贝一份，这些拷贝放在 descriptor 变量后面，注意这里拷贝的并不是对象本身，而是指向这些对象的指针变量。invoke 函数为何需要把块对象作为参数传进来，原因就在于，执行块是，要从内存中吧这些捕获到的变量读出来。

#### 全局块、栈块及堆块

定义块的时候，其所占的内存区域是分配在栈中的。这就是说，块只在定义它的那个范围有效（函数或类中）。所以我们设置属性的时候，对于block来说设置为copy，进行copy操作后会将块拷贝到堆上面，这样就可以在定义它的范围之外使用了。而且，一旦复制到堆上，块就成了带引用计数的对象了。后续的复制操作都不会真的执行复制，只是递增对象的引用计数。

```
typedef void (^CopyBlock)(void);
@property (nonatomic, copy) CopyBlock aCopyBlock;
@property (nonatomic, copy) CopyBlock aAnotherBlock;

- (void)copyBlock{
    __block int a = 0;
    void (^block)(void) = ^{
        a = 10;
    };
    NSLog(@"%@", block); // <__NSMallocBlock__: 0x600000249db0>
    self.aCopyBlock = block;
    self.aAnotherBlock = block;
    NSLog(@"%@ %@", self.aCopyBlock, self.aAnotherBlock); // <__NSMallocBlock__: 0x600000249db0> <__NSMallocBlock__: 0x600000249db0>
}
```

**栈块**

使用定义范围内的变量的block，但是赋值给block变量之后会变成 \<__NSMallocBlock__: 0x60c00024eee0>

```
int a = 0;
NSLog(@"%@", ^{NSLog(@"%d", a);}); //  <__NSStackBlock__: 0x7ffeeede9a60>
NSLog(@"%@", ^{a = 10;}); //  <__NSStackBlock__: 0x7ffeed927a38>
```

**堆块**

栈块赋值给block变量后，会变成堆块(感觉自动进行一次拷贝操作)。

```
__block int a = 0;
NSLog(@"%@", ^{a = 10;}); // <__NSStackBlock__: 0x7ffeef751a70>
block = ^{NSLog(@"%d", a);};
NSLog(@"%@", block); // <__NSMallocBlock__: 0x6000000559f0>
```

这个例子能看出那个拷贝操作

```
typedef void (^CopyBlock)(void);
@property (nonatomic, copy) CopyBlock aCopyBlock;
@property (nonatomic, copy) CopyBlock aAnotherBlock;

- (void)copyBlock{
    __block int a = 0;
    void (^block)(void);
    block = ^{NSLog(@"%d", a);};
    NSLog(@"%@", block);// <__NSMallocBlock__: 0x60c000050e60>
    self.aCopyBlock = block;
    self.aAnotherBlock = [block copy];
    NSLog(@"%@ %@", self.aCopyBlock, self.aAnotherBlock); // <__NSMallocBlock__: 0x60c000050e60> <__NSMallocBlock__: 0x60c000050e60>
}
```

**全局块**

全局块就是没有捕获状态的块（比如可访问范围内的变量）。块所使用的整个内存区域，在编译期已经完全确定了，所以全局块可以声明在全局内存里，而不需要每次用到的时候于栈中创建。

全局块的拷贝操作是空操作，因为全局块不会被系统回收。相当于单例。

```
typedef void (^CopyBlock)(void);
@property (nonatomic, copy) CopyBlock aCopyBlock;
@property (nonatomic, copy) CopyBlock aAnotherBlock;

- (void)copyBlock{
    __block int a = 0;
    void (^block)(void);
    block = ^{
        NSLog(@"__NSGlobalBlock__");
    };
    NSLog(@"%@", block); // <__NSGlobalBlock__: 0x10b4b1220>
    self.aCopyBlock = block;
    self.aAnotherBlock = [block copy];
    NSLog(@"%@ %@", self.aCopyBlock, self.aAnotherBlock); // <__NSGlobalBlock__: 0x10b4b1220> <__NSGlobalBlock__: 0x10b4b1220>
}
```
虽然block类型不一样，但是地址是相同的。


### 第39条：为常用的块类型创建 typedef

目的是为了增加代码可读性。

```
typedef void (^CopyBlock)(void);
@property (nonatomic, copy) CopyBlock aCopyBlock;

- (void)useTypeDef
{
    CopyBlock block = ^{
        
    };
}

- (void)useCopyBlock:(CopyBlock)copyBlock
{
    if (copyBlock) {
        copyBlock();
    }
}
```

并且当你后面需要修改这个block的参数时候，直接修改这里，然后编译，可以找到所有你用过的地方（因为编译会报错），然后依次替换就好了。

如果用之前的方式，你可能找不全你想要改的地方，代码过于复杂的话。

如果有好几个类都要执行相似但各有区别的异步任务，而这几个类又不能放入同一个继承体系的时候，那么，每一个类就应该有自己的块类型，即使块中的参数类型都相同。这样可以保证如果某个类有新的需求需要改参数的时候，不用去动其他类的块。

例如：

```
typedef void (^MINAccountStoreSaveCompletionHandler)(BOOL success, NSError *error);
typedef void (^MINAccountStoreRequestAccessCompletionHandler)(BOOL success, NSError *error);
```

### 第39条：用 handler 块降低代码分散程度

## 系统框架

### 第48条：多用块枚举，少用for循环

#### 为什么不用for循环

当for循环遍历NSDictionary 和 NSSet，无法通过整数下标直接访问其中的值。

所以我们需要先通过 allKeys 或 allObjects去得到一个数组，通过遍历数组来遍历。多创建的数组就是额外不必要的开销。

#### 使用NSEnumerator来遍历 Objective-C 1.0

**数组**

```
- (void)enumerateArr:(NSArray *)array
{
    NSEnumerator *enumerator = [array objectEnumerator];
    // 反向遍历
//    NSEnumerator *enumerator = [array reverseObjectEnumerator];
    id object;
    while ((object = [enumerator nextObject]) != nil) {
        // Do something with "object"
    }
}
```

**字典**

```
- (void)enumerateDictionary:(NSDictionary *)dictionary
{
    NSEnumerator *enumerator = [dictionary objectEnumerator];
    // 反向遍历
//    NSEnumerator *enumerator = [dictionary reverseObjectEnumerator];
    id object;
    while ((object = [enumerator nextObject]) != nil) {
        // Do something with "object"
    }
}
```

**NSSet**

```
- (void)enumerateSet:(NSSet *)set
{
    NSEnumerator *enumerator = [set objectEnumerator];
    // 反向遍历
//    NSEnumerator *enumerator = [set reverseObjectEnumerator];
    id object;
    while ((object = [enumerator nextObject]) != nil) {
        // Do something with "object"
    }
}
```

#### 快速遍历 for...in 

缺点是无法直接拿到对象下标

**数组**

```
- (void)enumerateArrByForIn:(NSArray *)array
{
    for (id object in array) {
        // Do something with "object"
    }
}
```

**字典**

```
- (void)enumerateDictionaryByForIn:(NSDictionary *)dictionary
{
    for (id object in dictionary) {
        // Do something with "object"
    }
}
```

**NSSet**

```
- (void)enumerateSetByForIn:(NSSet *)set
{
    for (id object in set) {
        // Do something with "object"
    }
}
```

**块遍历**

```
    /*
     typedef NS_OPTIONS(NSUInteger, NSEnumerationOptions) {
     NSEnumerationConcurrent = (1UL << 0), // 并行执行块，通过GCD实现
     NSEnumerationReverse = (1UL << 1), // 反序遍历
     };
     */
```

**数组**

```
- (void)enumerateArrUsingBlock:(NSArray *)array
{
    BOOL shouldStop = YES;
    [array enumerateObjectsUsingBlock:^(id  _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
        if (shouldStop) {
            *stop = YES;
        }
    }];
    [array enumerateObjectsWithOptions: NSEnumerationReverse usingBlock:^(id  _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
        if (shouldStop) {
            *stop = YES;
        }
    }];
}
```

**字典**

```
- (void)enumerateDictionaryUsingBlock:(NSDictionary *)dictionary
{
    BOOL shouldStop = YES;
    [dictionary enumerateKeysAndObjectsUsingBlock:^(id  _Nonnull key, id  _Nonnull obj, BOOL * _Nonnull stop) {
        if (shouldStop) {
            *stop = YES;
        }
    }];
    [dictionary enumerateKeysAndObjectsWithOptions: NSEnumerationConcurrent usingBlock:^(id  _Nonnull key, id  _Nonnull obj, BOOL * _Nonnull stop) {
        if (shouldStop) {
            *stop = YES;
        }
    }];
}
```

**NSSet**

```
- (void)enumerateSetUsingBlock:(NSSet *)set
{
    BOOL shouldStop = YES;
    [set enumerateObjectsUsingBlock:^(id  _Nonnull obj, BOOL * _Nonnull stop) {
        if (shouldStop) {
            *stop = YES;
        }
    }];
    // 反向
    [set enumerateObjectsWithOptions: NSEnumerationReverse usingBlock:^(id  _Nonnull obj, BOOL * _Nonnull stop) {
        if (shouldStop) {
            *stop = YES;
        }
    }];
}
```
### 第49条：对自定义其内存管理语义的collection使用无缝桥接

首先先了解Objective-C的系统库中属于collection类的，有数组、字典、set等。

Foundation框架定义了这些collection以及其他各种collection所对应的Objective-C类

与之对应的CoreFoundation框架也定义了一套C语言的API，用于操作这些collection以及其他各种collection的数据结构。

NSArray （Foundation框架中表示Objective-C类）

CFArray （Corefoundation框架中等价物）

无缝桥接（toll—free bridging）是一个可让两个类型平滑转换的技术

```
    NSArray *array = @[@1, @2, @3, @4];
    CFArrayRef cfArray = (__bridge_retained CFArrayRef) array;
    NSLog(@"size = %li", CFArrayGetCount(cfArray));
    NSArray *transferArray = (__bridge_transfer NSArray*)cfArray;
    NSLog(@"transfer size = %lu", (unsigned long)transferArray.count);
```

__bridge ：ARC仍然具备这个Objective-C对象的所有权

__bridge_retained：交出对象的所有权，上面代码的cfArray，在被用完之后需要加上CFRelease(cfArray)以释放其内存。(__bridge_retained \<CF type>)

__bridge_transfe：将collection数据结构转换为Objective-C对象，并使得对象获得所有权。所以如果前面用__bridge会报错，因为用这个cfArray不拥有对象所有权，也交不出来。 (__bridge_transfer \<Objective-C type>)

在自定义不同语义的NSMutableDictionary，对CFDictionaryCreateMutable 的了解

```
CFMutableDictionaryRef CFDictionaryCreateMutable(CFAllocatorRef allocator, CFIndex capacity, const CFDictionaryKeyCallBacks *keyCallBacks, const CFDictionaryValueCallBacks *valueCallBacks);

/*
	CFAllocatorRef allocator 表示要使用的内存分配器，通常传入NULL，表示采用默认的分配器
	CFIndex capacity 字典初始化大小，它并不会限制字典的最大容量。跟 NSMutableDictionary 的 + (instancetype)dictionaryWithCapacity:(NSUInteger)numItems; 差不多
	const CFDictionaryKeyCallBacks *keyCallBacks 这个默认有两个const 可以传入 （ kCFTypeDictionaryKeyCallBacks 和 kCFCopyStringDictionaryKeyCallBacks ）
		* kCFTypeDictionaryKeyCallBacks 表示键设置的时候需要是CFTypes
		* kCFCopyStringDictionaryKeyCallBacks 表示键设置时候需要是CFStrings并且可以被拷贝
	const CFDictionaryValueCallBacks *valueCallBacks 这里也有一个const 可以传入 kCFTypeDictionaryValueCallBacks
		* kCFTypeDictionaryValueCallBacks 表示值设置的时候需要是CFTypes
*/

/*
CFDictionaryKeyCallBacks 结构体
typedef struct {
    CFIndex				version;
    CFDictionaryRetainCallBack		retain;
    CFDictionaryReleaseCallBack		release;
    CFDictionaryCopyDescriptionCallBack	copyDescription;
    CFDictionaryEqualCallBack		equal;
    CFDictionaryHashCallBack		hash;
} CFDictionaryKeyCallBacks;
CFDictionaryValueCallBacks 结构体
typedef struct {
    CFIndex				version;
    CFDictionaryRetainCallBack		retain;
    CFDictionaryReleaseCallBack		release;
    CFDictionaryCopyDescriptionCallBack	copyDescription;
    CFDictionaryEqualCallBack		equal;
} CFDictionaryValueCallBacks;

version 参数目前应设为0。可能会修改此结构体，预留该值以表示版本号。

typedef const void *	(*CFDictionaryRetainCallBack)(CFAllocatorRef allocator, const void *value);

第一个参数参考上面，第二个参数表示传入的键或值，返回类型const void * 说明上面应该传一个函数指针。这些都可以自定义的。

typedef void		(*CFDictionaryReleaseCallBack)(CFAllocatorRef allocator, const void *value);

跟前面类似，表示release的时候回调的函数。

typedef CFStringRef	(*CFDictionaryCopyDescriptionCallBack)(const void *value);

typedef Boolean		(*CFDictionaryEqualCallBack)(const void *value1, const void *value2);

typedef CFHashCode	(*CFDictionaryHashCallBack)(const void *value);

*/
```

自定义一个其他语义的NSMutableDictionary

```
const void* MINRetainCallBack(CFAllocatorRef allocator, const void *value)
{
    return CFRetain(value);
//    return value;
}

void MINReleaseCallBack(CFAllocatorRef allocator, const void *value)
{
    CFRelease(value);
}

- (void)testCustomDictionary
{
    CFDictionaryKeyCallBacks keyCallBack = {0, MINRetainCallBack, MINReleaseCallBack, NULL, CFEqual, CFHash};
    CFDictionaryValueCallBacks valueCallBack = {0, MINRetainCallBack, MINReleaseCallBack, NULL, CFEqual};
    CFMutableDictionaryRef cfDictionary = CFDictionaryCreateMutable(kCFAllocatorDefault, 1, &keyCallBack, &valueCallBack);
//    CFStringRef cfString = CFSTR("key");
    const void *cKeyArr[] = {CFSTR("key1"), CFSTR("key2")};
    const void *cValueArr[] = {CFSTR("value1"), CFSTR("value2")};
    CFArrayRef cfKeyArray = CFArrayCreate(0, cKeyArr, (CFIndex)2, NULL);
    CFArrayRef cfValueArray = CFArrayCreate(0, cValueArr, (CFIndex)2, NULL);
    CFDictionaryAddValue(cfDictionary, cfKeyArray, cfValueArray);
    NSMutableDictionary *dictionary = (__bridge_transfer NSMutableDictionary *)cfDictionary; // 一般情况下我们的key值都是string，现在可以设置对象了
    NSLog(@"%@", dictionary);
}
```

这里改变了键值所对应的retain回调函数对key的处理方式，不是拷贝key了而是保留key

设置回调的时候，copyDescription 取值为NULL， 因为采用默认的实现就很好。而equal 与 hash 回调函数分别设置为  CFEqual 和 CFHash，因为这二者采用的方式与NSMutableDictionary 的默认实现相同。CFEqual 最终会调用 NSObject 的 “isEqual:”方法，而CFHash 则会调用 hash 方法。 

键值所对应的的retain 和 release 回调函数指针分别指向 MINRetainCallBack 和 MINReleaseCallBack 函数。一般我们创建的 NSMutableDictionary 加入键和值是，字典会自动“拷贝”键并保留“值”。这里我们将键的方式改成了“保留”键。可以做到，当键值不能被拷贝的时候不会报错。

开发者可以通过这样的方式创建一个自定义语义的Objective-C类对象。

**要点**

* 通过无缝桥接技术，可以在Foundation框架中的Objective-C对象与CoreFoundation框架中的C语言数据结构之间来为转换
* 在CoreFoundation层面创建collection时，可以指定许多回调函数，这些函数表示此collection应如何处理其元素。然后，可运用无缝桥接技术，将其转换成具备特殊内存管理语义的Objective-C collection。

### 第50条：构建缓存时选用 NSCahche 而非 NSDictionary

NSCache 的优势 :

[NSCache相关基础参考](http://southpeak.github.io/2015/02/11/cocoa-foundation-nscache/) 

1. 当系统资源将要耗尽的时候，它可以自动减删缓存，先行删减“最久未使用的”对象，但是可以在它的NSCacheDelegate代理方法中去对删除对象处理（存入Sqlite或CoreData）
2. NSCache 不会拷贝键，而是持有它。set 在 NSCache 中的对象可以结合\< NSDiscardableContent >协议中的四个方法，让其对象在不被我们使用的时候，可以将其丢弃，以让程序占用更少的内容。通过设置 NSCache 的 evictsObjectsWithDiscardedContent 属性开启或关闭这种方式(默认是开启的)。后面会有相关方法的详细解释。
  * - (BOOL)beginContentAccess;
  * - (void)endContentAccess;
  * - (void)discardContentIfPossible;
  * - (BOOL)isContentDiscarded; 	
3. 线程安全，多个线程可以同时访问NSCache
4. 可以设置最大开销(最大对象个数或者最大内存使用大小)，大于这个开销会删减对象。

如果缓存使用得当，应用程序的响应速度就能提高。只有那种“重新计算起来很费事的”数据，才值得放入缓存，比如那些需要从网络获取或从磁盘读取的数据


#### NSDiscardableContent 协议

一个NSDiscardableContent对象的生命周期依赖于一个“counter”变量。一个NSDiscardableContent对象实际是一个可清理内存块，这个内存记录了对象当前是否被其它对象使用。如果这块内存正在被读取，或者仍然被需要，则它的counter变量是大于或等于1的；当它不再被使用时，就可以丢弃，此时counter变量将等于0。当counter变量等于0时，如果当前时间点内存比较紧张的话，内存块就可能被丢弃。

这个跟对象引用计数无关。这个“counter”变量只用于记录对象是否被其它对象使用。所以这个“counter”变量应该由我们自己来创建。

所以在初始化对象的时候，我们需要将这个“counter”变量初始化为1。从这个点开始，我们就需要去跟踪counter变量的状态。通过协议声明的两个方法：beginContentAccess和endContentAccess

* - (BOOL)beginContentAccess; //增加counter变量 函数的返回值如果是YES，则表明可丢弃内存仍然可用且已被成功访问；否则返回NO。
* - (void)endContentAccess; // 该方法会减少对象的counter变量，通常是让对象的counter值变回为0，这样在对象的内容不再被需要时，就要以将其丢弃。所以在你还想使用这个对象的时候，要让counter变量大于0
* - (void)discardContentIfPossible; // 当对象将要被丢弃的时候回调用这个方法，我们也可以主动调用这个方法，去做一些存储的事情，如果这个对象需要被存储在本地的话。这个方法会在NSCache代理方法后面执行。
* - (BOOL)isContentDiscarded; // 会在我们需要调用这个NSDiscardableContent对象调用。这里我们可以根据count变量的值去判断，这个对象是否被丢弃，如果被丢弃就返回NO，如果没有就返回YES。
	* 如果返回 NO，会去响应NSCache代理方法，然后会调用discardContentIfPossible方法。
	* 如果返回 YES。

这里要注意当我们调用 NSCache 的objectForKey方法的时候，会默认去调用key值所对应的对象的isContentDiscarded方法，如果返回NO，则不会执行beginContentAccess和endContentAccess方法了，而是去掉用 NSCache 对象的willEvictObject:代理方法，和丢弃对象的discardContentIfPossible方法。


下面会简单的构建一个NSDiscardableContent对象

```
@interface CacheObject : NSObject <NSDiscardableContent>
@property (nonatomic, assign) int counter;
@property (nonatomic, assign) int num;
@end
#import "CacheObject.h"

@interface CacheObject() 
@end

@implementation CacheObject
- (instancetype)init
{
    if (self = [super init]) {
        _counter = 1;
    }
    return self;
}

- (BOOL)beginContentAccess
{
    self.counter += 1;
    NSLog(@"obj = %@ num = %d count = %d beginContentAccess", self, self.num, self.counter);
    if (self.counter > 0) {
        return YES;
    }
    return NO;
}

- (void)endContentAccess
{
    self.counter -= 1;
    NSLog(@"obj = %@ num = %d count = %d  endContentAccess", self, self.num, self.counter);
}

- (void)discardContentIfPossible
{
    NSLog(@"obj = %@ num = %d count = %d  discardContentIfPossible", self, self.num, self.counter);
}

- (BOOL)isContentDiscarded
{
    NSLog(@"obj = %@ num = %d count = %d  isContentDiscarded", self, self.num, self.counter);
    if (self.counter > 0) {
        return NO;
    }
    return YES;
}
```

简单的使用

```
- (void)TestNSCache
{
    NSCache *cache = [[NSCache alloc] init];
    cache.delegate = self;
//    self.cache.countLimit = 2; // 0 为不限制,这里如果不注释的话，第二个for循环就会开始丢弃对象了，注释后，会在这个函数执行完后丢弃所有对象，因为NSCache对象被释放了
//    cache.evictsObjectsWithDiscardedContent = NO; // 默认为YES。 如果注释取消后，第二个for循环取出对象时不会去调用isContentDiscarded方法去判断对象是否被释放了
    for (int i = 0; i < 4; i++) {
        CacheObject *object = [[CacheObject alloc] init]; // 初始化后counter为1
        object.num = i;
        [cache setObject: object forKey: [NSNumber numberWithInt: i]];
        NSLog(@"setting data %@ %d", object ,object.num);
//        [obj endContentAccess]; 如果这里执行了，下面的beginContentAccess和endContentAccess不会执行
        NSLog(@"--------------");
    }
    for (int i = 0; i < 4; i++) {
        CacheObject *obj = [cache objectForKey: [NSNumber numberWithInt:i]];
        [obj beginContentAccess]; // 让其增加，保证其他地方减一的时候，这里不会被释放
        NSLog(@"using data %@ %d", obj ,obj.num);
        [obj endContentAccess]; // 让其减一，当其他地方再调用时，如果为0，就回去执行丢弃的代理方法和discardContentIfPossible方法了
        NSLog(@"***************");
    }
}

#pragma mark - NSCacheDelegate
- (void)cache:(NSCache *)cache willEvictObject:(id)obj
{
    CacheObject *object = (CacheObject *)obj;
    NSLog(@"willEvictObject obj = %@ num = %d",object, object.num);
}
```

打印

```
2018-04-22 15:59:53.131463+0800 enumerateTest[6407:219822] setting data <CacheObject: 0x60800000deb0> 0
2018-04-22 15:59:53.131622+0800 enumerateTest[6407:219822] --------------
2018-04-22 15:59:53.131749+0800 enumerateTest[6407:219822] setting data <CacheObject: 0x60000000c6c0> 1
2018-04-22 15:59:53.131833+0800 enumerateTest[6407:219822] --------------
2018-04-22 15:59:53.131924+0800 enumerateTest[6407:219822] setting data <CacheObject: 0x60000000c760> 2
2018-04-22 15:59:53.132004+0800 enumerateTest[6407:219822] --------------
2018-04-22 15:59:53.132110+0800 enumerateTest[6407:219822] setting data <CacheObject: 0x60000000c780> 3
2018-04-22 15:59:53.132188+0800 enumerateTest[6407:219822] --------------
2018-04-22 15:59:53.132290+0800 enumerateTest[6407:219822] obj = <CacheObject: 0x60800000deb0> num = 0 count = 1  isContentDiscarded
2018-04-22 15:59:53.132401+0800 enumerateTest[6407:219822] obj = <CacheObject: 0x60800000deb0> num = 0 count = 2 beginContentAccess
2018-04-22 15:59:53.132499+0800 enumerateTest[6407:219822] using data <CacheObject: 0x60800000deb0> 0
2018-04-22 15:59:53.132633+0800 enumerateTest[6407:219822] obj = <CacheObject: 0x60800000deb0> num = 0 count = 1  endContentAccess
2018-04-22 15:59:53.132728+0800 enumerateTest[6407:219822] ***************
2018-04-22 15:59:53.132813+0800 enumerateTest[6407:219822] obj = <CacheObject: 0x60000000c6c0> num = 1 count = 1  isContentDiscarded
2018-04-22 15:59:53.132930+0800 enumerateTest[6407:219822] obj = <CacheObject: 0x60000000c6c0> num = 1 count = 2 beginContentAccess
2018-04-22 15:59:53.133102+0800 enumerateTest[6407:219822] using data <CacheObject: 0x60000000c6c0> 1
2018-04-22 15:59:53.133257+0800 enumerateTest[6407:219822] obj = <CacheObject: 0x60000000c6c0> num = 1 count = 1  endContentAccess
2018-04-22 15:59:53.133426+0800 enumerateTest[6407:219822] ***************
2018-04-22 15:59:53.133630+0800 enumerateTest[6407:219822] obj = <CacheObject: 0x60000000c760> num = 2 count = 1  isContentDiscarded
2018-04-22 15:59:53.133811+0800 enumerateTest[6407:219822] obj = <CacheObject: 0x60000000c760> num = 2 count = 2 beginContentAccess
2018-04-22 15:59:53.133989+0800 enumerateTest[6407:219822] using data <CacheObject: 0x60000000c760> 2
2018-04-22 15:59:53.134164+0800 enumerateTest[6407:219822] obj = <CacheObject: 0x60000000c760> num = 2 count = 1  endContentAccess
2018-04-22 15:59:53.134312+0800 enumerateTest[6407:219822] ***************
2018-04-22 15:59:53.134583+0800 enumerateTest[6407:219822] obj = <CacheObject: 0x60000000c780> num = 3 count = 1  isContentDiscarded
2018-04-22 15:59:53.134754+0800 enumerateTest[6407:219822] obj = <CacheObject: 0x60000000c780> num = 3 count = 2 beginContentAccess
2018-04-22 15:59:53.134942+0800 enumerateTest[6407:219822] using data <CacheObject: 0x60000000c780> 3
2018-04-22 15:59:53.135135+0800 enumerateTest[6407:219822] obj = <CacheObject: 0x60000000c780> num = 3 count = 1  endContentAccess
2018-04-22 15:59:53.135314+0800 enumerateTest[6407:219822] ***************
2018-04-22 15:59:53.135562+0800 enumerateTest[6407:219822] willEvictObject obj = <CacheObject: 0x60800000deb0> num = 0
2018-04-22 15:59:53.135657+0800 enumerateTest[6407:219822] obj = <CacheObject: 0x60800000deb0> num = 0 count = 1  discardContentIfPossible
2018-04-22 15:59:53.135786+0800 enumerateTest[6407:219822] willEvictObject obj = <CacheObject: 0x60000000c6c0> num = 1
2018-04-22 15:59:53.135962+0800 enumerateTest[6407:219822] obj = <CacheObject: 0x60000000c6c0> num = 1 count = 1  discardContentIfPossible
2018-04-22 15:59:53.136138+0800 enumerateTest[6407:219822] willEvictObject obj = <CacheObject: 0x60000000c760> num = 2
2018-04-22 15:59:53.136264+0800 enumerateTest[6407:219822] obj = <CacheObject: 0x60000000c760> num = 2 count = 1  discardContentIfPossible
2018-04-22 15:59:53.136426+0800 enumerateTest[6407:219822] willEvictObject obj = <CacheObject: 0x60000000c780> num = 3
2018-04-22 15:59:53.136602+0800 enumerateTest[6407:219822] obj = <CacheObject: 0x60000000c780> num = 3 count = 1  discardContentIfPossible
```

### 第51条：精简initialize 与 load 的实现方法

有时候，类必须先执行某些初始化操作，然后才能正常使用。在 Objective-C 中，对大多数类都继承自 NSObject 这个根类，而该类有两个方法，可以用来实现这种初始化操作

**load** 方法，对于加入运行期系统中的每个类（class）以及分类（category）来说，必定会调用这个方法，而且仅调用一次。当包含类或分类的程序库载入系统时，就会执行此方法。

* 在 iOS 应用程序中，这个时候通常指的是应用启动的时候
* 在 MAC OS X 应用程序更自由一点，它们可以使用“动态加载”之类的特性，等应用程序启动好之后再去加载程序库。

特点：

1. 调用顺序，先调用超类的，再调用子类，如果类中还有分类，会先调类里面的，再调分类的，分类调用顺序跟超类和子类无关。
2. 调用时会阻塞线程，所以复杂的操作不要在这里进行。
3. 不会覆盖超类的实现，各自执行自己的实现，如果没实现不执行。

尽量不使用这个方法,这个方法只要有self，就先去调用initialize方法。

**initialize** 方法，程序会在首次使用该类之前调用，且只调用一次。它是由运行期系统调用的，绝不应该通过代码直接调用。

initialize 方法只应该用来设置内部数据。不要调用其他方法，即使是自己类中的方法。

应用地方，无法再编译期设定的全局变量的初始化，比如初始化一个全局的可变数组。整数可以编译期定义，可变数组不行，因为它是一个Objective-C 对象，所以创建实例之前必须先激活运行期系统。

特点：

1. 从运行期系统完整度上来讲，此时可以安全使用并调用任意类中的任意方法。
2. 运行期系统也能确保 initialize 方法一定会在“线程安全的环境”中执行，就是说只有执行的那个线程可以操作类或类实例，其他线程都要先阻塞，等待它执行完成。
3. 如果超类实现了 initialize 方法，子类未实现，会执行父类的方法。
4. 调用顺序也是先父类再子类




