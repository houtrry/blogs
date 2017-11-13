# 第三章 View的事件体系

## 3.1 准备知识

### 坐标相关  
* getX/getY返回的是相对于当前View的左上角的x和y的坐标, 而getRawX和getRawY返回的是相对于手机屏幕左上角的x和y坐标.

### TouchSlop
TouchSlop是系统所能识别出的被认为是最小滑动距离, 也就是如果两次滑动之间的距离小于这个常量, 那么系统就不认为你是在进行滑动操作.  
获取方法: ViewConfiguration.get(getContext()).getScaledTouchSlop();

### VelocityTracker
用于追踪手指在滑动过程中的速度, 包括水平和竖直方向的速度.  
注意: 不需要使用的时候, 需要调用clear方法来重置并回收内存.  

```

velocityTracker.clear();  
velocityTracker.recycle();

```
### GestureDetector
用于辅助检测用户的单击/滑动/长按/双击等行为.  
建议: 如果只是监听滑动相关的, 建议自己实现onTouchEvent, 如果要监听双击这种行为的话, 那么就是用GestureDetector.  

### Scroller
用于实现View的弹性滑动., 实现有过渡效果的滑动.


## 3.2 View的滑动


## 事件的分发流程

## 事件冲突的处理