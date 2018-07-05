- [产生原因](#产生原因)
- [简述](#简述)
- [向Activity添加片段](#向activity添加片段)
  - [在Activity的布局文件内声明Fragment](#在activity的布局文件内声明fragment)
  - [或者通过编程方式将Fragment添加到某个现有ViewGroup](#或者通过编程方式将fragment添加到某个现有viewgroup)
  - [添加没有UI的Fragment](#添加没有ui的fragment)
- [管理片段]





### 产生原因


### 简述

Fragment（**片段**） 可视为 Activity 的模块化组成部分，它具有自己的生命周期。Fragment 必须始终嵌入在 Activity 中，其生命周期直接受宿主 Activity 生命周期的影响。

每个 Fragment 都可设计为可重复使用的模块化 Activity 组件，可以将一个 Fragment 加入多个 Activity . 因此，应该采用可复用式设计，避免直接从某个 Fragment 直接操纵另一个 Fragment . 因为模块化 Fragment 可以通过更改 Fragment 的组合方式来适应不同的屏幕尺寸。在设计可同时支持平板电脑和手机的应用时，可以在不同的布局配置中重复使用 Fragment , 以根据可用的屏幕空间优化用户体验。 例如，在手机上，如果不能在同一 Activity 内储存多个 Fragment , 可能必须利用单独 Fragment 来实现单窗格 UI .

当 Activity 正在运行（处于已恢复生命周期状态）时，可独立操纵每个 Fragment , 如添加或移除它们。当执行此类 Fragment 事务时，也可以将其添加到由 `Activity 管理的返回栈` — **Activity 中的每个返回栈条目都是一条已发生 Fragment 事务的记录**。返回栈让用户可以通过按返回按钮撤消 Fragment 事务（后退）。


### 向Activity添加片段

#### 在Activity的布局文件内声明Fragment

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal">

    <fragment
        android:id="@+id/list"
        android:name="com.zch.learnbase.modules.fragment.ArticleListFragment"
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:layout_weight="1" />

    <fragment
        android:id="@+id/detail"
        android:name="com.zch.learnbase.modules.fragment.ArticleDetailFragment"
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:layout_weight="2" />
</LinearLayout>
```

- <fragment\> 中的 android:name 属性指定要在布局中实例化的 Fragment 类。
- 当系统创建此 Activity 布局时，会实例化在布局中指定的每个 Fragment , 并为每个 Fragment 调用 onCreateView() 方法，以检索每个 Fragment 的布局。系统会直接插入 Fragment 返回的 View 来替代 <fragment\> 元素。

> 每个 Fragment 都需要一个唯一的标识符，重启 Activity 时，系统可以使用该标识符来恢复 Fragment（还可以使用该标识符来捕获 Fragment 以执行某些事务，如将其移除）。

**可以通过三种方式为 Fragment 提供唯一的标识符：**

- 为 android:id 属性提供唯一 ID
- 为 android:tag 属性提供唯一字符串
- 如果未给以上两个属性提供值，系统会使用容器视图的 ID

#### 或者通过编程方式将Fragment添加到某个现有ViewGroup

```Java
FragmentManager fragmentManager = getFragmentManager();
FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
ArticleListFragment fragment = new ArticleListFragment();
fragmentTransaction.add(R.id.fragment_container, fragment);
fragmentTransaction.commit();
```

可以在 Activity 运行期间随时将 Fragment 添加到 Activity 布局中。在 Activity 中执行 Fragment 事务（如添加、移除或替换片段），必须使用 FragmentTransaction 中的 API . 一旦通过 FragmentTransaction 做出了更改，就必须调用 commit() 以使更改生效。

#### 添加没有UI的Fragment

还可以使用 Fragment 为 Activity 提供后台行为，而不显示额外 UI . 只能通过 `add (Fragment fragment,  String tag)` 的方式添加，用 tag 做唯一标识符。获取该 Fragment 则需要使用 `findFragmentByTag()` . 由于它并不与 Activity 布局中的视图关联，因此**不会收到对 onCreateView() 的调用**。因此，不需要实现该方法。

将没有 UI 的 Fragment 用作后台工作线程的示例 Activity 位于：SDK 示例（通过 Android SDK 管理器提供）中，以 <sdk_root>/APIDemos/app/src/main/java/com/example/android/apis/app/FragmentRetainInstance.java 形式位于您的系统中。