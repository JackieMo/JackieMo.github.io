---
layout: post
title: NSLayoutConstraint
date: 2017-12-06 10:05:14.000000000 +09:00
---

## 什么是NSLayoutConstraint?

众所周知，我们可以在xib中用Auto Layout的方式给控件添加约束，此外苹果还给我们提供了代码的方式来添加约束：NSLayoutConstraint。

## 如何使用NSLayoutConstraint?

在使用NSLayoutConstraint前，我们需要知道一个小的知识点：Autoresizing Mask。在 使用Auto Layout时，首先需要将视图的 `setTranslatesAutoresizingMaskIntoConstraints` 设为NO。这个属性默认为YES。当它为YES是，运行时系统会自动将Autoresizing Mask转换为Auto Layout的约束，这些约束很有可能会和我们自己添加的产生冲突。我们常常会忘了做这一步，然后引起的约束报错就是这样的：

![](http://p0bkxzmll.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-12-15%20%E4%B8%8B%E5%8D%883.46.40.png)


在xib中，如果我们勾选了`use auto layout`，则编译器会自动帮我们关闭Autoresizing Mask，如果是使用代码添加约束，则需要手动关闭Autoresizing Mask。

`setTranslatesAutoresizingMaskIntoConstraints` 这个方法是交给被添加约束的视图来执行的，关闭该视图的Autoresizing Mask。在添加约束前，就应该关闭该属性。

## 使用NSLayoutConstraint为视图添加约束

Auto Layout 中约束对应的类为 NSLayoutConstraint，一个 NSLayoutConstraint 实例代表一条约束。

NSLayoutConstraint有两个方法，我们主要介绍 constraintWithItem:也是最常用的：

```
+(instancetype)constraintWithItem:(id)view1 
                        attribute:(NSLayoutAttribute)attr1 
                        relatedBy:(NSLayoutRelation)relation 
                           toItem:(nullable id)view2 
                        attribute:(NSLayoutAttribute)attr2 
                       multiplier:(CGFloat)multiplier 
                         constant:(CGFloat)c;
```

前三个和被约束的视图有关，后四个和他的父视图有关。这个方法连起来可以翻译如下：view1的某个属性（attr1）等于view2的某个属性（attr2）的值的多少倍（multiplier）加上某个常量（constant）。描述的是一个view与另外一个view的位置和大小约束关系。其中属性attribute有上、下、左、右、宽、高等，关系relation有小于等于、等于、大于等于。需要注意的是，小于等于 或 大于等于 优先会使用 等于 关系，如果 等于 不能满足，才会使用 小于 或 大于。例如设置一个 大于等于100 的关系，默认会是 100，当视图被拉伸时，100 无法被满足，尺寸才会变得更大。

假如我们设计一个简单的页面。一个子view在父view中，其中子view的上下左右边缘都离父view的边缘40个像素。这个我们该如何写呢？如下:

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    self.view.backgroundColor = [UIColor yellowColor];
    
    
    UIView *subView = [[UIView alloc] init];
    subView.backgroundColor = [UIColor redColor];
    // 在设置约束前，先将子视图添加进来
    [self.view addSubview:subView];
    
    // 使用autoLayout约束，禁止将AutoresizingMask转换为约束
    [subView setTranslatesAutoresizingMaskIntoConstraints:NO];
    
    // 设置subView相对于VIEW的上左下右各40像素
    NSLayoutConstraint *constraint1 = [NSLayoutConstraint constraintWithItem:subView attribute:NSLayoutAttributeTop relatedBy:NSLayoutRelationEqual toItem:self.view attribute:NSLayoutAttributeTop multiplier:1.0 constant:40];
    NSLayoutConstraint *constraint2 = [NSLayoutConstraint constraintWithItem:subView attribute:NSLayoutAttributeLeft relatedBy:NSLayoutRelationEqual toItem:self.view attribute:NSLayoutAttributeLeft multiplier:1.0 constant:40];
    // 由于iOS坐标系的原点在左上角，所以设置右边距使用负值
    NSLayoutConstraint *constraint3 = [NSLayoutConstraint constraintWithItem:subView attribute:NSLayoutAttributeBottom relatedBy:NSLayoutRelationEqual toItem:self.view attribute:NSLayoutAttributeBottom multiplier:1.0 constant:-40];
    
    // 由于iOS坐标系的原点在左上角，所以设置下边距使用负值
    NSLayoutConstraint *constraint4 = [NSLayoutConstraint constraintWithItem:subView attribute:NSLayoutAttributeRight relatedBy:NSLayoutRelationEqual toItem:self.view attribute:NSLayoutAttributeRight multiplier:1.0 constant:-40];
    
    // 将四条约束加进数组中
    NSArray *array = [NSArray arrayWithObjects:constraint1, constraint2, constraint3, constraint4 ,nil];
    // 把约束条件设置到父视图的Contraints中
    [self.view addConstraints:array];
}

```
我们再添加一个需求：
子view在父view的中间，且子view长200，高100。

```
// ssubView在subView中间，宽高200，100
    UIView *ssubView = [[UIView alloc] init];
    ssubView.backgroundColor = [UIColor blackColor];
    [self.view addSubview:ssubView];
    [ssubView setTranslatesAutoresizingMaskIntoConstraints:NO];
    
    // 加约束
    NSLayoutConstraint *sContraint1 = [NSLayoutConstraint constraintWithItem:ssubView attribute:NSLayoutAttributeCenterX relatedBy:NSLayoutRelationEqual toItem:self.view attribute:NSLayoutAttributeCenterX multiplier:1.0 constant:0];
    NSLayoutConstraint *sContraint2 = [NSLayoutConstraint constraintWithItem:ssubView attribute:NSLayoutAttributeCenterY relatedBy:NSLayoutRelationEqual toItem:self.view attribute:NSLayoutAttributeCenterY multiplier:1.0 constant:0];
    NSLayoutConstraint *sConstraint3 = [NSLayoutConstraint constraintWithItem:ssubView attribute:NSLayoutAttributeWidth relatedBy:NSLayoutRelationGreaterThanOrEqual toItem:nil attribute:NSLayoutAttributeNotAnAttribute multiplier:1.0 constant:200];
    NSLayoutConstraint *sConstraint4 = [NSLayoutConstraint constraintWithItem:ssubView attribute:NSLayoutAttributeHeight relatedBy:NSLayoutRelationGreaterThanOrEqual toItem:nil attribute:NSLayoutAttributeNotAnAttribute multiplier:1.0 constant:100];
    
    NSArray *array2 = [NSArray arrayWithObjects:sContraint1, sContraint2,sConstraint3,sConstraint4, nil];
    [self.view addConstraints: array2];
    
```


这里有几个注意点

- 如果是设置view自身的属性，不涉及到与其他view的位置约束关系。比如view自身的宽、高等约束时，方法constraintWithItem:的第四个参数view2（secondItem）应设为nil；且第五个参数attire（secondAttribute）应设为NSLayoutAttributeNotAnAttribute 。

- 在设置宽和高这两个约束时，relatedBy参数使用的是 NSLayoutRelationGreaterThanOrEqual，而不是 NSLayoutRelationEqual。因为 Auto Layout 是相对布局，所以通常你不应该直接设置宽度和高度这种固定不变的值，除非你很确定视图的宽度或高度需要保持不变。



