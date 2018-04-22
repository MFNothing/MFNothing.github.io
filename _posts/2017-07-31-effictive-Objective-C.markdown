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
* 有时会使用惰性初始化（重写getter方法，在getter方法中初始化实例变量）技术配置某份数据，这种情况下，需要通过属性来读取数据。否则实例
* 变量永远不会初始化。

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
## 系统框架

### 多用块枚举，少用for循环

#### 为什么不用for循环

当for循环遍历NSDictionary 和 NSSet，无法通过整数下标直接访问其中的值。

所以我们需要先通过 allKeys 或 allObjects去得到一个数组，通过遍历数组来遍历。多创建的数组就是额外不必要的开销。

#### 使用NSEnumerator来遍历 Objective-C 1.0

##### 数组

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

##### 字典

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

#### NSSet

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

##### 数组

```
- (void)enumerateArrByForIn:(NSArray *)array
{
    for (id object in array) {
        // Do something with "object"
    }
}
```

##### 字典

```
- (void)enumerateDictionaryByForIn:(NSDictionary *)dictionary
{
    for (id object in dictionary) {
        // Do something with "object"
    }
}
```

#### NSSet

```
- (void)enumerateSetByForIn:(NSSet *)set
{
    for (id object in set) {
        // Do something with "object"
    }
}
```

#### 块遍历

```
    /*
     typedef NS_OPTIONS(NSUInteger, NSEnumerationOptions) {
     NSEnumerationConcurrent = (1UL << 0), // 并行执行块，通过GCD实现
     NSEnumerationReverse = (1UL << 1), // 反序遍历
     };
     */
```

##### 数组

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

##### 字典

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

#### NSSet

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
### 构建缓存时选用 NSCahche 而非 NSDictionary

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






