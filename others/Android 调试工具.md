## 目录
- [性能调优工具Hugo](#性能调优工具hugo)
- [强大的调试器工具Stetho](#强大的调试器工具stetho)
- [调试手机中数据库Android-Debug-Database](#调试手机中数据库android-debug-database)
- [Findbugs](#findbugs)



### 性能调优工具Hugo

在进行 Android 性能调优、减少应用卡顿时，寻找可优化的 code 是一个必要的过程。如何发现应用中的耗时任务甚至是耗时函数呢，如果可以在 log 中打印**每个方法的执行时间**，甚至把执行方法时的**输入输出参数**同时打印，绝对是非常棒的功能。

幸运的是 Jake Wharton 大神已经做出了这样的工具：[Hugo](https://github.com/JakeWharton/hugo)。

**具体使用方法步骤：**

**1、添加依赖**

- 在 Project 的 gradle 中的 buildscript 领域中的 dependencies 领域里面添加如下代码：

```groovy
classpath 'com.jakewharton.hugo:hugo-plugin:1.2.1'
```

- 在 Module 的 gradle 的顶部添加如下代码：

```groovy
apply plugin: 'com.jakewharton.hugo'
```

**2、在代码里使用**

只需在需要打印信息的方法上增加 `@DebugLog` 即可：

```Java
public class MainActivity extends Activity {

    private ArrayList<User> mUserList = new ArrayList<>();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        initUserList(3000);
    }

    @DebugLog
    private boolean initUserList(int count) {
        User user = null;
        for (int i = 0; i < count; i++) {
            user = new User("" + i, "test_" + i);
            mUserList.add(user);
        }
        return true;
    }
}
```

以上例子输出结果如下：

```Java
02-07 12:08:51.227 20972-20972/com.zhi.hello V/MainActivity: ⇢ initUserList(count=3000)
02-07 12:08:51.237 20972-20972/com.zhi.hello V/MainActivity: ⇠ initUserList [7ms] = true
```

> 打印日志只会在 debug builds 发生，注解也不会在任何编译后的 class 文件出现。完全不影响非 debug builds .

### 强大的调试器工具Stetho

[Stetho，一个功能强大的 Android 应用调试桥](http://www.jianshu.com/p/38d8324b126a)

### 调试手机中数据库Android-Debug-Database

- step1

在 application 的 module 里面添加依赖

```Groovy
debugCompile 'com.amitshekhar.android:debug-db:1.0.0'
```

- step2

启动应用，然后在 logcat 中看到这么一行

```Java
D/DebugDB: Open http://xxx.xxx.xxx.xxx:8080 in your browser
```

把它复制到你电脑的浏览器，你就可以看到你的 App 中的数据库，和 SharedPreferences .

### Findbugs

[FindBugs在AndroidStudio上的应用](http://blog.csdn.net/fengyuzhengfan/article/details/50775170)

[Android Studio 插件--Findbugs](http://blog.csdn.net/atangsir/article/details/50731550)