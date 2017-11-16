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

### scrollBy和scrollTo  

### View动画和属性动画  
View动画是对View的影像做的操作, 它并不能真正改变View的位置参数, 包括宽高, 并且如果希望动画后的状态得以保留还必须将fillAfter属性设置为true, 否则, 动画完成后, 其动画效果会消失.  

### 改变布局参数  
也就时修改LayoutParams.注意ViewGroup.MarginLayoutParams这个支持margin属性的LayoutParams, 它是ViewGroup.LayoutParams的子类, 是LinearLayout.LayoutParams以及RelativeLayout.LayoutParams等的父类.  

## 3.3 弹性滑动  

###  使用Scroller

###  通过动画

###  使用延时策略

### ViewDragHelper

## 3.4 事件的分发机制
### 3.4.1  点击事件的传递规则
#### dispatchTouchEvent  
返回值表示当前View以及当前View的子View会不会消费当前事件. 注意: 这里并不仅仅是表示当前View.  

#### onInterceptTouchEvent  
在dispatchTouchEvent中调用. 用来判断是否拦截某个事件. 注意:如果当前View拦截了某个事件, 那么在同一个事件序列(DOWN-->UP当做同一个事件序列)中, 此方法不会再次调用.  

#### onTouchEvent
在dispatchTouchEvent中调用. 用来处理点击事件.返回结果表示是否消耗当前事件.如果不消耗, 那么在同一个事件序列中, 当前View无法再次接收事件.

#### onTouch和onTouchEvent
当一个View需要处理事件时, 如果它设置了OnTouchListener, 那么, OnTouchListener中的onTouch方法会被回调. 这时, 如果onTouch返回true, 那么onTouchEvent将不会被调用, 如果onTouch返回false, onTouchEvent会被调用.也就是View设置的onTouchListener的优先级比onTouch高.  

#### 事件传递规则的一些结论:  
1. 一个事件序列指从手指接触屏幕到手指离开屏幕的过程中产生的一系列事件.  
2. 正常情况下, 一个事件序列只能被一个View消耗.  
3. 某个View一旦决定拦截, 那么这个事件序列都只能由它来处理, 并且它的onInterceptTouchEvent不会再被调用.
4. 某个View一旦开始处理事件, 如果它不消耗ACTION_DOWN事件(即onTouchEvent返回了false), 那么, 同一个事件序列的其他事件都不会再交给它处理.  
5. 如果View不消耗出ACTION_DOWN以外的其它事件, 那么这个点击事件会消失, 此时父元素的onTouchEvent并不会被调用, 并且当前View可以持续收到后续的事件, 最终这些消失的事件会传递给Activity处理.( 我理解的是在onTouchEvent中, ACTION_DOWN返回了true, 但其它的都返回了false的情况).  
6. ViewGroup默认不拦截任何事件.
7. View中没有onInterceptTouchEvent方法, 一旦有事件传递给它, 就会执行它的onTouchEvent方法.  
8. View的onTouchEvent默认会消耗事件(返回true), 除非它是不可点击的(clickable和longClickable同时为false).  
9. View的enable属性不影响onTouchEvent的默认返回值.  
10. 通过requestDisallowInterceptTouchEvent方法可以在子元素中干预父元素的事件分发过程, 但是ACTION_DOWN事件除外.

### 3.4.2 事件分发源码解析

#### 事件的分发过程
* Activity-->Window(其实是它的实现PhoneWindow)-->DecorView(Fragment的子类, 包含TitleView和ContentView, 其中, ContentView就是Activity中setContent方法设置的内容)-->ViewGroup...  
* onInterceptTouchEvent不是每次事件都会被调用, 如果我们想提前处理所有的点击事件, 要选择dispatchTouchEvent方法, 只有这个方法能确保每次都会调用, 当然, 前提是, 事件能够传递到当前ViewGroup.  

## 3.5 事件冲突

