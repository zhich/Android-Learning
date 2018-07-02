- [生命周期](#生命周期)
- [启动模式](#启动模式)





### 生命周期

Activity 的生命周期回调方法有：onCreate() , onStart() , onResume() , onPause() , onStop() , onRestart() , onDestroy() . 

- onCreate()

  Activity 正在被创建。这是生命周期的第一个方法，在整个生命周期中只会被调用一次，一般在此做一些初始化工作。参数 savedInstanceState 保存 Activity 因异常情况而被销毁前的状态，可利用此参数做一些数据恢复的操作，若 Activity 正常启动，savedInstanceState 为 null .

- onStart()

  Activity 正在被启动。此时 Activity 已经可见，但还没出现在前台，还无法与用户交互。

- onResume()

  Activity 启动完成。此时 Activity 已经可见，并出现在前台，可以与用户交互了。该方法在 Activity 的整个生命周期中可能会多次被调用到。

- onPause()

  Activity 正在被停止。在此可做一些存储数据、停止动画等工作，但注意不能太耗时，因为这会影响到新 Activity 的显示，onPause 必须执行完，新 Activity 的 onResume 才会执行。

- onStop()

  Activity 即将停止。当前 Activity 不可见时回调此方法。在此处可释放全部用户使用不到的数据，可做一些稍微重量级的回收工作，同样不能太耗时，如对注册广播的解注册，对一些状态数据的存储。此时 Activity 还不会被销毁掉，而是保持在内存中，但随时都会被回收。

- onRestart()

  Activity 正在重新启动。一般情况下，当当前 Activity 从不可见重新变为可见状态时，onRestart 就会被调用。这种情况一般由用户行为所导致，比如用户按 Home 键切换到桌面或者用户打开一个新 Activity , 此时当前 Activity 就会暂停，也就是 onPause 和 onStop 被执行了，接着用户又回到这个 Activity , 就会导致该 Activity 的 onRestart 被调用。

- onDestroy

  Activity 即将被销毁。这是 Activity 生命周期中的最后一个回调，在此可做一些回收工作和最终的资源释放。

> 通常情况下：onCreate() 和 onDestroy() 成对存在；onStart() 和 onStop() 成对存在；onResume() 和 onPause() 成对存在。

![](https://github.com/zhich/git_resource/blob/master/Android-Learning/activity_lifecycle.png)