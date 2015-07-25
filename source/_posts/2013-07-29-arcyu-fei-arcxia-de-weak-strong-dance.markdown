---
layout: post
title: "ARC与非ARC下的Weak-Strong Dance"
date: 2013-07-29 17:16
comments: true
categories: iOS
---
## ARC
在使用block过程中，经常会遇到`retain cycle`的问题，例如：
```objc
- (void)dealloc
{
	[[NSNotificationCenter defaultCenter] removeObserver:_observer];
}

- (void)loadView
{
	[super loadView];
			
	_observer = [[NSNotificationCenter defaultCenter] addObserverForName:@"testKey"
																  object:nil
																   queue:nil
															  usingBlock:^(NSNotification *note) {
		[self dismissModalViewControllerAnimated:YES];	
	}];
}
```
在block中用到了self，self会被block retain，而\_observer会copy一份该block，就是说\_observer间接持有self，同时当前的self也会retain \_observer，最终导致self持有_observer，\_observer持有self，形成`retain cycle`。
<!-- more -->
对于在block中的`retain cycle`，在2011 WWDC Session #322 (Objective-C Advancements in Depth)有一个解决方案`weak-strong dance`，很漂亮的名字。其实现如下：
```objc
- (void)dealloc
{
	[[NSNotificationCenter defaultCenter] removeObserver:_observer];
}

- (void)loadView
{
	[super loadView];
			
	__weak TestViewController *wself = self;
	_observer = [[NSNotificationCenter defaultCenter] addObserverForName:@"testKey"
														     	  object:nil
														   		   queue:nil
															  usingBlock:^(NSNotification *note) {
		TestViewController *sself = wself;
		[sself dismissModalViewControllerAnimated:YES];
	}];
}
```
在block中使用self之前先用一个`__weak`变量引用self，导致block不会retain self，打破retain cycle，然后在block中使用wself之前先用`__strong`类型变量引用wself，以确保使用过程中不会dealloc。简而言之就是推迟对self的retain，在使用时才进行retain。这有点像lazy loading的意思。

注：iOS5以下没有`__weak`，则需使用`__unsafe_unretained`。

## 非ARC
在非ARC环境中，显然之前的使用的`__weak`或`__unsafe_unretained`将会是无效的，那么我们需使用另外一种方法来代替，这里就需要用到`__block`。

`__block`在ARC和非ARC中有点细微的差别（[Automatic Reference Counting : Blocks](http://www.mikeash.com/pyblog/friday-qa-2011-09-30-automatic-reference-counting.html)）：

- 在ARC中，`__block`会自动进行retain
- 在非ARC中，`__block`不会自动进行retain

因此首先要注意的一点就是用`__block`打破`retain cycle`的方法仅在非ARC下有效，下面是非ARC的`weak-strong dance`：
```objc
- (void)dealloc
{
	[[NSNotificationCenter defaultCenter] removeObserver:_observer];
	[_observer release];

	[super dealloc];
}

- (void)loadView
{
	[super loadView];
			
	__block TestViewController *bself = self;
	_observer = [[NSNotificationCenter defaultCenter] addObserverForName:@"testKey"
					            								  object:nil
																   queue:nil
																 ngBlock:^(NSNotification *note) {
		[bself retain];
		[bself dismissModalViewControllerAnimated:YES];
		[bself release];
	}];
}
```
将self赋值为`__block`类型变量，在非ARC中`__block`类型变量不会进行retain，从而打破retain cycle，然后在使用bself前进行retain，以确保在使用过程中不会dealloc。