### 常见的滑动冲突场景
1. 外部滑动方向和内部滑动方向不一致.比如横向ViewPager包裹竖直方向的RecyclerView或者ListView(ViewPager包裹Fragment, Fragment包裹更小的RecyclerView或者ListView)
2. 外部滑动方向和内部滑动方向一致, 比如横向的ViewPager包裹横向的ViewPager(ViewPager包裹Fragment, Fragment包裹更小的ViewPager)
3. 上述两种情况的嵌套

### 处理规则
1. 对于场景1, 可以根据是水平滑动还是竖直滑动来判断由谁来拦截事件.而判断水平还是竖直滑动的方法①判断水平方向和竖直方向上滑动距离的大小②判断判断水平方向和竖直方向上滑动速度的大小
2. 根据具体的业务来做判断
3. 根据具体的业务来做判断

### 滑动冲突的解决方式

#### 外部拦截法

所谓外部拦截法是指点击事件都事先经过父容器的拦截处理, 如果父容器需要此事件就拦截, 如果不需要就不拦截, 这样就可以解决滑动冲突的问题.外部拦截法需要重写父容器的onInterceptTouchEvent方法, 在该方法内部做相应的拦截即可.  

```

public boolean onInterceptTouchEvent(MotionEvent event) {
	boolean intercepted = false;
	int x = (int)event.getX();
	int y = (int)event.getY();
	switch() {
		case MotionEvent.ACTION_DOWN:{
			intercepted = false;
			break;
		}
		case MotionEvent.ACTION_MOVE:{
			intercepted = false;
			if(父容器需要当前点击事件) {
				intercepted = true;
			} else {
				intercepted = false
			}
			break;
		}
		case MotionEvent.ACTION_UP:{
			intercepted = false;
			break;
		}
		default:
			break;
	}
	mLastXIntercept = x;
	mLastYIntercept = y;
}

```

在onInterceptTouchEvent方法中, 首先是ACTION_DOWN这个事件, 父容器必须返回false, 即不拦截ACTION_DOWN事件, 这是因为一旦父容器拦截了ACTION_DOWN, 那么同一个事件序列的后续事件ACTION_MOVE和ACTION_UP都会直接交给父容器处理, 这个事件没法再传递给子元素了.其次, ACTION_MOVE, 父容器可以根据需要拦截. 最后ACTION_MOVE事件, 父容器必须返回false.  
如果事件交由子元素处理, 但是父容器的ACTION_UP返回了true, 会导致子元素无法接收ACTION_UP事件, 这个时候子元素的onClick事件就无法触发.

#### 内部拦截法

内部拦截法是指父容器不拦截任何事件, 所有的事件都传递给子元素, 如果子元素需要此事件就直接消耗掉, 否则, 就交给父容器进行处理.这种方法需要配合requestDisallowInterceptTouchEvent方法使用. 伪代码如下, 需要重写子元素的dispatchTouchEvent方法:  

```

public boolean dispatchTouchEvent(MotionEvent event) {
	int x = (int) event.getX();
	int y = (int) event.getY();

	switch(event.getAction()) {
		case MOtionEvent.ACTION_DOWN:{
			parent.requestDisallowInterceptTouchEvent(true);
			break;
		}
		case MOtionEvent.ACTION_DOWN:{
			int deltaX = x - mLastX;
			int deltaY = y - mLastY;
			if(父容器需此类点击事件) {
				parent.requestDisallowInterceptTouchEvent(false);
			}
			break;
		}
		case MOtionEvent.ACTION_DOWN:{
			break;
		}
		default:
			break;
	}

}

```

除了子元素需要做处理以外, 父元素也需要默认拦截除了ACTION_DOWN以外的其他事件, 这样当子元素调用parent.requestDisallowInterceptTouchEvent(false);方法时, 父元素才能继续拦截所有事件.父元素所做的修改如下:  

```

public boolean onInterceptTouchEvent(MotionEvnet event) {
	int action = event.getAction();
	if(action == MotionEvent.ACTION_DOWN) {
		return false;
	} else {
		return true;
	}
}

```