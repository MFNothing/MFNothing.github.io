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

加锁类似于这样

```
- (NSString *)someString
{
    @synchronized(self) {
        return _minString;
    }
}

- (void)setSomeString:(NSString *)someString
{
    @synchronized(self){
        _minString = [someString copy];
    }
}
```

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

在Objective-C中比较两个对象时使用==操作符，比较的是两个指针本身而非对象。如果需要做对象比较，需要使用NSObject协议中声明的“isEqual:”方法来进行比较。

**NSObject 协议中有两个用于判断等同性的关键方法:**

```
- (BOOL)isEqual:(id)object;

- (NSUInteger)hash;
```

这两个方法的默认实现是：当且仅当其“指针值”完全相同时，这两个对象才相等。

比如有下面这个类：

```
@interface MINPerson : NSObject
@property (nonatomic, copy) NSString *firstName;
@property (nonatomic, copy) NSString *lastName;
@property (nonatomic, assign) NSUInteger age;
@end
```

它的 “isEqual:”方法可以写成：

```
- (BOOL)isEqual:(id)object
{
    if (self == object) {
        return YES;
    }
    if ([self class] != [object class] ) {
        return NO;
    }
    MINPerson *otherPerson = (MINPerson *)object;
    if (![_firstName isEqualToString: otherPerson.firstName]) {
        return NO;
    }
    if (![_lastName isEqualToString: otherPerson.lastName]) {
        return NO;
    }
    if (_age != otherPerson.age) {
        return NO;
    }
    return YES;
}
```
**覆写 hash 方法：**

```
- (NSUInteger)hash
{
    return 100999;
}
```

覆写 hash 方法的时候需要注意，如果我们同一个类的对象返回同一个固定值，在 collection 中使用这种对象将产生性能问题。因为 collection 在检索哈希表时，会用对象的哈希码做索引。假如某个 collection 是用 set 实现的，那么 set 可能会根据哈希码把对象分装到不同的数组（后文也称箱子（bin））。在向 set 中添加新对象是，要根据其哈希码找到与之相关的那个数组，然后遍历其中元素，看数组中已有的对象是否和将要添加的新对象相等。所以如果set里面有100000那个类的对象时，需要将这100000个对象全部扫描一遍。

```
- (NSUInteger)hash
{
    NSString *stringToHash = [NSString stringWithFormat: @"%@:%@:%@", _firstName, _lastName, _age];
    return [stringToHash hash];
}
```

这种写法需要负担创建字符串的开销，所以比返回单一值要慢。把这种对象添加到 collection 中时，也会产生性能问题，因为想要添加，必须先计算其哈希码。

```
- (NSUInteger)hash
{
    NSUInteger firstNameHash = [_firstName hash];
    NSUInteger lastNameHash = [_lastName hash];
    NSUInteger ageHash = _age;
    return firstNameHash ^ lastNameHash ^ ageHash;
}
```

这种做法技能保持较高效率，又能使生成的哈希码在一定范围内，而不会过于频繁地重复。

#### 特定类具有的等同性判定方法

如果经常需要判断等同性，那么可能会自己来创建等同性判定方法，因为无须检测参数类型，所以能大大提升检测速度。

好处：

* 令代码看上去更美观、更易读，此动机与 NSString 类“isEqualToString:”方法创建缘由相似，纯粹为了装点门面。
* 不用检查两个受测对象的类型。

在编写判定方法时，也应一并覆写“isEqual:”方法。

```
- (BOOL)isEqualToPerson:(MINPerson *)otherPerson {
    if (self == otherPerson) {
        return YES;
    }
    if (![_firstName isEqualToString: otherPerson.firstName]) {
        return NO;
    }
    if (![_lastName isEqualToString: otherPerson.lastName]) {
        return NO;
    }
    if (_age != otherPerson.age) {
        return NO;
    }
    return YES;
}

- (BOOL)isEqual:(id)object
{
    if ([self class] == [object class]) {
        return [self isEqualToPerson:(MINPerson *)object];
    }else {
        return [super isEqual: object];
    }
}
```

**等同性判定的执行深度**

创建等同性判定方法时，需要决定是根据整个对象来判断等同性，还是仅根据其中几个字段判断。

我们可以根据自己的需求去决定怎样判定两个对象是否相同。

**容器中可变的等同性**

把某个对象放入 collection 之后，就不应该改变其哈希码了。collection 会把各个对象按照其哈希码分装到不同的“箱子数组”中。如果某对象放入“箱子”之后哈希码又变了，那么其现在所处的箱子对于他来说就是“错误”的。（第18条会解释为何要将对象做成不可变的）

```
    NSMutableArray *arrayA = [@[@1, @2] mutableCopy];
    [set addObject: arrayA];
    NSLog(@"set = %@", set);
    NSMutableArray *arrayB = [@[@1, @2] mutableCopy];
    [set addObject: arrayB];
    NSLog(@"set = %@", set);
    NSMutableArray *arrayC = [@[@1] mutableCopy];
    [set addObject: arrayC];
    NSLog(@"set = %@", set);
    [arrayC addObject: @2];
    NSLog(@"set = %@", set);
    NSSet *setB = [set copy];
    NSLog(@"setB = %@", setB);
```

打印

```
2018-05-15 16:19:28.135930+0800 Object Message[7876:302013] set = {(
        (
        1,
        2
    )
)}
2018-05-15 16:19:28.136085+0800 Object Message[7876:302013] set = {(
        (
        1,
        2
    )
)}
2018-05-15 16:19:28.136225+0800 Object Message[7876:302013] set = {(
        (
        1
    ),
        (
        1,
        2
    )
)}
2018-05-15 16:19:28.136349+0800 Object Message[7876:302013] set = {(
        (
        1,
        2
    ),
        (
        1,
        2
    )
)}
2018-05-15 16:19:28.136490+0800 Object Message[7876:302013] setB = {(
        (
        1,
        2
    )
)}
```

这里看出我们改变arrayC的内容，set的内容会被破坏，set本来就是存放不可重复的对象，这样存放的对象重复了，然后set复制后，又不重复了。改变 collection 中的对象可能会出现的问题是无法预测的。

**要点**

* 若想检测对象的等同性，请提供“isEqual:”与 hash 方法。
* 相同的对象必须具有相同的哈希码，但是两个哈希码相同的对象却未必相同。
* 不要盲目地逐个检测每条属性，而是应该依照具体需求来指定检测方案
* 编写 hash 方法时，应该使用计算速度快而且哈希码碰撞几率低的算法。

### 第9条：以“类族模式”隐藏实现细节

“类族”是一种可以隐藏“抽象基类“背后实现细节的模式。Objective-C 的系统框架中普遍使用此模式。比如 UIButton。

```
+ (instancetype)buttonWithType:(UIButtonType)buttonType;
```

这样绘制出来的button，用户不需要考虑按钮的绘制方式等实现细节。使用者只需要明白如果创建按钮，如果设置像”标题“这样的属性，如果增加触摸动作的目标对象等问题就好。

这种根据 type 切换绘制方法，如果 type 有很多，就会变得麻烦，我们可以将其绘制方法放到相关子类中去。使用”类族“模式就可以灵活应对多个类，将它们的实现细节隐藏在抽象基类后面，以保持接口简洁。用户无须创建子类实例，只需要调用基类方法来创建。

**创建类族**

```
typedef NS_ENUM(NSUInteger, MINEmployeeType) {
    MINEmployeeTypeDeveloper,
    MINEmployeeTypeDesigner,
    MINEmployeeTypeFinance,
};

@interface MINEmployee : NSObject
@property (nonatomic, copy) NSString *name;
@property (nonatomic, assign) NSUInteger salary;

+ (MINEmployee *)employeeWithType:(MINEmployeeType)type;
- (void)doADaysWork;
@end

#import "MINEmployeeFinance.h"
#import "MINEmployeeDesigner.h"
#import "MINEmployeeDeveloper.h"

@implementation MINEmployee
+ (MINEmployee *)employeeWithType:(MINEmployeeType)type
{
    switch (type) {
        case MINEmployeeTypeDeveloper:
            return [MINEmployeeDeveloper new];
            break;
        case MINEmployeeTypeDesigner:
            return [MINEmployeeDesigner new];
            break;
        case MINEmployeeTypeFinance:
            return [MINEmployeeFinance new];
        default:
            break;
    }
}
- (void)doADaysWork
{
    
}
@end
```

子类分别覆写 doADaysWork 方法。

Objective-C 这门语言没有办法指明某个基类是”抽象的“。于是，开发者通常会在文档中写明类的用法。这种情况下，基类接口一般没有名为 init 的成员方法，这暗示该类的实例也许不应该由用户直接创建。

**Cocoa 里的类族**

系统框架中有许多类族。大部分 collection 类都是某个类族的抽象基类。在传统的类族模式中，通常只有一个类具备”公共接口“，这个类就是类族中的抽象基类。

检查某对象是否位于类族中，不要直接检测两个”类对象“是否相同，而应该采用下列代码

```
    id someEmployee = [MINEmployeeFinance new];
    if ([someEmployee isKindOfClass: [MINEmployee class]]) {
        
    }
```

**isKindOfClass 和 isMemberOfClass的区别**

* isKindOfClass : 返回一个布尔值，该值能够判断出对象是否为某类或其派生类的实例。
* isMemberOfClass : 返回一个布尔值，该值能够判断出对象是否为某个特定类的实例。

**要点**

* 类族模式可以把实现细节隐藏在一套简单的公共接口后面。
* 系统框架中经常使用类族。（UIButton）
* 从类族的公共抽象基类中继承子类时要当心，若有开发文档，则应首先阅读。

### 第10条：在既有类中使用关联对象存放自定义数据

通过关联对象，向对象中存放一些我们需要的信息。

其实就是给某个对象增加一个对象属性。主要方法包括: 

* 设置关联对象
	
	```
	void objc_setAssociatedObject(id _Nonnull object, const void * _Nonnull key, id _Nullable value, objc_AssociationPolicy policy);
	object 被关联的对象
	key 关联对象的键，后面获取关联的对象就是用这个键
	value 关联对象，设置为nil，会根据键去清除那个键对应的关联对象
	police 关联对象的语义
	```
* 获取关联对象

	```
	id _Nullable objc_getAssociatedObject(id _Nonnull object, const void * _Nonnull key);
	object 被关联的对象
	key 关联对象的键，通过这个键去取对应的关联对象
	```
* 移除所有关联对象

	```
	void objc_removeAssociatedObjects(id _Nonnull object);
	object 被关联的对象
	```
	
其中 objc_AssociationPolicy 包括

* OBJC_ASSOCIATION_ASSIGN 对应属性的 assign
* OBJC_ASSOCIATION_RETAIN_NONATOMIC 对应属性的 nonatomic, retain
* OBJC_ASSOCIATION_COPY_NONATOMIC 对应属性的 copy, nonatomic
* OBJC_ASSOCIATION_RETAIN 对应属性的 retain
* OBJC_ASSOCIATION_COPY 对应属性的 copy

#### 简单使用

```
#import <objc/runtime.h>

// 关联对象

NSArray *array = [[NSArray alloc] initWithObjects: @"obj", nil];

// 设置被关联对象

NSObject *obj = [[NSObject alloc] init];
objc_setAssociatedObject(obj, @"NSObjectArrayKey", array, OBJC_ASSOCIATION_COPY);

// 获取关联对象

NSArray *getArr = objc_getAssociatedObject(obj, @"NSObjectArrayKey");
NSLog(@"%@", getArr);

// 移除被关联对象，下面两种方式都可以
   
objc_removeAssociatedObjects(obj);
//objc_setAssociatedObject(obj, @"NSObjectArrayKey", nil, OBJC_ASSOCIATION_COPY);
```
打印

```
2018-05-22 14:42:02.932239+0800 关联对象[8810:211450] (
    obj
)
```

**要点**

* 可以通过“关联对象”机制来把两个对象连起来。
* 定义关联对象时可指定内存管理语义，用以模仿定义属性时采用的“拥有关系”与“非拥有关系”。
* 只有在其他做法不可行时才应选用关联对象，因为这种做法通常会引入难于查找的 bug。

