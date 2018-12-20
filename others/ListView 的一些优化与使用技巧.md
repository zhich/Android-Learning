## 目录
- [优化](#优化)
  - [基本原则](#基本原则)
  - [item监听事件复用](#item监听事件复用)
- [使用技巧](#使用技巧)
  - [设置item间的分割线](#设置item间的分割线)
  - [取消item的点击效果](#取消item的点击效果)
  - [设置ListView显示位置](#设置listview显示位置)
  - [遍历ListView中所有item](#遍历listview中所有item)
  - [处理空ListView](#处理空listview)
  - [自动显示与隐藏布局的ListView](#自动显示与隐藏布局的listview)



### 优化

#### 基本原则

- 如果自定义适配器，那么在 getView 方法中要考虑方法传进来的参数 contentView 是否为 null , 如果为 null 就创建 contentView 并返回，如果不为 null 则直接使用。在这个方法中尽可能少创建 view ;
- 给 contentView 设置 tag（setTag()），传入一个 ViewHolder 对象，用于缓存要显示的数据，可以达到图像数据异步加载的效果；
- 如果 listview 需要显示的 item 很多，就要考虑分页加载。比如一共要显示 100 条或者更多的时候，我们可以考虑先加载 20 条，等用户拉到列表底部的时候再去加载接下来的 20 条。



#### item监听事件复用

比如在一个 MainActivity 中有一个 ListView , ListView 中的每一个 item 显示用户名、登录按钮、注册按钮。其中，对登录按钮和注册按钮设置监听事件。

> tip : 我们只需要在 convertView 为 null 时设置监听事件即可，不需要在每次执行 getView 时重复设置，避免重复创建监听事件对象。

MainActivity.java 的代码：

```java
public class MainActivity extends Activity {

    private ListView lvUsers;
    private ArrayList<User> userList = new ArrayList<>();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        lvUsers = (ListView) this.findViewById(R.id.main_lv_users);
        initData();
        lvUsers.setAdapter(new UserAdapter(this, userList));
    }

    /**
     * 初始化数据
     */
    private void initData() {
        User user = null;
        for (int i = 0; i < 30; i++) {
            user = new User("" + i, "test_" + i);
            userList.add(user);
        }
    }

}
```

activity_main.xml 的代码：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context="com.zhi.hello.MainActivity">

    <ListView
        android:id="@+id/main_lv_users"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
</LinearLayout>
```

item_user.xml 的代码：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal">

    <TextView
        android:id="@+id/itemUser_tv_name"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginLeft="20dp"
        tools:text="name" />

    <Button
        android:id="@+id/itemUser_btn_login"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginLeft="20dp"
        android:text="登录" />

    <Button
        android:id="@+id/itemUser_btn_register"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginLeft="20dp"
        android:text="注册" />
</LinearLayout>

```

User.java 的代码：

```java
public class User {

    public String id;//用户id
    public String name;//用户名

    public User() {

    }

    public User(String id, String name) {
        this.id = id;
        this.name = name;
    }
}
```

UserAdapter.java 的代码：

```java
public class UserAdapter extends BaseAdapter {

    private Context context;
    private LayoutInflater inflater;

    private ArrayList<User> userList = new ArrayList<>();

    public UserAdapter(Context context, ArrayList<User> userList) {
        this.context = context;
        this.inflater = (LayoutInflater) context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
        this.userList = userList;
    }

    @Override
    public int getCount() {
        return null == userList ? 0 : userList.size();
    }

    @Override
    public Object getItem(int position) {
        return null == userList ? null : userList.get(position);
    }

    @Override
    public long getItemId(int position) {
        return position;
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        final ViewHolder holder;
        if (null == convertView) {
            holder = new ViewHolder();
            convertView = inflater.inflate(R.layout.item_user, null);
            holder.tvName = (TextView) convertView.findViewById(R.id.itemUser_tv_name);
            holder.btnLogin = (Button) convertView.findViewById(R.id.itemUser_btn_login);
            holder.btnRegister = (Button) convertView.findViewById(R.id.itemUser_btn_register);
            convertView.setTag(holder);
            holder.btnLogin.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    int position = CommonUtils.getIntTagFromView(v); // 取出登录按钮的位置 position
                    if (position != -1) {
                        Toast.makeText(context, userList.get(position).name + "...登录", Toast.LENGTH_SHORT).show();
                    }
                }
            });
            holder.btnRegister.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    int position = CommonUtils.getIntTagFromView(v); // 取出注册按钮的位置 position
                    if (position != -1) {
                        Toast.makeText(context, userList.get(position).name + "...注册", Toast.LENGTH_SHORT).show();
                    }
                }
            });
        } else {
            holder = (ViewHolder) convertView.getTag();
        }
        // 把登录按钮和注册按钮的位置 position 设置到它们各自控件的 tag 里面去
        holder.btnLogin.setTag(Integer.valueOf(position));
        holder.btnRegister.setTag(Integer.valueOf(position));

        User user = userList.get(position);
        if (null != user) {
            holder.tvName.setText(user.name);
        }

        return convertView;
    }

    class ViewHolder {
        TextView tvName;
        Button btnLogin;
        Button btnRegister;
    }

}
```

UserAdapter 中关键的代码是：

```java
// 把登录按钮和注册按钮的位置 position 设置到它们各自控件的 tag 里面去
holder.btnLogin.setTag(Integer.valueOf(position));
holder.btnRegister.setTag(Integer.valueOf(position));
```

每次执行 getView 方法都把位置 position 设置给即将需要使用的 View（这里为登录按钮和注册按钮）。

CommonUtils.java 的代码：

```java
public class CommonUtils {

    /**
     * @param view
     * @return the value when call {@link View#setTag(Object)}, if not found,
     * return -1
     */
    public static int getIntTagFromView(View view) {
        Object objTag = view.getTag();
        if (objTag != null && objTag instanceof Integer) {
            Integer i = (Integer) objTag;
            return i.intValue();
        }

        return -1;
    }

}
```

### 使用技巧

#### 设置item间的分割线

divider : 设置分割线，可以为颜色或者图片资源。

dividerHeight : 设置分割线的高度。



#### 取消item的点击效果

```xml
android:listSelector=”#00000000” 或者 android:listSelector=”@android:color/transparent”
```



#### 设置ListView显示位置

```Java
mListView.setSelection(N);
```



#### 遍历ListView中所有item

ListView 作为一个 ViewGroup , 为我们提供了操纵子 View 的各种方法。

最常用的就是通过 getChildAt() 来获取第 i 个子 View .

```Java
for(int i = 0 ; i<mListView.getChildCount(); i++){
    View view =mListView.getChildAt(i);
}
```



#### 处理空ListView

ListView 提供了一个方法 setEmptyView(View emptyView) , 通过这个方法可以给 ListView 设置一个在空数据下显示的默认提示界面。布局和 Activity 中的代码如下。

```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >
    <ListView
        android:id="@+id/listview"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
    <ImageView
        android:id="@+id/empty_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:src="@+id/mipmap/ic_launcher" />
</FrameLayout>
```

```Java
ListView mListView = (ListView)this.findViewById(R.id.listview);
mListView.setEmptyView(findViewById(R.id.empty_view));
```



#### 自动显示与隐藏布局的ListView

当用户对 ListView 进行上拉操作时标题栏消失，而进行下拉操作时标题栏又重现。

Activity 代码：

```Java
public class ScrollHideListView extends Activity {
    private Toolbar mToolbar;
    private ListView mListView;
    private String[] mStr = new String[20];
    private int mTouchSlop;
    private float mFirstY;
    private float mCurrentY;
    private int direction;
    private ObjectAnimator mAnimator;
    private boolean mShow = true;
    View.OnTouchListener myTouchListener = new View.OnTouchListener() {
        @Override
        public boolean onTouch(View v, MotionEvent event) {
            switch (event.getAction()) {
                case MotionEvent.ACTION_DOWN:
                    mFirstY = event.getY();
                    break;
                case MotionEvent.ACTION_MOVE:
                    mCurrentY = event.getY();
                    if (mCurrentY - mFirstY > mTouchSlop) {
                        direction = 0;// down
                    } else if (mFirstY - mCurrentY > mTouchSlop) {
                        direction = 1;// up
                    }
                    if (direction == 1) {
                        if (mShow) {
                            toolbarAnim(1);//hide
                            mShow = !mShow;
                        }
                    } else if (direction == 0) {
                        if (!mShow) {
                            toolbarAnim(0);//show
                            mShow = !mShow;
                        }
                    }
                    break;
                case MotionEvent.ACTION_UP:
                    break;
            }
            return false;
        }
    };
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.scroll_hide);
        // 获取系统默认的最低滑动距离，超过它系统就定义为滑动状态
        mTouchSlop = ViewConfiguration.get(this).getScaledTouchSlop();
        mToolbar = (Toolbar) findViewById(R.id.toolbar);
        mListView = (ListView) findViewById(R.id.listview);
        for (int i = 0; i < mStr.length; i++) {
            mStr[i] = "Item " + i;
        }
        View header = new View(this);
        // 获取系统 Actionbar 的高度，并设置给 header
        header.setLayoutParams(new AbsListView.LayoutParams(
                AbsListView.LayoutParams.MATCH_PARENT,
                (int) getResources().getDimension(
                        R.dimen.abc_action_bar_default_height_material)));
        mListView.addHeaderView(header);
        mListView.setAdapter(new ArrayAdapter<String>(
                ScrollHideListView.this,
                android.R.layout.simple_expandable_list_item_1,
                mStr));
        mListView.setOnTouchListener(myTouchListener);
    }
	// 控制布局显示隐藏的动画
    private void toolbarAnim(int flag) {
        if (mAnimator != null && mAnimator.isRunning()) {
            mAnimator.cancel();
        }
        if (flag == 0) {
            mAnimator = ObjectAnimator.ofFloat(mToolbar,
                    "translationY", mToolbar.getTranslationY(), 0);
        } else {
            mAnimator = ObjectAnimator.ofFloat(mToolbar,
                    "translationY", mToolbar.getTranslationY(),
                    -mToolbar.getHeight());
        }
        mAnimator.start();
    }
}
```

布局代码：

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">
    <ListView
        android:id="@+id/listview"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:headerDividersEnabled="false" />
    <android.support.v7.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"
        android:background="@android:color/holo_blue_light" />
</RelativeLayout>
```

Google 已经推荐使用 Toolbar 控件取代 ActionBar 了，因为它更加灵活，在使用时，一定要注意使用的 theme 一定是要 NoActionBar 的，否则会引起冲突，同时不要忘记引入编译。

```groovy
dependencies{
    compile fileTree(include: ['*.jar'], dir:'libs')
    compile'com.android.support:appcompat-v7:21.0.3'
}
```

