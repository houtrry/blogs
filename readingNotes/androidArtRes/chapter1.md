## 第一章: Activity的生命周期和启动模式

### 1.1 Activity的生命周期的全面分析

#### 1.1.1 典型情况下的生命周期分析
* 如果新Activity采用了透明主题, 那么当前的Activity不会回调onStop
* onStart和onStop 与onResume和onPause的区别:</br>
    onStart和onStop是从Activity是否可见的角度来回调的  </br>
    onResume和onPause是从是否位于前台这个角度来回调的  </br>
* A页面启动B页面生命周期: onPause(A) -> onCreate(B) -> onStart(B) -> onPause(B) -> onStop(A)
* 不能在onPause中做耗时操作, 以便新的Activity尽快显示并切换到前台

#### 1.1.2 异常情况下的生命周期分析
1. 资源相关的系统配置发生改变导致Activity被杀死并重新创建
  * 当系统配置发生改变后, Activity会被销毁, 其onPause/onStop/onDestroy均会被调用. 同时系统会调用onSaveInstanceState来保存当前Activity的状态. 这个方法的调用时机实在onStop之前, 它和onPause没有既定的时序关系, 既可能在onPause之前调用, 也可能在之后调用.
  * Activity重新创建后, 系统会调用onRestoreInstanceState. 从时许上来说, onRestoreInstanceState的调用时机在onStart之后.
  * 选择在onCreate或者onRestoreInstanceState恢复数据的区别: onCreate的参数Bundle savedInstanceState可能为空, 需要做非空判断, 而onRestoreInstanceState的Bundle savedInstanceState一定是有值的. 官方文档建议用onRestoreInstanceState去恢复数据.
2. 资源内存不足导致低优先级的Activity被杀死
  * 前台Activity----正在和用户交互的Activity
  * 可见但非前台Activity----比如Activity中弹出一个对话框, 导致Activity可见但是位于后台无法和用户直接交互
  * 后台Activity----已经被暂停的Activity, 比如执行了onStop
3. 如果某项内容发生改变后, 我们不想让系统重新创建Activity, 可以给Activity指定configChanges属性.