---
layout: post
title:  RoboGuice学习笔记
date:   2016-03-25 16:45:55
category: "学习"
---

RoboGuice是基于Guice库开发，目的为Android提供一套简单易用的依赖注入框架，而Google Guice提供了Java平台上一个轻量级的 Dependency injection 框架，并可以支持开发Android应用。它降低了代码中new和Factory方法的使用，采用@Inject来替代，虽然有时候还需要写一些Factory方法，不过我们的代码不会依赖这些代码来创建实例，这使得单元测试和代码重用变得更方便。（这个以后慢慢体会吧）

一、接入

在build.gradle添加依赖

```java
dependencies {
   	provided 'org.roboguice:roboblender:3.0.1'
   	compile 'org.roboguice:roboguice:3.0.1'
}
```
	
二、基本的注解

	1、使用@ContentView来注入绑定布局
	2、使用@InjectView来注入绑定控件
	3、使用@InjectResource来注入绑定资源res，如：String资源、color资源、Animation资源
	4、使用@Inject来注入对象和服务
	
然后综合上面几个注解来一个例子：

首先是布局

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
   	xmlns:tools="http://schemas.android.com/tools"
   	android:layout_width="match_parent"
   	android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context="cn.zhouchaoyuan.myapplication.MainActivity">

    <TextView
       	android:id="@+id/text_view"
       	android:layout_width="wrap_content"
       	android:layout_height="wrap_content"
       	android:text="Hello World!" />
</LinearLayout>
```

接下来是Activity的代码，要继承自RoboActivity或者RoboFragmentActivity

```java
@ContentView(R.layout.activity_main)
public class MainActivity extends RoboActivity {

   	@InjectView(R.id.text_view)
   	TextView textView;

   	@InjectResource(R.string.author_name)
   	String author_name;

   	@Inject
   	NotificationManager notificationManager;
 
   	@Override
   	protected void onCreate(Bundle savedInstanceState) {
       	super.onCreate(savedInstanceState);
       	textView.setText(author_name);
       	notificationManager.cancelAll();
   	}
}	
```

然后是string资源文件：

```xml
<resources>
   	<string name="app_name">MyApplication</string>
   	<string name="author_name">zhouchaoyuan</string>
</resources>
```

最后运行可以看到与我们平常的效果是一样的，对于Fragment也可以是这样的，只不过需要注意视图注入需要在onViewCreated后使用。

对象的注入稍微复杂一些，大概就是说我们不需要知道对象是怎么被初始化的，我们只要知道对象的某个状态就行，下面是一个对象注入的例子：

```java
@ContentView(R.layout.activity_main)
public class MainActivity extends RoboActivity {

   	@InjectView(R.id.text_view)
   	TextView textView;

   	@Inject
   	User user;

   	@Override
   	protected void onCreate(Bundle savedInstanceState) {
       	super.onCreate(savedInstanceState);
       	if(user.isInvalidate()){
           	textView.setText(user.getName());
       	}
       	else{
           	textView.setText("名字无效啊!");
       	}
   	}
}

@ContextSingleton
class User{
   	String name;
   	private Context context;

   	//使用这个构造器来初始化对象,需要一个Context,这个参数会通过Activity提交给容器管理者,否则使用默认构造器
   	@Inject
   	public User(Context context){
       	this.context = context;
       	init();
   	}

   	public boolean isInvalidate(){
       	return !TextUtils.isEmpty(name);
   	}

   	private void init() {
       	SharedPreferences sharedPreferences = PreferenceManager.getDefaultSharedPreferences(context);
       	name = sharedPreferences.getString("name", null);
   	}

   	public String getName() {
       	return name;
   	}

}
```
	
上面用到了@ContextSingleton注解，表示User随Context的生命周期销毁而销毁，如果这里改为@Singleton，那么User的生命周期将是整个应用的生命周期，如果两个Activity都使用了该注解，那么产生的对象将是同一个。另外，对一个对象使用@FragmentSingleton，那么在Fragment的作用域之内这个对象也只会被初始化一次。

三、注解自定义对象

这里的对象绑定指的是将一个接口或类绑定到一个子类、对象、或对象提供容器上。当我们注入这个接口或类时，默认会根据绑定的类别初始化这个接口的实现。

1、首先我们需要在AndroidManifest.xml进行绑定，像下面这样：

```xml
<application ...>
   	<meta-data android:name="roboguice.modules"
		android:value="cn.zhouchaoyuan.myapplication.MyModule" />                
</application>
```

2、创建一个继承AbstractModule的类，这个类和上面在AndroidManifest注册的时候的名字一样，然后在configure里面进行绑定就可以了，如下：

先创建一个接口：

```java
public interface GsonProvider {
   	Gson get();
}
```

之后创建一个类实现该接口：

```java
public class GsonProviderImp implements GsonProvider{
   	@Override
   	public Gson get() {
       	return new GsonBuilder().create();
   	}
}
```

然后在自定义的Module里面进行绑定：

```java
public class MyModule extends AbstractModule{
   	@Override
   	protected void configure() {
       	bind(GsonProvider.class).to(GsonProviderImp.class);
   	}
}
```
	
最后看Activity里面的代码：

```java
@ContentView(R.layout.activity_main)
public class MainActivity extends RoboActivity{

