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

One day, our product owner requested the following: "*I want to make our application more exciting and visually attractive, by slightly changing the design*". 

Our task:"Create rounded corners over the UITabBar"
![goal in detail]({{ root_url }} /images/2015-09-01-how-create-rounded-corners-on-top-of-uitabbar/goal2.png)

Let's code

The first approach: Set cornerRadius in all our the controllers of the UITabBarController:
```
    view.layer.cornerRadius = 8.0

```

But the result was as expected: Rounded corners on top of my views as well:
![eror one]({{ root_url }} /images/2015-09-01-how-create-rounded-corners-on-top-of-uitabbar/step_1.png)

The second attempt: Use masks
```
        let roundedPath = UIBezierPath(roundedRect: bounds, byRoundingCorners: corners, cornerRadii: CGSize(width: radius, height: radius))
        let maskLayer = CAShapeLayer()
        maskLayer.frame = bounds
        maskLayer.path = roundedPath.CGPath
        layer.mask = maskLayer

```

After some research, we understood that each controller in our UITabBarController is a UINavigationController. As result, rounded corners disappear after pushing any next controller.
The third try: We applied it for all main views. We created an extension of UIView and used it where it is required.

```
extension UIView {
    func makeCornerRadius(radius: CGFloat = 8, corners: UIRectCorner) {
		...
    }
}
```

Cool, seems it worked! However, after testing we understood that it didn't work with the UIScrollView, since you can see a black area while scrolling.

![eror two]({{ root_url }} /images/2015-09-01-how-create-rounded-corners-on-top-of-uitabbar/step_2.png)

Finally, we decided to recalculate the mask path for UIScrollView:
```
	public override func layoutSublayersOfLayer(layer: CALayer!) {
        super.layoutSublayersOfLayer(layer)
        if let maskLayer = layer.mask as? CAShapeLayer {
            maskLayer.path = UIBezierPath(roundedRect: layer.bounds, byRoundingCorners: UIRectCorner.BottomRight | .BottomLeft, cornerRadii: CGSize(width: 8.0, height: 8.0)).CGPath
        }
    }
```

Annd yes, it worked! No bugs or issues. 

*Mission complete*


**This is a solid solution, however if you find a way to improve it. Don't hesitate to let us know.**


