- [Activity正常生命周期流程](#activity正常生命周期流程)
- [Activity异常情况生命周期分析](#activity异常情况生命周期分析)
- [Activity生命周期附加说明](#activity生命周期附加说明)
- [configChanges属性的应用](#configchanges属性的应用)
- [Activity启动模式](#activity启动模式)
  - [四种加载模式](#四种加载模式)
  - [LaunchMode附加说明](#launchmode附加说明)
  - [Activity的Flags](#activity的flags)
- [IntentFilter的匹配规则](#intentfilter的匹配规则)
  - [action的匹配规则](#action的匹配规则)
  - [category的匹配规则](#category的匹配规则)
  - [data的匹配规则](#data的匹配规则)






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

![Mou icon](https://github.com/zhich/git_resource/blob/master/Android-Learning/activity_lifecycle.png)

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

我们在开发项目的过程中，会涉及到该应用中多个 Activity 组件之间的跳转，或者夹带其它应用的可复用的 Activity . 例如我们可能希望跳转到原来某个 Activity 实例，而不是产生大量重复的 Activity . 这样就需要我们为 Activity 配置特定的加载模式，而不是使用默认的加载模式。

#### 四种加载模式

- standard 模式

  **标准模式**。这是默认模式，每次激活 Activity 时都会创建 Activity 实例，并放入任务栈中，不管这个实例是否存在。一个任务栈中可以有多个实例，每个实例也可以属于不同的任务栈。在这种模式下，谁启动了这个 Activity , 那么这个 Activity 就运行在启动它的那个 Activity 所在的任务栈中。**启动的生命周期为：onCreate() -> onStart() -> onResume()** .

- singleTop 模式

  **栈顶复用模式**。如果在任务的栈顶正好存在该 Activity 的实例，就重用该实例（同时 **onNewIntent** 方法会被回调，通过该方法的 Intent 参数我们可以取出当前请求的信息），此时这个 Activity 的**生命周期顺序为：onPause() ->onNewIntent()->onResume()** , 否则就会创建新的 Activity 实例并放入栈顶，即使栈中已经存在该 Activity 的实例，只要不在栈顶，都会创建新的实例。此时**生命周期顺序为：onCreate()->onStart()->onResume()** .


- singleTask 模式

  **栈内复用模式**。这是一种单实例模式，如果在栈中已经有该 Activity 的实例，就重用该实例（会调用实例的 **onNewIntent()**）。具体地说，当一个具有 singleTask 模式的 Activity 请求启动后，如 Activity A , 系统首先会寻找是否存在 A 想要的任务栈，若不存在，则重新创建一个任务栈，然后创建 A 的实例后把 A 放入栈中。若存在，这时要看 A 是否在栈中有实例存在，若有实例存在，那么系统就会把 A 调到栈顶（此时还会把 A 上面的实例移除出栈）并调用它的 onNewIntent 方法，若没有实例存在，则创建 A 的实例并把 A 压入栈中。

- singleInstance 模式

  **单实例模式**。这是一种加强的 singleTask 模式，**它除了具有 singleTask 的所有特性外，还加强了一点，那就是具有此种模式的 Activity 只能单独存在于一个任务栈中**。它在一个新栈中创建该 Activity 的实例，并让多个应用共享该栈中的该 Activity 实例。一旦该模式的 Activity 实例已经存在于某个栈中，任何应用再激活该 Activity 时都会重用该栈中的实例（会同时调用实例的 **onNewIntent()**）。其效果相当于多个应用共享一个应用，不管谁激活该 Activity 都会进入同一个应用中。

> 设置启动模式的位置在 AndroidManifest.xml 文件中 Activity 元素的 Android:launchMode 属性。

#### LaunchMode附加说明

1. 使用 **TaskAffinity** 属性指定一个 Activity 所需要的任务栈的名字，默认情况下，所有 Activity 所需的任务栈的名字为**应用的包名**。TaskAffinity 属性主要和 singleTask 启动模式或者 allowTaskReparenting 属性配对使用。

2. 任务栈分为**前台任务栈**和**后台任务栈**，后台任务栈中的 Activity 位于暂停状态，用户可通过切换将后台任务栈再次调到前台。

#### Activity的Flags

除了可以在 manifest 中设置 Activity 的启动模式，也可以通过设置 Intent 的 flag 标识来设定 Activity 的启动模式。

常用的有：FLAG_ACTIVITY_NEW_TASK、FLAG_ACTIVITY_SINGLE_TOP、FLAG_ACTIVITY_CLEAR_TOP

- FLAG_ACTIVITY_NEW_TASK

  相当于 **singleTask** 启动模式。

- FLAG_ACTIVITY_SINGLE_TOP

  相当于 **singleTop** 启动模式。

- FLAG_ACTIVITY_CLEAR_TOP

  设置此标识的 Activity 在启动时，如果当前的任务栈内存在此 Activity 实例，则跳转到此实例，并清除掉在此实例上面的所有 Activity 实例，此时此 Activity 实例位于任务栈的栈顶。

### IntentFilter的匹配规则

启动 Activity 分为**显示**和**隐式**调用，显示调用需要明确指定被启动对象的组件信息，包括包名和类名，而隐式调用则不需要明确指定组件信息。**当显示和隐式调用共存时以显示调用为主**。显示调用很简单，这里只介绍隐式调用。隐式调用需要 Intent 能够匹配目标组件的 IntentFilter 中所设置的过滤信息，如果不匹配将无法启动目标 Activity , IntentFilter 中的过滤信息有 action、category、data .

一个过滤列表中的 action、category 和 data 可以有多个，所有的 action、category、data 分别构成不同类别，同一类别的信息共同约束当前类别的匹配过程。**只有一个 Intent 同时匹配 action、category、data 类别才算完全匹配，只有完全匹配才能成功启动目标 Activity** . 一个 Activity 中可以有多个 intent-filter , 一个 Intent 只要能匹配任何一组 intent-filter 即可成功启动对应的 Activity .

#### action的匹配规则

action 是一个字符串，系统预定义了一些 action , 同时我们也可以在应用中定义自己的 action . 一个过滤规则中可以有多个 action . **action 的匹配要求 Intent 中的 action 必须存在且和过滤规则中的其中一个 action 完全相同即可匹配成功**，若 Intent 中没有指定 aciotn , 则匹配失败。另外，action是 区分大小写的。

#### category的匹配规则

category 是一个字符串，系统预定义了一些 category , 同时我们也可以在应用中自定义自己的 category . **category 的匹配规则是，只要 Intent 中出现了 categoty , 不管有几个 category , 对于每一个 category , 它必须是过滤规则中已经定义了的 category** , 系统在调用 startActivity 或者 startActivityForResult 的时候会默认给 Intent 加上 **android.intent.category.DEFAULT** 这个 category . 因此，为了我们的 activity 能够接收隐式调用，就必须在 intent-filter 中指定这个默认的 category .

#### data的匹配规则

data 的匹配规则和 action 类似，它要求 Intent 中必须含有 data 数据，并且 data 数据能够完全匹配过滤规则中的某一个 data . 这里说的完全匹配是指**过滤规则中出现的 data 部分也出现在了 Intent 中的 data 中**。




**参考资料**

- Android开发艺术探索










