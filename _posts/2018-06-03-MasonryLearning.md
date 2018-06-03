---
layout:     post
title:      "Masonry 源码学习"
subtitle:   "Masonry 源码学习"
date:       2018-06-03 12:00:00
author:     "MIN"
header-img: "img/post-bg-nextgen-web-pwa.jpg"
header-mask: 0.3
catalog:    true
tags:
    - Application
---

## Masonry 源码学习

### 简述

Masonry 与其它的第三方开源框架一样选择了使用分类的方式为 UIKit 添加一个方法 mas_makeConstraint, 这个方法接受了一个 block, 这个 block 有一个 MASConstraintMaker 类型的参数, 这个 maker 会持有一个约束的数组, 这里保存着所有将被加入到视图中的约束.

我们通过链式的语法配置 maker, 设置它的 left right 等属性, 比如说 make.left.equalTo(view), 其实这个  left equalTo 还有像 with offset 之类的方法都会返回一个 MASConstraint 的实例, 所以在这里才可以用类似 Ruby 中链式的语法.

在配置结束后, 首先会调用 maker 的 install 方法, 而这个 maker 的 install 方法会遍历其持有的约束数组, 对其中的每一个约束发送 install 消息. 在这里就会使用到在上一步中配置的属性, 初始化 NSLayoutConstraint 的子类 MASLayoutConstraint 并添加到合适的视图上.

视图的选择会通过调用一个方法 mas_closestCommonSuperview: 来返回两个视图的最近公共父视图.