### 第11条：理解 objc_msgSend 的作用

#### 了解运行期

由于 Objective-C 是 C 的超集，所以最好先理解 C 语言的函数调用方式。C 语言使用“静态绑定”，也就是说，在编译期就能决定运行时所应调用的函数。以下面代码为例

```
#import <stdio.h>

void printHello() {
    printf("Hello");
}

void printGoodbye() {
    printf("Goodbye");
}

void doTheThing(int type) {
    if (type == 0) {
        printHello();
    }else {
        printGoodbye();
    }
}
```

上面的代码，如果不考虑“内联”（编译时，类似宏替换，使用函数体替换调用处的函数名，编译器对代码的优化），那么编译器在编译代码的时候就已经知道程序中有 printHello 和 printGoodbye 这两个函数了，于是会直接生成调用这些函数的指令。而函数地址实际上市硬编码在指令之中的。

```
#import <stdio.h>

void printHello() {
    printf("Hello");
}

void printGoodbye() {
    printf("Goodbye");
}

void doTheThing(int type) {
    void (*fnc)(void);
    if (type == 0) {
        fnc = printHello;
    }else {
        fnc = printGoodbye;
    }
    fnc();
}
```

这时候就得使用“动态绑定”了，因为所要调用的函数知道运行期才能确定。在第一个例子中，if 和 else 语句里都有函数调用指令，编译期可以直接硬编码地址，等调用的时候直接根据 type 调到指定函数体里，而在第二个例子中只有一个函数调用指令，待调用的函数地址无法硬编码在指令中，而是要在运行期读出来。

#### 了解 objc_msgSend

在 Objective-C 中，如果向某对象传递消息，那就会使用动态绑定机制来决定需要调用的方法。

给对象发消息一般是这样的形式

```
id returnValue = [someObject messageName: parameter];
```

上面例子中，someObject 叫做“接收者”，messageName 叫做 “选择子”（Selector）。选择子与参数合起来成为“消息”。

编译期看到此消息后，会将其转换为一条标准的 C 语言函数调用，所调用的函数是消息传递机制中的核心函数，叫做 objc_msgSend。

```
id _Nullable objc_msgSend(id _Nullable self, SEL _Nonnull op, ...)
```

这是个可变参数函数，能接收两个或两个以上的参数。第一个参数代表接收者，第二个参数代表选择子（SEL 是选择子的类型），后续参数就是消息中的那些参数，其顺序不变。

最终编译会把刚才那个例子中的消息转换为如下函数:

```
id returnValue = objc_msgSend(someObject, @selector(messageName:), parameter);
```

objc_msgSend 函数会依据接收者与选择子的类型来调用适当的方法。

其过程如下 :

1. 先在接收者所属的类中搜寻其“方法列表”，如果能找到与选择子名称相符的方法就跳至其实现代码。如果没有执行下一步。
2. 再沿着集成体系继续向上查找，等找到合适的方法之后再跳转。如果还是找不到，执行第三步。
3. 执行“消息派发”（message forwarding）操作，具体见第12条。

objc_msgSend 会将匹配的结果缓存在“快速映射表”里面，每个类都有这样一块缓存，若是后面还有相同的消息，就可以很快执行。但是这个还是达不到“静态绑定的函数调用操作”那样迅速。

**其他边界情况**，就是一些特殊情况。需要交给 Objective-C 运行环境中的另一些函数处理 :

* objc_msgSend_stret 如果待发送的消息要返回结构体，那么可以交给此函数处理。但是返回的结构体太大，会由另一个函数派发。那个函数会通过分配在栈上的某个变量来处理。
* objc_msgSend_fpret 如果消息返回的是浮点数，那么可以交给此函数处理。这个函数是为了处理 x85 等架构 CPU 中某些令人稍觉惊讶的奇怪状况。某些架构的 CPU 调用函数时需要对“浮点数寄存器”做特殊处理。
* objc_msgSendSuper 如果要给超类发消息，那么可以交给此函数处理。例如 [super message: parameter]。

Objective-C 对象的每个方法都可以视为简单的 C 函数，其原型如下 :

```
<return_type> Class_selector(id self, SEL _cmd, ...)
```

真正的函数名与上面写的可能不太一致。每个类中都有一张表格，其中的指针都会指向这种函数，而选择子的名称则是查表时所用的“键”。objc_msgSend 等函数正是通过这张表格来寻找应该执行的方法并跳至其实现的。

这个函数的原型跟 objc_msgSend 函数很像，是为了利用 “尾函数优化”技术，令“跳至方法实现”这一操作变得更简单。

如果某函数的最后一项操作是调用另外一个函数，那么就可以运用“尾函数优化”技术。编译器会生成调转至另一函数所需的指令码，而且不会向调用栈中推入新的“栈帧”。

只有当某函数的最后一个操作仅仅是调用其他函数而不会将其返回值另做他用是，才能执行“尾调用优化”。

这个优化对 objc_msgSend 非常关键，如果不这么做的话，那么会调用 Objective-C 方法之前，都需要为 objc_msgSend 函数准备“栈帧”，可以在“栈踪迹”中可以看到这种 “栈帧”。不优化可能导致“栈溢出”现象。

“尾函数优化”可以看下面两段代码

```
int fun(int n)
{
    if (n == 0) {
        return 1;
    }
    return fun(n-1) * n;
}
```

这个例子中函数返回值被用来做计算了，所以无法做尾递归优化，所以当 n 很大的时候，会栈溢出。

```
int fun1(int n, int prev, int next)
{
    if (n == 1) {
        return next;
    }
    return fun1(n-1, next, next + prev);
}
```

像这个符合“尾递归优化”的，就可以使用“尾递归优化”技术进行优化，保证“栈帧”。即使不做优化，它能够计算的 n 值也比上一个大一些，因为它所需的栈更少。

#### 简单使用 objc_msgSend

现在能够正常使用 objc_msgSend 函数，有两种方式 :

* ((void(*)(id, SEL, id))objc_msgSend)(obj, @selector(senMssage:), array);
* 选中项目 - Project - Build Settings - ENABLE_STRICT_OBJC_MSGSEND  将其设置为 NO 

推荐使用上面的方式调用。

```
#import <objc/message.h>

@interface MINObject : NSObject
- (void)senMssage:(NSArray *)arr;
@end

@implementation MINObject
- (void)senMssage:(NSArray *)arr
{
    NSLog(@"senMssage :%@", arr);
}
@end

- (void)userSendMsgSend
{
    NSArray *array = [[NSArray alloc] initWithObjects: @"obj", nil];
    MINObject *obj = [[MINObject alloc] init];
    SEL sel = @selector(senMssage:);
    ((void(*)(id, SEL, id))objc_msgSend)(obj, @selector(senMssage:), array);
    // objc_msgSend(obj, sel, array);
}

```

**要点**

* 消息由接收者、选择器及参数构成。给某对象“发送消息”也就相当于在该对象上“调用方法”。
* 发给某对象的全部消息都要由“动态消息派发系统”来处理，该系统会查出对应的方法，并执行其代码。

### 第12条：理解消息转发机制

当接收消息的对象无法响应消息的时候，就会启动“消息转发”机制，我们可以在此过程中告诉对象应该如何处理未知消息。其处理过程如下 :

![](/img/in-mpost/Effective-Objective-C/消息转发流程.png)

#### 动态方法解析（resolveInstanceUnkownMethod 和 resolveClassMethod）

对象在接收无法解读的消息后，首先将调用其所属类的下列类方法 : 

```
+ (BOOL)resolveInstanceMethod:(SEL)sel;
```

该方法的参数就是那个未知的选择子，其返回值为 Boolean 类型，表示这个类是否能够新增一个实例方法以处理此选择子。在继续往下执行转发机制之前，本类有机会新增一个处理此选择子的方法。如果未实现的不是实例方法而是类方法，运行期系统会调用另一个方法 :

```
+ (BOOL)resolveClassMethod:(SEL)sel;
```

**简单使用**

```
#import <Foundation/Foundation.h>

@interface MINObject : NSObject

@end

#import "MINObject.h"
#import <objc/runtime.h>

@implementation MINObject

- (void)resolveInstanceUnkownMethod:(NSString *)string
{
    NSLog(@"resolveInstanceUnkownMethod : %@", string);
}

void resolveClassUnknownCMethod(id self, SEL _cmd)
{
    NSLog(@"resolveClassUnknownMethod");
}

+ (void)resolveClassUnknownObMethod
{
    NSLog(@"resolveClassUnknownMethod");
}

+ (BOOL)resolveInstanceMethod:(SEL)sel
{
    NSString *selString = NSStringFromSelector(sel);
    if ([selString isEqualToString: @"sendInstanceUnknownMethod:"]) {
        class_addMethod([self class], sel, class_getMethodImplementation([self class],  @selector(resolveInstanceUnkownMethod:)), "v@:@");
        return YES;
    }
    return [super resolveClassMethod: sel];
}

+ (BOOL)resolveClassMethod:(SEL)sel
{
    NSString *selString = NSStringFromSelector(sel);
    if ([selString isEqualToString: @"sendClassUnknownMethod"]) {
        
//        class_addMethod(objc_getMetaClass("MINObject"), sel, (IMP)resolveClassUnknownCMethod, "v@:");
        
        class_addMethod(object_getClass(self) , sel, class_getMethodImplementation(object_getClass(self),  @selector(resolveClassUnknownObMethod)), "v@:@");
        return YES;
    }
    return [super resolveClassMethod: sel];
}

@end
```

使用

```
- (void)useResolveMethod
{
    [MINObject performSelector: @selector(sendClassUnknownMethod)];
    MINObject *obj = [[MINObject alloc] init];
    [obj performSelector: @selector(sendInstanceUnknownMethod:) withObject: @"hello"];
}
```

这里要提几个地方 :

