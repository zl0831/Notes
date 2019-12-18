# Android笔记

## Fragment

### Fragment的加载

几种加载方式：

>1. 静态加载
>2. 动态加载
>3. 通过FragmentAdapter加载

&emsp;

* **静态加载**

直接将创建好的fragment写在xml布局文件里面,一般这种方式比较少用

```xml
// 具体代码
// xml布局文件
<fragment
    android:id="@+id/myFragment" // id必须要有
    android:name="com.zl.testactivity.fragment.TestFragmentOne" // name == 我们自己创建的Fragment类
    android:layout_width="wrap_content"
    android:layout_height="match_parent"/>
```

* **动态加载**

通过FragmentManager来实现动态加载

```java
// 具体代码
FragmentManager manager = getSupportFragmentManager();
FragmentTransaction transaction = manager.beginTransaction();
// 加载当前显示的Fragment
transaction.replace(R.id.relativeLayout_main, new MyFragment());
transaction.commit(); // 提交创建Fragment请求
```

这儿有个小细节，即**add和replace**区别,在此简单的描述下：
>add是把一个fragment添加到一个容器container 里。
replace是先remove掉相同id的所有fragment，然后再add当前的这个fragment。
在大部分情况下，这两个的表现基本相同。
>
>网上很多源码分析的，这儿就不写了，就写一个总结性的描述：
在一般的情况下使用add和replace，并没有什么区别，replace会先查找FragmentManager中是否包含相同mContainerId 的fragment，如果有就先remove它，然后再add它。
注意它是根据containerViewId来进行比较的，并不是依据对象是否相等来判断的。在这一点上还需要注意区别。

* **通过FragmentAdapter加载**

这种方式需要引入下面这个包（多个界面相互切换）:
`implementation 'com.android.support:design:26.0.+'`

然后在xml布局中使用TabLayout和ViewPager：

```xml
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    // 注意这个TabLayout只有添加了design依赖之后才会出现
    <android.support.design.widget.TabLayout
        android:id="@+id/testFragmentTablayout"
        android:layout_width="match_parent"
        android:layout_height="80sp"
        android:background="#1569b9"/>
    <android.support.v4.view.ViewPager
        android:id="@+id/testFragmentViewPager"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>
</LinearLayout>
```

xml基本就这些代码，其他的在java代码里面写就成：

```java
// 创建Adapter
public class TestFragmentAdapter extends FragmentPagerAdapter {
    // 用来添加Fragment
    private List<Fragment> fragments;
    // 用来存储Fragment标题
    private String title[] = {"第一个Fragment","第二个Fragment"};

    public void setFragments(List<Fragment> fragments) {
        this.fragments = fragments;
        notifyDataSetChanged();
    }

    public TestFragmentAdapter(FragmentManager fm) {
        super(fm);
    }

    @Override
    public Fragment getItem(int position) {
        return fragments.get(position);
    }

    @Override
    public int getCount() {
        return fragments != null ? fragments.size() : 0;
    }

    @Override
    public CharSequence getPageTitle(int position) {
        // 设置Fragment标题
        return title[position];
    }
}
```

```java
// 使用adapter
// 初始化TabLayout、ViewPager绑定适配器添加之前创建好的Fragment到集合中之后设置TabLayout与ViewPager
mTabLayout = (TabLayout) findViewById(R.id.testFragmentTablayout);
mViewPager = (ViewPager) findViewById(R.id.testFragmentViewPager);
// 创建Fragment集合
List<Fragment> fragments = new ArrayList<>();
// 将Fragment添加到集合
fragments.add(new TestFragmentOne());
fragments.add(new TestFragmentTwe());
// 初始化适配器
TestFragmentAdapter adapter = new TestFragmentAdapter(getSupportFragmentManager());
adapter.setFragments(fragments);
mViewPager.setAdapter(adapter);
// 设置ViewPager
mTabLayout.setupWithViewPager(mViewPager);
```

通过adapter创建Fragment的基本流程就是这样。

每次都要新建一个adapter类比较麻烦，我习惯直接临时new一个：

```java
//这儿adapter也可以不用单独创建，我喜欢直接在ViewPager.setAdapter（）时new一个即可
mViewPager.setAdapter(new FragmentPagerAdapter(getChildFragmentManager()) {
    @Override
    public Fragment getItem(int position) {
       return fragments.get(position);
    }
    @Override
    public int getCount() {
        return fragments.size();
    }
    @Override
    public CharSequence getPageTitle(int position) {
        return titles.get(position);
    }
});
```

## Bundle

### 参数传递

* **传递Map和List**,其他基本类型的传递，bundle都给封装好了，直接用就成

利用SerializableMap来传递map和list，需要将数据对象序列化

```java
//自定义序列化数据对象
public class SerializableData<T> implements Serializable {

    private List<T> list;
    private Map<String,T> map;

    public List<T> getList() {
        return list;
    }
    public void setList(List<T> list) {
        this.list = list;
    }
    public Map<String,T> getMap()
    {
        return map;
    }
    public void setMap(Map<String,T> map)
    {
        this.map=map;
    }

}
```

```java
// 传递map
Bundle bundle = new Bundle();
SerializableData map = new SerializableData<Object>();
Map mapData = new HashMap<String,Object>();
map.setMap(mapData);
bundle.putSerializable("map", map);

// 传递list
SerializableData list = new SerializableData<Map>();
List listData = new ArrayList<HashMap<String,Object>>();
list.setList(listData);
bundle.putSerializable("list", list);
```

解析bundle获取map和list

```java
SerializableData data = (SerializableData) bundle.getSerializable("map");
Map map = data.getMap();
SerializableData list = (SerializableData) bundle.getSerializable("list");
List list = list.getList();
```

传递List还有另外一种方法，将数据对象定义为Parcelable类型,相对麻烦些

```java
public class Group implements Parcelable{
    private int id;
    private String name;

    public String getId() {
        return id;
    }
    public void setId(String id) {
        this.id = id;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }

    @Override
    public void writeToParcel(Parcel arg0, int arg1) {
        // TODO Auto-generated method stub
        // 1.必须按成员变量声明的顺序封装数据，不然会出现获取数据出错
        // 2.序列化对象
        arg0.writeString(id);
        arg0.writeString(name);
    }

    // 1.必须实现Parcelable.Creator接口,否则在获取Person数据的时候，会报错，如下：
    // android.os.BadParcelableException:
    // Parcelable protocol requires a Parcelable.Creator object called  CREATOR on class com.um.demo.Person
    // 2.这个接口实现了从Percel容器读取Person数据，并返回Person对象给逻辑层使用
    // 3.实现Parcelable.Creator接口对象名必须为CREATOR，不如同样会报错上面所提到的错；
    // 4.在读取Parcel容器里的数据事，必须按成员变量声明的顺序读取数据，不然会出现获取数据出错
    // 5.反序列化对象
    public static final Parcelable.Creator<Gate> CREATOR = new Creator(){
        @Override
        public Gate createFromParcel(Parcel source) {
            // TODO Auto-generated method stub
            // 必须按成员变量声明的顺序读取数据，不然会出现获取数据出错
            Gate p = new Gate();
            p.setId(source.readString());
            p.setName(source.readString());
            return p;
        }

        @Override
        public Gate[] newArray(int size) {
            // TODO Auto-generated method stub
            return new Gate[size];
        }
    };
}
```

用bundle传递Parcelable对象

```java
// 传递数据
Bundle bundle = new Bundle();
List groups = new ArrayList<Group>();
bundle.putParcelableArrayList("groups", groups);

//获取Parcelable对象
groups = bundle.getParcelableArrayList("groups");
```
