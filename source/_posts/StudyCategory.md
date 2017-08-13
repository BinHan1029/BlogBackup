---
title: 关于 Objective-C：Category 的一点理解
date: 2017-02-18 20:11:07
tags:
- 移动开发
- iOS
---

category 是 OC 2.0后添加的新特性，主要作用是给已经存在的类添加方法，当然秉着类的单一职责原则和接口隔离原则我们通常会将不同功能的类扩展分别写在不同的文件中，这样不仅减少了单个文件的体积，同时可以对 category 进行按需加载。

### category 的定义
所有的 OC 类和对象在 runtime 层其实都是 strcut，比如 objc_class、objc_object， 甚至 block(__block_impl) 还定义有自己的 isa 指针，category 也是如此。

``` objc
typedef struct category_t {
    const char *name;
    classref_t cls;
    struct method_list_t *instanceMethods;
    struct method_list_t *classMethods;
    struct protocol_list_t *protocols;
    struct property_list_t *instanceProperties;
} category_t;
```
<!-- more -->

定义中包含了：

1. 类的名字（name）
1. 类（cls）
1. category中所有给类添加的实例方法的列表（instanceMethods）
1. category中所有添加的类方法的列表（classMethods）
1. category实现的所有协议的列表（protocols）
1. category中添加的所有属性（instanceProperties）

这里我们着重关心一下 instanceMethods，因为 category 中给类添加的实例方法最终就会放在这个列表中，如果有你多个 category，那么在程序加载前会将所有的实例方法列表遍历组装成一个大的实例方法列表，最终会添加到 objc_class 的 methodLists 中，而 category 中的方法会在新的方法列表前面，所以当我们给一个对象发送消息时，会优先查找到 category 中的方法，当然在这之前会优先查找一下方法缓存 objc_cache。

### category 中的"覆盖"
如果 category 定义跟系统类方法重名会出现什么情况呢？
这里我们给 NSString 扩展添加一个系统同名的方法 

``` objc
- (unichar)characterAtIndex:(NSUInteger)index;
```

首先如果我们真的这么做的话，编译器直接就会报警告:
Category is implementing a method which will also be implemented by its primary class.

> If the name of a method declared in a category is the same as a method in the original class, or a 
> method in another category on the same class (or even a superclass), the behavior is undefined as to 
> which method implementation is used at runtime.
 
这是苹果 [Category Method Name Clashes](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/CustomizingExistingClasses/CustomizingExistingClasses.html) 中间一段的解释，所以我们在给标准的 Cocoa 类添加重名类方法在运行时可能会出现问题。

如果我们忽略这个警告，代码是可以正常编译且运行的，而且通过 class_copyMethodList 获取列表后然后遍历所有的方法：

``` objc
unsigned int methodCount;
Method *methodList = class_copyMethodList([NSString class], &methodCount);
for (NSInteger i = 0; i < methodCount; i++)
{
	Method method = methodList[i];
	NSString *methodName = [NSString stringWithCString:sel_getName(method_getName(method)) encoding:NSUTF8StringEncoding];
	NSLog(@"methodName = %@", methodName);
}
```
会发现列表中存在两个 characterAtIndex 方法，这里如果我们进一步通过遍历调用 characterAtIndex 方法的话，结果两次执行的都是系统类方法的实现，而我们自己 category 中的方法是无法被执行的，也就是说如果我们 category 中的方法跟系统的方法重名的话，那么方法在 class_copyMethodList 中会存在两份，但调用仍旧执行系统的定义的。

如果 category 方法名跟系统类扩展方法重名会是一样的结论么？这里测试发现，同样运行时会有两个同名方法，但是如果调用的话会执行我们定义的，当然具体调用谁取决于 category 编译时的先后顺序，如果我们编写了多个类扩展，则可以在 Compile Sources 中进行修改。

当然前面也说了标准的 Cocoa 类是这样，如果是自定义类的类方法和类扩展方法重名的话，调用执行的话依旧是扩展中的方法，这跟在 class_copyMethodList 方法列表查找的顺序也是一致的。

### category 的一些姿势

###### 替代继承
	
继承很大的一个缺点就是打破了类的封装，基类完全向子类暴露了实现细节。且继承是一种高耦合的设计，违背了面向对象的设计原则。将类的实现分散到便于管理的数个分类之中，这样既便于管理，还可以隐藏实现细节。另外这样也更便于调试，因为分类中的方法有自己的 symbol name，当抛出错误时，可以快速根据提示进行查找。
	
###### hook 原方法

我们可以在类加载 load 的时候进行 hook。典型的比如页面无埋点统计，我们只需要替换到系统控制器的几个生命周期函数既可以。
或者还可以做的更多，比如数组越界防崩溃，KVO 成对的添加和移除监听，因为多线程下的重复添加或移除将直接导致应用 crash。

###### 添加关联属性
属性是数据封装的一种方式，但是在分类中是无法合成实例变量的，当然有源码的 extension 可以添加属性且自动生成对应的实例变量。
分类中添加属性编译期既会抛出警告，当然可以通过 @dynamic 暂时让编译器忽略，在运行时通过消息转发机制拦截此方法那么或许可以采取这种做法。比如一个只读属性，只实现它的get方法是完全没有问题的。

关联对象可以解决分类中无法合成实例变量的问题，当然这里需要特别注意内存管理。另外一点我们还可以通过一些特殊的姿势给关联属性添加 weak 对象，这可能需要一点小技巧，虽然 objc_AssociationPolicy 并没有提供 weak 内存管理方式。比如关联 block，在 block 中引用 weak 对象。当然通过这种方式，我们还可以实现数组及字典中“弱引用”别的元素。

``` objc
-(void)setCurrentController:(UIViewController *)currentController
{
     objc_setAssociatedObject(self, @selector(currentController), currentController, OBJC_ASSOCIATION_RETAIN);
}

-(UIViewController *)currentController
{
    return objc_getAssociatedObject(self, _cmd);
}
```
###### private 分类
一些经常使用的 API 可能非常适合在程序库内使用，但是又不便于暴露给外部，创建 private 分类的头文件不一起对外公开，这样就可以实现私有方法。当然通过 linkmap 我们还是可以查看到所有的第三方库定义的所有方法。

### 静态库中的 category
由于 Objective-C 并不为每个函数定义 linker symbol，它只为每个类生成 linker symbol，也就是说类被编译链接后，类扩展将不会被链接，那么在运行时调用扩展的方法会抛出异常 exception “selector not recognized”，所以在编译时还需要 Other linker flags 下设置参数 -all_load / -force_load。

相关资料：

[https://tech.meituan.com/DiveIntoCategory.html](https://tech.meituan.com/DiveIntoCategory.html)

[http://blog.csdn.net/wlq861025/article/details/51888782](http://blog.csdn.net/wlq861025/article/details/51888782)