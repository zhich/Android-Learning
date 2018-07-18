- [产生原因](#产生原因)
- [简述](#简述)
- [创建Fragment](#创建fragment)
  - [添加用户界面](#添加用户界面)
  - [创建布局](#创建布局)
  - [向Activity添加片段](#向activity添加片段)
    - [在Activity的布局文件内声明Fragment](#在activity的布局文件内声明fragment)
    - [或者通过编程方式将Fragment添加到某个现有ViewGroup](#或者通过编程方式将fragment添加到某个现有viewgroup)
    - [添加没有UI的Fragment](#添加没有ui的fragment)
- [管理Fragment](#管理fragment)
- [管理Fragment回退栈](#管理fragment回退栈)
- [Fragment常用的API](#fragment常用的api)
- [执行Fragment事务](#执行fragment事务)
- [Fragment生命周期](#fragment生命周期)
- [与Activity通信](#与activity通信)
- [Fragment状态的持久化](#fragment状态的持久化)


## 产生原因

Android 在 Android 3.0（API 级别 11）中引入了 Fragment（**片段**），主要是为了给大屏幕（如平板电脑）上更加动态和灵活的 UI 设计提供支持。由于平板电脑的屏幕比手机屏幕大得多，因此可用于组合和交换 UI 组件的空间更大。利用 Fragment 实现此类设计时，无需管理对视图层次结构的复杂更改。 通过将 Activity 布局分成 Fragment , 可以在运行时修改 Activity 的外观，并在由 Activity 管理的返回栈中保留这些更改。


## 简述

Fragment 可视为 Activity 的模块化组成部分，它具有自己的生命周期。Fragment 必须始终嵌入在 Activity 中，其生命周期直接受宿主 Activity 生命周期的影响。

每个 Fragment 都可设计为可重复使用的模块化 Activity 组件，可以将一个 Fragment 加入多个 Activity . 因此，应该采用可复用式设计，避免直接从某个 Fragment 直接操纵另一个 Fragment . 因为模块化 Fragment 可以通过更改 Fragment 的组合方式来适应不同的屏幕尺寸。在设计可同时支持平板电脑和手机的应用时，可以在不同的布局配置中重复使用 Fragment , 以根据可用的屏幕空间优化用户体验。 例如，在手机上，如果不能在同一 Activity 内储存多个 Fragment , 可能必须利用单独 Fragment 来实现单窗格 UI .

当 Activity 正在运行（处于已恢复生命周期状态）时，可独立操纵每个 Fragment , 如添加或移除它们。当执行此类 Fragment 事务时，也可以将其添加到由 `Activity 管理的返回栈` — **Activity 中的每个返回栈条目都是一条已发生 Fragment 事务的记录**。返回栈让用户可以通过按返回按钮撤消 Fragment 事务（后退）。


## 创建Fragment

要创建一个 Fragment 必须扩展 Fragment 类（或已有的其子类 DialogFragment、ListFragment、PreferenceFragment）。

- DialogFragment

  显示浮动对话框。使用此类创建对话框可有效地替代使用 Activity 类中的对话框帮助程序方法，因为您可以将片段对话框纳入由 Activity 管理的片段返回栈，从而使用户能够返回清除的片段。

- ListFragment

  显示由适配器（如 SimpleCursorAdapter）管理的一系列项目，类似于 ListActivity . 它提供了几种管理列表视图的方法，如用于处理点击事件的 onListItemClick() 回调。

- PreferenceFragment

  以列表形式显示 Preference 对象的层次结构，类似于 PreferenceActivity . 这在为您的应用创建“设置” Activity 时很有用处。


### 添加用户界面

Fragment 通常用作 Activity 用户界面的一部分，将其自己的布局融入 Activity . 要想为 Fragment 提供布局，必须实现 onCreateView() 回调方法，Android 系统会在 Fragment 需要绘制其布局时调用该方法。此方法返回的 View 必须是 Fragment 布局的根视图。

> 如果是 ListFragment 的子类，则默认实现会从 onCreateView() 返回一个 ListView，因此无需实现它。

### 创建布局

```Java
public static class ExampleFragment extends Fragment {
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        return inflater.inflate(R.layout.example_fragment, container, false);
    }
}
```

传递至 onCreateView() 的 container 参数是 Fragment 布局将插入到的父 ViewGroup（来自 Activity 的布局）。savedInstanceState 参数是在恢复 Fragment 时，提供上一 Fragment 实例相关数据的 Bundle .

inflate() 方法带有三个参数：

- 您想要扩展的布局的资源 ID
- 将作为扩展布局父项的 ViewGroup
- 指示是否应该在扩展期间将扩展布局附加至 ViewGroup（第二个参数）的布尔值。（在本例中，其值为 false , 因为系统已经将扩展布局插入 container — 传递 true 值会在最终布局中创建一个多余的视图组。）

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

## 管理Fragment

要管理 Fragment , 需要使用 FragmentManager , FragmentManager 执行的操作包括：

- 通过 findFragmentById() 或 findFragmentByTag() 获取 Activity 中存在的 Fragment .
- 通过 popBackStack()（模拟用户发出的返回命令）将 Fragment 从返回栈中弹出。
- 通过 addOnBackStackChangedListener() 注册一个侦听返回栈变化的侦听器。

## 管理Fragment回退栈

- 跟踪回退栈状态

  ```Java
  public class MyClass implements FragmentManager.OnBackStackChangedListener
    
    @Override
    public void onBackStackChanged() {
    
    }
    // ...
    // 添加回退栈监听接口
    getSupportFragmentManager().addOnBackStackChangedListener(this);
    // ...
  }
  ```

- 管理回退栈

  - FragmentTransaction.addToBackStack(String) // 将一个刚刚添加的 Fragment 加入到回退栈中
  - getSupportFragmentManager().getBackStackEntryCount() // 获取回退栈中实体数量
  - getSupportFragmentManager().popBackStack(String name, int flags) // 根据 name 立刻弹出栈顶的 Fragment
  - getSupportFragmentManager().popBackStack(int id, int flags) // 根据 id 立刻弹出栈顶的 Fragment

## Fragment常用的API

- android.support.v4.app.Fragment 主要用于定义 Fragment
- android.support.v4.app.FragmentManager 主要用于在 Activity 中操作 Fragment , 可以使用 FragmentManager.findFragmenById、FragmentManager.findFragmentByTag 等方法去找到一个 Fragment
- android.support.v4.app.FragmentTransaction 保证一系列 Fragment 操作的原子性
- 主要的操作都是 FragmentTransaction 的方法（一般我们为了向下兼容，都使用 support.v4 包里面的 Fragment）

  ```Java
  getFragmentManager() // Fragment 若使用的是 support.v4 包中的，那就使用 getSupportFragmentManager 代替
  ```
- FragmentTransaction 的一些操作方法

## 执行Fragment事务

在 Activity 中使用 Fragment 的一大优点是，可以根据用户行为通过它们执行添加、移除、替换以及其他操作。 提交给 Activity 的每组更改都称为事务，可以使用 FragmentTransaction 中的 API 来执行一项事务。也可以**将每个事务**保存到由 Activity 管理的返回栈内，从而让用户能够回退 Fragment 更改（类似于回退 Activity）。

```Java

Fragment newFragment = new ArticleListFragment();
FragmentTransaction transaction = getFragmentManager().beginTransaction();

// 往 Activity 中添加一个 Fragment
transaction.add();

// 从 Activity 中移除一个 Fragment
transaction.remove();

// 使用另一个 Fragment 替换当前的，实际上就是 remove() 然后 add() 的合体
transaction.replace();

// 隐藏当前的 Fragment , 仅仅是设为不可见，并不会销毁
transaction.hide();

// 显示之前隐藏的 Fragment
transaction.show();

// 将以上一组事务保存到返回栈，以便用户能够通过按返回按钮撤消事务并回退到上一 Fragment
transaction.addToBackStack(null);

transaction.commit(); //提交一个事务
```

**说明**

- 每个事务都是想要同时执行的一组更改。可以使用 add()、remove() 和 replace() 等方法为给定事务设置想要执行的所有更改。然后，要想将事务应用到 Activity , 必须调用 commit() .
- 调用 commit() 之前，可调用 addToBackStack() , 以将事务添加到 Fragment 事务返回栈，该返回栈由 Activity 管理，允许用户通过按返回按钮返回上一 Fragment 状态。
- 如果向事务添加了多个更改（如有一个 add() 或 remove()），并且调用了 addToBackStack() , 则在调用 commit() 前应用的所有更改都将作为单一事务添加到返回栈，并且返回按钮会将它们一并撤消。
- 如果向同一容器添加多个 Fragment , 则您添加 Fragment 的顺序将决定它们在视图层次结构中的出现顺序。
- 如果没有在执行移除 Fragment 的事务时调用 addToBackStack() , 则事务提交时该 Fragment 会被销毁，用户将无法回退到该 Fragment . 如果调用了 addToBackStack() , 系统会停止该 Fragment , 并在用户回退时将其恢复。
- 对于每个 Fragment 事务，都可以通过在提交前调用 setTransition() 来应用过渡动画。
- 调用 commit() 不会立即执行事务，而是在 Activity 的 UI 线程可以执行该操作时再安排其在线程上运行。不过，如有必要，也可以从 UI 线程调用 executePendingTransactions() 以立即执行 commit() 提交的事务。通常不必这样做，除非其他线程中的作业依赖该事务。
- 只能在 Activity 保存其状态（用户离开 Activity）之前使用 commit() 提交事务。如果试图在该时间点后提交，则会引发异常。 这是因为如需恢复 Activity , 则提交后的状态可能会丢失。 对于丢失提交无关紧要的情况，请使用 commitAllowingStateLoss() .


## Fragment生命周期

- Fragment必须依存于Activity

![Mou icon](https://github.com/zhich/git_resource/blob/master/Android-Learning/activity_fragment_lifecycle_0.png)

- Fragment依附于Activity的生命状态

![Mou icon](https://github.com/zhich/git_resource/blob/master/Android-Learning/activity_fragment_lifecycle.png)

**Fragment生命周期回调方法含义**

- public void onAttach(Context context)

  在 Fragment 已与 Activity 关联时调用 onAttach 方法。从该方法起就可通过 Fragment.getActivity 方法获取与 Fragment 关联的 Activity 对象。此时由于 Fragment 的控件尚未初始化，因此不能操纵控件。

- public void onCreate(Bundle savedInstanceState)

  onCreate 方法在 onAttach 执行完后马上执行。在该方法中可以读取保存的状态，获取、初始化一些数据，可在 Bundle 对象获取一些从 Activity 传递过来的数据。

- public View onCreateView(LayoutInflater inflater, ViewGroup container,Bundle savedInstanceState)

  在该方法中会创建在 Fragment 显示的 View . inflater 用来装载布局文件；container 是 <fragment\> 标签的父标签对应对象；savedInstanceState 可获取 Fragment 保存的状态，为 null 表示未保存。

- public void onViewCreated(View view,Bundle savedInstanceState)

  创建完 Fragment 中的 View 后会立即调用该方法。参数 view 就是 onCreateView 方法返回的 View 对象。

- public void onActivityCreated(Bundle savedInstanceState)

  该方法在 Activity 的 onCreate 方法执行完之后调用，表示窗口已经初始化完成。在该方法中可以通过 getActivity().findViewById(Id) 来操纵 Activity 中的 view 了。

- public void onStart()

  调用该方法时，Fragment 已经可见了，但还无法与用户交互。

- public void onResume()

  调用该方法时，Fragment 已经可以与用户交互了。

- public void onPause()

  Fragment 活动正在暂停或者它的操作正在 Activity 中被修改，不再与用户交互。在此可做一些需要临时暂停的工作，如保存音乐播放的进度，然后在 onResume 中恢复。

- public void onStop()
  
  Fragment 活动正在停止或者它的操作正在 Activity 中被修改，不再对用户可见。

- public void onDestoryView()

  移除在 onCreateView 方法中创建的 View 时调用。

- public void onDestroy()

  做一些最后清理 Fragment 的状态。

- public void onDetach()

  取消 Fragment 与 Activity 的关联时调用。

  
## 与Activity通信

Fragment 可通过 getActivity() 访问 Activity 实例，并轻松地执行在 Activity 布局中查找 View 等任务。

```Java
View listView = getActivity().findViewById(R.id.list);
```

Activity 也可以使用 findFragmentById() 或 findFragmentByTag() , 通过从 FragmentManager 获取对 Fragment 的引用来调用 Fragment 中的方法。

```Java
ExampleFragment fragment = (ExampleFragment) getFragmentManager().findFragmentById(R.id.example_fragment);
```

> Fragment 与Activity 之间的交互可以通过 Fragment.setArguments(Bundle args) 以及 Fragment.getArguments() 来实现。

**创建对 Activity 的事件回调**

在某些情况下，可能需要通过 Fragment 与 Activity 共享事件。执行此操作的一个好方法是，在 Fragment 内定义一个回调接口，并要求宿主 Activity 实现它。当 Activity 通过该接口收到回调时，可以根据需要与布局中的其它 Fragment 共享这些信息。

例如，如果一个新闻应用的 Activity 有两个 Fragment , 一个用于显示文章列表（FragmentA），另一个用于显示文章详情（FragmentB），那么 FragmentA 必须在列表项被选定后告知 Activity , 以便它告知 FragmentB 显示该文章详情。

```Java
public static class FragmentA extends ListFragment {

    OnArticleSelectedListener mListener;
    ...

    @Override
    public void onAttach(Activity activity) {
        super.onAttach(activity);
        try {
            mListener = (OnArticleSelectedListener) activity;
        } catch (ClassCastException e) {
            throw new ClassCastException(activity.toString() + " must implement OnArticleSelectedListener");
        }
    }
    ...

    // Container Activity must implement this interface
    public interface OnArticleSelectedListener {
        public void onArticleSelected(Uri articleUri);
    }
    ...
}
```

宿主 Activity 会实现 OnArticleSelectedListener 接口并复写 onArticleSelected() , 将来自FragmentA 的事件通知 FragmentB . 为确保宿主 Activity 实现此接口，FragmentA 的 onAttach() 回调方法会通过转换传递到 onAttach() 中的 Activity 来实例化 OnArticleSelectedListener 的实例。如果 Activity 未实现接口，则片段会引发 ClassCastException .

实现时，mListener 成员会保留对 Activity 的 OnArticleSelectedListener 实现的引用，以便 FragmentA 可以通过调用 OnArticleSelectedListener 接口定义的方法与 Activity 共享事件。

## Fragment状态的持久化

由于 Activity 会经常性地发生配置变化，所以依附于它的 Fragment 就可能需要将其状态保存起来。有两个常用的方法可将 Fragment 的状态持久化。

1. 通过 onSaveInstanceState 与 onRestoreInstanceState 保存和恢复状态。

2. 让 Android 自动帮我们保存 Fragment 状态。

   在 Activity 中保存 Fragment 的方法：FragmentManager.putFragment(Bundle bundle, String key, Fragment fragment) ; 在 Activity 中获取所保存的 Fragment 的方法：FragmentManager.getFragment(Bundle bundle, String key) .

   这个方法仅仅能够保存 Fragment 中的控件状态，比如说 EditText 中用户已经输入的文字（注意！在这里，控件需要设置一个 id , 否则 Android 将不会为我们保存控件的状态），而 Fragment 中需要持久化的变量依然会丢失，此时就需要利用方法 1 了。

以下为状态持久化的事例代码：

**Activity 代码**

```Java
     FragmentA fragmentA;

     @Override
     protected void onCreate(@Nullable Bundle savedInstanceState) {
         super.onCreate(savedInstanceState);
         setContentView(R.layout.fragment_activity);

         if( savedInstanceState != null ){
             fragmentA = (FragmentA) getSupportFragmentManager().getFragment(savedInstanceState,"fragmentA");
         }
         init();
     }

     @Override
     protected void onSaveInstanceState(Bundle outState) {
         if( fragmentA != null ){
             getSupportFragmentManager().putFragment(outState,"fragmentA",fragmentA);
         }

         super.onSaveInstanceState(outState);
     }
```

**FragmentA 代码**

```Java
     @Nullable
     @Override
     public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
         if ( null != savedInstanceState ){
             String savedString = savedInstanceState.getString("string");
         }
         View root = inflater.inflate(R.layout.fragment_a,null);
         return root;
     }

 	@Override
     public void onSaveInstanceState(Bundle outState) {
         outState.putString("string","anAngryAnt");
         super.onSaveInstanceState(outState);
     }
```










