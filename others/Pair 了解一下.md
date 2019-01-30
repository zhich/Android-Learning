- [介绍](#介绍)
- [用法](#用法)
- [使用场景](#使用场景)










### 介绍

**Pair** 的字面意思是“一对”、“一双”，瞄一眼它的源码，果不其然，里面只有两个字段 `first` 与 `second` .

```Java
public class Pair<F, S> {
    public final F first;
    public final S second;

    public Pair(F first, S second) {
        this.first = first;
        this.second = second;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Pair)) {
            return false;
        }
        Pair<?, ?> p = (Pair<?, ?>) o;
        return Objects.equals(p.first, first) && Objects.equals(p.second, second);
    }

    // ...

    public static <A, B> Pair <A, B> create(A a, B b) {
        return new Pair<A, B>(a, b);
    }
}
```

### 用法

它的使用也是非常简单的：

```Java
// 两种方式都可以创建 Pair 实例，而第二种方式内部实际上也是使用第一种方式创建
Pair pair1 = new Pair<Integer, String>(1, "111"); // 第一种方式创建
Pair pair2 = Pair.create(1, 111); // 第二种方式创建
Pair pair3 = Pair.create(1, 111);

Log.e(TAG, pair1.first.toString()); // 1
Log.e(TAG, pair1.second.toString()); // 111
Log.e(TAG, pair1.second.equals("111") + ""); // true
Log.e(TAG, pair1.second.equals(111) + ""); // false

Log.e(TAG, pair1.equals(pair2) + ""); // false
Log.e(TAG, pair2.equals(pair3) + ""); // true
```

从以上示例可知：

- Pair 的 first 获取的是第一个位置的数据，second 获取的是第二个位置的数据；
- Pair 的 equals 比较的是 first 与 second 值是否同时 equals .

说到 `equals` , 上面的源码只是 android.util 包下 Pair 类的 equals 方法，由于 android.support.v4.util 包下也有 Pair 类，通过比较，两个包下的 Pair 类只有 equals 方法有所不同，其它方法无异。

```Java
// android.util 包下
public boolean equals(Object o) {
    if (!(o instanceof Pair)) {
        return false;
    }
    Pair<?, ?> p = (Pair<?, ?>) o;
    return Objects.equals(p.first, first) && Objects.equals(p.second, second);
}

// android.support.v4.util 包下
public boolean equals(Object o) {
    if (!(o instanceof Pair)) {
        return false;
    } else {
        Pair<?, ?> p = (Pair)o;
        return ObjectsCompat.equals(p.first, this.first) && ObjectsCompat.equals(p.second, this.second);
    }
}
```

ObjectsCompat 类里面的 equals 方法：

```Java
public static boolean equals(@Nullable Object a, @Nullable Object b) {
    if (VERSION.SDK_INT >= 19) {
        return Objects.equals(a, b);
    } else {
        return a == b || a != null && a.equals(b);
    }
}
```

Objects 是 Java7 以后才有的类，而 Android 是从 4.4 开始支持 JDK7 编译的，因此为了兼容 4.4 之前的版本，在 v4 中加入了一个不依赖 JDK7 的 Pair 类。

### 使用场景

在**既要以键值对的方式存储数据列表，同时在输出时保持顺序**的情况下，我们可以使用 Pair 搭配 ArrayList 实现。

**场景一：**

假如我们需要生成 n 个按钮，而每个按钮都有 code 值、展示文本内容的 content 值，当我们点击其中一个按钮后就根据 code 值去做指定的事情（如网络请求）。

```Java
ArrayList<Pair<String,String>> dataList = new ArrayList();
```

**场景二：**

记录推送过来的消息，我们可以用 Pair 的 first 记录消息到达的时间戳，second 记录消息体。

```Java
ArrayList<Pair<Long,Message>> dataList = new ArrayList();
```