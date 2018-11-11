- [前言](#前言)
- [ViewModel使用](#viewmodel使用)
- [ViewModel的生命周期](#viewmodel的生命周期)
- [Fragment之间共享数据](#fragment之间共享数据)





### 前言

在 Android 中，ViewModel 的作用就是在 **UI 控制器**（ 如 Activity、Fragment）的生命周期中保存和管理 UI 相关的数据。ViewModel 保存的数据在配置更改（如屏幕旋转）后会依然存在，不会丢失。

在屏幕旋转的时候，Activity 会重建，为了不让数据丢失，我们通常的做法是在 `onSaveInstanceState()` 方法中通过 bundle 保存数据，然后在 `onCreate()` 或 `onRestoreInstanceState()` 方法中取出 bundle 来恢复数据。然而，这种方式有一定的局限性，它只适用于**可序列化然后反序列化**的少量数据，对于 Bitmap 等比较大的数据就不适用了。

另一方面，UI 控制器通常需要做一些耗时的异步调用操作，并且需要去管理这些调用。UI 控制器需要确保系统在销毁后去清理掉这些异步调用，以避免潜在的内存泄漏，这种管理方式需要大量的维护工作。而且，在配置更改后重建对象是很浪费资源的，因为该对象可能必须重新发出之前已经发出过的调用。

UI 控制器一般只负责显示和处理用户操作，加载数据库数据或网络数据的工作应该委托给其它类，这样会让测试工作更加容易地进行。因此，**将视图数据相关操作从 UI 控制器逻辑中分离出来是很有必要。**

### ViewModel使用

比如，一个 ViewModelActivity 需要展示一个 User 的列表数据，那么可以定义一个 UserViewModel 来获取数据，然后在 ViewModelActivity 中创建一个 UserViewModel 对象来获取到 User 的列表数据。

```Kotlin
class UserViewModel : ViewModel() {

    private lateinit var users: MutableLiveData<List<User>>

    fun getUsers(): LiveData<List<User>> {
        if (!::users.isInitialized) {
            users = MutableLiveData()
            loadUsers()
        }
        return users
    }

    private fun loadUsers() {
        // Do an asynchronous operation to fetch users .
        Thread(Runnable {
            Thread.sleep(3000)
            // 在子线程发送值用 postValue , 否则用 setValue .
            users.postValue(listOf(User("1", "AA"), User("2", "BB")))
        }).start()
    }
}
```

```Kotlin
class ViewModelActivity : AppCompatActivity() {

    private val TAG = "ViewModelActivity"

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_view_model)

        // 就算配置更改（如屏幕旋转）了，获取到的 userViewModel 对象还会是上一次的 UserViewModel 对象
        val userViewModel = ViewModelProviders.of(this).get(UserViewModel::class.java)

        // 这里的 this 需要用实现了 LifecycleOwner 的类的 this . 如 AppCompatActivity、FragmentActivity
        userViewModel.getUsers().observe(this, Observer {
            Log.e(TAG, it.toString())
            // 打印结果：[User(id=1, name=AA), User(id=2, name=BB)]
        })
    }
}
```

查看源码可知，ViewModelProviders.of(this) 获取了一个全新的 ViewModelProvider 对象，

```Kotlin
public static ViewModelProvider of(@NonNull FragmentActivity activity,
            @Nullable Factory factory) {
        Application application = checkApplication(activity);
        if (factory == null) {
            factory = ViewModelProvider.AndroidViewModelFactory.getInstance(application);
        }
        return new ViewModelProvider(ViewModelStores.of(activity), factory);
    }
```

ViewModelProvider 对象调用 get() 方法获取到我们需要的 ViewModel 对象。追踪一下 get() 方法可以知道，ViewModel 对象是存储在一个 ViewModelStore 类的对象中的，该类里面使用 HashMap 来保存和获取 ViewModel . 

```Kotlin
ViewModel viewModel = mViewModelStore.get(key);
```

获取 ViewModel 使用的 key 相对具体的 ViewModel 类是不会变化的，因此从 ViewModelStore 中取出的 ViewModel 对象也不会变。包括在配置更改后也可以获取到之前的 ViewModel .

当宿主 Activity 调用了 finish() 方法，系统会调用 ViewModel 对象的 onCleared() 方法来让它清理掉资源，到这里之后 ViewModel 才会被释放掉。

> ViewModel 里面不要引用 View、或者任何持有 Activity 类的 context , 否则会引发内存泄漏问题。

当 ViewModel 需要 Application 类的 context 来获取资源、查找系统服务等，可以继承 **AndroidViewModel** 类。 

```Kotlin
class MyAndroidViewModel(application: Application) : AndroidViewModel(application) {

    private val app
        get() = getApplication<Application>()

    fun getStatus(code: Int): String {
        return when (code) {
            1 -> app.resources.getString(R.string.be_late) // 迟到
            2 -> app.resources.getString(R.string.leave_early) // 早退
            else -> app.resources.getString(R.string.absenteeism) // 旷工
        }
    }
}
```

```Kotlin
val myAndroidViewModel = ViewModelProviders.of(this).get(MyAndroidViewModel::class.java)
Log.e(TAG, myAndroidViewModel.getStatus(2))
// 打印结果：早退
```

### ViewModel的生命周期

ViewModel 会一直保留在内存中，直到 Activity / Fragment 在以下情况下才会销毁：

- 宿主 Activity 被 finish 后调用 onDestroy 方法。
- 宿主 Fragment 被 detached 后调用 onDetach 方法。

下图展示了一个 Activity 经历了旋转然后调用 finish 的各种生命周期状态，同时展示了关联了该 Activity 的 ViewModel 的生命周期。（UI 控制器是 Fragment 的情况也类似。）

![Mou icon](http://pcckwdbix.bkt.clouddn.com/viewmodel-lifecycle.png)

### Fragment之间共享数据

假设我们有这样的需求：在一个 MasterFragment 中有一个 User 列表，点击列表项后将点中的 User 对象传递给 DetailFragment 用于展示详细的 User 信息。

我们一般的做法是：在两个 Fragment 中定义一些通信接口，并且宿主 Activity 需要把它们绑定起来，这样做相当繁琐。并且两个 Fragment 还需要处理另外的 Fragment 尚未创建或者可见的场景。

为了避免以上繁琐的做法，我们可以通过两个 Fragment 之间共享一个 ViewModel 的方式来实现数据通信。

```Kotlin
class SharedViewModel : ViewModel() {

    val selected = MutableLiveData<User>()

    fun select(user: User) {
        selected.value = user
    }
}
```

```Kotlin
class MasterFragment : Fragment() {

    private val dataList = listOf(User("1", "张三"), User("2", "李四"), User("3", "王五"))

    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View? {
        return inflater.inflate(R.layout.fragment_master, container, false)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        var model = activity?.run {
            ViewModelProviders.of(this).get(SharedViewModel::class.java)
        } ?: throw Exception("Invalid Activity")

        lvMaster.adapter = ArrayAdapter<User>(
                activity,
                android.R.layout.simple_expandable_list_item_1,
                dataList)

        lvMaster.setOnItemClickListener { _, _, position, _ ->
            model.select(dataList[position])
        }
    }
}
```

```Kotlin
class DetailFragment : Fragment() {

    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View? {
        return inflater.inflate(R.layout.fragment_detail, container, false)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        var model: SharedViewModel = activity?.run {
            ViewModelProviders.of(this).get(SharedViewModel::class.java)
        } ?: throw Exception("Invalid Activity")
        
        model.selected.observe(this, Observer<User> { item ->
            tvDetail.setText("${item?.id} : ${item?.name}")
        })
    }
}
```

> 需要特别注意，两个 Fragment 都需要使用它们的宿主 Activty 的 this 来获取 ViewModelProviders ， 这样才确保它们获取到的是同一个 ViewModel 对象。

这种数据通信的方式有以下几个好处：

- 宿主 Activity 不需要做任何的事情，也完全不知道 Fragment 之间的通信；
- 一个 Fragment 不需要知道另一个 Fragment 中除了 ViewModel 契约之外的其它事情，哪怕另一个 Fragment 消失了，它也继续保持正常工作；
- 每个 Fragment 都有自己的生命周期，它们之间互不影响，哪怕某一个 Fragment 被其它 Fragment 替换了，UI 还是会继续工作，没有任何问题。


[文中 Demo GitHub 地址](https://github.com/zhich/AndroidJetpackDemo)