   	@Inject
   	GsonProvider gsonProvider;
    
   	@Override
   	protected void onCreate(Bundle savedInstanceState) {
       	super.onCreate(savedInstanceState);
       	String str = gsonProvider.get().toJson("123");
   	}
}
```

其实我们通过注解GsonProviderImp也可以实现同样的功能，如下：

```java
@Inject
GsonProviderImp gsonProviderImp;
```
    
假如我们要提供不同的Gson，这时候我们应该怎么做呢，其实只要再实现GsonProvider这个类，然后在configure绑定就行了，如下：

```java
public class GsonProviderImpWithNULL implements  GsonProvider{
   	@Override
   	public Gson get() {
       	return new GsonBuilder().serializeNulls().create();
   	}
}
```

然后自定的Module如下：

```java
public class MyModule extends AbstractModule{
   	@Override
   	protected void configure() {
        bind(GsonProvider.class).annotatedWith(Names.named("normal")).to(GsonProviderImpWithNULL.class);
        bind(GsonProvider.class).annotatedWith(Names.named("serializeNulls")).to(GsonProviderImp.class);
   	}
}
```

然后在MainActivity就可以通过@Named提供不同的GsonProvider初始化了：

```java
@ContentView(R.layout.activity_main)
public class MainActivity extends RoboActivity{

   	@Named("normal")
   	@Inject
   	GsonProvider gsonProvider;

   	@Named("serializeNulls")
   	@Inject
   	GsonProvider gsonProviderWithNull;

   	@Override
   	protected void onCreate(Bundle savedInstanceState) {
       	super.onCreate(savedInstanceState);
       	String str = gsonProvider.get().toJson(new A());
       	Log.e("1207020203",str);
       	str = gsonProviderWithNull.get().toJson(new A());
       	Log.e("1207020203",str);
   	}
}

class A{
   	B b;
   	class B{}
}
```

输出下面的内容：

	03-25 02:38:54.940 909-909/cn.zhouchaoyuan.myapplication E/1207020203: {"b":null}
	03-25 02:38:54.940 909-909/cn.zhouchaoyuan.myapplication E/1207020203: {}
	
可以看到我们根据不同注解可以提供不同功能的GsonProvider.

四、注解自定义View

首先我们给自己自定义一个非常简单的View：

```java
public class CustomView extends LinearLayout {

   	@InjectView(R.id.custom_text)
   	TextView textView;

   	public CustomView(Context context, AttributeSet attrs) {
       	super(context, attrs);
       	init(context);
   	}

    private void init(Context context) {
       	inflate(context, R.layout.custom_layout, this);
       	RoboGuice.getInjector(context).injectMembers(this);
       	RoboGuice.getInjector(context).injectViewMembers(this);
   	}

   	@Override
   	protected void onFinishInflate() {
       	super.onFinishInflate();
       	textView.setText("zhouchaoyuan");
   	}
}
```

然后在activity_main使用这个自定义的view：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
   	xmlns:tools="http://schemas.android.com/tools"
   	android:layout_width="match_parent"
   	android:layout_height="match_parent"
   	android:orientation="vertical"
   	tools:context="cn.zhouchaoyuan.myapplication.MainActivity">

   	<cn.zhouchaoyuan.myapplication.CustomView
       	android:id="@+id/custom_id"
       	android:layout_width="wrap_content"
       	android:layout_height="wrap_content"
       	android:orientation="vertical" />
</LinearLayout>
```

然后在MainActivity里面使用注解注入这个view：

```java
@ContentView(R.layout.activity_main)
public class MainActivity extends RoboActivity{

   	@InjectView(R.id.custom_id)
    CustomView customView;

   	@Override
   	protected void onCreate(Bundle savedInstanceState) {
       	super.onCreate(savedInstanceState);
       	TextView textView = new TextView(this);
       	textView.setText("acjiji");
       	customView.addView(textView);
   	}
}
```
	
可以看到CustomView也被注入了。

五、其他常用方法

RoboGuice还提供了IntentExtra的获取，但是注意，如果标注@InjectExtra的value没有找到对应的数据，则app会crash，如果允许获取不到extra，则必须将optional = true。
除此之外，RoboGuice提供了直接获取图中对象的方法，如下的Gson对象获取。
 
```java 
@ContentView(R.layout.activity_second)
public class SecondActivity extends RoboFragmentActivity {
  
	@InjectExtra(value = "pull", optional = true)
	String pull;
  
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		Gson gson = RoboGuice.getInjector(this).getInstance(Gson.class);
		String demoStr = gson.toJson(pull);   
	}
}
```

[参考链接](
https://github.com/roboguice/roboguice/wiki/Your-First-View-Injection)