[TOC]
# 关于我的仓库

- 这篇文章是我为面试准备的iOS基础知识学习中的一篇
- 我将准备面试中找到的所有学习资料，写的Demo，写的博客都放在了这个仓库里[iOS-Engineer-Interview](https://github.com/KevinAshen/iOS-Engineer-Interview)
- 欢迎star👏👏
- 其中的博客在简书，CSDN都有发布
- 博客中提到的相关的代码Demo可以在仓库里相应的文件夹里找到

# 前言
- 本文设计到关于iOS 属性关键字的内容，作为面试指定必考考点，会尽量多从各个关键字的作用，区别进行讲述
- weak关键字的主要分析都放在runtime中的weak解析里面，这篇会尽量把copy好好分析下
- 根据《禅与 Objective-C 编程艺术》中的观点，我会按照原子性，读写权限和内存管理的顺序来介绍
- 这里可以插一嘴，这么写的意思由于我们都知道三类属性关键字我们重要程度/修改可能性是按照内存管理 > 读写权限 >原子操作
- 那么按照人体工程学法则【把这个关键词记住，可以装个逼👍】，作为程序员的我们，一般在看一行属性的时候会习惯性地从右往左进行查看，那么就会先看到最重要的内存管理
- 细节就是魔鬼
# 原子性

- 原子性就nonatomic和atomic两个关键字

## atomic与nonatomic

- atomic是默认的原子操作关键字，atomic也就代表其具有原子性
- atomic 属性关键字会给该 property 的 getter和setter方法加锁，但不能保证线程一定安全并且会带来更多损耗。
- 而nonatomic就是非原子性的，不会给setter，getter方法加锁，这也是我们默认的方法
- 原因：

  1. 使用atomic也无法保证线程一定能安全了
  2. 对于涉及到多线程安全的场景，我们常常是自己去额外加同步
  3. 这样也可以加快运行速度

## 为什么会出现线程不安全

- 这部分主要要看哲神的博客：[从atomic关键字说到多线程安全（内含iOS给代码加锁方法总结）](https://blog.csdn.net/qq_42347755/article/details/97753122)
- 补充下地址总线的知识：地址总线 (Address Bus；又称：位址总线) 属于一种电脑总线 （一部份），是由CPU 或有DMA 能力的单元，用来沟通这些单元想要存取（读取/写入）电脑[内存](https://baike.baidu.com/item/内存/103614)元件/地方的实体位址。数据总线的宽度，随可寻址的内存组件大小而变，决定有多少的内存可以被访问。
- atomic所说的线程安全只是保证了getter和setter存取方法的线程安全，并不能保证整个对象是线程安全的。
- 这里推荐的哲神的文章其中对于地址总线方面的看法还是一个猜想，到底怎么样也是没人知道

# 读写权限

- 读写权限这一块很好理解，就是一个readwrite以及一个readonly
- readwrite：其修饰的属性拥有“获取方法”（getter）与“设置方法”（setter）。
- readonly：其修饰的属性仅拥有获取方法。
- 这一块可以联想到runtime源码中，关于一个对象结构体中要有rw以及ro结构体，他们其实就是readwrite以及readonly的区别

# 内存管理

- 终于到重头戏了，其实对于上面这两个部分，敲过一段时间的iOS开发者心里都会有数，像原子性，无脑nonatomic，像读写权限，一般直接默认不写，真正有难度，比较复杂的都在内存管理部分
- 下面先把最重要的四个：assgin，strong，weak，copy讲一下

## 内存管理四天王——ASWC

### assgin

- assgin用于基本数据类型的默认属性关键字，它等于就是啥也没干，不会使得引用计数加一，赋值的时候就是简单的赋值，简单来说，就是用于不需要非对象
- 这里要讲清楚一个概念，在探究内存管理属性关键字的时候，会大量涉及到引用计数+1的概念，这里要注意，都指的是赋值的时候会不会引用计数加一。
- 什么意思呢？之前有是看了看组里大一的同学学习，印象很深的有一个同学写了个NSInteger *obj = xxxx
- 这样的写法可能是来自于认为以NS开头的都是OC对象，可惜的是，如果查看过NSInteger的定义，就这么一句：typedef long NSInteger;
- 所以，我们讲到的基本数据类型其实就是非对象的数据类型，学到现在，你看到已经知道，指针是存在栈区，对象本身是存在堆区，两者本身不是连在一起的
- 而指针里面会存着对象的地址，也就是指针指向了对象
- 霹雳吧啦讲了一大堆，我们拉回来看下assgin的作用，由于是直接赋值，不改变引用计数，这就限制了我们只能用它来来修饰非对象
- 假如我们使用它来修饰一个对象，首先会报警告【Assigning retained object to unsafe property; object will be released after assignment】；我们假设有A，B两个指针指向某个对象，在有引用计数管理的情况下，其引用计数不为0就不会释放，而如果我们用的是assgin，就会导致该对象直接被释放了。A，B指针没有被置nil，出现野指针
- 而用它来修饰基本数据类型，由于它们都是被分配在栈上的，栈的内存由系统总结自动处理回收

### strong

- strong是对象的默认内存管理关键字，在学过ARC中的strong修饰符后，基本就不会有什么问题了
- 是对象的默认属性关键字，此特质表明该属性定义了一种“持有关系”，为这种属性设置新值时，设置方法会先保留（retain）新值，并释放（release）旧值，然后再将新值设置上去。
- 这个感觉真没啥好说的，看学长面试讲到strong都很尬

### weak

- 此特质表明该属性定义了一种“非持有关系”。为这种属性设置新值时，设置方法既不保留（retain）新值，也不释放（release）旧值。此特质同assign类似，然而在属性对象所指的对象遭到摧毁（dealloc）时，属性值也会清空（置nil）。
- weak要深挖就参看我runtime系列文章中相关部分就行

### copy

- 终于到重头戏copy了，我们好好研究下
- 此特质表达的所属关系与strong类似。然而设置方法并不保留新值，而是将其拷贝（copy）
- 这一部分推荐看哲神的[详解iOS开发中复制对象](https://blog.csdn.net/qq_42347755/article/details/89057135)
- NSString、NSArray、NSDictionary 等等经常使用copy关键字，是因为他们有对应的可变类型：NSMutableString、NSMutableArray、NSMutableDictionary；
- 理解：这里我认为意思就是由于NSString存在可变类型NSMutableString，那么假设我们整了一个NSString类型的属性A，又有一个NSMutableString类型的可变字符串B，我们把B赋给A；假如我们的A使用strong的话，他就是直接指针赋值，A，B指向同一块内存，这样我们修改B，同时就会修改A；而当我们使用的是copy的时候，这样我们A持有就是一个B的不可变副本，那么不管B怎么改，A都是不变的

## 杂鱼小虾
- retain：在ARC时代，strong等于retain，引用计数会加一
- unsafe_unretained：此特质的语义和assign相同，但它适用于“对象类型”，该特质表达一种“非持有关系”，当目标对象销毁时，属性值不会自动清空（unsafe），这一点与weak有区别。

# 总结

## 默认关键字

- 基本数据：atomic,readwrite,assign
- 普通的 OC 对象: atomic,readwrite,strong

## weak与assgin的区别
- weak 策略在属性所指的对象遭到摧毁时,系统会将 weak 修饰的属性对象的指针指向 nil,在 OC 给 nil 发消息是不会有什么问题的;如果使用 assign 策略在属性所指的对象遭到摧毁时,属性对象指针还指向原来的对象,由于对象已经被销毁,这时候就产生了野指针,如果这时候在给此对象发送消息,很容造成程序奔溃
- assigin 可以用于修饰非 OC 对象,而 weak 必须用于 OC 对象。