* object_getClass : 这个获取的元对象的 Class，跟[self Class]不同，在 resolveClassMethod 方法中的添加方法，必须是添加类方法，所以必须获取这个。
* “v@:@” : class_addMethod 方法中的描述添加方法参数和返回值的字符数组。这里表示的返回 void 传入 (id, _cmd, id)。具体参考文档 [Type Encodings
](http://lbsyun.baidu.com/index.php?title=iossdk/sdkiosdev-download) 。
* 我们声明的类方法和实例方法默认是会有两个参数的 (id self, SEL _cmd)。

#### 备援接收者（forwardingTargetForSelector）

当前接收者如果不能处理这个选择子（SEL），运行期这个时候会向当前接收者发消息，问它能不能把这条消息转给其他接收者来处理。

```
- (id)forwardingTargetForSelector:(SEL)aSelector;
```

注意这里是实例方法，所以是类对象发的消息，不能被类对象内部处理，以及动态解析才会到这里。返回一个可以处理这个消息的对象。

**简单使用**

在上面的 MINObject 添加下面的方法

```
#import "RedundancyObject.h"

- (id)forwardingTargetForSelector:(SEL)aSelector
{
    NSString *selString = NSStringFromSelector(aSelector);
    if ([selString isEqualToString: @"OnlyRedundancyObjectCanProcess"]) {
        RedundancyObject *object = [[RedundancyObject alloc] init];
        return object;
    }
    return nil;
}
```

新增处理这个消息的类

```
#import <Foundation/Foundation.h>

@interface RedundancyObject : NSObject

@end

#import "RedundancyObject.h"

@implementation RedundancyObject

- (void)OnlyRedundancyObjectCanProcess
{
    NSLog(@"OnlyRedundancyObjectCanProcess");
}

@end
```

使用

```
- (void)useForwardTarget
{
    MINObject *obj = [[MINObject alloc] init];
    [obj performSelector: @selector(OnlyRedundancyObjectCanProcess)];
}

```
#### 完整消息转发（forwardInvocation 和 methodSignatureForSelector）

如果上面还没能处理消息，唯一能做的就是启用完整的消息转发机制了。

首先创建 NSInvocation 对象，把与尚未处理的那条消息有关的全部细节封于其中。此对象包含选择子、目标（target）及参数。

在触发 NSInvocation 对象时，“消息派发系统”（message-dispatch system）将亲自出马，把消息指派给目标对象。


```
- (void)forwardInvocation:(NSInvocation *)invocation;
```

这方法简单实现：只需要改变调用目标，使得消息在新目标上得以调用即可。但是这样实现跟“备援接收者”没啥区别。

有用的实现：在触发消息前，先以某种方式改变消息内容，比如追加一个参数，或是改变选择子，等等。

如果发现某调用操作不应由本类处理，则需调用超类的同名方法。这样继承体系中每个类都有机会处理此调用请求，直至 NSObject。如果 NSObject 还不能处理，就会调用 “doesNotRecognizeSelector:”以抛出异常。

```
- (void)forwardInvocation:(NSInvocation *)anInvocation
{
    NSLog(@"forwardInvocation");
    RedundancyObject *object = [[RedundancyObject alloc] init];
    [object performSelector: anInvocation.selector];
}

- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector
{
    if (aSelector == @selector(OnlyInvocationObjCanProcess)) {
        
        //        return [NSMethodSignature methodSignatureForSelector: aSelector];
        
        // 注意为啥不能用上面那个创建，因为上面那个方法本来就是找不到，才过来的，你还用它创建 NSMethodSignature， 能被创建出来才有鬼
        
        return [NSMethodSignature signatureWithObjCTypes: "v@:"];
    }
    return [super methodSignatureForSelector: aSelector];
}
```

在调用这个方法之前，我们需要重写 methodSignatureForSelector，返回一个 NSMethodSignature （方法签名，描述方法有哪些参数和返回值的），来协助消息转发，如果不实现这个方法，无法到达 forwardInvocation 方法里。

#### 简单了解一下 NSInvocation 和 NSMethodSignature

通过 NSInvocation 调用方法。

```
- (void)userInvocationDoSomeThing
{
    NSMethodSignature *methodSign = [self methodSignatureForSelector: @selector(doSomeThing:)];
    NSInvocation *invocation = [NSInvocation invocationWithMethodSignature:methodSign];
    [invocation setTarget: self];
    [invocation setSelector: @selector(doSomeThing:)];
    NSString *argument = @"doSomeThing";
    [invocation setArgument: &argument atIndex: 2];
    [invocation invoke];
}

- (void)doSomeThing:(NSString *)someThingString
{
    NSLog(@"%@", someThingString);
}
```

**要点**

* 若对象无法响应某个选择子，则进入消息转发流程。
* 通过运行期的动态方法解析功能，我们可以在需要用到某个方法时再将其加入类中。
* 对象可以把其无法解读的某些选择子转交给其他对象来处理。
* 经过上述两步之后，如果还是没办法处理选择子，那就启动完整的消息转发机制。
* 步骤越往后处理消息需要付出的代价会越来越大。

### 第13条：用“方法调配技术”调试“黑盒方法”

简单来说就是通过运行期方法交换两个方法。

类的方法列表会把选择子的名称映射到相关的方法实现之上，使得“动态消息派发系统”能够据此找到应该调用的方法。这些方法均以函数指针的形式来表示，这种指针叫做 IMP，其原型如下：

```
id (*IMP)(id, SEL, ...)
```

交换方法的方法

```
void method_exchangeImplementations(Method _Nonnull m1, Method _Nonnull m2) 
```

Method，在类定义中表示方法的不透明类型。

此函数的两个参数表示带交换的两个方法实现，可以通过下面的方法获得

```
Method _Nullable class_getInstanceMethod(Class _Nullable cls, SEL _Nonnull name)
```

注意上面的获取的是实例方法的实现，不要获取类方法的了。

简单使用，交换 NSString 的小写字体和大写字体两个方法。

```
- (void)exchangeMethod
{
    Method lowercaseStringMethod = class_getInstanceMethod([NSString class], @selector(lowercaseString));
    Method uppercaseStringMethod = class_getInstanceMethod([NSString class], @selector(uppercaseString));
    method_exchangeImplementations(lowercaseStringMethod, uppercaseStringMethod);
    NSString *testString = @"MFNothing";
    NSLog(@"lowercaseString : %@", [testString lowercaseString]);
    NSLog(@"uppercaseString : %@", [testString uppercaseString]);
}
```

打印

```
2018-05-28 11:06:47.472281+0800 方法调配[7631:218547] lowercaseString : MFNOTHING
2018-05-28 11:06:47.472430+0800 方法调配[7631:218547] uppercaseString : mfnothing
```

通常我们通过这一手段来为既有方法的实现增添新功能。比如想要在调用 lowercaseString 时记录某些信息，这是就可以通过交换方法来达成此目标。

新增分类，添加一个用于交换的方法。

```
#import <Foundation/Foundation.h>

@interface NSString (MINAdditions)

- (NSString *)min_myLowercaseString;

@end

#import "NSString+MINAdditions.h"

@implementation NSString (MINAdditions)

- (NSString *)min_myLowercaseString
{
    NSString *lowerString = [self min_myLowercaseString];
    NSLog(@"min_myLowercaseString : %@", lowerString);
    return lowerString;
}

@end
```

注意这里并不会出现死循环，当我们在使用的时候，调用 lowercaseString 方法的时候，进入这里，然后调用 min_myLowercaseString 方法的时候，进入的其实是原来 lowercaseString 的方法实现的地方，不是这里。

```

- (void)addSomeLogInLowercaseStringMethod
{
    Method lowercaseStringMethod = class_getInstanceMethod([NSString class], @selector(lowercaseString));
    Method min_myLowercaseStringMethod = class_getInstanceMethod([NSString class], @selector(min_myLowercaseString));
    method_exchangeImplementations(lowercaseStringMethod, min_myLowercaseStringMethod);
    NSString *testString = @"MFNothing";
    [testString lowercaseString];
}

---打印---

2018-05-28 11:23:20.936679+0800 方法调配[7959:232560] min_myLowercaseString : mfnothing
```

**要点**

* 在运行期，可以向类中新增或替换选择子所对应的方法实现。
* 使用另一份实现来替换原有的方法实现，这道工序叫做“方法调配”，开发者常用此项技术向原有实现中添加新功能。
* 一般来说，只有调试程序的时候才需要在运行期修改方法实现，这种做法不宜滥用。

### 第14条：理解“类对象”的用意

```
/// An opaque type that represents an Objective-C class.
typedef struct objc_class *Class;

/// Represents an instance of a class.
struct objc_object {
    Class _Nonnull isa  OBJC_ISA_AVAILABILITY;
};

/// A pointer to an instance of a class.
typedef struct objc_object *id;
```

从上面我们可以知道 id 是一个指向类实例的指针。在 Objective-C 中，id 可以指代任意的 Objective-C 对象类型。

objc_object 代表一个类实例。

Class 表示Objective-C类的不透明类型。

每个对象结构体（objc_object）的首个成员是 Class 类型的变量。该变量定义了对象所属的类，通常称为“is a”指针。

Class 在运行期程序库的头文件中的定义 ：

```
struct objc_class {
    Class _Nonnull isa  OBJC_ISA_AVAILABILITY;

#if !__OBJC2__
    Class _Nullable super_class                              OBJC2_UNAVAILABLE;
    const char * _Nonnull name                               OBJC2_UNAVAILABLE;
    long version                                             OBJC2_UNAVAILABLE;
    long info                                                OBJC2_UNAVAILABLE;
    long instance_size                                       OBJC2_UNAVAILABLE;
    struct objc_ivar_list * _Nullable ivars                  OBJC2_UNAVAILABLE;
    struct objc_method_list * _Nullable * _Nullable methodLists                    OBJC2_UNAVAILABLE;
    struct objc_cache * _Nonnull cache                       OBJC2_UNAVAILABLE;
    struct objc_protocol_list * _Nullable protocols          OBJC2_UNAVAILABLE;
#endif

} OBJC2_UNAVAILABLE;
```

此结构体存放类的“元数据”。首个变量也是 isa 指针，这说明 Class 本身亦为 Objective-C 对象。

* isa 指向类对象所属类型，这个累叫做“元类”，用来表述类对象本身所具备的元数据。比如类方法就定义于此。
* super_class 定义本类的超类。
* 每个类仅有一个“类对象”，而每个“类对象”仅有一个与之相关的“元类”。

![](/img/in-mpost/Effective-Objective-C/isa.png)

super_class 指针确立了继承关系，而 isa 指针描述了实例所属的类。

#### 在类继承体系中查询类型信息

前面介绍过下面两种方法来判断类是否相同。

* isKindOfClass : 返回一个布尔值，该值能够判断出对象是否为某类或其派生类的实例。
* isMemberOfClass : 返回一个布尔值，该值能够判断出对象是否为某个特定类的实例。

像这种类型信息查询的方法使用 isa 指针获取对象所属的类，然后通过 super_class 指针在继承体系中游走。



**要点**

* 每个实例都有一个指向 Class 对象的指针，用以表明其类型，而这些 Class 对象则构成了类的继承体系。
* 如果对象类型无法在编译期确定，那么就应该使用类型信息查询方法来探知。
* 尽量使用类型信息查询方法来确定对象类型，而不要直接比较类对象，因为某些对象可能实现消息转发功能。

## 接口与API设计
## 协议与分类

### 第23条：通过委托与数据源协议进行对象间通信

如果设置代理出现下面的错误，是因为协议没有从 NSObject 继承。

```
No known instance method for selector 'respondsToSelector:'
```

我们通常可以把委托对象能否响应某个协议方法这一信息缓存起来，以优化程序效率。

```
@class MINNetworkFetcher;
@protocol MINNetworkFetcherDelegate <NSObject>
@optional
- (void)networkFetcher:(MINNetworkFetcher *)fetcher
         didReciveData:(NSData *)data;
- (void)networkFetcher:(MINNetworkFetcher *)fetcher
         didFailWithError:(NSError *)error;
- (void)networkFetcher:(MINNetworkFetcher *)fetcher
         didUpdateProgressTo:(float)progress;
@end

@interface MINNetworkFetcher : NSObject
@property (nonatomic, weak) id<MINNetworkFetcherDelegate> delegate;
@end
```

```
@interface MINNetworkFetcher()
{
    struct {
        unsigned int didReciveData : 1;
        unsigned int didFailWithError : 1;
        unsigned int didUpdateProgressTo : 1;
    }_delegateFlags;
}
@end

@implementation MINNetworkFetcher

- (void)respondsDelegate
{
    if (_delegateFlags.didReciveData) {
        [self.delegate networkFetcher: self didReciveData: [NSData data]];
    }
    if (_delegateFlags.didFailWithError) {
        [self.delegate networkFetcher: self didFailWithError: [NSError errorWithDomain: @"error" code: 0 userInfo: nil]];
    }
    if (_delegateFlags.didUpdateProgressTo) {
        [self.delegate networkFetcher: self didUpdateProgressTo: 10];
    }
}

// 在设置代理的时候，去判断代理对象能不能响应协议中的所有方法

- (void)setDelegate:(id<MINNetworkFetcherDelegate>)delegate
{
    _delegate = delegate;
    _delegateFlags.didReciveData = [_delegate respondsToSelector:@selector(networkFetcher:didReciveData:)];
    _delegateFlags.didFailWithError = [_delegate respondsToSelector:@selector(networkFetcher:didFailWithError:)];
    _delegateFlags.didUpdateProgressTo = [_delegate respondsToSelector:@selector(networkFetcher:didUpdateProgressTo:)];
}
@end
```

### 第24条：将类的实现代码分散到便于管理的数个分类之中

优点：

* 对现有类进行扩展：比如：可以扩展Cocoa touch框架中的类，在类目中增加的方法会被子类继承，而且在运行时跟其他的方法没有区别。
* 作为子类的替代手段：不需要定义和使用一个子类，可以通过类目直接向已有的类里增加方法。
* 对类中的方法归类：利用category把一个庞大的类划分为小块来分别进行开发，从而更好地对类中的方法进行更新和维护。

缺点：

* 不能添加实例变量
* 类目中的方法会覆盖子类中名字相同的方法，它们拥有更高的优先级

**要点**

* 使用分类机制把类的实现代码划分成易于管理的小块
* 将应该视为“私有”的方法归入名为 Private 的分类中，以隐藏实现细节。

### 第25条：总是为第三方类的分类名称加前缀

分类机制通常用于向无源码的既有类中新增功能。将分类方法加入类中这一操作是在运行期系统加载分类时完成的。运行期系统会把分类中所实现的每个方法都加入类的方法列表。如果类中本来就有此方法，而分类又实现了一次，那么分类中的方法会覆盖原来那一份实现代码。

但是这样可能会产生多次覆盖，有两个分类都实现了这个方法，这时分类谁最晚加载，方法实现就是谁，所以会产生一些问题。

要解决此问题，一般的做法是：已命名空间来区别各个分类的名称与其中所定义的方法。在 Objective-C 中实现命名空间功能，只有一个方法，就是给相关名称都加上某个公用的前缀。

```
@interface NSString (MIN_HTTP)

// Encoade a string with URL encoding

- (NSString *)min_urlEncodedString;

@end
```

虽然两个分类重名，也不会出错。即便加了前缀，也难保其他分类不会覆盖你所写的方法，然而几率却小了很多，因为其他程序库很小会和你选用同一个前缀。

**要点**

* 向第三方类中添加分类时，总应给其名称加上你专用的前缀。
* 向第三方类中添加分类时，总应给其他的方法名加上你专用的前缀。

### 第26条：勿在分类中声明属性

分类是的目标是拓展类的功能，而非封装数据。

分类里面可以使用属性，但是由于分类无法合成与属性相关的实例变量，所以开发者需要在分类中为该属性实现存取方法。或者将属性的存取方法声明为 @dynamic。

**要点**

* 把封装数据所用的全部属性都定义在主接口。
* 在 “class-continuation”之外的其他分类中，可以定义存取方法，蛋尽量不要定义属性。

### 第二十七条：使用“class-continuation 分类” 隐藏实现细节


"class-continuation 分类" 和普通的分类不同，它必须定义在其所接续的那个类的实现文件里。

特点

* 是唯一能声明实例变量的分类，不是属性
* 没有特定的实现文件，其中声明的方法都应该定义在类的主实现文件里
* 没有分类名字

"class-continuation 分类" 就是.m 中写的
	
```
// 这个部分就是 class-continuation

@interface MINClass（）
{
// 块，实例变量声明的地方
	
}
@end
// 上面的部分是 class-continuation,下面是具体的实现

@implementation MINClass
{
// 括号里面的表示实现块，这里也可以声明实例变量,属性不能在块中声明

	NSString *justString;
}
@end
```
	
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

#include "SomeCppClass.h" 
// C++类

@interface EOCClass : NSObject
{
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

@interface EOCClass ()
{
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

### 第28条：通过协议提供匿名对象

```
@property (nonatomic, weak) id<MINNetworkFetcherDelegate> delegate;
```

该属性的类型是 id\<MINNetworkFetcherDelegate>，所以实际上任何类的对象都能充当这一属性，即便该类不继承自 NSObject 也可以，只要遵循 MINNetworkFetcherDelegate 协议就行。对于具备此属性的类来说，delegate 就是“匿名”的。

```
#import <Foundation/Foundation.h>

@protocol MINDataBaseConnection <NSObject>
- (void)connect;
- (void)disconnect;
- (void)isConnected;
- (NSArray *)performQuery:(NSString *)query;
@end

@interface MINDatabseManager : NSObject
+ (id)sharedInstance;
- (id<MINDataBaseConnection>)connectionWithIdentifier:(NSString *)indentifier;
@end
```

注意这个方法 connectionWithIdentifier: 方法。这个方法返回处理数据库连接的类，由于我们使用匿名对象，所以我们可以根据indentifier返回不同框架的类去交给调用者，让他们处理数据库操作。只是这些返回对象都是遵从 MINDataBaseConnection 协议的。

**要点**

* 协议可在某种程度上上提供匿名类型。具体的对象类型可以淡化成遵从某协议的 id 类型，协议里规定了对象所应实现的方法。
* 使用匿名对象来隐藏类型名称（或类名）。
* 如果具体类型不重要，重要的是对象能够响应（定义在协议里的）特定方法，那么可以使用匿名对象来表示。

## 内存管理

### 第29条：理解引用计数

在 MRC 中，我们可以通过NSObject 协议声明了的三个方法操作计数器，以递增或递减其值：

* retain 递增引用计数
* release 递减引用计数
* autorelease 等稍后清理autorelease pool的时候再递减引用计数

查看引用计数的方法叫 retainCount

![](/img/in-mpost/Effective-Objective-C/retainCount.png)

上图演示了对象自创建出来之后，经历一个“保留”，两次“释放”操作的过程。

对象在“解除分配”之后，只是放回“可用内存池”。如果执行NSLog时尚未覆写对象内存，那么该对象仍然有效，这时程序不会奔溃。

#### 自动释放池

调用autorelease，此方法会在稍后递减计数，通常是在下一次“事件循环”时递减，不过也可能执行得更早些。

此特性很有用，尤其是在方法中返回对象时更应该用它。

#### 保留环（循环引用）

对于循环中的每个对象来说，至少还有另外一个对象引用它，这将导致内存泄露。（这里泄露的意思是：没有正确释放已经不再使用的内存）

通常采用“弱引用”来解决此问题。

### 第30条：以ARC简化引用计数

使用ARC是，引用计数实际还是要执行的，只不过保留和释放操作现在是由ARC自动为你添加。

由于ARC会自动执行 retain、release、autorelease 等操作，所以直接在ARC下调用这些内存管理方法是非法的。(这里说的是调用，dealloc可以重写)

* retain
* release
* autorelease
* dealloc

#### 使用ARC时必须遵循的方法命名规则

方法名以下列词语开头，则其返回的对象归调用者所有：

* alloc
* new
* copy
* mutableCopy

归调用者所有的意思是：调用上述四种方法的那段代码要负责释放方法所返回的对象。

```
- (MINPerson *)newPerson
{
    MINPerson *newPerson = [[MINPerson alloc] init];
    return newPerson;
}

- (MINPerson *)somePerson
{
    MINPerson *person = [[MINPerson alloc] init];
    return person;
}

- (void)doSomething {
    MINPerson *personOne = [self newPerson];
    MINPerson *personTwo = [self somePerson];
    
    /* 当这段代码执行完后，ARC需要去清理它们。
       personOne归这段代码所有，所以它需要执行release
       而personTwo不归这个代码所有，所以它不需要执行release
    */
}
```

使用ARC还有其他好处，它可以执行一些手工操作很难甚至无法完成的优化。例如，在编译期，ARC会把能够相互抵消的retain、release和autorelease 操作约简。如果发现在同一个对象上执行了多次“保留”和“释放”操作，那么ARC有时会可以成对地移除这两个操作。

```
// _myperson 是一个强引用变量

_myPerson = [MINPerson personWithName:@"MIN"];
```

由于实例变量是一个强引用，所以编译器会在设置其值的时候执行一次保留操作。上面的代码下MRC中与下面的等效：

```
MINPerson *tmp = [MINPerson personWithName:@"MIN"];
_myPerson = [tmp retain];
```

此时应该可以看出，“personWithName:”方法里的 autorelease 与上段代码中的 retain 都是多余的。为提升性能，可将而这删去。但是ARC考虑到“向后兼容性”，以兼容那些不使用ARC的代码。

不过，ARC 可以运行期检测到这一对多余的操作，也就是 autorelease 及紧跟其后的 retain。

为了优化代码，在方法返回自动释放的对象时，要执行一个特殊函数。此时不直接调用对象的 autorelease 方法，而是改为调用 objc_autoreleaseReturnValue。此函数会检视方法返回之后将要执行的那段代码。若发现那段代码在返回的对象上执行 retain 操作，则设置全局数据结构（此数据结构的具体内容因处理器而异）中的一个标志位，而不执行 autorelease 操作。与之相似，如果方法返回了一个自动释放的对象，而调用方法的代码要保留这个对象，那么此时不直接执行retain，而是改为执行 objc_retainAutoreleaseReturnValue函数。此函数要检测刚才提到的标志位，如果设置了，则不执行 retain 操作。设置标志位，要比调用 autorelease 和 retain 更快。

#### 变量的内存管理语义

ARC 与会处理局部变量与实例变量的内存管理。默认情况下，每个变量都是指向对象的强引用。

在应用程序中，可用下列修饰符来改变局部变量与实例变量的语义：

* __strong: 默认语义，保留此值
* __unsafe_unretained: 不保留此值，这么做可能不安全，因为等到再次使用变量时，其对象已经被回收了，而变量还指向那块内存
* __weak: 不保留此值，但是变量可以安全使用，因为如果系统把这个对象回收了，那么变量也会指向nil
* __autoreleasing: 把对象“按引用传递”给方法时，使用这个特殊的修饰符。此值在方法返回时自动释放。

我们经常会给局部变量加上修饰符，用以打破由“块”所引入的“循环引用”。块会自动保留其所捕获的全部对象（参考第40条），而如果这其中有某个对象又保留块本身，那么就可能导致“循环引用”。可以通过 __weak 来打破。

```
NSURL *url = [NSURL URLWithString:@"MFNothing.cn"];
MINNetworkManager *networkManager = [[MINNetworkManager alloc] initWithURL: url];
MINNetworkManager *__weak weakNetworkManager = networkManager;
[networkManager startWithCompletion:^(BOOL success){
	NSLog(@"Finish from %@", weakNetworkManager.url);
}];
```

#### ARC如何清理实例变量

在手动管理时，凡是具备强引用的变量，都必须释放，我们可能会编写类似下面的代码。

```
-  (void)dealloc{
	[_foo release];
	[_bar release];
	[super dealloc];
}
```

ARC会在dealloc 方法中插入这些代码。而不需要手动编写 dealloc 方法了，因为ARC会借用 Objective-C++的一项特性来生成清理例程。回收Objective-C++对象时，待回收的对象会调用所有C++对象的析构函数。编译器如果发现某个对象含有C++对象，就会生成名为.cxx_destruct 的方法。而ARC则借助此特性，在该方法中生成清理内存所需的代码。

不过，如果有非Objective-C 的对象，比如CoreFoundation 中的对象或是由malloc()分配在堆中内存，那么仍然需要清理。ARC 环境下，dealloc 方法可以像这样写：

```
- (void)dealloc {
	CFRelease(_coreFoundationObject);
	free(_heapAllocatedMemoryBlob);
}
```

**要点**

* ARC 只负责管理Objective-C对象的内存。尤其注意：CoreFoundation对象不归ARC管理，开发者必须适时调用 CFRetain/CFRelease。
* ARC 管理对象生命周期的办法基本上就是：在合适的地方插入“保留”及“释放”操作。在ARC 环境下，变量的内存管理语义可以通过修饰符指明，而原来则需要手工执行“保留”及“释放”操作。

### 第31条：在 dealloc 方法中只释放引用并解除监听

对象在经历起生命期后，最终会被系统所回收，这时就要执行dealloc 方法了。在每个对象的生命期内，此方法仅执行一次。系统会在适当的时候调用它，所以我们不应该去自己调用的dealloc 方法。而且一旦调用过dealloc 之后，对象就不在有效，后续方法调用均是无效的。

dealloc 方法主要就是释放对象所拥有的引用，也就是把所有 Objective-C 对象都释放掉，ARC 会通过自动生成的 .cxx_destruct 方法，在 dealloc 中为你自动添加这些释放代码。对象所拥有的其他非 Objective-C 对象也需要释放，但是需要我们自己去操作。

在 dealloc 方法中，通常还需要把原来配置过的观测行为都清理掉。

```
- (void)dealloc {
	CFRelease(coreFoundationObject);
	[[NSNotificationCenter defaultCenter] removeObserver: self];
}
```

这里如果不使用ARC的话，在最后还需要调用 "[super dealloc];"和释放所有持有的对象。

dealloc 方法不应该去释放开销较大或系统内稀缺的资源，例如文件描述符、套接字、大块内存等。通常做法，实现另外一个方法，当应用程序用完资源对象后，就调用此方法。

像某对象管理着连接服务器用的套接字。

```
#import <Foundation/Foundation.h>

@interface MINServerConnection: NSObject
- (void)open:(NSString *)address;
- (void)close;
```

在清理方法而不是 dealloc 方法中清理资源还有一个原因，就是系统并不保证每个创建对象的对象的 dealloc 都会执行。极个别情况下，当应用程序终止是，仍有对象处于存活状态，这些对象没有收到 dealloc 消息。由于应用程序终止之后，其占用的资源也会返回操作系统，所以实际上这些对象也就等于消亡了。不调用 dealloc 方法是为了优化程序效率。

在 MAC OS X 及 iOS 应用程序所对应的 application delegate 中，都含有一个会于程序终止时调用的方法。如果一定要清理某些对象，那么可在此方法中调用对象的 “清理方法”。

在 MAC OS X 系统里，应用程序终止是会调用 NSApplicationDelegate 之中的下述方法：

```
- (void)applicationWillTerminate:(NSNotification *)notification
```
而在 iOS 系统中，应用程序终止是会调用 UIApplicationDelegate 之中的下述方法：

```
- (void)applicationWillTerminate:(UIApplication *)application
```

在 dealloc 中为了避免开发者忘记调用“清理方法”，也要调用“清理方法”。

```
- (void)close {
	/*clean up resources*/
	_closed = YES;
}

- (void)dealloc {
	if (!_closed) {
		NSLog(@"ERROR: close was not called before dealloc");
		[self close];
	}
}
```

在编写 dealloc 方法时还需注意，不要在里面随便调用其他方法。前面是为了查错调用。如果调用的方法还需要异步需要执行某些任务，可能当任务回调时，对象已经被摧毁了，这时会导致许多问题，且经常使程序崩溃。

在 dealloc 里也不要调用属性的存取方法，因为有人可能会覆写这些方法，并于其中做一些无法在回收阶段安全执行的操作。此外，属性可能正处于 KVO 机制的监控下，该属性的观察者可能会在属性值改变是 “保留” 或使用这个即将回收的对象。这种做法会令运行期系统的状态完全失调，从而导致一些莫名其妙的错误。

**要点**

* 在 dealloc 方法里，应该做的事情就是释放指向其他对象的引用，并取消原来订阅的“键值观测（KVO）”或 NSNotificationCenter 等通知，不要做其他事情。
* 如果对象持有文件描述符等系统资源，那么应该专门编写一个方法来释放这种资源。这样的类要和其使用者约定：用完资源后必须调用close方法。
* 执行异步任务的方法不应在 dealloc 里调用，只能在正常状态下执行的那些方法也不应该在 dealloc 里调用，因为此时对象已处于正在回收的状态了。

### 第32条：编写“异常安全代码”时留意内存管理问题

在当前运行期系统中，C++ 和 Objective-C 的异常相互兼容，也就是说，从其中一门语言抛出的异常能够用另外一门语言所编写的“异常处理代码”来捕获。纯C中没有异常。

Objective-C 的错误模型表明，异常只应该发生在严重错误后抛出。

当我们使用第三方库而此程序库抛出的异常又不受你控制时，就需要捕获及处理异常了。此外当用使用系统库时，有时也会出现异常，比如使用KVO时，注销一个尚未注册的“观察者”。

在 try 块中，如果先保留了某个对象，然后在释放他之前又抛出了异常，那么除非 catch 块能够处理此问题，否则对象所占内存就将泄露。C++ 的析构函数由 Objective-C 的异常处理例程来运行。 这对 C++ 对象很重要，由于抛出异常会缩短其生命期，所以发送异常时必须析构，不然就会泄露，而文件句柄等系统资源因为没有正确清理，所以就更容易因此而泄露了。

在 MRC 下管理

```
    NSObject *obj;
    @try {
        obj = [[NSObject alloc] init];
        [obj performSelector: @selector(doSomehingThatMayThrow) withObject: nil];
    }
    @catch (...){
        NSLog(@"something wrong");
    }
    @finally {
        [obj release];
    }
```
这里 @finally 保证在出现异常的时候回执行一次，但是也要引用obj，所以要将它移到 @try 块外面，像上面一样。

在 ARC 环境下，问题会更加严重，我们无法直接调用 release 操作，所以无法向 MRC 那样吧释放操作移到 @finally 块中。在默认情况ARC不会自动处理，因为这样做需要计入大量样板代码，以便追踪待清理的对象，从而在抛出异常时将其释放。这些代码会严重影响运行期的性能，即便不抛异常时也如此。并且会增加应用程序大小。

在 Objective-C++ 模式时，编译器会自动把 -fobjc-arc-exceptions 标志打开。

如果手动管理引用计数，而且必须捕获异常时，要设法保证所编写的代码能把对象正确清理干净。若使用 ARC 且必须捕获异常，则需打开编译器的 
 -fobjc-arc-exceptions 标志打开。

但最重要的的是：在发现大量异常捕获操作时，应该考虑重构代码。

**要点**

* 捕获异常是，一定要注意将 try 块内所创建的对象清理干净。
* 在默认情况下，ARC 不生成安全处理异常所需要的清理代码。开启编译器标志后(-fobjc-arc-excaptions 跟那个-fno-objc-arc设置位置应该一样)，可生成这种代码，不过会导致应用程序变大，而且会降低运行效率。


### 第33条：以弱引用避免循环引用

弱引用的方式

* unfase_unretained：用unfase_unretained 修饰的属性特质，其语义同 assign 特质等价。assign 通常只用于“原生类型”（int、float、结构体等），unsafe_unreturned 则多用于对象类型。
* weak：它与 unfase_unretained 的作用完全相同，然而，系统只要把属性回收，属性值就会自动设为nil

```
@property (nonatomic, unsafe_unretained) NSObject *unsafeObj;
@property (nonatomic, weak) NSObject *weakObj;
```


**要点**

* 将某些引用设为weak，可避免出现“循环引用”
* weak 引用可以自动清空，也可以不自动清空。自动清空是随着 ARC 而引入的新特性，由运行期系统来实现。在具备自动清空功能的弱引用上，可以随意读取数据，因为这种引用不会指向已经释放的对象。

### 第34条：以“自动释放块”降低内存峰值

释放对象有两种方式：

* 一种是调用 release 方法，使其保留计数立即递减；
* 另一种是调用 autorelease 方法，将其加入“自动释放池”中。自动释放池用于存放那些需要在稍后某个时刻释放的对象。清空（drain）自动释放池时，系统会向其中的对象发送 release 消息。

创建自动释放池的语法：

```
    @autoreleasepool {
        
    }
```

Mac OS X 与 iOS 应用程序分别运行于 Cocoa 及 Cocoa Touch 环境中。系统会自动穿甲一些默认有自动释放池的线程，例如主线程或是“大中枢派发”（Grand Central Dispatch，GCD）机制中的线程，每次执行“事件循环”时，就会将其清空。因此，不需要自己来创建“自动释放池块”。

通常只有一个地方需要创建自动释放池，那就是在 main 函数里，我们用自动释放池来包裹应用程序的主入口点。

```
int main(int argc, char * argv[]) {
    @autoreleasepool {
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}
```

这个池可以理解成最外围捕捉全部自动释放对象所用的池，如果注释掉这个自动释放池，那么由 UIApplicationMain 函数所自动释放的那些对象，就没有自动释放的池可以容纳了。虽然会末尾恰好就是应用程序的终止点，而此时操作系统会把程序所占用的全部内存都释放掉。

**自动释放池可以嵌套**

```
    @autoreleasepool {
        NSString *string = [NSString stringWithFormat: @"1 = %i", 1];
        @autoreleasepool {
            NSNumber *number = [NSNumber numberWithInt: 1];
        }
    }
```

string 放在外围的自动释放池中，而 number 则放在里层的自动释放池中。

自动释放池机制就像“栈”一样。系统创建好自动释放池之后，就将其推入栈中，而清空自动释放池，则相当于将其从栈中弹出。在对象上执行自动释放操作，就等于将其放入栈顶的那个池里。

利用自动释放池，降低应用程序在执行循环时的内存峰值

```
NSArray *databaseRecords = /* ... */;
NSMutableArray *people = [NSMutableArray new];
for (NSDictionary *record in databaseRecords) {
	@autoreleasepool {
		MINPerson *person = [[MINPerson alloc] initWithRecord: record];
		[people addObject: person];
	}
}
```

如果不加这个这个自动释放池，这些自动释放的对象会放在线程的主池里面。而它们本来应该被提早回收的。

尽管自动释放池的开销不太大，但毕竟还是有的，所以尽量不要建立额外的自动释放池。首先的监控内存用量，判断有没有需要解决的问题。

在ARC之前还有一个老式写法，使用 NSAutoreleasePool 对象（只能在MRC中使用）。

```
NSArray *databaseRecords = /* ... */;
NSMutableArray *people = [NSMutableArray new];
int i = 0;

NSAutoreleasePool *pool = [[NSAutoreleasePool alloc] init];
for (NSDictionary *record in databaseRecords) {
	MINPerson *person = [[MINPerson alloc] initWithRecord: record];
	[people addObject: person];
	if (++i == 10) {
		[pool drain];
		i = 0;
	}
}
```

相比之下 @autoreleasepool 更方便使用（在作用域外不可使用），NSAutoreleasePool还需要指定在什么时候释放（容易出现提前释放）。

**要点**

* 自动释放池排布在栈中，对象收到 autorelease 消息后，系统将其放入最顶端的池里。
* 合理运用自动释放池，可降低应用程序的内存峰值。
* @autoreleasepool 这种新式写法能创建出更为轻便的自动释放池。

### 第35条：用“僵尸对象”调试内存管理问题

程序在运行的时候，偶尔崩溃，可能是我们向已回收的对象发送消息。奔溃与否完全取决于对象所占内存有没有为其他的内存所覆写。

启用“僵尸对象”这个功能之后，运行期系统会把所有已经回收的实例转化为特殊的“僵尸对象”，而不会真正回收。这种对象所在的核心内存无法重用，因此不可能遭到覆写。僵尸对象收到消息（即执行方法之后）会抛出异常，其中准确说明了发送过来的消息，并描述了回收之前的那个对象。僵尸对象是调试内存管理问题的最佳方式。

从上面的那段话可以看出这个“僵尸对象”的功能可以保证我们不能使用被回收的对象所在的那块内存，并且使用回收后的对象的方法会抛出异常。

通过在 Xcode -> Product -> Scheme -> Edit Scheme -> Diagnostics -> Memory Management -> Zombie Objects 开启功能。

在系统即将回收对象时，如果发现通过环境变量启动了僵尸对象功能，那么将执行一个附加步骤，把对象转化为僵尸对象，而不彻底回收。

僵尸类是从名为 \_NSZombie_ 的模板类复制出来的。

整个转换过程，就是在 NSObject 的 dealloc 方法中，运气期系统如果发现 NSZombieEnabled 环境变量已经设置了（即启用了“僵尸对象”功能），就会 “调配”（swizzle，用替换感觉更好，参加第13条）成一段将 OriginailClass 变为 \_NSZombie_OriginailClass 的代码（OriginailClass代指被回收对象所属的类）。

这里保证了 dealloc 并没有释放掉被回收的对象，虽说内存泄露了，但是这只是调试。制作正式发布的应用程序的时不会把这项功能打开，也就不会存在内存泄露了。

\_NSZombie_ 类（以及所有从该类拷贝出来的类）并未实现任何方法。此类没有超类，因此和 NSObject 一样，也是个“根类”，该类只有一个实例变量，叫做 isa，所有 Objective-C 根类都必须有这个变量。由于这个僵尸类没有实现任何方法，所以发给它的消息都要走“完整的消息转发机制”（参见第12条）。

在完整的消息转发机制中，___forwarding___ 是核心，它首先要做的事情包括检查该接受消息的对象所属的类。若名称的前缀为 \_NSZombie_，则表明消息接受对象是僵尸对象，需要特殊处理。会打印一条消息，僵尸对象所收到的消息及原来的类，然后程序终止。


使用 MRC 来执行下面的代码。

```
@interface MINClass : NSObject
- (void)logSomeThing;
@end
@implementation MINClass
- (void)logSomeThing
{
    NSLog(@"self = %@", self);
}
@end

#import <objc/runtime.h>

- (void)testObj
{
    MINClass * obj = nil;
    obj = [[MINClass alloc] init];
    NSLog(@"Brfore release:");
    [obj logSomeThing];
    [self printClassInfo: obj];
    [obj release];
    NSLog(@"After release:");
    [self printClassInfo: obj];
    [obj logSomeThing];
}

- (void)printClassInfo:(id)obj
{
    Class cls = object_getClass(obj);
    Class superClass = class_getSuperclass(cls);
    NSLog(@"=== %s : %s ===", class_getName(cls), class_getName(superClass));
}
```

**要点**

* 系统在回收对象时，可以不将其真的回收，而是把它转化为僵尸对象。通过环境变量 NSZoobieEnabled 可开启这个功能。
* 系统会修改对象的 isa 指针，令其指向特殊的僵尸类（指向一个新创建的拷贝僵尸类的类），从而使该对象变为僵尸对象。僵尸类能够响应所有的选择子，响应方式为：打印一条包含消息内容及其接受者的消息，然后终止应用程序。

### 第36条：不要使用 retainCount

retainCount 方法所返回的保留计数只是某个给定时间点的值，但是该方法并未考虑到系统会稍后把自动释放池清空，因而不会将后续的释放操作从返回值中减去。

所以如果我们用此时的 retainCount 返回的值去判断是否释放该对象，会出问题。

```
while ([Object retainCount]) {
	[Object release];
}
```

上面的写法为考虑到自动释放操作，只是不停的通过释放操作降低保留计数，直至对象对系统所回收。假如此对象也在自动释放池里，那么稍后系统清空池子的时候还要把它再释放一次，而这将导致程序崩溃。

第二个错误在于：retainCount 可能永远不会返回0，因为有时系统会优化对象的释放操作，在保留计数还是1的时候就把它回收了。这样while就死循环了。

```
- (void)testRetainCount
{
    NSString *string = @"string";
    NSLog(@"string retainCount = %lu", [string retainCount]);
    
    NSNumber *numberI = @1;
    NSLog(@"numberI retainCount = %lu", [numberI retainCount]);
    
    NSNumber *numberF = @3.14;
    NSLog(@"numberF retainCount = %lu", [numberF retainCount]);
}
```

```
2018-05-08 10:08:27.542838+0800 memory management[2035:69502] string retainCount = 18446744073709551615
2018-05-08 10:08:27.542954+0800 memory management[2035:69502] numberI retainCount = 9223372036854775807
2018-05-08 10:08:27.543044+0800 memory management[2035:69502] numberF retainCount = 1
```

第一个对象保留计数是 2^64 - 1，第二个对象保留计数是 2^63 - 1。由于二者皆为“单例对象”，所以其保留计数都很大。系统会尽可能把 NSString 实现成单例对象。如果字符串像上面那种形式，是一个编译器常量，编译器会把 NSString 对象所表示的数据放到应用程序的二进制文件里，运行程序时就可以直接用，无需创建 NSString 对象了。

NSNumber 类似，使用了一种叫做“标签指针”（tagged pointer）的概念来标注特定类型的数值（这里一般是整形）。这种做法不使用 NSNumber 对象，而是把与数值有关的全部信息都放在指针值里面。运行期系统会在消息派发期间检测这种标签指针，并对它执行响应操作，使其行为看起来和真正的 NSNumber 对象一样。

另外，对于这种对象的保留及释放操作都是“空操作”（no-op）。

**要点**

* 对象的保留计数看似有用，实则不然，因为任何给定时间点的“绝对保留计数”都无法反映对象的生命期的全貌。
* 引入 ARC 之后，retainCount 方法就正式废止了，在 ARC 下调用该方法会导致编译器错误。

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
    NSLog(@"%@", block);  // <__NSMallocBlock__: 0x600000249db0>
    
    self.aCopyBlock = block;
    self.aAnotherBlock = block;
    NSLog(@"%@ %@", self.aCopyBlock, self.aAnotherBlock);  // <__NSMallocBlock__: 0x600000249db0> <__NSMallocBlock__: 0x600000249db0>

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
NSLog(@"%@", ^{a = 10;}); 
// <__NSStackBlock__: 0x7ffeef751a70>
block = ^{NSLog(@"%d", a);};
NSLog(@"%@", block); 
// <__NSMallocBlock__: 0x6000000559f0>
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
    NSLog(@"%@", block);
    // <__NSMallocBlock__: 0x60c000050e60>
    
    self.aCopyBlock = block;
    self.aAnotherBlock = [block copy];
    NSLog(@"%@ %@", self.aCopyBlock, self.aAnotherBlock); 
    // <__NSMallocBlock__: 0x60c000050e60> <__NSMallocBlock__: 0x60c000050e60>

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


### 第38条：为常用的块类型创建 typedef

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

在创建对象时，可以使用内联的 handler 块将相关业务逻辑一并声明。

在有多个实例需要监控时，如果采用委托模式，那么经常需要根据传入的对象来切换（增加代码阅读难度），而改用 handler 块来实现，则可直接将块与相关对象放在一起。

使用代理的方式

```
// 类

#import <Foundation/Foundation.h>

@class MINNetWorkFetcherUsingDelegate;

@protocol MINNetWorkFetcherDelegate <NSObject>
- (void)networkFetcher:(MINNetWorkFetcherUsingDelegate *)networkFetcher
     didFinishWithData:(NSData *)data
                 error:(NSError *)error;
@end

@interface MINNetWorkFetcherUsingDelegate : NSObject
@property (nonatomic, weak) id<MINNetWorkFetcherDelegate> delegate;
- (instancetype)initWithUrl:(NSURL *)url;
- (void)start;
@end

// 使用

- (void)fetchFooData
{
    NSURL *url = [NSURL URLWithString: @"http://url.com/foo.data"];
    _fooFetcher = [[MINNetWorkFetcherUsingDelegate alloc] initWithUrl: url];
    _fooFetcher.delegate = self;
    [_fooFetcher start];
}

- (void)fetchBarData
{
    NSURL *url = [NSURL URLWithString: @"http://url.com/bar.data"];
    _barFetcher = [[MINNetWorkFetcherUsingDelegate alloc] initWithUrl: url];
    _barFetcher.delegate = self;
    [_barFetcher start];
}

#pragma mark - MINNetWorkFetcherDelegate

- (void)networkFetcher:(MINNetWorkFetcherUsingDelegate *)networkFetcher didFinishWithData:(NSData *)data error:(NSError *)error
{
    if (networkFetcher == _fooFetcher) {
        
    }else if (networkFetcher == _barFetcher) {
        
    }
}
```

使用 handler 的方式

```
// 类

#import <Foundation/Foundation.h>

typedef void (^MINNetworkFetcherCompletionHandler)(NSData *data, NSError *error);

@interface MINNetworkFetcherUsingBlock : NSObject
- (instancetype)initWithUrl:(NSURL *)url;
- (void)startWithCompletionHandler:(MINNetworkFetcherCompletionHandler) handler;
@end

- (void)fetchFooData
{
    NSURL *url = [NSURL URLWithString: @"http://url.com/foo.data"];
    MINNetworkFetcherUsingBlock *fooFetcher = [[MINNetworkFetcherUsingBlock alloc] initWithUrl: url];
    [fooFetcher startWithCompletionHandler:^(NSData *data, NSError *error) {
        if (error) {
            
        }else {
            
        }
    }];
}

- (void)fetchBarData
{
    NSURL *url = [NSURL URLWithString: @"http://url.com/bar.data"];
    MINNetworkFetcherUsingBlock *barFetcher = [[MINNetworkFetcherUsingBlock alloc] initWithUrl: url];
    [barFetcher startWithCompletionHandler:^(NSData *data, NSError *error) {
        if (error) {
            
        }else {
            
        }
    }];
}
```

设计API时如果用到了 handler 块，那么可以增加一个参数，使调用者可通过此参数来决定应该把块安排在哪个队列上执行。

```
// 这个可以参考通知的方式，默认不传 queue，则在默认线程执行。

- (id <NSObject>)addObserverForName:(nullable NSNotificationName)name
                             object:(nullable id)obj
                              queue:(nullable NSOperationQueue *)queue
                         usingBlock:(void (^)(NSNotification *note))block
                         
```

### 第40条：用块引用其所属对象时不要出现保留环（循环引用）

如果块所捕获的对象直接或间接的保留了块本身，那么就得当心保留环的问题。

一定要找个适当的时机解除保留环，而不能把责任推给API调用者。即不要让使用块里面去做操作。

```
#import <Foundation/Foundation.h>

typedef void (^MINNetworkFetcherCompletionHandler)(NSData *data, NSError *error);

@interface MINNetworkFetcherUsingBlock : NSObject
@property (nonatomic, strong, readonly) NSURL *url;
- (instancetype)initWithUrl:(NSURL *)url;
- (void)startWithCompletionHandler:(MINNetworkFetcherCompletionHandler) handler;
@end

#import "MINNetworkFetcherUsingBlock.h"

@interface MINNetworkFetcherUsingBlock ()
@property (nonatomic, strong, readwrite) NSURL *url;
@property (nonatomic, copy) MINNetworkFetcherCompletionHandler completionHandler;
@property (nonatomic, strong) NSData *downloadData;
@end

@implementation MINNetworkFetcherUsingBlock
- (instancetype)initWithUrl:(NSURL *)url
{
    if (self = [super init]) {
        _url = url;
    }
    return self;
}

- (void)startWithCompletionHandler:(MINNetworkFetcherCompletionHandler)handler
{
    self.completionHandler = handler;
    // Start the request
    // Request sets downloadedData property
    // When request is finished, p_requestCompleted is called
}

- (void)p_requestCompleted {
    if (_completionHandler) {
        _completionHandler(_downloadData, nil);
    }
    // 这里将放弃持有block，避免循环引用，最重要的地方
    self.completionHandler = nil;
}
@end
```

### 第41条：多用派发队列，少用同步锁

这里先简单了解一下队列的相关知识，如果了解，可以跳过这段。

**GCD(Grand central Dispacth)**

调控线程，是一个block

将任务(操作)放入队列

任务：

* 同步(sync),阻塞当前线程并等待block中的任务执行完毕(即等我执行完，你再执行)，在执行（这个只是阻塞当前线程，所以如果是并行队列，可能多个任务会在多个线程中执行，排在同一个线程下的任务才会被阻塞）
* 异步(async),不阻塞当前线程，当前线程继续进行(不用等我，你们走你们的)

队列(存放任务的地方)：

* 串行队列 : 从队列中依次取出，根据取出顺序依次执行(dispatch_get_main_queue)，所以这个不会去管任务是同步还是异步的，要等这个任务执行完了，才执行下一个。
* 并行队列 : 从队列中依次取出，放入其他线程一起执行(dispatch_get_global_queue)

队列的创建方式：

* dispatch_queue_t queue = dispatch_queue_create("name", DISPATCH_QUEUE_SERIAL); // 串行
* dispatch_queue_t queue = dispatch_queue_create("name", DISPATCH_QUEUE_CONCURRENT); // 并行
* dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT,0);
* dispatch_get_main_queue();

主线程异步执行

```
dispatch_async(dispatch_get_main_queue(), ^{...});
```

主线程同步执行

```
dispatch_sync(dispathc_get_main_queue(), ^{...});
```

**栅拦块**，必须单独执行，不能与其他块并行，并行队列会等所有块执行完后才会执行这个块。(注意这里queue的)

```
* void dispatch_barrier_async(dispatch_queue_t queue, dispatch_block_t block);
* void dispatch_barrier_sync(dispatch_queue_t queue, DISPATCH_NOESCAPE dispatch_block_t block);
```

共同点：

1. 等待在它前面插入队列的任务先执行完
2. 等待他们自己的任务执行完再执行后面的任务

不同点：

1. dispatch_barrier_sync将自己的任务插入到队列的时候，需要等待自己的任务结束之后才会继续插入被写在它后面的任务，然后执行它们
2. dispatch_barrier_async将自己的任务插入到队列之后，不会等待自己的任务结束，它会继续把后面的任务插入到队列，然后等待自己的任务结束后才执行后面任务。

```
- (void)createBarrierQueue
{
    dispatch_queue_t globalQueue = dispatch_queue_create( "concurrent_queue", DISPATCH_QUEUE_CONCURRENT);
    //dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    for (int i = 0; i < 10; i++) {
        dispatch_sync( globalQueue, ^{
            NSLog(@"*****%d", i);
        });
    }
    dispatch_barrier_sync(globalQueue, ^{
        NSLog(@"barrier_queue");
    }); // 这个会等到前面执行完再去把后面的任务块加入队列，所以 11111会在后面才打印
//    dispatch_barrier_async(globalQueue, ^{
//        NSLog(@"barrier_queue");
//    }); // 这个不会等到前面执行完再去把后面的任务块加入队列，所以 11111会在前面就打印
    NSLog(@"11111");
    for (int i = 0; i < 10; i++) {
        dispatch_sync( globalQueue, ^{
            NSLog(@"------%d", i);
        });
    }
}
```

**要点**

1. 使用GCD来表述同步语义，这种做法要比使用@synchronized块或NSLock对象更简单。
2. 将同步和异步派发结合起来，可以实现与普通加锁机制一样的同步行为，而这么做不会阻塞异步派发的线程。
3. 使用同步队列及栅拦块，可以令同步行为更加高效。

### 第42条：多用 GCD，少用 performSelector 系列方法

当通过下面方式去使用 performSelector 时，会发生警告

* PerformSelector may cause a leak because its selector is unknown

编译器由于不知道方法名是否有返回值，所以就没办法运用ARC的内存管理规则来判定返回值是不是应该释放。鉴于此，ARC采取了比较谨慎的做法，就是不添加释放操作。然而这么做可能会导致内存泄露，因为方法在返回对象时可能已经将其保留了。（一般返回对象的时候会对它进行一个Autorelease操作，当没有调用它时会自动释放，但是这里就没有可能就不会去做这个操作了）

```
- (void)createPerformSelectorWithIndex:(NSInteger)index
{
    SEL selector;
    if (index == 0) {
        selector = @selector(newObject);
    }else if (index == 1) {
        selector = @selector(copyObject);
    }else {
        selector = @selector(logObject);
    }
    id ret = [self performSelector: selector];
}
```

这个ret对象在前面两个有返回值的情况下，ret 对象应由这段代码来释放。

performSelector系列方法能处理的选择太过于局限，选择的返回类型及发送给方法的参数个数都受到限制。

如果想把任务放到另外一个线程上执行，那么最好不要用performSelector系列方法，而是应该把任务封装到块里，然后用GCD的相关方式来实现。

例如，延后执行某项任务

```
- (void)delayDoSomeThing
{
    // 使用 performSelector
    [self performSelector: @selector(doSomeThing) withObject: nil afterDelay: 5.0];
    
    // 使用 GCD
    dispatch_time_t time = dispatch_time(DISPATCH_TIME_NOW, (int64_t)(5.0 * NSEC_PER_SEC));
    dispatch_after(time, dispatch_get_main_queue(), ^{
        [self doSomeThing];
    });
}
```

简单了解一些 NSThread

**NSThread**

```
--- 初始化方式 ---
1.动态方法
- (id)initWithTarget:(id)target selector:(SEL)selector object:(id)argument;  

// 初始化线程  
NSThread *thread = [[NSThread alloc] initWithTarget:self selector:@selector(run) object:nil];  
// 设置线程的优先级(0.0 - 1.0，1.0最高级)  
thread.threadPriority = 1;  
// 开启线程  
[thread start];  

/*
selector ：线程执行的方法，这个selector最多只能接收一个参数
target ：selector消息发送的对象
argument : 传给selector的唯一参数，也可以是nil
*/

2.静态方法
+ (void)detachNewThreadSelector:(SEL)selector toTarget:(id)target withObject:(id)argument; 

[NSThread detachNewThreadSelector:@selector(run) toTarget:self withObject:nil];  
// 调用完毕后，会马上创建并开启新线程  

3.隐式创建线程的方法
[self performSelectorInBackground:@selector(run) withObject:nil];
```
```

--- 线程操作 ---

1. 获取当前线程
NSThread *current = [NSThread currentThread]; 

2. 获取主线程
NSThread *main = [NSThread mainThread];  

3. 暂停当前线程
// 暂停2s  
[NSThread sleepForTimeInterval:2];  
// 或者  
NSDate *date = [NSDate dateWithTimeInterval:2 sinceDate:[NSDate date]];  
[NSThread sleepUntilDate:date];  
```
```
--- 线程间的通信 ---

1. 在指定线程上执行操作
[self performSelector:@selector(run) onThread:thread withObject:nil waitUntilDone:YES];  

2. 在主线程上执行操作
[self performSelectorOnMainThread:@selector(run) withObject:nil waitUntilDone:YES]; 

3. 在当前线程执行操作
[self performSelector:@selector(run) withObject:nil];
```

```
--- 优缺点 ---

1. 优点：NSThread比其他两种多线程方案较轻量级，更直观地控制线程对象

2. 缺点：需要自己管理线程的生命周期，线程同步。线程同步对数据的加锁会有一定的系统开销
```


### 第43条：掌握 GCD 及操作队列的使用时机

在解决多线程与任务管理问题时，派发队列并非唯一方案。

操作队列（NSOperation）提供了一套高层的 OBjective—C API，能实现纯 GCD 所具备的绝大部分功能，而且还能完成一些更为复杂的操作，那些操作若改用 GCD 来写，则需另外编写代码。

使用 NSOperation 及 NSOperationQueue 的好处

* 取消某个操作。在 NSOperation 对象上调用 cancel 方法，该方法会设置对象内的标志位，用以表示此任务不需执行，不过，已启动的任务无法取消。
* 指定操作间的依赖关系。一个操作可以依赖其他多个操作。开发者能够指定操作之间的依赖关系，使特定的操作必须在另一个操作顺利执行完毕后方可执行。
* 可以通过 KVO 监控 NSOperation 对象的属性。通过 isCanclled 属性判断任务是否取消，通过 isFinished 属性来判断任务是否已经完成。
* 指定操作的优先级。（@property NSOperationQueuePriority queuePriority 这个属性）
* 重用 NSOperation 对象。可以通过设计类中的方法，达到创建一个可以复用的类，让这里 NSOperation 类可以再代码中多次使用。

简单了解 NSOperation 

**NSOperation/NSOperationQueue**


```
1.概述：
NSOperation的作用是实现多线程编程。
NSOperation与NSOperationQueue实现多线程编程的基本步骤为：
（1）先将一个需要的操作封到NSOperation中。
（2）将NSOperation添加到NSOperationQueue中。
（3）系统自动将NSOperationQueue的NSOperation取出。
（4）NSOperation中封装的操作会被放到一条新线程上执行。
2.注意：
NSOperation如同NSObject一样，是个抽象类，所以你必须使用继承于它的子类去封装你需要的操作。
使用子类有3中：
（1）NSInvocationOperation
（2）NSBlockOperation
（3）自定义继承于NSOperation的子类，实现方法。

```
**NSInvocationOperation**

```
 NSInvocationOperation *operation = [[NSInvocationOperation alloc]initWithTarget:self selector:@selector(doSomeThing) object:nil];
 
[operation start];
// 这里是在主线程中执行
```

**NSBlockOperation**

```
NSBlockOperation *operation2 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"%@", [NSThread currentThread]);
    }];
    // 添加额外操作
    [operation2 addExecutionBlock:^{
        NSLog(@"%@", [NSThread currentThread]);
    }];
    
    [operation2 start];
//  只要操作数大于2 ，就会执行异步操作
/*
2016-08-12 11:09:55.413 多线程[998:173197] <NSThread: 0x7fe103405b80>{number = 1, name = main}
2016-08-12 11:09:55.413 多线程[998:173233] <NSThread: 0x7fe1036081e0>{number = 2, name = (null)}
*/
// 从第二打印看出，额外操作是在另外一个线程中执行的
```

**NSOperationQueue**

```
// 主队列

NSOperationQueue *queue = [NSOperationQueue mainQueue];
 
// 非主队列

NSOperationQueue *queue = [[NSOperationQueue alloc] init];
```


```
--- 创建方式 ---

// 创建NSInvocationOperation对象
    NSInvocationOperation *operation1 = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(testAction) object:nil];
    // block方式创建
    NSBlockOperation *operation2 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"test2%@", [NSThread currentThread]);
    }];
    // 创建NSOperationQueue
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    // 把操作添加到队列中
    [queue addOperation:operation1];
    [queue addOperation:operation2];
    // 另一种添加到队列方式
    [queue addOperationWithBlock:^{
        NSLog(@"test3%@", [NSThread currentThread]);
    }];
// 从queue中拿出来的operation都是异步执行的。

// 添加一组操作
    NSArray *array =[ [NSArray alloc]initWithObjects:operation2,operation1, nil];
    
    [queue addOperations:array waitUntilFinished:YES];
    
```
```
--- 添加依赖关系 ---

// 这样就可以设置执行先后顺序
NSOperationQueue *queue = [[NSOperationQueue alloc] init];  
  
NSBlockOperation *operation1 = [NSBlockOperation blockOperationWithBlock:^(){  
    NSLog(@"执行第1次操作，线程：%@", [NSThread currentThread]);  
}];  
  
NSBlockOperation *operation2 = [NSBlockOperation blockOperationWithBlock:^(){  
    NSLog(@"执行第2次操作，线程：%@", [NSThread currentThread]);  
}];  
// operation1依赖于operation2, 所以operation1 会等到 operation2执行了之后才会执行  
[operation1 addDependency:operation2];  
  
[queue addOperation:operation1];  
[queue addOperation:operation2];  
```

```
--- 设置队列并发数 ---

// 每次只能执行一个操作  
queue.maxConcurrentOperationCount = 1;  
// 或者这样写  
[queue setMaxConcurrentOperationCount:1]; 
// 注意这里并不决定操作执行顺序，执行顺序还是跟operation的优先级和是否准备好有关,相同优先级还是可以依靠先进先出的原则串行的。
```

```
--- 等待operation完成 ---

// 这样也可以设定并发顺序

NSBlockOperation *opB = [NSBlockOperation blockOperationWithBlock:^{
　　　[opA waitUntilFinished]; //opB线程等待直到opA执行结束（正常结束或被取消）
        [self operate];
    }];
// 当然最好还是添加依赖实现

// 阻塞当前线程，等待queue的所有操作执行完毕  
[queue waitUntilAllOperationsAreFinished];  
// 使用方法跟上面一样，添加到你想要等待queue在其之前完成的队列当中
// 跟 GCD 中的栅拦一样，队列前面的先执行，在这行后面添加的操作后执行

    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    for (int i = 0; i < 10; i++) {
        NSInvocationOperation *operation = [[NSInvocationOperation alloc] initWithTarget: self selector: @selector(doSomeThing) object: nil];
        [queue addOperation: operation];
    }
    [queue waitUntilAllOperationsAreFinished];
    for (int i = 0; i < 3; i ++) {
        NSInvocationOperation *operation = [[NSInvocationOperation alloc] initWithTarget: self selector: @selector(doAnotherThing) object: nil];
        [queue addOperation: operation];
    }
```

### 第44条：通过 Dispatch Group 机制，根据系统资源状况来执行任务

dispatch group 是 GCD 的一项特性，能够把任务分组。 调用者可以等待这组任务执行完毕，也可以在提供回调函数自后继续往下执行，这组任务完成时，调用者会得到通知。

并行和并发的区别

* 并行：多个任务在多个CPU上同时执行，即同一时刻有多个事情在做
* 并发：多个任务在同一个CPU上交替执行，即同一个时刻只能有一个事情在做

简单使用

```
- (void)createGroupDispatch
{
    dispatch_group_t group = dispatch_group_create();
    for (int i = 0; i < 10; i++) {
        dispatch_group_async( group, dispatch_get_global_queue( DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
            NSLog(@"%d", i);
        });
    }
    
    // 监听任务前面的任务是否执行完成
    
    dispatch_group_notify(group, dispatch_get_main_queue(), ^{
        NSLog(@"finish");
    });
}
```
**dispatch_group_enter 和 dispatch_group_wait**

指定任务所属的dispatch group，可以使用在网络请求时，当你想要知道网络是否完成时，在你的网络请求前插入第一句，完成的时候插入第二句，然后就只需要监听所有任务是否完成就可以了。

```
void dispatch_group_enter(dispatch_group_t group);
void dispatch_group_leave(dispatch_group_t group);
```

**dispatch_group_wait**

等待某个线程,会阻塞当前线程，等待timeout的时间，所以你设置DISPATCH_TIME_FOREVER的时候，它会等前面加入group的任务全部执行完成之后才会进行下一步。

```
    dispatch_group_t group = dispatch_group_create();
    for (int i = 0; i < 10; i++) {
        dispatch_group_async( group, dispatch_get_global_queue( DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
            NSLog(@"start %d",i);
            [NSThread sleepForTimeInterval: 10];
            NSLog(@"end %d", i);
        });
    }
    
    // 如果dispatch group 所需要的时间小于 timeout，则返回0，否则返回非0值
    
    long result = dispatch_group_wait(group, DISPATCH_TIME_FOREVER);
    if (result == 0) {
        NSLog(@"group finish");
    }else {
        NSLog(@"gourp is Process");
    }
```

```
2018-05-03 09:51:40.089314+0800 GCD[50816:1863051] start 2
2018-05-03 09:51:40.089313+0800 GCD[50816:1863054] start 1
2018-05-03 09:51:40.089314+0800 GCD[50816:1863052] start 0
2018-05-03 09:51:40.089323+0800 GCD[50816:1863053] start 3
2018-05-03 09:51:40.089355+0800 GCD[50816:1863059] start 4
2018-05-03 09:51:40.089414+0800 GCD[50816:1863061] start 5
2018-05-03 09:51:40.089438+0800 GCD[50816:1863062] start 6
2018-05-03 09:51:40.089469+0800 GCD[50816:1863063] start 7
2018-05-03 09:51:40.089515+0800 GCD[50816:1863064] start 8
2018-05-03 09:51:40.089531+0800 GCD[50816:1863065] start 9
2018-05-03 09:51:50.092983+0800 GCD[50816:1863052] end 0
2018-05-03 09:51:50.092983+0800 GCD[50816:1863051] end 2
2018-05-03 09:51:50.092983+0800 GCD[50816:1863053] end 3
2018-05-03 09:51:50.092983+0800 GCD[50816:1863061] end 5
2018-05-03 09:51:50.092983+0800 GCD[50816:1863059] end 4
2018-05-03 09:51:50.092983+0800 GCD[50816:1863054] end 1
2018-05-03 09:51:50.093044+0800 GCD[50816:1863063] end 7
2018-05-03 09:51:50.093044+0800 GCD[50816:1863062] end 6
2018-05-03 09:51:50.093049+0800 GCD[50816:1863064] end 8
2018-05-03 09:51:50.093049+0800 GCD[50816:1863065] end 9
2018-05-03 09:51:50.094299+0800 GCD[50816:1862995] group finish
```

```
long dispatch_group_wait(dispatch_group_t group, dispatch_time_t timeout);

// dispatch_time_t

        dispatch_time(<#dispatch_time_t when#>, <#int64_t delta#>)
        第一个参数是从什么时间开始,一般直接传 DISPATCH_TIME_NOW; 表示从现在开始
        第二个参数表示具体的时间长度(不能直接传 int 或 float), 可以写成这种形式 (int64_t)3* NSEC_PER_SEC
        
        #define NSEC_PER_SEC 1000000000ull  每秒有1000000000纳秒
        #define NSEC_PER_MSEC 1000000ull    每毫秒有1000000纳秒
        #define USEC_PER_SEC 1000000ull     每秒有1000000微秒
        #define NSEC_PER_USEC 1000ull       每微秒有1000纳秒
        
        注意 delta 的单位是纳秒! 
        1秒的写作方式可以是 1* NSEC_PER_SEC; 1000* NSEC_PER_MSEC; USEC_PER_SEC* NSEC_PER_USEC
```

```
// 如果dispatch group 所需要的时间小于 timeout，则返回0，否则返回非0值

    long result = dispatch_group_wait(group, dispatch_time(DISPATCH_TIME_NOW, 1 * NSEC_PER_SEC));
    if (result == 0) {
        NSLog(@"group finish");
    }else {
        NSLog(@"gourp is Process");
    }
```

**dispatch_group_notify** 

可以等待所有group的任务完成之后，使块在特定线程执行，注意这个你在group的任意位置写都会等加入这个group的所有任务完成的才执行。

```
void dispatch_group_notify(dispatch_group_t group,
	dispatch_queue_t queue,
	dispatch_block_t block);
```

**dispatch_apply**

反复执行一定次数，每次传给块的参数值都会递增，从 0 开始，直至iterations-1。

```
- (void)createDispatchApply
{
    dispatch_queue_t queue = dispatch_get_global_queue( DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    dispatch_apply(10, queue, ^(size_t i) {
        NSLog(@"%zu", i);
    });
}
```

### 第45条：使用 dispatch_once 来执行只需要运行一次的线程安全的代码

通常单例编写，使用锁来

```
@implementation SynchronizedObj

+ (instancetype)sharedInstance
{
    return [[self alloc] init];
}

+ (instancetype)allocWithZone:(struct _NSZone *)zone
{
    static id _sharedInstance;
    @synchronized(self) {
        if (_sharedInstance == nil) {
            _sharedInstance = [super allocWithZone: zone];
        }
    }
    return _sharedInstance;
}
@end
```

dispatch_once 的写法，系统会判断这个代码执行过没，如果执行过，就不会再执行了。

```
@implementation DispatchOnceObj

static id _sharedInstance = nil;

+ (instancetype)sharedInstance
{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        _sharedInstance = [[self alloc] init];
    });
    return _sharedInstance;
}

+ (instancetype)allocWithZone:(struct _NSZone *)zone
{

    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        _sharedInstance = [super allocWithZone: zone];
    });
    return _sharedInstance;
}

@end
```

### 第47条：不要使用 dispatch_get_current_queue

dispatch_get_current_queue 返回的是当前的队列，但是当前队列是最里层的那个队列。

```
- (void)createDispatchGetCurrentQueue
{
    dispatch_queue_t queueA = dispatch_queue_create("queue A", DISPATCH_QUEUE_SERIAL);
    dispatch_queue_t queueB = dispatch_queue_create("queue B", DISPATCH_QUEUE_SERIAL);
    dispatch_set_target_queue(queueB, queueA);
    static int kQueueSpecific;
    CFStringRef queueSepcificValue = CFSTR("queuA");
    dispatch_queue_set_specific(queueA, &kQueueSpecific, (void*)queueSepcificValue, (dispatch_function_t)CFRelease);
    dispatch_sync(queueA, ^{
        NSLog(@"queueA");
        dispatch_block_t block = ^{
            NSLog(@"queueAA");
        };
        CFStringRef retrievedValue = dispatch_get_specific(&kQueueSpecific);
        if (retrievedValue) {
            block();
        }else {
            dispatch_sync(queueB, ^{
            });
        }
    });
}
```

* dispatch_get_current_queue 函数的行为常常与开发者所预期的不同。此函数已经废弃，只应做调试之用。
* 由于派发队列是按层级来组织的，所以无法单用某个队列对象来描述当前“当前队列”这一概念。
* dispatch_get_current_queue 函数用于解决由于不可重入的代码所引发的死锁，然而能用此函数解决的问题，通常也能改用“队列特定数据”来解决。

#### dispatch_set_target_queue

第一个参数表示要变更优先级的队列，第二个参数表示优先级参考右边的队列

```
void dispatch_set_target_queue(dispatch_object_t object,
		dispatch_queue_t _Nullable queue);
```

第一个作用，是改变队列的优先级。

dispatch_queue_create函数生成的DisPatch Queue不管是Serial DisPatch Queue还是Concurrent Dispatch Queue，执行的优先级都与默认优先级的Global Dispatch queue相同，即是下面的globalQueue 的优先级跟 serialQueue 和 concurrentQueue的优先级相同。

```
dispatch_queue_t globalQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

dispatch_queue_t serialQueue = dispatch_queue_create("SERIAL", DISPATCH_QUEUE_SERIAL);

dispatch_queue_t concurrentQueue = dispatch_queue_create("CONCURRENT", DISPATCH_QUEUE_CONCURRENT);

```

第二个作用，创建队列的层次体系，让不同队列中的任务同步执行。创建一个串行队列，然后其他队列的target都指向它。（这里需要知道，多个的串行队列的任务，是同时执行的，在不指定target的时候）

```
- (void)testTargetQueue2 {
    //创建一个串行队列queue1
    dispatch_queue_t queue1 = dispatch_queue_create("test.1", DISPATCH_QUEUE_SERIAL);
    //创建一个并行队列queue2
    dispatch_queue_t queue2 = dispatch_queue_create("test.2", DISPATCH_QUEUE_CONCURRENT);
    // target队列
    dispatch_queue_t targetQueu = dispatch_queue_create("test.1", DISPATCH_QUEUE_SERIAL);
    //使用dispatch_set_target_queue()实现队列的动态调度管理
    dispatch_set_target_queue(queue1, targetQueu);
    dispatch_set_target_queue(queue2, targetQueu);
    
    for (NSInteger i = 0; i < 3; i++) {
        dispatch_async(queue1, ^{
            NSLog(@"queue1:,%ld\n%@", i, [NSThread currentThread]);
        });
        dispatch_async(queue2, ^{
            NSLog(@"queue2:,%ld\n%@", i, [NSThread currentThread]);
        });
        dispatch_async(queue1, ^{
            NSLog(@"queue3:,%ld\n%@", i, [NSThread currentThread]);
        });
    }
}
```
这里可以发现queue2会让queue1中的所有任务全部执行完之后，才会开始执行。这里queue2是串行或并行队列并不会影响这个打印结果。先加入的任务的队列会被先执行完。下一个队列会等这个队列的任务全部执行之后才执行。

```
2018-05-03 14:40:23.011185+0800 GCD[56271:2073269] queue1:,0
<NSThread: 0x608000279c40>{number = 3, name = (null)}
2018-05-03 14:40:23.011350+0800 GCD[56271:2073269] queue3:,0
<NSThread: 0x608000279c40>{number = 3, name = (null)}
2018-05-03 14:40:23.011483+0800 GCD[56271:2073269] queue1:,1
<NSThread: 0x608000279c40>{number = 3, name = (null)}
2018-05-03 14:40:23.011626+0800 GCD[56271:2073269] queue3:,1
<NSThread: 0x608000279c40>{number = 3, name = (null)}
2018-05-03 14:40:23.011756+0800 GCD[56271:2073269] queue1:,2
<NSThread: 0x608000279c40>{number = 3, name = (null)}
2018-05-03 14:40:23.011936+0800 GCD[56271:2073269] queue3:,2
<NSThread: 0x608000279c40>{number = 3, name = (null)}
2018-05-03 14:40:23.012061+0800 GCD[56271:2073269] queue2:,0
<NSThread: 0x608000279c40>{number = 3, name = (null)}
2018-05-03 14:40:23.012200+0800 GCD[56271:2073269] queue2:,1
<NSThread: 0x608000279c40>{number = 3, name = (null)}
2018-05-03 14:40:23.012340+0800 GCD[56271:2073269] queue2:,2
<NSThread: 0x608000279c40>{number = 3, name = (null)}
```


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




