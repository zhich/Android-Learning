- [in & out 怎么记？](#in & out 怎么记？)
- [为啥要使用 in & out ？](#为啥要使用 in & out ？)
- [记住 in & out 的另一种方式](#记住 in & out 的另一种方式)





当我们在 Kotlin 中定义泛型时，我们会发现它需要使用到 `in` 和 `out` 两个关键字来定义。从形式上来讲，这是一种定义「逆变」和「协变」的方式。

那啥叫逆变？啥叫协变？可以参考下维基百科的定义：[协变与逆变](https://zh.wikipedia.org/wiki/%E5%8D%8F%E5%8F%98%E4%B8%8E%E9%80%86%E5%8F%98)

### in & out 怎么记？

**out（协变）**

如果泛型类**只将**泛型类型作为函数的返回（输出），那么使用 out：

```Kotlin
interface Production<out T> {
    fun produce(): T
}
```

可以称之为生产类/接口，因为它主要是用来生产（produce）指定的泛型对象。因此，我们可以简单地这样记忆：

**produce = output = out**

**in（逆变）**

如果泛型类**只将**泛型类型作为函数的入参（输入），那么使用 in：

```Kotlin
interface Consumer<in T> {
    fun consume(item: T)
}
```

可以称之为消费者类/接口，因为它主要是用来消费（consume）指定的泛型对象。因此我们可以简单地这样记忆：

**consume = input = in**

**invariant（不变）**

如果泛型类既将泛型类型作为函数参数，又将泛型类型作为函数的输出，那么既不用 out 也不用 in：

```Kotlin
interface ProductionConsumer<T> {
    fun produce(): T
    fun consume(item: T)
}
```

### 为啥要使用 in & out ？

举个例子，我们定义下汉堡类对象，它是一种快餐，也是一种食物。

```Kotlin
open class Food
open class FastFood : Food() 
class Burger : FastFood()
```

#### 汉堡生产者

根据上面定义的生产（Production）接口，我们可以进一步扩展它们来生产食物、快餐和汉堡：

```Kotlin
class FoodStore : Production<Food> {
    override fun produce(): Food {
        println("Produce food")
        return Food()
    }
}

class FastFoodStore : Production<FastFood> {
    override fun produce(): FastFood {
        println("Produce fast food")
        return FastFood()
    }
}

class InOutBurger : Production<Burger> {
    override fun produce(): Burger {
        println("Produce burger")
        return Burger()
    }
}
```

现在，我们可以这样赋值：

```Kotlin
val production1 : Production<Food> = FoodStore()
val production2 : Production<Food> = FastFoodStore()
val production3 : Production<Food> = InOutBurger()
```

显然，汉堡商店属于快餐商店，也属于食物商店。

> **因此，对于 out 类型，我们能够将使用子类泛型的对象赋值给使用父类泛型的对象。**

如果我们修改如下，那么就会出错了，因为食物或快餐商店是可以生产汉堡，但不一定仅仅生产汉堡：

```Kotlin
val production1 : Production<Burger> = FoodStore()  // Error
val production2 : Production<Burger> = FastFoodStore()  // Error
val production3 : Production<Burger> = InOutBurger()
```

#### 汉堡消费者

根据上面定义的消费（Consumer）接口，我们可以进一步扩展它们来消费食物、快餐和汉堡：

```Kotlin
class Everybody : Consumer<Food> {
    override fun consume(item: Food) {
        println("Eat food")
    }
}

class ModernPeople : Consumer<FastFood> {
    override fun consume(item: FastFood) {
        println("Eat fast food")
    }
}

class American : Consumer<Burger> {
    override fun consume(item: Burger) {
        println("Eat burger")
    }
}
```

我们可以将人类、现代人、美国人指定为汉堡消费者，所以可以这样赋值：

```Kotlin
val consumer1 : Consumer<Burger> = Everybody()
val consumer2 : Consumer<Burger> = ModernPeople()
val consumer3 : Consumer<Burger> = American()
```

不难理解，汉堡的消费者可以是美国人，也可以是现代人，更可以是人类。

> **因此，对于 in 泛型，我们能够将使用父类泛型的对象赋值给使用子类泛型的对象。**

反之，如果我们修改如下，就会出现错误，因为汉堡的消费者不仅仅是美国人或现代人。

```Kotlin
val consumer1 : Consumer<Food> = Everybody()
val consumer2 : Consumer<Food> = ModernPeople()  // Error
val consumer3 : Consumer<Food> = American()  // Error
```

### 记住 in & out 的另一种方式

- 父类泛型对象可以赋值给子类泛型对象，用 in；
- 子类泛型对象可以赋值给父类泛型对象，用 out。



参考资料：

[In and out type variant of Kotlin](https://medium.com/@elye.project/in-and-out-type-variant-of-kotlin-587e4fa2944c)

[Kotlin 泛型中的 in 和 out](https://zhuanlan.zhihu.com/p/32583310)

