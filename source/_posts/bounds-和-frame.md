title: bounds 和 frame
date: 2015-03-16 08:47:36
tags: iOS
categories: 技术

---

# frame
* The frame rectangle, which describes the view’s location and size in **its superview’s coordinate system**.
* This rectangle defines the size and position of the view in its superview’s coordinate system. You use this rectangle during layout operations to size and position the view. Setting this property changes the point specified by the **center** property and the size in the **bounds** rectangle accordingly. The coordinates of the frame rectangle are always specified in points.
* Changing the frame rectangle **automatically redisplays the receiver** without invoking the drawRect: method. If you want the drawRect: method invoked when the frame rectangle changes, set the contentMode property to UIViewContentModeRedraw.
* Changes to this property can be animated. However, **if the transform property contains a non-identity transform, the value of the frame property is undefined and should not be modified.** In that case, you can reposition the view using the **center** property and adjust the size using the **bounds** property instead.

#bounds
* The bounds rectangle, which describes **the view’s location and size in its own coordinate system**.
* On the screen, the bounds rectangle represents the same visible portion of the view as its frame rectangle. By default, the origin of the bounds rectangle is set to (0, 0) but you can change this value to display different portions of the view.** The size of the bounds rectangle is coupled to the size of the frame rectangle, so that changes to one affect the other.Changing the bounds size grows or shrinks the view relative to its center point. **The coordinates of the bounds rectangle are always specified in points.
* 改变父视图的bounds，其子类的位置也会跟着变动。

#center
* The center of the **frame**.
* The center is specified within **the coordinate system of its superview** and is measured in points. Setting this property changes the values of the **frame** properties accordingly.
* Changes to this property can be animated. Use the beginAnimations:context: class method to begin and the commitAnimations class method to end an animation block.

# transform
* Specifies the transform applied to the receiver, relative to the center of its bounds.
* **The origin of the transform is the value of the center property**, or the layer’s anchorPoint property if it was changed. (Use the layer property to get the underlying Core Animation layer object.) The default value is CGAffineTransformIdentity.
* Changes to this property can be animated. Use the beginAnimations:context: class method to begin and the commitAnimations class method to end an animation block. The default is whatever the center value is (or anchor point if changed)
* **If this property is not the identity transform**, the value of the frame property is undefined and therefore should be ignored.