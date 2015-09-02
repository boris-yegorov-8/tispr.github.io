---
layout: post
title: "How to create rounded corners on a top of  UITabBar"
date: 2015-09-01 15:41:58 -0700
comments: true
categories: [Swift, iOS]
author: Andrei Pitsko
github-url: https://github.com/Pitsko
twitter-url: https://twitter.com/AndreyPitsko
---

In one very sunny and nice day we got the next requirement from our Product Owner: "*I want to change a bit design of our application, to make application more beautiful and exciting*". 

We understood out task:"we should do rounded corners over UITabBar"
![goal in detail]({{ root_url }} /images/2015-09-01-how-create-rounded-corners-on-top-of-uitabbar/goal2.png)

Let's code

First my idea: we can set cornerRadius in all our controllers of UITabBarController:
```
    view.layer.cornerRadius = 8.0

```

But result was expected. We got rounded corners on top of my views as well:
![eror one]({{ root_url }} /images/2015-09-01-how-create-rounded-corners-on-top-of-uitabbar/step_1.png)

Second my idea: we can try to use masks
```
        let roundedPath = UIBezierPath(roundedRect: bounds, byRoundingCorners: corners, cornerRadii: CGSize(width: radius, height: radius))
        let maskLayer = CAShapeLayer()
        maskLayer.frame = bounds
        maskLayer.path = roundedPath.CGPath
        layer.mask = maskLayer

```

After some checking I understood that each of controller in our UITabBarController is UINavigationController. As result rounded corners disappear after pushing any next controller.
So we need to apply it for all main views. I decided to create extension of UIView and use it where it is required

```
extension UIView {
    func makeCornerRadius(radius: CGFloat = 8, corners: UIRectCorner) {
		...
    }
}
```

Cool, Seems it works. But again, after testing I understood that it didn't work with UIScrollView, because during scrolling you can see black area.

![eror two]({{ root_url }} /images/2015-09-01-how-create-rounded-corners-on-top-of-uitabbar/step_2.png)

ok, than I decided to recalculate mask path for UIScrollView:
```
	public override func layoutSublayersOfLayer(layer: CALayer!) {
        super.layoutSublayersOfLayer(layer)
        if let maskLayer = layer.mask as? CAShapeLayer {
            maskLayer.path = UIBezierPath(roundedRect: layer.bounds, byRoundingCorners: UIRectCorner.BottomRight | .BottomLeft, cornerRadii: CGSize(width: 8.0, height: 8.0)).CGPath
        }
    }
```

and yes, It is final solution. I didn't find any bugs and issues. 

*Mission completed*


**We sure that this solution works, but probably there is some way to improve it. Let me know**


