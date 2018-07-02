- [Activity正常生命周期流程](#activity正常生命周期流程)
- [Activity异常情况生命周期分析](#activity异常情况生命周期分析)
- [Activity生命周期附加说明](#activity生命周期附加说明)
- [configChanges属性的应用](#configchanges属性的应用)
- [Activity启动模式](#activity启动模式)





### Activity正常生命周期流程

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

### Activity异常情况生命周期分析

- 系统资源配置发生改变导致 Activity 被杀死并重新创建

  当系统配置发生变化（如旋转屏幕），Activity 会被销毁，其 onPause、onStop、onDestroy 均会被调用，同时由于 Activity 是在异常情况下终止的，系统会调用 **onSaveInstanceState** 来保存当前 Activity 的状态。**onSaveInstanceState 方法的调用时机是在 onStop 之前**，它和 onPause 没有既定的时序关系。当 Activity 被重建后，系统会调用 **onRestoreInstanceState** , 并把 Activity 销毁时的 onSaveInstanceState 方法所保存的 Bundle 对象作为参数同时传递给 onRestoreInstanceState 和 onCreate 方法。因此可以通过 onRestoreInstanceState 和 onCreate 方法来判断 Activity 是否被重建了，如果被重建了，我们就可以取出之前保存的数据并恢复，从时序上说，**onRestoreInstanceState 的调用时机在 onStart 之后。**

  系统只在 Activity 异常终止的时候才会调用 onSaveInstanceState 和 onRestoreInstanceState 来存储和恢复数据，其他情况不会触发这个过程，但是按 Home 键或启动新 Activity 仍然会触发 onSaveInstanceState 的调用。

  在 onSaveInstance 和 onRestoreInstanceState 方法中，系统自动为我们做了一定的恢复工作。当 Activity 在异常情况下需要重建时，系统会默认保存了当前 Activity 的视图结构，并且在 Activity 重启后为我们恢复这些数据，如文本框中用户的输入数据、ListView 滚动位置等。

  **在 onCreate 和 onRestoreInstanceState 中接收保存的数据的区别是**：onRestoreInstanceState 一旦被调用，其参数 savedIntanceState 一定有值，而不需要额外判断它是否为空；但是 onCreate 如果是正常启动的话，其参数 saveInstanceState 为 null , 所以必须额外判断。官方文档建议采用 onRestoreInstance 去恢复数据。

- 资源内存不足导致低优先级的 Activity 被杀死

  Activity 按优先级从高到低如下：

  - **前台 Activity** , 正在与用户交互的 Activity，优先级最高。
  - **可见但非前台 Activity** , 如 Activity 中弹出一个对话框，导致 Activity 可见但是位于后台无法与用户直接交互。
  - **后台 Activity** , 已经被停止的 Activity , 如执行了 onStop , 优先级最低。

  当系统内存不足时，系统就会按照上述优先级去杀死目标 Activity 所在的进程，并在后续通过 onSaveInstanceState 和 onRestoreInstanceState 来存储和恢复数据。如果一个进程中没有四大组件在执行，那么这个进程很快被系统杀死，因此，**一些后台工作不适合脱离四大组件而独自运行在后台中，这样进程很容易被杀死**。比较好的方法是将后台工作放在 Service 中从而保证进程有一定的优先级，这样就不会轻易被系统杀死。

### Activity生命周期附加说明

1. 当用户打开新的 Activity 或切换到桌面时，回调如下：onPause --> onStop . 但是有一种情况，如果新 Activity 采用了**透明主题**，那么当前 Activity 不会调用 onStop .

2. 不能在 onPause 中做重量级的操作，因为必须 onPause 执行完成后新启动的 Activity 才能 Resume .

### configChanges属性的应用

1. 防止屏幕旋转时 Activity 重启

```xml
android:configChanges="orientation | screenSize"
```

有了上面的设置，系统会调用 Activity 的 onConfigurationChanged 方法，此时我们可以做一些自己的特殊处理。

### Activity启动模式
