#Masonry自动布局框架的使用

###Masonry介绍

Masonry是一个对系统NSLayoutConstraint进行封装的第三方自动布局框架，采用链式编程的方式提供给开发者API。系统AutoLayout支持的操作，Masonry都支持，相比系统API功能来说，Masonry是有过之而无不及。

Masonry采取了链式编程的方式，代码理解起来非常清晰易懂，而且写完之后代码量看起来非常少。

github地址：[https://github.com/SnapKit/Masonry](https://github.com/SnapKit/Masonry)

####集成方式
CocoaPods:在Podfile中加入以下代码，执行pod install 命令即可
>pod 'Masonry'

####注意事项

- 在使用Masonry添加约束之前，需要在addSubview之后才能使用，否则会导致崩溃。
- 在添加约束时初学者经常会出现一些错误，约束出现问题的原因一般就是两种：约束冲突和缺少约束。对于这两种问题，可以通过调试和log排查。
- 之前使用Interface Builder添加约束，如果约束有错误直接就可以看出来，并且会以红色或者黄色警告体现出来。而Masonry则不会直观的体现出来，而是以运行过程中崩溃或者打印异常log体现，所以这也是手写代码进行AutoLayout的一个缺点。


####基础使用

基础api

````
mas_makeConstraints()    添加约束
mas_remakeConstraints()  移除之前的约束，重新添加新的约束
mas_updateConstraints()  更新约束
 
equalTo()       参数是对象类型，一般是视图对象或者mas_width这样的坐标系对象
mas_equalTo()   和上面功能相同，参数可以传递基础数据类型对象，可以理解为比上面的API更强大
 
width()         用来表示宽度，例如代表view的宽度
mas_width()     用来获取宽度的值。和上面的区别在于，一个代表某个坐标系对象，一个用来获取坐标系对象的值

````

更新约束

````
- (void)updateConstraintsIfNeeded  调用此方法，如果有标记为需要重新布局的约束，则立即进行重新布局，内部会调用updateConstraints方法
- (void)updateConstraints          重写此方法，内部实现自定义布局过程
- (BOOL)needsUpdateConstraints     当前是否需要重新布局，内部会判断当前有没有被标记的约束
- (void)setNeedsUpdateConstraints  标记需要进行重新布局
````

####常用方法

设置内边距

````
/** 
 设置yellow视图和self.view等大，并且有10的内边距。
 注意根据UIView的坐标系，下面right和bottom进行了取反。所以不能写成下面这样，否则right、bottom这两个方向会出现问题。
 make.edges.equalTo(self.view).with.offset(10);
 
 除了下面例子中的offset()方法，还有针对不同坐标系的centerOffset()、sizeOffset()、valueOffset()之类的方法。
 */
[self.yellowView mas_makeConstraints:^(MASConstraintMaker *make) {
    make.left.equalTo(self.view).with.offset(10);
    make.top.equalTo(self.view).with.offset(10);
    make.right.equalTo(self.view).with.offset(-10);
    make.bottom.equalTo(self.view).with.offset(-10);
}];
````
通过insets简化设置内边距的方法

````
// 下面的方法和上面例子等价，区别在于使用insets()方法。
[self.blueView mas_makeConstraints:^(MASConstraintMaker *make) {
    // 下、右不需要写负号，insets方法中已经为我们做了取反的操作了。
    make.edges.equalTo(self.view).with.insets(UIEdgeInsetsMake(10, 10, 10, 10));
}];
````

更新约束

````
// 设置greenView的center和size，这样就可以达到简单进行约束的目的
[self.greenView mas_makeConstraints:^(MASConstraintMaker *make) {
    make.center.equalTo(self.view);
    // 这里通过mas_equalTo给size设置了基础数据类型的参数，参数为CGSize的结构体
    make.size.mas_equalTo(CGSizeMake(300, 300));
}];
 
// 为了更清楚的看出约束变化的效果，在显示两秒后更新约束。
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2.f * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
    [self.greenView mas_updateConstraints:^(MASConstraintMaker *make) {
        make.centerX.equalTo(self.view).offset(100);
        make.size.mas_equalTo(CGSizeMake(100, 100));
    }];
});
````

大于等于和小于等于某个值的约束

````
[self.textLabel mas_makeConstraints:^(MASConstraintMaker *make) {
    make.center.equalTo(self.view);
    // 设置宽度小于等于200
    make.width.lessThanOrEqualTo(@200);
    // 设置高度大于等于10
    make.height.greaterThanOrEqualTo(@(10));
}];
//label会根据文字内容的多少自动调节大小（需设置lines为0）
````

使用基础数据类型当做参数

````
/** 
 如果想使用基础数据类型当做参数，Masonry为我们提供了"mas_xx"格式的宏定义。
 这些宏定义会将传入的基础数据类型转换为NSNumber类型，这个过程叫做封箱(Auto Boxing)。
  
 "mas_xx"开头的宏定义，内部都是通过MASBoxValue()函数实现的。
 这样的宏定义主要有四个，分别是mas_equalTo()、mas_offset()和大于等于、小于等于四个。
 */
[self.redView mas_makeConstraints:^(MASConstraintMaker *make) {
    make.center.equalTo(self.view);
    make.width.mas_equalTo(100);
    make.height.mas_equalTo(100);
}];
````


设置约束优先级

````
/** 
 Masonry为我们提供了三个默认的方法，priorityLow()、priorityMedium()、priorityHigh()，这三个方法内部对应着不同的默认优先级。
 除了这三个方法，我们也可以自己设置优先级的值，可以通过priority()方法来设置。
 */
[self.redView mas_makeConstraints:^(MASConstraintMaker *make) {
    make.center.equalTo(self.view);
    make.width.equalTo(self.view).priorityLow();
    make.width.mas_equalTo(20).priorityHigh();
    make.height.equalTo(self.view).priority(200);
    make.height.mas_equalTo(100).priority(1000);
}];
````
````
Masonry也帮我们定义好了一些默认的优先级常量，分别对应着不同的数值，优先级最大数值是1000。
static const MASLayoutPriority MASLayoutPriorityRequired = UILayoutPriorityRequired;
static const MASLayoutPriority MASLayoutPriorityDefaultHigh = UILayoutPriorityDefaultHigh;
static const MASLayoutPriority MASLayoutPriorityDefaultMedium = 500;
static const MASLayoutPriority MASLayoutPriorityDefaultLow = UILayoutPriorityDefaultLow;
static const MASLayoutPriority MASLayoutPriorityFittingSizeLevel = UILayoutPriorityFittingSizeLevel;
````
设置约束比例

````
// 设置当前约束值乘以多少，例如这个例子是redView的宽度是self.view宽度的0.2倍。
[self.redView mas_makeConstraints:^(MASConstraintMaker *make) {
    make.center.equalTo(self.view);
    make.height.mas_equalTo(30);
    make.width.equalTo(self.view).multipliedBy(0.2);
}];
````

更多内容详见[原文](http://www.jianshu.com/p/ea74b230c70d)




