- [函数定义与使用](#函数定义与使用)
- [函数特点](#函数特点)
- [函数选择](#函数选择)




## 函数定义与使用
### run()

**定义**

```Java
/**
 * Calls the specified function [block] and returns its result.
 */
@kotlin.internal.InlineOnly
public inline fun <R> run(block: () -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return block()
}
```

**示例**

```Java
fun runTest1() {
    var name = "AA"
    run {
        val name = "BB"
        Log.e(TAG, name) // BB
    }
    Log.d(TAG, name) // AA
}
```

> run() 函数在 runTest1 函数中又提供了自己的作用域，并且 run() 函数中可以重新定义一个 name 变量，该变量只存在于 run() 函数中。以下介绍的几个函数和 run() 函数同理，都是提供了自己的作用域。

```Java
fun runTest2() {
    var success = true
    var result = run {
        if (success) {
            "200"
        } else {
            "404"
        }
    }
    Log.d(TAG, result) // 200
}
```

> **run() 返回作用域中的最后一个对象**。

### T.run()

**定义**

```Java
/**
 * Calls the specified function [block] with `this` value as its receiver and returns its result.
 */
@kotlin.internal.InlineOnly
public inline fun <T, R> T.run(block: T.() -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return block()
}
```

**示例**

```Java
fun tRunTest() {
    val result = "ABCDEF".run {
        Log.d(TAG, "字符串的长度为$length") // 字符串的长度为6
        substring(2)
    }
    Log.d(TAG, result) // CDEF
}
```

> T.run() 中通过 this 来获取 "ABCDEF" 对象，然后输出 length . **T.run() 返回作用域中的最后一个对象**。

### with()

**定义**

```Java
/**
 * Calls the specified function [block] with the given [receiver] as its receiver and returns its result.
 */
@kotlin.internal.InlineOnly
public inline fun <T, R> with(receiver: T, block: T.() -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return receiver.block()
}
```

**示例**

```Java
fun withTest() {
    val result = with("ABCDEF") {
        substring(2)
    }
    Log.d(TAG, result) // CDEF
}
```

> **with() 返回作用域中的最后一个对象**。

### T.let()

**定义**

```Java
/**
 * Calls the specified function [block] with `this` value as its argument and returns its result.
 */
@kotlin.internal.InlineOnly
public inline fun <T, R> T.let(block: (T) -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return block(this)
}
```

**示例**

```Java
fun letTest() {
    val result = "ABCDEF".let {
        it.substring(2) // it 代表 "ABCDEF"
    }
    Log.d(TAG, result) // CDEF
}
```

> **T.let() 返回作用域中的最后一个对象**。

### T.also()

**定义**

```Java
/**
 * Calls the specified function [block] with `this` value as its argument and returns `this` value.
 */
@kotlin.internal.InlineOnly
@SinceKotlin("1.1")
public inline fun <T> T.also(block: (T) -> Unit): T {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    block(this)
    return this
}
```

**示例**

```Java
fun alsoTest() {
    val  result = "ABCDEF".also {
        it.substring(2) // it 代表 "ABCDEF"
    }
    Log.d(TAG, result) // ABCDEF
}
```

> **T.also() 返回原来的对象不变**。

### T.apply()

**定义**

```Java
/**
 * Calls the specified function [block] with `this` value as its receiver and returns `this` value.
 */
@kotlin.internal.InlineOnly
public inline fun <T> T.apply(block: T.() -> Unit): T {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    block()
    return this
}
```

**示例**

```Java
fun applyTest() {
    val result = "ABCDEF".apply {
        this.substring(2) // this 代表 "ABCDEF"
    }
    Log.d(TAG, result) // ABCDEF
}
```

> **T.apply() 返回原来的对象不变**。

## 函数特点
### T.run()、T.run()、T.also()、T.apply() 函数

> xxx 表示函数名

```Java
class MyClass {
    fun test() {
        val str = "AA"
        val result = str.xxx {
            print(this) // 接收者
            print(it) // 传参
            100 // 返回值
        }
        print(result)
    }
}
```

| 函数           | 接收者（this）  | 传参（it）| 返回值（result）      |
| ------------- |---------------|----------|---------------------|
| T.run()       |   "AA"        | 编译错误   | 作用域中的最后一个对象  |
| T.let()       | this@Myclass  |  "AA"    | 作用域中的最后一个对象  |
| T.also()      | this@Myclass  |  "AA"    |   "AA" 对象（本身）   |
| T.apply()     | "AA"          | 编译错误  |   "AA" 对象（本身）   |


### run() 与 with(T) 函数

```Java
class MyClass {
    fun runTest() {
        var result = run {
            print(this) // 接收者
            print(it) // 传参
            100 // 返回值
        }
        print(result)
    }

    fun withTest() {
        val str = "AA"
        var result = with(str) {
            print(this) // 接收者
            print(it) // 传参
            100 // 返回值
        }
        print(result)
    }
}
```

| 函数           | 接收者（this） | 传参（it）| 返回值（result）      |
| ------------- |---------------|----------|---------------------|
| run()         | this@Myclass  | 编译错误  | 作用域中的最后一个对象  |
| with()        | "AA"          | 编译错误  | 作用域中的最后一个对象  |

### 函数特点汇总

| 函数           | 接收者（this）  | 传参（it）| 返回值（result）      |
| ------------- |---------------|----------|---------------------|
| T.run()       |   "AA"        | 编译错误  | 作用域中的最后一个对象  |
| run()         | this@Myclass  | 编译错误  | 作用域中的最后一个对象  |
| with()        | "AA"          | 编译错误  | 作用域中的最后一个对象  |
| T.let()       | this@Myclass  |  "AA"    | 作用域中的最后一个对象  |
| T.also()      | this@Myclass  |  "AA"    |   "AA" 对象（本身）   |
| T.apply()     | "AA"          | 编译错误  |   "AA" 对象（本身）   |

## 函数选择

![Mou icon](http://pcckwdbix.bkt.clouddn.com/20.png)



参考资料：

[Kotlin基础 — 操作符：run、with、let、also、apply、takeIf、takeUnless、repeat](https://blog.csdn.net/Love667767/article/details/79376813)

[Kotlin 操作符：run、with、let、also、apply 的差异与选择](https://www.jianshu.com/p/01e28c4cc730)

[聊一聊Kotlin扩展函数run,with,let,also和apply的使用和区别](https://blog.csdn.net/ljd2038/article/details/79576091)


