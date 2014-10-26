---
layout: post
title: "Developing iOS 7 Apps for iPhone and iPad - 学习笔记"
description: 
headline: 
date:   2014-09-20 20:33
category: 机械境
tags: []
image: 
  feature: 
comments: true
mathjax: 
---

## Lecture 4 - Foundation and Attributed Strings

### 1. 强制类型转换是不安全的（它会欺骗编译器），在做强制类型转换之前，我们必须检查这个id指向的对象来自哪个类。
Casting is unsafe, we have to check which class the id is from before doing this.
<!-- more -->
{% highlight objc %}
￼@interface Vehicle
- (void)move;
@end
@interface Ship : Vehicle
- (void)shoot;
@end
Ship *s = [[Ship alloc] init];
[s shoot];
[s move];
Vehicle *v = s;
[v shoot];
id obj = ...;
[obj shoot];
[obj someMethodNameThatNoObjectAnywhereRespondsTo];
// Object Typing
￼￼NSString *hello = @"hello";
[hello shoot];
Ship *helloShip = (Ship *)hello;
[helloShip shoot]; // We’ve forced the compiler to ignore the object type by “casting” in line. “All’s well,” the compiler thinks. Guaranteed crash at runtime.
{% endhighlight %}

### 2. NSObejct 的 description 方法
description method of NSObject  
为了比如调试的目的，实现这个方法是很方便的。
It's very convient to implement this method in your custom class, for purpose such as debugging.

### 3. NSObject 的 copy 和 mutablecopy 方法
copy and mutablecopy methods of NSObejct   
NSObject 或者其他子类 的这两个方法一般是没有被实现的，除了 **NSArray** 和 **NSDictionary** (包括 mutable 类型)  
These two method of NSObject and its subclass are not implemented in general except **NSArray** & **NSDictionary** (including mutable ones).  
{% highlight objc %}
- (id)copy; // not all objects implement mechanism (raises exception if not)
- (id)mutableCopy; // not all objects implement mechanism (raises exception if not)
{% endhighlight %}

### 4. 利用好 Selector  
Take advantage of Selector  
Method testing methods take a selector (SEL)
Special @selector() directive turns the name of a method into a selector  
{% highlight objc %}
if ([obj respondsToSelector:@selector(shoot)]) {
    [obj shoot];
} else if ([obj respondsToSelector:@selector(shootAt:)]) {
    [obj shootAt:target];
}
{% endhighlight %}
SEL is the Objective-C “type” for a selector  
{% highlight objc %}
SEL shootSelector = @selector(shoot);
SEL shootAtSelector = @selector(shootAt:);
SEL moveToSelector = @selector(moveTo:withPenColor:);
{% endhighlight %}  
如果你有一个 SEL 类型的变量，你也可以请求一个对象使用它...  
If you have a SEL, you can also ask an object to perform it ...  
Using the `performSelector:` or `performSelector:withObject: methods in NSObject [obj performSelector:shootSelector];`
`[obj performSelector:shootAtSelector withObject:coordinate];`  
Using makeObjectsPerformSelector: methods in NSArray
{% highlight objc %}
[array makeObjectsPerformSelector:shootSelector]; // cool, huh?
[array makeObjectsPerformSelector:shootAtSelector withObject:target]; // target is an id
{% endhighlight %}  
￼In UIButton, `- (void)addTarget:(id)anObject action:(SEL)action ...;` 
{% highlight objc %}
[button addTarget:self action:@selector(digitPressed:) ...];
{% endhighlight %}  

### 5. 当心 Property List
Take care of Property List 

* The term “Property List” just means a collection of collections   
It’s just a phrase (not a language thing). It means any graph of objects containing only:  
NSArray, NSDictionary, NSNumber, NSString, NSDate, NSData (or mutable subclasses thereof) 
* An NSArray is a Property List if all its members are too
So an NSArray of NSString is a Property List.
So is an NSArray of NSArray as long as those NSArray’s members are Property Lists.
* An NSDictionary is one only if all keys and values are too  
An NSArray of NSDictionarys whose keys are NSStrings and values are NSNumbers is one.
* Why define this term?
Because the SDK has a number of methods which operate on Property Lists.
Usually to read them from somewhere or write them out to somewhere. Example:
`- (void)writeToFile:(NSString *)path atomically:(BOOL)atom;`  
This can (only) be sent to an NSArray or NSDictionary that contains only Property List objects.

## Lecture 5 - Protocol, Block, Animation

### 1. 协议略  

### 2. Block 外部变量是只读的，如果想实现读写，变量前需要加 __block 关键字  

{% highlight objc %}
__block BOOL stoppedEarly = NO;
double stopValue = 53.5;
[aDictionary enumerateKeysAndObjectsUsingBlock:^(id key, id value, BOOL *stop) {
    NSLog(@“value for key %@ is %@”, key, value);
    if ([@“ENOUGH” isEqualToString:key] || ([value doubleValue] == stopValue)) {
￼*stop = YES;
        stoppedEarly = YES;  // this is legal now
} }];
{% endhighlight %}

### 3. Block 会对调用的对象产生强引用，直到Block执行结束。这也容易导致循环引用。
正确做法：  
{% highlight objc %}
__weak MyClass *weakSelf = self; // even though self is strong, weakSelf is weak
Now if we reference weakSelf inside the block, then the block will not strongly point to self ... [self.myBlocks addObject:^ {
    [weakSelf doSomething];
}];
{% endhighlight %}

## Lecture 9 - Animation & Autolayout

