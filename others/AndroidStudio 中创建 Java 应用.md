先新建一个 Project , 然后新建一个 Module ;

新建 Module 时选择 Java Library , 然后 Next 直到 Finish ;

在 MyClass 里添加静态的 main 函数：

```Java
package com.zch;

public class MyClass {

    public static void main(String[] args) {
        System.out.println("Hello World.");
    }

}
```

此时还不能直接运行或者调试，需要配置一下。

点击菜单的 Run -> Edit Configuration , 然后点击 `+` 选择 Application .

![a.png](https://github.com/zhich/AndroidDev/blob/master/picture/android-studio-java-config.png)