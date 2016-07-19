---
layout: post
title:  ViewPager中的Fragment与普通Fragment在生命周期上有何不同
date:   2016-07-18 14:54:54
category: "学习"
---

今天在看ppt的时候看到“ViewPager中的Fragment与普通Fragment在生命周期上有何不同”这么一个问题，哇！一看就知道肯定有很大的不同，那到底有什么不同呢？

###普通的Fragment的生命周期

普通的Fragment的生命周期在下面这张图就基本能概括了

![普通的Fragment的生命周期](https://raw.githubusercontent.com/zhouchaoyuan/zhouchaoyuan.github.io/master/images/lifecycle.png)

###ViewPager中的Fragment的生命周期

对于ViewPager中的Fragment是由Adapter控制的，对于在屏幕中的Fragment肯定是处于onResume状态的，那么处于左右两侧的Fragment又是怎么样的状态呢？我们来写一个Demo看看情况吧！

####一个测试用的Fragment

xml文件：

```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent" android:layout_height="match_parent">
    <TextView
        android:text="Fragment"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />

</FrameLayout>
```

Fragment源代码：

```java
import android.content.Context;
import android.os.Bundle;
import android.support.annotation.Nullable;
import android.support.v4.app.Fragment;
import android.util.Log;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
/**
 * Created by zhouchaoyuan on 16/7/18.
 */
public class TestUIFragment extends Fragment {
    private int id = 0;
    public TestUIFragment(int id){
        super();
        this.id = id;
    }
    @Override
    public void onAttach(Context context) {
        super.onAttach(context);
        Log.e(Consts.EXAMPLE_TAG, "TestUIFragment"+id+".onAttach");
    }
    @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Log.e(Consts.EXAMPLE_TAG, "TestUIFragment"+id+".onCreate");
    }
    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        Log.e(Consts.EXAMPLE_TAG, "TestUIFragment"+id+".onCreateView");
        return inflater.inflate(R.layout.fragment_layout, container, false);
    }
    @Override
    public void onViewCreated(View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        Log.e(Consts.EXAMPLE_TAG, "TestUIFragment"+id+".onViewCreated");
    }
    @Override
    public void onStart() {
        super.onStart();
        Log.e(Consts.EXAMPLE_TAG, "TestUIFragment"+id+".onStart");
    }
    @Override
    public void onResume() {
        super.onResume();
        Log.e(Consts.EXAMPLE_TAG, "TestUIFragment"+id+".onResume");
    }
    @Override
    public void onPause() {
        super.onPause();
        Log.e(Consts.EXAMPLE_TAG, "TestUIFragment"+id+".onPause");
    }
    @Override
    public void onStop() {
        super.onStop();
        Log.e(Consts.EXAMPLE_TAG, "TestUIFragment"+id+".onStop");
    }
    @Override
    public void onDestroyView() {
        super.onDestroyView();
        Log.e(Consts.EXAMPLE_TAG, "TestUIFragment"+id+".onDestroyView");
    }
    @Override
    public void onDestroy() {
        super.onDestroy();
        Log.e(Consts.EXAMPLE_TAG, "TestUIFragment"+id+".onDestroy");
    }
    @Override
    public void onDetach() {
        super.onDetach();
        Log.e(Consts.EXAMPLE_TAG, "TestUIFragment"+id+".onDetach");
    }
}

```

上面的代码几乎重写了Fragment的所有方法，然后嘛，我们就需要写一个带有ViewPager的Activity了，如下：

```java
public class ViewpagerActivity extends AppCompatActivity {
    private List<Fragment> fragments;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_viewpager);
        ViewPager pager = (ViewPager) findViewById(R.id.viewPager);
        init_data();
        pager.setAdapter(new ViewPagerAdapter(this, fragments, new String[]{"啦啦啦","啦啦啦","啦啦啦","啦啦啦"},
                getSupportFragmentManager()));
    }
    private void init_data(){
        fragments = new ArrayList<>();
        fragments.add(new TestUIFragment(1));
        fragments.add(new TestUIFragment(2));
        fragments.add(new TestUIFragment(3));
        fragments.add(new TestUIFragment(4));
    }
}

```

然后也xml布局如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="cn.zhouchaoyuan.example.ViewpagerActivity">
    <android.support.v4.view.ViewPager
        android:id="@+id/viewPager"
        android:layout_width="match_parent"
        android:layout_height="match_parent"></android.support.v4.view.ViewPager>

</LinearLayout>
```

最后还需要一个Adapter，因为用到了ViewPager，代码如下：

```java
public class ViewPagerAdapter extends FragmentPagerAdapter {
    private String[] titles;
    private List<Fragment> list;
    private Context context;
    public ViewPagerAdapter(Context context,List<Fragment> list,String[] titles,FragmentManager fm) {
        super(fm);
        this.titles = titles;
        this.list = list;
        this.context = context;
    }
    @Override
    public Fragment getItem(int position) {
        return list.get(position);
    }
    @Override
    public int getCount() {
        return list.size();
    }
    @Override
    public CharSequence getPageTitle(int position) {
        return  titles[position];
    }
```

然后就到了激动人心看结果的时刻了，启动Activity，我们将看到什么呢，内容如下：

```java
07-17 14:43:39.338 1900-1900/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment1.onAttach
07-17 14:43:39.338 1900-1900/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment1.onCreate
07-17 14:43:39.338 1900-1900/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment1.onCreateView
07-17 14:43:39.338 1900-1900/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment1.onViewCreated
07-17 14:43:39.338 1900-1900/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment1.onStart
07-17 14:43:39.338 1900-1900/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment1.onResume
07-17 14:43:39.338 1900-1900/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment2.onAttach
07-17 14:43:39.338 1900-1900/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment2.onCreate
07-17 14:43:39.338 1900-1900/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment2.onCreateView
07-17 14:43:39.338 1900-1900/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment2.onViewCreated
07-17 14:43:39.338 1900-1900/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment2.onStart
07-17 14:43:39.338 1900-1900/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment2.onResume
```

通过输出可以看到Fragment1和Fragment2都被实例化了，生命周期都走到了onResume，这个时候Fragment1显示在屏幕上，我们接着滑动到第二个Fragment，即Fragment2，这个时候又输出什么呢？如下：

```java
07-17 14:45:52.194 1900-1900/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment3.onAttach
07-17 14:45:52.194 1900-1900/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment3.onCreate
07-17 14:45:52.194 1900-1900/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment3.onCreateView
07-17 14:45:52.194 1900-1900/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment3.onViewCreated
07-17 14:45:52.194 1900-1900/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment3.onStart
07-17 14:45:52.198 1900-1900/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment3.onResume
```

通过输出可以看到当Fragment2显示出来的时候，Fragment3也同样被提前初始化了，走到onResume的状态，到目前为止，除了在ViewPager中会提前初始化Fragment好像还没有什么太大的不同，让我们接着走下一步吧，滑动到第三个Fragment，即Fragment3，看看这个时候又会怎么样？如下：

```java
07-17 14:49:50.314 1900-1900/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment1.onPause
07-17 14:49:50.314 1900-1900/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment1.onStop
07-17 14:49:50.314 1900-1900/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment1.onDestroyView
07-17 14:49:50.314 1900-1900/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment4.onAttach
07-17 14:49:50.314 1900-1900/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment4.onCreate
07-17 14:49:50.314 1900-1900/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment4.onCreateView
07-17 14:49:50.314 1900-1900/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment4.onViewCreated
07-17 14:49:50.314 1900-1900/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment4.onStart
07-17 14:49:50.314 1900-1900/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment4.onResume
```

这一次好像有一点不同了，可以看到Fragment1的onPause、onStop、onDestroyView分别被调用了，然后Fragment4和往常一样走到了onResume，这个时候是Fragment3显示出来了，让我们切回Fragment2吧，看看有什么不同？结果如下：

```java
07-17 14:53:55.914 1900-1900/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment1.onCreateView
07-17 14:53:55.914 1900-1900/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment1.onViewCreated
07-17 14:53:55.914 1900-1900/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment4.onPause
07-17 14:53:55.914 1900-1900/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment4.onStop
07-17 14:53:55.914 1900-1900/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment4.onDestroyView
07-17 14:53:55.914 1900-1900/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment1.onStart
07-17 14:53:55.914 1900-1900/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment1.onResume
```

当切回Fragment2时，即Fragment2显示在屏幕上，可以看到Fragment1的生命周期还是有很大的不同的，它分别调用了onCreateView和onViewCreated，接着调用了Fragment4的onPause、onStop、onDestroyView，然后又重新调用了Fragment1的生命周期函数onStart和onResume了，可以看到这里确实有很大的不同。


###总结

- 当Fragment被实例化过之后，如果已经不处于当前Fragment的刚好左/右两侧，那么它的onPause、onStop、onDestroyView会分别被调用
- 当调用了onDestroyView的Fragment又重新被实例化的时候（即进入当前页的刚好左/右两侧），onCreateView和onViewCreated会被调用，然后刚好退出左/右两侧的Fragment走onPause、onStop、onDestroyView，之后那个刚好进入当前页的刚好左/右两侧的Fragment再走onStart和onResume

###更新

上面的demo是基于左右缓存各一页的情况，不过Viewpager还提供了一个`setOffscreenPageLimit(int limit)`这样的方法给我们，用来设置左右两个的缓存页面要多少页（默认limit是1），这样我们更好奇的就是这期间Fragment的生命周期还会不会用什么不同吗？

首先我们先来看看`pager.setOffscreenPageLimit(2);`是怎么样的吧，运行demo，看到如下输出：

```java
07-18 22:48:29.487 3707-3707/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment1.onAttach
07-18 22:48:29.487 3707-3707/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment1.onCreate
07-18 22:48:29.487 3707-3707/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment1.onCreateView
07-18 22:48:29.487 3707-3707/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment1.onViewCreated
07-18 22:48:29.487 3707-3707/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment1.onStart
07-18 22:48:29.487 3707-3707/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment1.onResume
07-18 22:48:29.487 3707-3707/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment2.onAttach
07-18 22:48:29.487 3707-3707/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment2.onCreate
07-18 22:48:29.487 3707-3707/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment2.onCreateView
07-18 22:48:29.487 3707-3707/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment2.onViewCreated
07-18 22:48:29.487 3707-3707/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment3.onAttach
07-18 22:48:29.487 3707-3707/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment3.onCreate
07-18 22:48:29.487 3707-3707/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment3.onCreateView
07-18 22:48:29.487 3707-3707/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment3.onViewCreated
07-18 22:48:29.487 3707-3707/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment2.onStart
07-18 22:48:29.487 3707-3707/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment2.onResume
07-18 22:48:29.487 3707-3707/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment3.onStart
07-18 22:48:29.487 3707-3707/cn.zhouchaoyuan.example E/zhouchaoyuan happen in: TestUIFragment3.onResume
```

输出还是挺多的，可以看到Fragment1的生命周期相比之前是limit是1的时候没有太大的变化，不过Fragment2和Fragment3的生命周期交替调用了，Fragment2调用了onAttach-->onCreate-->onCreateView-->onViewCreated，之后Fragment3进行了同样的过程，然后Fragment2再次调用onStart-->onResume，然后Fragment3又进行了一样的过程。其实这个结果还是挺郁闷了，为什么缓存的Fragment的生命周期不一次性走到onResume呢？也许是onViewCreated调用了之后就把视图创建了，这样是先把所有Fragment的视图都创建了，然后再去走onStart和onResume？（是不是要几个todo？感觉一时半会出不来结果！）


好了，接下来试一试`pager.setOffscreenPageLimit(0);`吧，运行，卧槽，居然和最开始的时候一样，我还以为这样不会缓存呢，然后查看一下源码，看到如下部分：

```java
    private static final int DEFAULT_OFFSCREEN_PAGES = 1;
    public void setOffscreenPageLimit(int limit) {
        if (limit < DEFAULT_OFFSCREEN_PAGES) {
            Log.w(TAG, "Requested offscreen page limit " + limit + " too small; defaulting to " +
                    DEFAULT_OFFSCREEN_PAGES);
            limit = DEFAULT_OFFSCREEN_PAGES;
        }
        if (limit != mOffscreenPageLimit) {
            mOffscreenPageLimit = limit;
            populate();
        }
    }
```
发现0和1的时候是一样的，应该默认缓存是1，小于1的都忽略，这就是为什么和最开始的结果一样了。

###番外篇－－－－Fragment数据保存

onSaveInstanceState在很多情况下都不在调用，为啥？见[这里](http://mp.weixin.qq.com/s?__biz=MzA4NTQwNDcyMA==&mid=2650661714&idx=2&sn=cdebaec04ddf630b024b659402497d95&scene=0#wechat_redirect)，不过onDestroyView在几乎所有的情况下都会调用，那么我们保存是否就考虑从onDestroyView入手，如下是保存数据的一些操作：

```java
	Bundle savedState;//每一个Fragment对应一个Bundle
    private Bundle saveState(){//具体保存数据的操作
        Bundle state = new Bundle();
        state.putString("author","zhouchaoyuan");
        return state;
    }
    private void saveStateToArguments(){
        savedState = saveState();
        if(savedState != null){
            Bundle bundle = getArguments();
            if(bundle == null) return ;
            bundle.putBundle("savedViewState",savedState);
        }
    }
    @Override
    public void onSaveInstanceState(Bundle outState) {//在onSaveInstanceState保存数据
        super.onSaveInstanceState(outState);
        saveStateToArguments();
    }
    @Override
    public void onDestroyView() {//在onSaveInstanceState保存数据
        super.onDestroyView();
        saveStateToArguments();
    }
```

从上面可以看到们选择把数据保存在Arguments中，把一些状态放进Bundle。

接下来是恢复数据的部分，如下：

```java
    private void restoreState(){//恢复数据的真正操作部分
        if(savedState != null){
            String author = savedState.getString("author");
        }
    }
    private boolean restoreStateFromArguments(){//返回false代表第一次进入
        Bundle bundle = getArguments();
        if(bundle == null)return false;
        savedState = bundle.getBundle("");
        if (savedState != null){
            restoreState();
            return true;
        }
        return false;
    }
    public void onActivityCreated(@Nullable Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        if(!restoreStateFromArguments()){
            //第一次进入做一些初始化操作
        }
    }
```

通过上面两个就能比较完美的保存Fragment里面的数据了。