---
layout:     post                    # 使用的布局（不需要改）
title:      Learning Autoresizing Mask               # 标题
subtitle:    #副标题
date:       2017-11-19              # 时间
author:     Xiongbin Zhao                     # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - iOS
---

<!--As an iOS developer, we spend a lot of time handling the frame and constraints for our views in order to show the correct and perfect content/animation for our user. Apple introduced Auto Layout to help developer make their app easily fit into different size of devices. However, we still have the option to use Autoresizing to dynamically change the frame of our views based on different devices' sizes.

However, with the existence of both techniques, sometimes, we will see some wired behaviours for our views when trying to position and size them. These wired behaviours are produced from the conflict between Autoresizing and Auto Layout. Today, we will look at how Autoresizing and Auto Layout work so that we will have a better idea what's happening when seeing those unexpected behaviours of our views.

##Frame and Constraints
To start with, let's review again what is Frame and Constraints of a UIView.

###Frame

Frame is a property which describes the view's location and size in its superview's coordinate system.

###Constraints:
Constraint is a set of rule between different elements on your screen to govern the size of position.

So, at runtime, system will use Constraints to calculate the Frame for your views. After using Constraints, Frame is a result of your Constraints.


##What is Autoresizing


###What Autoresizing can do
In short, we can use autoresizing to defined the UI Based relationship of a view and its super view. As a result, when a super view's frame changed, its subviews will get notified and subviews' frame will be changed in order to safatify the Autoresizing rules we've defined.

Two most important properties for Autoresizing are **autoresizingMask** and **translateAutoresizingMaskIntoConstraints** in UIView.




For margins, height and width, if we didn't specify the autoreszingMask, the value will be fixed. For example, if we set the autoresizingMask of our view to `UIViewAutoresizingFlexibleWidth`, when its super view's frame change, the width of our view will change in order to keep margins and height the same number because they are not flexible.-->





# Autoresizing
After Auto Layout was introduced, it become the first choice for iOS developer trying to make their apps look perfect on different screen. However, I think it is still necessary to understand autoresizing, which is a auto sizing technique predate Auto Layout. The reasons for this are:

1. Autoresizing is still an option for doing Auto sizing
2. Autoresizing will have conflict with Auto Layout if we use them at the same time. It is helpful to understand this technique so that we can better troubleshooting when seeing some wired bbehaviours of our views.

## Why look at this issue
There was a view I needed to use in multiple TableView/CollectionView cell. In order to reuse this view, I created a class and xib file for this view, so that I can create a generic cell add then add the custom view as subview.

However, after creating the view from xib and updated its frame, the size of the view became different after it shown up in the cell. The frame of my subview was changed during runtime, which is caused by autoresizing.

## What is autoresizing
We can use a example to demonstrate the use case of autoresizing. Assumpting that we want to add a view to our screen, which will always have 10 spacing between the view and our screen's edges, and also have 100 points spacing to the Top.
### Without Autoresizing
```
// Screen size for iPhone Plus is (414, 736)
UIView *subView = [[UIView alloc] initWithFrame:CGRectMake(10, 100, 394, 200)];
subView.backgroundColor = [UIColor redColor];
[self.view addSubview:subView];
```
The code above will create our desired view for us. However, the issue about this approach is that when we rotate our phone from portarit to landscape, it will keep the same size (394, 200). But our goal is to keep the spacing between edges as 10 points, which means on an iPhone plus, the size should be changed to (716, 200).

To handle this, we need to override this method to change the subview's size.

```
override func viewWillTransitionToSize(size: CGSize, withTransitionCoordinator coordinator: UIViewControllerTransitionCoordinator)

```
### With Autoresizing
To handle case like this, Apple introduced Autoresizing. Autoresizing is a inner constraints system created by system at runtime. Instead of positioning our subview using frame, let's think about these six values:

* Left Margin
* Right Margin
* Top Margin
* Bottom Margin
* Height
* Width

Using these six values, we can position and size a subview in its super view. Let's use Autosizing to handle the same example:

```
// Screen size for iPhone Plus is (414, 736)
UIView *subView = [[UIView alloc] initWithFrame:CGRectMake(10, 100, 394, 200)];
subview.translateAutoresizingIntoConstraints = YES;
subview.autoresizingMask = UIViewAutoresizingWidthFlexible;
subView.backgroundColor = [UIColor redColor];
[self.view addSubview:subView];
```
By setting the correct autoresizingMask of our subview, when the device is rotated, our subview will keep its sides margins and height, but increase the width in order to keep the margins because we set our autoresizingMask to UIViewAutoresizingWidthFlexible which means our width is flexible. Let's look at these two properties: **translateAutoresizingMaskIntoConstraints** and **autoresizingMask**
####- translateAutoresizingMaskIntoConstraints
```
@property(nonatomic) BOOL translatesAutoresizingMaskIntoConstraints NS_AVAILABLE_IOS(6_0); // Default YES
```
**translatesAutoresizingMaskIntoConstraints** is a bool value to specify if we want to use autoresizing for our view. The default value of this property is YES when we create a UIView from code, as a result, system at runtime, will create constraints for our view based on our autoresizingMask.


####- autoresizingMask
autoresizingMask is a set of rules we can define to constraint our UIView.

```
//
//  UIView.h
//  UIKit

typedef NS_OPTIONS(NSUInteger, UIViewAutoresizing) {
    UIViewAutoresizingNone                 = 0,
    UIViewAutoresizingFlexibleLeftMargin   = 1 << 0,
    UIViewAutoresizingFlexibleWidth        = 1 << 1,
    UIViewAutoresizingFlexibleRightMargin  = 1 << 2,
    UIViewAutoresizingFlexibleTopMargin    = 1 << 3,
    UIViewAutoresizingFlexibleHeight       = 1 << 4,
    UIViewAutoresizingFlexibleBottomMargin = 1 << 5
};
```

`UIViewAutoresizingNone`: There's no autoresizing rules for our view and its super view. So, onced we set a frame for our view, our view will keep the exact frmae regardless of its super view's frame change.

`UIViewAutoresizingFlexibleLeftMargin`: The left padding between our view and its super view is flexible.

`UIViewAutoresizingFlexibleWidth`: The left width for our view is flexible.

`UIViewAutoresizingFlexibleRightMargin`: The right padding between our view and its super view is flexible.

`UIViewAutoresizingFlexibleTopMargin`: The top padding between our view and its super view is flexible.

`UIViewAutoresizingFlexibleHeight`: The height for our viewis flexible.

`UIViewAutoresizingFlexibleBottomMargin`: The bottom padding between our view and its super view is flexible.

## Conclusion
Without Autosizing, by setting frame, we are only setting the absolute size and position of our views. With Autosizing, system will create constraints for us based on autoresizingMask and frame we set.
