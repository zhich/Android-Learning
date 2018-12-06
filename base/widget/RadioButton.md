- [设置默认选中项](#设置默认选中项)
  - [在xml中设置](#在xml中设置)
  - [在代码中设置](#在代码中设置)
  - [根据下标设置](#根据下标设置)
- [设置点击事件](#设置点击事件)







### 设置默认选中项

#### 在xml中设置

```xml
<RadioGroup
    android:id="@+id/rgMap"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:checkedButton="@id/rbSateLliteMap"
    android:orientation="horizontal"
    android:paddingTop="10dp"
    android:paddingBottom="10dp">

    <RadioButton
        android:id="@+id/rbCommonMap"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:scaleX="0.8"
        android:scaleY="0.8"
        android:text="普通地图"
        android:textSize="16sp" />


    <RadioButton
        android:id="@+id/rbSateLliteMap"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:scaleX="0.8"
        android:scaleY="0.8"
        android:text="卫星地图"
        android:textSize="16sp" />
</RadioGroup>
```

在 RadioGroup 标签的 **android:checkedButton** 属性中设置默认选中的 id

#### 在代码中设置

```Kotlin
rgMap.check(R.id.rbSateLliteMap)
```

#### 根据下标设置

```Kotlin
(rgMap.getChildAt(selectedIndex) as RadioButton).isChecked = true
```

### 设置点击事件

```Kotlin
rgMap.setOnCheckedChangeListener { _, checkedId ->
    when (checkedId) {
        R.id.rbCommonMap -> {
            // ...
        }
        R.id.rbSateLliteMap -> {
            // ...
        }
    }
}
```