### 1. 3种方法设置约束(Constraints)
1. ib里使用辅助线布局，然后使用 **Reset to Suggested Constraints** (⌥⇪⌘=)
2. 使用ib里的 **Add New (Alignment) Constraints** 按钮
3. 按住 Control(⌃) 从一个 View 拖动到另一个 View

### 2. 动画的几种类型
1. UIView的几个重要的属性的动画  Animation of important UIView properties 
**注意： 事实上，属性的改变是立刻执行的，但是屏幕上却是慢慢变化。** 
**IMPORTANT: The changes are made immediately, but appear on-screen over time (i.e. not instantly).**  
UIView‘s class method(s)  `animationWithDuration:...`
2. UIView外形的任何改变的动画 Animation of the appearance of arbitrary changes to a UIView  
By flipping or dissolving or curling the entire view.
UIView’s class method `transitionWithView:...`  
3. Dynamic Animator  
Specify the “physics” of animatable objects (usually UIViews).  
Gravity, pushing forces, attachments between objects, collision boundaries, etc. Let the physics happen!  

### 3. UIView 动画
- Changes to certain UIView properties can be animated over time  
**frame**  
**transform** (translation, rotation and scale)  
**alpha** (opacity)  
- Done with UIView class method and blocks  
The class method takes animation parameters and an animation block as arguments.
The animation block contains the code that makes the changes to the UIView(s).
Most also have a “completion block” to be executed when the animation is done.
The changes inside the block are made immediately (even though they will appear “over time”).   
- Animation class method in UIView  
{% highlight objc %}
+ (void)animateWithDuration:(NSTimeInterval)duration
                      delay:(NSTimeInterval)delay
                    options:(UIViewAnimationOptions)options
                 animations:(void (^)(void))animations
                 completion:(void (^)(BOOL finished))completion;
{% endhighlight %}
- UIViewAnimationOptions  
{% highlight objc %}
  BeginFromCurrentState // interrupt other, in-progress animations of these properties
  AllowUserInteraction // allow gestures to get processed while animation is in progress
  LayoutSubviews // animate the relayout of subviews along with a parent’s animation
  Repeat // repeat indefinitely
  Autoreverse // play animation forwards, then backwards
  OverrideInheritedDuration // if not set, use duration of any in-progress animation
  OverrideInheritedCurve // if not set, use curve (e.g. ease-in/out) of in-progress animation
  AllowAnimatedContent // if not set, just interpolate between current and end state image
  CurveEaseInEaseOut // slower at the beginning, normal throughout, then slow at end
  CurveEaseIn // slower at the beginning, but then constant through the rest
  CurveLinear // same speed throughout
{% endhighlight %}
- Translation Animation  
**Sometimes you want to make an entire view modification at once**   
By flipping view over`UIViewAnimationOptionsTransitionFlipFrom{Left,Right,Top,Bottom}`  
Dissolving from old to new state `UIViewAnimationOptionsTransitionCrossDissolve`  
Curling up or down `UIViewAnimationOptionsTransitionCurl{Up,Down}`  
Just put the changes inside the animations block of this UIView class method ...
{% highlight objc %}
+ (void)transitionWithView:(UIView *)view
                  duration:(NSTimeInterval)duration
                   options:(UIViewAnimationOptions)options
                animations:(void (^)(void))animations
                completion:(void (^)(BOOL finished))completion;
{% endhighlight %}

**Animating changes to the view hierarchy is slightly different**  
Animate swapping the replacement of one view with another in the view hierarchy.
{% highlight objc %}
+ (void)transitionFromView:(UIView *)fromView
                    toView:(UIView *)toView
                  duration:(NSTimeInterval)duration
                   options:(UIViewAnimationOptions)options
                completion:(void (^)(BOOL finished))completion;
{% endhighlight %}
Include `UIViewAnimationOptionShowHideTransitionViews` if you want to use the hidden property.   
Otherwise it will actually remove fromView from the view hierarchy and add toView.

## Lecture - 17 Camera, Core Motion, Application Lifecycle

### 1. 当 App 的 UI 开始/结束时收到的事件
When your application’s UI starts/stops receiving events ...  
{% highlight objc %}
- (void)applicationDidBecomeActive:(UIApplication *)sender;
- (void)applicationWillResignActive:(UIApplication *)sender;
{% endhighlight %}
任何对象都可以接收到如下的通知中心广播。Everyone gets these radio station broadcasts ...
{% highlight objc %}
UIApplicationDidBecomeActiveNotification UIApplicationWillResignActiveNotification
{% endhighlight %}
它有可能发生在用户切换到其他 app 或者一个电话打了进来。  
These might happen because user switched to another app or maybe a phone call come in. Use these notifications to pause doing stuff in your UI and then restart it later.  

### 2. 当程序进入后台 / 前台...
When you enter the background ...  
你只有短短的几秒时间去处理余下工作。  
You only get a few seconds to respond to this.  
`- (void)applicationDidEnterBackground:(UIApplication *)sender;`  
and `UIApplicationDidEnterBackgroundNotification`  
如果你需要更多时间：  
If you need more time, it is possible (see beginBackgroundTaskWithExpirationHandler:).! This is a notification for you to clean up any significant resource usage, etc.!  
**You find out when you get back to the foreground too ...**  
our Application Delegate gets ...  
`- (void)applicationWillEnterForeground:(UIApplication *)sender;` and `UIApplicationWillEnterForegroundNotification`  
Generally you undo whatever you did in DidEnterBackground  
You’ll get `applicationDidBecomeActive:` soon after receiving the above.