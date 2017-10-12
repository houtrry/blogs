## 第一章: Activity的生命周期和启动模式

### 1.1 Activity的生命周期的全面分析

#### 1.1.1 典型情况下的生命周期分析
* 如果新Activity采用了透明主题, 那么当前的Activity不会回调onStop
* onStart和onStop 与onResume和onPause的区别:
    onStart和onStop是从Activity是否可见的角度来回调的
    onResume和onPause是从是否位于前台这个角度来回调的
* A页面启动B页面生命周期: onPause(A) -> onCreate(B) -> onStart(B) -> onPause(B) -> onStop(A)

#### 1.1.2 异常情况下的生命周期分析