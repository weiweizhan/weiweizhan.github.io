---
layout: post
title: iOS中nib文件的加载过程
---

在TableViewController中，经常看到这样的代码：

{% highlight sql %}
	- (NVClientAccountViewCell *)loadCell{
    NSArray *views = [[NSBundle mainBundle] loadNibNamed:@"NVClientAccountViewCell" owner:nil options:nil];
    return (NVClientAccountViewCell *)[views objectAtIndex:0];
} {% endhighlight %}

利用这段代码通过nib文件构造了TableViewCell。


#### nib文件的加载

1. 将nib文件的内容和引用资源文件加载到内存。

2. 解压nib文件中的object graph数据，针对不同对象，使用不同的方法初始化这些对象。符合NSCoding协议的对象利用initWithCoder:来初始化，这其中包括UIView和UIViewController的所有子类。自定义对象接收到init消息。

3. 重建nib文件中对象(action, outlet)之间的连接，包括file’s owner和其他占位对象。
outlet连接: 利用setValue:forKey方法重建各个outlet之间的连接。该方法寻找<font color="red">合适的accessor方法，否则失败</font>。设置outlet会对观察者发出KVO通知，这些通知可能在所有内部对象重建连接前发送，一定在awakeFromNib前发送。
action连接: 利用UIControl对象的addTarget:action:forControlEvents:方法来配置action。如果target为nil，action则由responder chain处理。

4. 向nib文件中定义了selector的合适对象发送awakeFromNib消息。只发送给nib加载代码初始化的接口对象，不发送给file’s owner, first responder和其他占位对象。

5. 展示。


#### 在nib文件中定义IBOutlet的三种方式

1. 将Outlet定义为@property，放在h文件中。
不推荐，该方法将IBOutlet完全暴露出来，不符合封装的要求。

2. 将Outlet定义为@property，通过category方式使其成为类的私有变量，放在m文件中。

{% highlight sql %}
@interface MyViewCell()
@property (weak, nonatomic) IBOutlet UILabel *label;
@property (weak, nonatomic) IBOutlet UIImageView *image;
@end{% endhighlight %}

3. 直接将Outlet设置成为类的私有变量。

{% highlight sql %}
@implementation MyViewCell{
     IBOutlet UILabel *titleLbl;
     IBOutlet UIImageView *iconImg;
}

- (void)setTitle:(NSString*)labelText{
    titleLbl.text = labelText;
}

- (void)setIcon:(NSString*) theIcon{
    iconImg.image = [UIImage imageNamed:theIcon];
}{% endhighlight %}

在加载nib文件过程中，重建outlet的时候，会调用outlet的getter和setter方法初始化outlet。如果在方法三中，将UILabel的名字设成title，就会将setTitle方法当成title重载后的setter方法，用它来初始化UILabel，造成outlet初始化失败，为nil。


[参考文献]

1. [Resource Programming Guide - nib files](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/LoadingResources/CocoaNibs/CocoaNibs.html)