以上文字引用 [这个大神的文章](https://draveness.me/ios-yuan-dai-ma-fen-xi-masonry)

### 从使用开始了解

```
[self.bottomView mas_makeConstraints:^(MASConstraintMaker *make) {
    make.left.equalTo(self);
    make.right.equalTo(self);
    make.top.equalTo(self.topView.mas_bottom);
    make.height.equalTo(self.topView);
}];
```

#### mas_makeConstraints: 函数

我们常用的三个函数，分别是设置，更新，重置约束。这里要注意第二个方法，只能更新已存在约束的 constant 属性。

```
- (NSArray *)mas_makeConstraints:(void(NS_NOESCAPE ^)(MASConstraintMaker *make))block;
- (NSArray *)mas_updateConstraints:(void(NS_NOESCAPE ^)(MASConstraintMaker *make))block;
- (NSArray *)mas_remakeConstraints:(void(NS_NOESCAPE ^)(MASConstraintMaker *make))block;
```

然后我们可以看到这个函数位于 UIView 分类中。这个分类不仅提供了以上的方法用于设置更改约束，还提供了 mas_left、mas_top等属性，以获取跟这个视图相关的 MASViewAttribute 对象，这个类用于描述约束。

方法实现

```
- (NSArray *)mas_makeConstraints:(void(^)(MASConstraintMaker *))block {
    self.translatesAutoresizingMaskIntoConstraints = NO;
    MASConstraintMaker *constraintMaker = [[MASConstraintMaker alloc] initWithView:self];
    block(constraintMaker);
    return [constraintMaker install];
}
```

translatesAutoresizingMaskIntoConstraints 属性设置为 NO，表明我们不想使用自动计算尺寸，用约束。如果不设置，约束会没有效果。

然后创建了一个 MASConstraintMaker 实例，然后将实例传入 block 中，最后调用实例的 install 方法为视图添加约束。


#### MASConstraintMaker 

```
Provides factory methods for creating MASConstraints. 
Constraints are collected until they are ready to be installed.
```

提供创建 masconstraint 的工厂方法。

所有约束都会被收集起来，直到它们最后调用 install 方法添加到视图上。

上面我们通过 initWithView: 方法创建了一个 MASConstraintMaker 实例。这个方法会初始化一个 maker，持有一个视图的弱引用。这里我们从上面可以知道，它持有的是需要添加约束的视图。并初始化一个可变数组 constraints，用于存储 MASConstraint 实例。

```
- (id)initWithView:(MAS_VIEW *)view {
    self = [super init];
    if (!self) return nil;
    
    self.view = view;
    self.constraints = NSMutableArray.new;
    
    return self;
}
```

#### block 里面执行的代码

在调用 `block(constraintMaker)` 其实是对 constraintMaker 进行配置，向可变数组 constraints 中添加对应的 MASConstraint 实例。

**make.left**

```
- (MASConstraint *)left {
    return [self addConstraintWithLayoutAttribute:NSLayoutAttributeLeft];
}

- (MASConstraint *)addConstraintWithLayoutAttribute:(NSLayoutAttribute)layoutAttribute {
    return [self constraint:nil addConstraintWithLayoutAttribute:layoutAttribute];
}

- (MASConstraint *)constraint:(MASConstraint *)constraint addConstraintWithLayoutAttribute:(NSLayoutAttribute)layoutAttribute {
    MASViewAttribute *viewAttribute = [[MASViewAttribute alloc] initWithView:self.view layoutAttribute:layoutAttribute];
    MASViewConstraint *newConstraint = [[MASViewConstraint alloc] initWithFirstViewAttribute:viewAttribute];
    if ([constraint isKindOfClass:MASViewConstraint.class]) {
        //replace with composite constraint
        NSArray *children = @[constraint, newConstraint];
        MASCompositeConstraint *compositeConstraint = [[MASCompositeConstraint alloc] initWithChildren:children];
        compositeConstraint.delegate = self;
        [self constraint:constraint shouldBeReplacedWithConstraint:compositeConstraint];
        return compositeConstraint;
    }
    if (!constraint) {
        newConstraint.delegate = self;
        [self.constraints addObject:newConstraint];
    }
    return newConstraint;
}
```

`make` 的 `left ` `right ` `top ` `bottom ` 等属性都最后会调用下面这个方法：

```
- (MASConstraint *)constraint:(MASConstraint *)constraint addConstraintWithLayoutAttribute:(NSLayoutAttribute)layoutAttribute;
```

这个方法先创建了一个 MASViewAttribute 类实例 viewAttribute。

这个 MASViewAttribute 类是存储视图和相关 NSLayoutAttribute 的不可变元组。用于描述约束等式左边或右边的部分。（比如这个例子就是用来描述左边的 make.left）

然后用这个 viewAttribute 来初始化另一个 MASViewConstraint 类实例 newConstraint。

这个类 MASViewConstraint 包含创建 NSLayoutConstraint 并将其添加到适当视图所需的属性。继承 MASConstraint 类。MASConstraint 类让我们可以通过链式结构来创建这个约束所需要的属性。如我们常用的 equalTo(...)

```
- (MASConstraint * (^)(id attr))equalTo;

下面会介绍，block作为返回值。
```
因为我们从这里传入的 `constraint` 为 nil，所以先跳过第一个 if 部分。看第二个，当 `constraint` 为空时，我们设置了 `newConstraint` 的代理为 maker，然后加入到 maker 的constraints数组中，然后返回。

这里就做完了 `make.left` 进行的所有工作，它会返回一个 MASConstraint，用于之后的配置。

**make.left.equalTo(self)**

在 `make.left` 返回 newConstraint 之后，我们会继续在这个链式的语法中调用下一个方法来指定约束的关系。

```
- (MASConstraint * (^)(id attr))equalTo;
- (MASConstraint * (^)(CGFloat offset))offset;
- (MASConstraint * (^)(CGFloat multiplier))multipliedBy;
- (MASConstraint * (^)(CGFloat divider))dividedBy;
```

上面的方法都在 `MASViewConstraint` 父类 `MASConstraint`中定义。

`MASConstraint` 是一个抽象类，其中有很多方法都必须在子类中覆写。Masonry 中有两个 `MASConstraint` 的子类，分别是 `MASViewConstraint` 和 `MASCompositeConstraint`。后者实际上是一些约束的集合。

然后我们具体来看 ```equalTo``` 的具体实现

```
- (MASConstraint * (^)(id))equalTo {
    return ^id(id attribute) {
        return self.equalToWithRelation(attribute, NSLayoutRelationEqual);
    };
}

- (MASConstraint * (^)(id, NSLayoutRelation))equalToWithRelation {
    return ^id(id attribute, NSLayoutRelation relation) {
        if ([attribute isKindOfClass:NSArray.class]) {
            NSAssert(!self.hasLayoutRelation, @"Redefinition of constraint relation");
            NSMutableArray *children = NSMutableArray.new;
            for (id attr in attribute) {
                MASViewConstraint *viewConstraint = [self copy];
                viewConstraint.layoutRelation = relation;
                viewConstraint.secondViewAttribute = attr;
                [children addObject:viewConstraint];
            }
            MASCompositeConstraint *compositeConstraint = [[MASCompositeConstraint alloc] initWithChildren:children];
            compositeConstraint.delegate = self.delegate;
            [self.delegate constraint:self shouldBeReplacedWithConstraint:compositeConstraint];
            return compositeConstraint;
        } else {
            NSAssert(!self.hasLayoutRelation || self.layoutRelation == relation && [attribute isKindOfClass:NSValue.class], @"Redefinition of constraint relation");
            self.layoutRelation = relation;
            self.secondViewAttribute = attribute;
            return self;
        }
    };
}
```

因为我传入的不是一个数组，所以跳过第一个 if 部分。因为我们为 equalTo 提供了参数 attribute 和布局关系 NSLayoutRelationEqual, 这两个参数会传递到 equalToWithRelation 中, 设置 constraint 的布局关系和 secondViewAttribute 属性, 为即将 maker 的 install 做准备。

注意这里的 `secondViewAttribute` 方法，不只是一个简单的赋值。

```
- (void)setSecondViewAttribute:(id)secondViewAttribute {
    if ([secondViewAttribute isKindOfClass:NSValue.class]) {
        [self setLayoutConstantWithValue:secondViewAttribute];
    } else if ([secondViewAttribute isKindOfClass:MAS_VIEW.class]) {
        _secondViewAttribute = [[MASViewAttribute alloc] initWithView:secondViewAttribute layoutAttribute:self.firstViewAttribute.layoutAttribute];
    } else if ([secondViewAttribute isKindOfClass:MASViewAttribute.class]) {
        MASViewAttribute *attr = secondViewAttribute;
        if (attr.layoutAttribute == NSLayoutAttributeNotAnAttribute) {
            _secondViewAttribute = [[MASViewAttribute alloc] initWithView:attr.view item:attr.item layoutAttribute:self.firstViewAttribute.layoutAttribute];;
        } else {
            _secondViewAttribute = secondViewAttribute;
        }
    } else {
        NSAssert(NO, @"attempting to add unsupported attribute: %@", secondViewAttribute);
    }
}
```

这里先判断了传入的 `secondViewAttribute` 是不是一个 `NSValue` 类，如果是，根据其类型，设置的是 `self` 而不是 `_secondViewAttribute` 的属性。

```
---这里对应传入的是 @10 或者 CGSizeMake(10, 10)---

make.left.equalTo(@10);
make.width.mas_equalTo(CGSizeMake(10, 10));
```

```
- (void)setLayoutConstantWithValue:(NSValue *)value {
    if ([value isKindOfClass:NSNumber.class]) {
        self.offset = [(NSNumber *)value doubleValue];
    } else if (strcmp(value.objCType, @encode(CGPoint)) == 0) {
        CGPoint point;
        [value getValue:&point];
        self.centerOffset = point;
    } else if (strcmp(value.objCType, @encode(CGSize)) == 0) {
        CGSize size;
        [value getValue:&size];
        self.sizeOffset = size;
    } else if (strcmp(value.objCType, @encode(MASEdgeInsets)) == 0) {
        MASEdgeInsets insets;
        [value getValue:&insets];
        self.insets = insets;
    } else {
        NSAssert(NO, @"attempting to set layout constant with unsupported value: %@", value);
    }
}
```

如果不是，看 `secondViewAttribute` 是不是一个 `UIView` 类型的实例，如果是，则初始化 `_secondViewAttribute`。

```
---这里对应传入的应该是一个视图---

make.left.equalTo(view);
```

如果不是，再判断 `secondViewAttribute` 是不是一个 `MASViewAttribute` 类型的实例，如果是则进入进行判断 `attr.layoutAttribute` 是不是已有的类型，如果不是，设置为 `self.firstViewAttribute.layoutAttribute`。如果是设置 `_secondViewAttribute = secondViewAttribute;`。

```
---这里对应传入的应该是 view.mas_XXX---

make.left.equalTo(view.mas_right);
```

如果以上都不满足，则抛出异常。

#### 最后 MASConstraintMaker 的 install 阶段

```
- (NSArray *)install {
    if (self.removeExisting) {
        NSArray *installedConstraints = [MASViewConstraint installedConstraintsForView:self.view];
        for (MASConstraint *constraint in installedConstraints) {
            [constraint uninstall];
        }
    }
    NSArray *constraints = self.constraints.copy;
    for (MASConstraint *constraint in constraints) {
        constraint.updateExisting = self.updateExisting;
        [constraint install];
    }
    [self.constraints removeAllObjects];
    return constraints;
}
``` 

第一个 if 的部分，是判断是否是 `mas_remakeConstraints`，如果是，就先遍历已经设置好的约束，并执行 `uninstall`。

然后遍历 `maker` 实例的 `constraints` 数组，来执行 `MASViewConstraint` 的 `install` 方法。

```
- (void)install {
    if (self.hasBeenInstalled) {
        return;
    }
    
    if ([self supportsActiveProperty] && self.layoutConstraint) {
        self.layoutConstraint.active = YES;
        [self.firstViewAttribute.view.mas_installedConstraints addObject:self];
        return;
    }
    
    MAS_VIEW *firstLayoutItem = self.firstViewAttribute.item;
    NSLayoutAttribute firstLayoutAttribute = self.firstViewAttribute.layoutAttribute;
    MAS_VIEW *secondLayoutItem = self.secondViewAttribute.item;
    NSLayoutAttribute secondLayoutAttribute = self.secondViewAttribute.layoutAttribute;

    // alignment attributes must have a secondViewAttribute
    // therefore we assume that is refering to superview
    // eg make.left.equalTo(@10)
    if (!self.firstViewAttribute.isSizeAttribute && !self.secondViewAttribute) {
        secondLayoutItem = self.firstViewAttribute.view.superview;
        secondLayoutAttribute = firstLayoutAttribute;
    }
    
    MASLayoutConstraint *layoutConstraint
        = [MASLayoutConstraint constraintWithItem:firstLayoutItem
                                        attribute:firstLayoutAttribute
                                        relatedBy:self.layoutRelation
                                           toItem:secondLayoutItem
                                        attribute:secondLayoutAttribute
                                       multiplier:self.layoutMultiplier
                                         constant:self.layoutConstant];
    
    layoutConstraint.priority = self.layoutPriority;
    layoutConstraint.mas_key = self.mas_key;
    
    if (self.secondViewAttribute.view) {
        MAS_VIEW *closestCommonSuperview = [self.firstViewAttribute.view mas_closestCommonSuperview:self.secondViewAttribute.view];
        NSAssert(closestCommonSuperview,
                 @"couldn't find a common superview for %@ and %@",
                 self.firstViewAttribute.view, self.secondViewAttribute.view);
        self.installedView = closestCommonSuperview;
    } else if (self.firstViewAttribute.isSizeAttribute) {
        self.installedView = self.firstViewAttribute.view;
    } else {
        self.installedView = self.firstViewAttribute.view.superview;
    }


    MASLayoutConstraint *existingConstraint = nil;
    if (self.updateExisting) {
        existingConstraint = [self layoutConstraintSimilarTo:layoutConstraint];
    }
    if (existingConstraint) {
        // just update the constant
        existingConstraint.constant = layoutConstraint.constant;
        self.layoutConstraint = existingConstraint;
    } else {
        [self.installedView addConstraint:layoutConstraint];
        self.layoutConstraint = layoutConstraint;
        [firstLayoutItem.mas_installedConstraints addObject:self];
    }
}
``` 

第一个 `if` 部分，先判断是否被设置过并激活，`active` 属性，只有当约束的这个属性为 YES 的时候，约束才会生效。

```
if (self.hasBeenInstalled) {
    return;
}
```

```
- (BOOL)hasBeenInstalled {
    return (self.layoutConstraint != nil) && [self isActive];
}

- (BOOL)isActive {
    BOOL active = YES;
    if ([self supportsActiveProperty]) {
        active = [self.layoutConstraint isActive];
    }

    return active;
}

- (BOOL)supportsActiveProperty {
    return [self.layoutConstraint respondsToSelector:@selector(isActive)];
}
```

第二个 `if` 部分判断，能不能使用 `active` 属性，因为 `iOS 8.0` 以下不能使用这个属性。判断是否设置过这个约束。如果两个都满足，则激活这个约束，并将自身添加 `self.firstViewAttribute.view` 的 `mas_installedConstraints` 属性中。注意这个 `mas_installedConstraints` 属性是一个我们给 `UIView` 新增的 `NSMutableSet`。用于记录这个 `view` 已经设置过得不重复的约束。

```
if ([self supportsActiveProperty] && self.layoutConstraint) {
    self.layoutConstraint.active = YES;
    [self.firstViewAttribute.view.mas_installedConstraints addObject:self];
    return;
}
```

如果以上都没有没有执行，那么开始下面的步骤。

取出第一个和第二个视图属性的 View 和 layoutAttribute。用于后面约束的参数设置。

```
MAS_VIEW *firstLayoutItem = self.firstViewAttribute.item;
    NSLayoutAttribute firstLayoutAttribute = self.firstViewAttribute.layoutAttribute;
    MAS_VIEW *secondLayoutItem = self.secondViewAttribute.item;
    NSLayoutAttribute secondLayoutAttribute = self.secondViewAttribute.layoutAttribute;
```

然后判断第一个视图属性的 `layoutAttribute` 是否是关于尺寸的并且第二个视图属性是不是为空。如果为空且与尺寸无关，则执行。设置第二个视图属性为第一视图属性相关的 `view` 的父视图，设置第二个视图属性的 `layoutAttribute` 与第一个视图相同。

```
if (!self.firstViewAttribute.isSizeAttribute && !self.secondViewAttribute) {
    secondLayoutItem = self.firstViewAttribute.view.superview;
    secondLayoutAttribute = firstLayoutAttribute;
}

- (BOOL)isSizeAttribute {
    return self.layoutAttribute == NSLayoutAttributeWidth
        || self.layoutAttribute == NSLayoutAttributeHeight;
}
```

以上的 `firstLayoutItem` `firstLayoutAttribute` `secondLayoutItem` `secondLayoutAttribute` 都是用来设置下面的约束的。

```
MASLayoutConstraint *layoutConstraint
        = [MASLayoutConstraint constraintWithItem:firstLayoutItem
                                        attribute:firstLayoutAttribute
                                        relatedBy:self.layoutRelation
                                           toItem:secondLayoutItem
                                        attribute:secondLayoutAttribute
                                       multiplier:self.layoutMultiplier
                                         constant:self.layoutConstant];
    
layoutConstraint.priority = self.layoutPriority;
layoutConstraint.mas_key = self.mas_key;
```

然后设置 MASViewConstraint 实例的 installedView 属性，这个属性是用来记录约束被添加到那个视图上面的。

```
if (self.secondViewAttribute.view) {
        MAS_VIEW *closestCommonSuperview = [self.firstViewAttribute.view mas_closestCommonSuperview:self.secondViewAttribute.view];
        NSAssert(closestCommonSuperview,
                 @"couldn't find a common superview for %@ and %@",
                 self.firstViewAttribute.view, self.secondViewAttribute.view);
        self.installedView = closestCommonSuperview;
    } else if (self.firstViewAttribute.isSizeAttribute) {
        self.installedView = self.firstViewAttribute.view;
    } else {
        self.installedView = self.firstViewAttribute.view.superview;
    }
```

第一个种情况是当第二个视图属性的 `view` 为不为空时，就去查找两个视图属性的 `view` 的共同的父视图，如果不能找到则抛出异常。如果找到则设置这个父视图为被添加约束的视图。

```
- (instancetype)mas_closestCommonSuperview:(MAS_VIEW *)view {
    MAS_VIEW *closestCommonSuperview = nil;

    MAS_VIEW *secondViewSuperview = view;
    while (!closestCommonSuperview && secondViewSuperview) {
        MAS_VIEW *firstViewSuperview = self;
        while (!closestCommonSuperview && firstViewSuperview) {
            if (secondViewSuperview == firstViewSuperview) {
                closestCommonSuperview = secondViewSuperview;
            }
            firstViewSuperview = firstViewSuperview.superview;
        }
        secondViewSuperview = secondViewSuperview.superview;
    }
    return closestCommonSuperview;
}
```
第二种情况是当第一个视图属性的 `ayoutAttribute` 是跟尺寸相关的。就设置被添加约束的视图为第一个视图属性的 `view`。

否则设置为第一个视图属性的 `view` 的父视图。

这里最后一段是用来判断是否是更新约束的，如果是则遍历上面那个 `installedView` 的所有约束，看其有没有是否与我们上面创建的约束除开 `constant` 其余都相同的已存在的约束。如果有则更新 `constant`，并设置 `self.layoutConstraint` 为已存在的约束。

如果没有添加约束到 `installedView` 视图中。并设置这个实例的 `layoutConstraint` 为上面我们创建的约束，并向 `firstLayoutItem.mas_installedConstraints` 添加 `self`。记住这里的 `self` 是 `MASViewConstraint` 类实例，包含创建 NSLayoutConstraint 并将其添加到适当视图所需的属性。


最后清空 `constraints` 数组。

以上就是整个添加过程。

#### 其他问题

**make.left.equal(view).with.offset(30) 中的 with 和 offset**

`with` 和 `and` 都是直接返回不做修改，为了使链式结构好看。

`offset` 是用来设置 `layoutConstant` 属性。

```
// MASConstraint.m

- (MASConstraint *)with {
    return self;
}

- (MASConstraint *)and {
    return self;
}
// MASViewConstraint.m

- (void)setOffset:(CGFloat)offset {
    self.layoutConstant = offset;
}
```

**MASCompositeConstraint**

`MASCompositeConstraint` 是一些 `MASConstraint` 的集合, 它能够提供一种更加便捷的方法同时为一个视图来添加多个约束.

通过 `make` 直接调用 `edges` `size` `center` 时, 就会产生一个 `MASCompositeConstraint` 的实例, 而这个实例会初始化所有对应的单独的约束.

```
// MASConstraintMaker.m

- (MASConstraint *)edges {
    return [self addConstraintWithAttributes:MASAttributeTop | MASAttributeLeft | MASAttributeRight | MASAttributeBottom];
}

- (MASConstraint *)size {
    return [self addConstraintWithAttributes:MASAttributeWidth | MASAttributeHeight];
}

- (MASConstraint *)center {
    return [self addConstraintWithAttributes:MASAttributeCenterX | MASAttributeCenterY];
}
```

这些属性都会调用 `addConstraintWithAttributes:` 方法, 生成多个属于 `MASCompositeConstraint` 的实例.

```
// MASConstraintMaker.m

NSMutableArray *children = [NSMutableArray arrayWithCapacity:attributes.count];

for (MASViewAttribute *a in attributes) {
   [children addObject:[[MASViewConstraint alloc] initWithFirstViewAttribute:a]];
}

MASCompositeConstraint *constraint = [[MASCompositeConstraint alloc] initWithChildren:children];
constraint.delegate = self;
[self.constraints addObject:constraint];
return constraint;
```


### 总结一下

**View+MASAdditions.h**

这个文件是一个 `UIView` 分类，提供我们约束相关的方法，包括设置，更新，重置约束。并提供大量属性获取一个 MASViewAttribute 类实例，获取的这个实例的 `view` 设置的都为调用的那个视图自身。

**MASViewAttribute.h**

继承自 NSObject。笔者称它为视图属性类。

这个文件就是上面那个实例的类文件，这个类就是用来存储一个视图，和那个视图要设置的布局属性（枚举类型）layoutAttribute。

还提供了一个方法，用于判断这个视图属性类中的 layoutAttribute 是否是 NSLayoutAttributeWidth 或者 NSLayoutAttributeHeight。

**MASConstraintMaker.h**

最上面的那个 `View+MASAdditions.h` 分类文件中的约束相关方法都创建了这个类。通过 block 的方式传递给调用者。

这个初始化方式会保存调用视图和创建一个可变数组，用于存储 `MASConstraint` 类对象。

然后这个类提供了大量属性（`left` `right`等）返回一个 `MASConstraint` 类对象。

创建这个  `MASConstraint` 类对象的过程中创建了 `MASViewAttribute` 对象用于保存调用视图和这个属性方法对应的 NSLayoutAttribute。然后通过这个视图对象来初始化 `MASConstraint` 类对象。

还提供了一个 `install` 用于遍历数组中的 `MASConstraint` 对象，并调用其 `install` 方法，注意这是 `MASConstraint` 对象的方法。这个方法用于根据 `MASConstraint` 对象中在设置好的属性来设置约束。

**MASConstraint.h**

抽象类，提供链式结构的方法，让调用者设置 `MASConstraint` 类对象的相关属性。

`MASViewConstraint` 和 `MASCompositeConstraint` 是其子类。

提供 `install` 和 `activate` 、`deactivate` 的方法，用于子类实现。

**MASViewConstraint.h**

这个子类中有我们创建约束的所有属性。

```
+ (instancetype)constraintWithItem:(id)view1
                         attribute:(NSLayoutAttribute)attr1
                         relatedBy:(NSLayoutRelation)relation
                            toItem:(nullable id)view2
                         attribute:(NSLayoutAttribute)attr2
                        multiplier:(CGFloat)multiplier
                          constant:(CGFloat);
                          
@property NSLayoutPriority priority;
```

* `view1` 与 `self.firstViewAttribute.item` 对应。
* `att1` 与 `self.firstViewAttribute.layoutAttribute` 对应。
* `relation` 与 `self.layoutRelation` 对应。
* `view2` 与 `self.secondViewAttribute.item` 对应。
* `att2` 与 `self.secondViewAttribute.layoutAttribute` 对应。
* `multiplier` 与 `self.layoutMultiplier`对应。
* `c` 与 `self.layoutConstant` 对应。
* `priority` 与 `self.layoutPriority` 对应。


### 理解 block 作为返回值，以达到链式结构的效果


我们在代码中经常可以看到下面这种代码：

```
make.top.equalTo(superview.mas_top).with.offset(padding);
```

其中 equalTo(...) 其实是使用的 block 作为返回值。

```
- (MASConstraint * (^)(id))equalTo {
    return ^id(id attribute) {
        return self.equalToWithRelation(attribute, NSLayoutRelationEqual);
    };
}
```

这样当我们使用这个函数的时候，传入的参数就会作为这个block的参数。你不传编译器会发出错误警告。

测试

```
#import <Foundation/Foundation.h>

@interface MINCatchObj : NSObject

- (int(^)(id inObj))testCatch;

@end

#import "MINCatchObj.h"

@implementation MINCatchObj

- (int (^)(id))testCatch
{
    return ^int(id catchObj){
        NSLog(@"%@", catchObj);
        return 1;
    };
}

@end
```

使用

```
#import "MINCatchObj.h"

- (void)testCatch
{
    MINCatchObj *obj = [[MINCatchObj alloc] init];
    obj.testCatch([UILabel new]);
}
```

```
2018-06-01 16:21:54.148284+0800 Masonry Learning[8046:275101] <UILabel: 0x7fa20e30f230; frame = (0 0; 0 0); userInteractionEnabled = NO; layer = <_UILabelLayer: 0x600000088520>>
```

### 其他文件中你会遇到的一些宏定义

#### NSAssert

断言（assertion）是指在开发期间使用的、让程序在运行时进行自检的代码（通常是一个子程序或宏）。断言为真，则表明程序运行正常，而断言为假，则意味着它已经在代码中发现了意料之外的错误。断言对于大型的复杂程序或可靠性要求极高的程序来说尤其有用。

使用

```
- (void)printMyName:(NSString *)myName  
{  
    NSAssert(myName != nil, @"名字不能为空！");  
    NSLog(@"My name is %@.",myName);  
} 
```

如果 myName 传输进去为空，程序会抛出错误，显示名字不能为空。
如果不为空，则正常执行

#### NSDictionaryOfVariableBindings 

```
#define NSDictionaryOfVariableBindings(...) _NSDictionaryOfVariableBindings(@"" # __VA_ARGS__, __VA_ARGS__, nil)
```

这是一个将你传入的参数变成字典的宏定义

```
NSDictionary *dic = NSDictionaryOfVariableBindings(greenView, redView);
```
打印

```
(lldb) po dic
{
    greenView = "<UIView: 0x7f9cc72458e0; frame = (0 0; 0 0); layer = <CALayer: 0x6040000321a0>>";
    redView = "<UIView: 0x7f9cc726f8d0; frame = (0 0; 0 0); layer = <CALayer: 0x60400022b940>>";
}
```

```
这里需要了解的是 __VA_ARGS__ 在预编译的时候会将括号中传入的参数直接替换
而 # 是会把后面的字符变成一个字符串 
比如

#define becomeStr(...) #__VA_ARGS__

NSLog(@"%s", becomeStr(greenView, redView));

打印 greenView, redView

所以这里宏定义在预编译的时候就变成了

_NSDictionaryOfVariableBindings(@"""greenView, redView", greenView, redView, nil);
```


#### 两个井号

连接前后字符的

```
#define MAS_ATTR_FORWARD(attr)  \
- (MASViewAttribute *)attr {    \
    return [self mas_##attr];   \
}

MAS_ATTR_FORWARD(top);

在预编译的时候就变成了

- (MASViewAttribute *) top {    \
    return [self mas_top];   \
}
```


#### va_start va_arg va_end

```
void va_start(va_list ap, last_arg)
```

这个宏初始化 ap 变量，它与 va_arg 和 va_end 宏是一起使用的。last_arg 是最后一个传递给函数的已知的固定参数，即省略号之前的参数。

```
type va_arg(va_list ap, type)
```

这个宏检索函数参数列表中类型为 type 的下一个参数。

也就是根据你传入的type去查找下一个参数，并返回那个参数。

```
void va_end(va_list ap)
```

这个宏允许使用了 va_start 宏的带有可变参数的函数返回。如果在从函数返回之前没有调用 va_end，则结果为未定义。


**简单使用**

```
int sum(int num_args, ...)
{
    int val = 0;
    va_list ap;
    int i;
    
    va_start(ap, num_args);
    for(i = 0; i < num_args; i++)
    {
        val += va_arg(ap, int);
    }
    va_end(ap);
    
    return val;
}


---调用----

NSLog(@"%d", sum(3, 10, 20, 30));

---打印---

60
```


