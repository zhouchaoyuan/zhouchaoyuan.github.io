---
layout: post
title:  Rxjava学习笔记
date:   2016-04-04 21:23:54
category: "学习"
---

听组里老大说项目中用了很多的Rxjava，然后就去看了看，果真的写了一坨坨看不懂的代码，所以趁这会有时间把Rxjava粗糙的学习一下（至少要看得懂代码吧）。看到别人的介绍，有人说“RxJava 真是太好用了”，而有人说“RxJava 真是太难用了”，果然自己真的成了懵逼了（什么鬼），但是我还是硬着头皮往下看了。好吧，其实我就是把Rxjava的用法整理一下。

###Before first

先导入使用的库：

	compile 'io.reactivex:rxjava:1.0.14' 
	compile 'io.reactivex:rxandroid:1.0.1'
	
他们在github的链接如下：

- [Rxjava](https://github.com/ReactiveX/RxJava)
- [RxAndroid](https://github.com/ReactiveX/RxAndroid)

###First:简述

Rxjava中两个核心的东西，Observables（被观察者，事件源）和Subscribers（观察者），而Observables发出一系列事件，Subscribers处理这些事件，且每一个Subscriber会根据Observable发出的事件回调`onNext()`,`onComplete()`,`onError()`这三个方法，他们的含义大致如下：

- `onNext()`: 每一发出一个事件，会回调这个方法。
- `onCompleted()`: 事件队列完结。Rxjava把每个事件单独处理，更把它们看做一个队列。并规定当不会再有新的`onNext()`发出时，需要触发`onCompleted()`方法作为标志。
- `onError()`: 事件队列异常。在事件处理过程中出异常时，`onError()`会被触发，其他所有事件都被终止，并且所以得异常都有走到这个方法来，我们在这里处理异常非常方便。

>**Attention**:`onCompleted()`和`onError()`二者也是互斥的，即在队列中调用了其中一个，就不应该再调用另一个。

**一、**通常情况下我们这样使用一个subscriber订阅一个Observable，如下：

	observable.subscribe(subscriber);//反过来了是吗？没错，就是这样子的

这个函数返回了一个Subscription，我们可以通过它取消两者的订阅关系，接下来，我们是这么创建一个Observable的：

```java
Observable observable = Observable.create(new Observable.OnSubscribe<String>() {
	@Override
    public void call(Subscriber<? super String> subscriber) {
        subscriber.onNext("I");
        subscriber.onNext("am");
        subscriber.onNext("superman");
        subscriber.onCompleted();
    }
});
```

又是这么创建一个subscriber(也可以是一个Observer，subscriber是实现了Observer的一个抽象类，使用Observer的时候它也会被转换成subscriber)的：

```java
Subscriber<String> subscriber = new Subscriber<String>() {
	@Override
	public void onCompleted() {}
	@Override
	public void onError(Throwable e) {}
	@Override
	public void onNext(String s) {
		Log.e("Hi",s);
	}
};
```

这几行代码的简单效果就是可以看到I am superman分行输出来了。

**二、**创建Observable也可以这么干：

a、使用public static final <T> Observable<T> from(@NotNull T[] array)

	Observable observable = Observable.from(new String[]{"I","am","superman"});
	
b、public static final <T> Observable<T> just(T...)

	Observable observable = Observable.just("I","am","superman");

我们可以看到Observable的创建被简化了很多，那么subscriber能否简化呢？答案是肯定的，我们使用ActionX这个类，其中X表示一个具体的阿拉伯数字，代表call这个方法有X个参数，比如我们可以这样用：

```java
Observable.from(new String[]{"I","am","superman"}).subscribe(new Action1<String>() {
	@Override
	public void call(String s) {
		Log.d("Hello",s);
	}
});
```
下面是observable.subscribe的几个常见的重载方法，分别对应onNext，onError，onCompleted：

	observable.subscribe(onNext);
	observable.subscribe(onNext, onError);
	observable.subscribe(onNext, onError, onCompleted);

以下是一个例子：

```java
Action1<String> onNextAction = new Action1<String>() {
    // onNext()
    @Override
    public void call(String s) {}
};
Action1<Throwable> onErrorAction = new Action1<Throwable>() {
    // onError()
    @Override
    public void call(Throwable throwable) {}
};
Action0 onCompletedAction = new Action0() {
    // onCompleted()
    @Override
    public void call() {}
};
observable.subscribe(onNextAction);//one
observable.subscribe(onNextAction, onErrorAction);//two
observable.subscribe(onNextAction, onErrorAction, onCompletedAction);//three
```

**三、**我们还可以使用[retrolambda](https://github.com/evant/gradle-retrolambda)进一步简化:

	Observable.just("zcy").subscribe(s-> Log.e("Hi",s));
	Observable.just("zcy").subscribe(s-> Log.e("Hi",s),throwable -> Log.e("",""));
	Observable.just("zcy").subscribe(s-> Log.e("Hi",s),throwable -> Log.e("",""),()-> Log.e("",""));
	
例子：

在资源中获取一张bitmap并显示在ImageView中，在出现异常的时候打印Toast报错

	ImageView imageView = new ImageView(this);
	Observable.just(BitmapFactory.decodeResource(getResources(),R.mipmap.ic_launcher))
		.subscribe(bitmap -> imageView.setImageBitmap(bitmap),
		throwable -> Log.e("Error","onError"));

###Second:操作符（Operators）

操作符用来解决Observable对象的变换问题，用于在Observable和最终的Subscriber之间修改Observable发出的事件

一、map操作符

我们使用map操作符过滤TextView成为一个字符串：

```java
Observable.just(textView).map(new Func1<TextView, String>() {
	@Override
    public String call(TextView textView) {
    	return (String)textView.getText();
	}
}).subscribe(s -> System.out.println(s));
```

上面的Func1<T,R>代表call的参数是T类型返回值是R类型的，然后通过Lambda简化如下：

	Observable.just(textView).map(textView1 -> (String) textView1.getText()).subscribe(s -> System.out.println(s));

二、flatMap操作符

假如我们有一个函数 Observable<List<String>> getJSON()，（这种方法在结合Retrofit使用的时候比较常见） 我们要取出每一个String，我们有很多种写法，如下：

```java
//第一种
getJson().subscribe(strings -> {
	for(String string :strings){
		System.out.println(string);
	}
});
//第二种
getJson().subscribe(strings -> {
	Observable.from(strings).subscribe(s -> System.out.println(s));
});
//第三种
getJson().flatMap(strings -> Observable.from(strings)).subscribe(s -> System.out.println(s));
```
可以看出第一种还是比较笨拙的(虽然看起来好直接)，还是第二种和第三种比较优雅，不过我们还是着重看第三种使用的flatMap，通过flatMap这个操作符把一个list转换成一个Observable，然后又可以采取像之前我们想要的操作（比如map什么的），这里无论想转换成什么，都可以达到一条链的结构，的确挺清晰的。这里map和flatMap是不同的，一个是一对一的关系，一个是一对多的关系。

三、filter和take操作符，我们来看看这个例子“在上面取出来的每一条字符串，我们再选取非空的前五条，然后存取到另外的一个数组并打印出来”，如下：

```java
getJson().flatMap(strings -> Observable.from(strings))
                .filter(s1 -> s1 != null).take(5).doOnNext(s2 -> array.add(s2))
                .subscribe(s -> System.out.println(s));
```

四、subscribeOn和observeOn操作符
我们想通过本地加载一段相当长的文字（耗时操作），然后加载到一个TextView上面，可以编写代码如下：

```java
retrofitService.getJson().subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(strings1 -> textView.setText(strings1));

```

如此Subscriber将运行在一个I/O线程上，结束以后，主线程上将发生View操作，更新TextView。

五、zip操作符

zip操作符是把两个observable提交的结果，严格按照顺序进行合并:

```java
Observable.zip(Observable.just(1, 2, 3), Observable.just(4, 5, 6, 7), new Func2<Integer, Integer, Integer>() {
		@Override
		public Integer call(Integer integer, Integer integer2) {
			return integer+integer2;
		}
	}).subscribe(new Subscriber<Integer>() {
		@Override
		public void onCompleted() {
			System.out.println("OK");
		}
		@Override
		public void onError(Throwable e) {
			System.out.println("error");
		}
		@Override
		public void onNext(Integer integer) {
			System.out.println(integer);
		}
});
```

然后使用lambda简化（还是挺优美的）：
        
```java
Observable.zip(Observable.just(1, 2, 3), Observable.just(4, 5, 6, 7), (a, b) -> a + b)
	.subscribe(res -> System.out.println(res), throwable -> System.out.println("error")
		, () -> System.out.println("OK"));
```

[更多的操作符](http://reactivex.io/documentation/operators.html#alphabetical)

###Fourth：线程控制

在Rxjava中，Scheduler（调度器）相当于线程调度器，它可以用来指定一段代码应该运行在什么样的线程，比如在Rxjava中有如下几种Scheduler：

- Schedulers.immediate(): 直接在当前线程运行，相当于不指定线程。这是默认的 Scheduler。
- Schedulers.newThread(): 总是启用新线程，并在新线程执行操作。
- Schedulers.io(): I/O 操作（读写文件、读写数据库、网络信息交互等）所使用的 Scheduler。行为模式和 newThread() 差不多，区别在于 io() 的内部实现是是用一个无数量上限的线程池，可以重用空闲的线程，因此多数情况下 io() 比 newThread() 更有效率。不要把计算工作放在 io() 中，可以避免创建不必要的线程。
- Schedulers.computation(): 计算所使用的 Scheduler。这个计算指的是 CPU 密集型计算，即不会被 I/O 等操作限制性能的操作，例如图形的计算。这个 Scheduler 使用的固定的线程池，大小为 CPU 核数。不要把 I/O 操作放在 computation() 中，否则 I/O 操作的等待时间会浪费 CPU。
- AndroidSchedulers.mainThread()： Android 专用，它指定的操作将在 Android 主线程运行。


我们可以看到上面subscribeOn的参数Schedulers.io()就是在io线程读取，而observeOn()的参数是AndroidSchedulers.mainThread()指明在主线程修改。

###Fifth:Retrofit和Rxjava的简单应用

简单的例子，使用Retrofit和Rxjava来获取https://api.douban.com/v2/movie/top250?start=0&count=10上面的json数据。

一、一般情况下在gradle.build需要以下的这几个东东：

	compile 'io.reactivex:rxandroid:1.1.0'
    compile 'io.reactivex:rxjava:1.1.0'
    compile 'com.squareup.retrofit2:adapter-rxjava:2.0.0-beta4'
    compile 'com.squareup.retrofit2:converter-gson:2.0.0-beta4'
    compile 'com.squareup.retrofit:retrofit:1.9.0'
    
二、对于Retrofit，我们需要创建一个接口并取名为MovieService

```java
public interface MovieService {
    @GET("top250")
    Observable<MovieEntity> getTopMovie(@Query("start") int start, @Query("count") int count);
}
```
其中MovieEntity是json数据对应的Object对象。

三、消费上面的接口，我们新建一个方法getTopMovie：

```java
public static void getTopMovie(Subscriber subscriber, int start, int count) {
	String baseUrl = "https://api.douban.com/v2/movie/";
	Retrofit retrofit = new Retrofit.Builder()
                .baseUrl(baseUrl)
                .addConverterFactory(GsonConverterFactory.create())
                .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
                .build();
	MovieService movieService = retrofit.create(MovieService.class);
	movieService.getTopMovie(start, count)
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(subscriber);
    }
```

四、Activity的布局有一个TextView，然后我们在Activity创建一个Subscriber，并调用getTopMovie就可以获取到数据了，如下：

```java
Subscriber subscriber = new Subscriber<MovieEntity>() {
	@Override
	public void onCompleted() {
		Log.e("acjiji", "OK");
	}
	@Override
	public void onError(Throwable e) {
		Log.e("acjiji", e.getMessage());
	}
	@Override
	public void onNext(MovieEntity movieEntity) {
                ((TextView) findViewById(R.id.textView)).setText(movieEntity.getTitle());
	}
};
getTopMovie(subscriber, 0, 10);
```
我们把获取的数据设置到TextView了，这样就是一个简单的的应用啦？

详细可参见：[RxJava 与 Retrofit 结合的最佳实践](http://gank.io/post/56e80c2c677659311bed9841)

###参考链接

- [Rxjava操作符](http://mushuichuan.com/2015/12/11/rxjava-operator-1/)
- [深入浅出RxJava](http://www.pythonnote.com/archives/shen-ru-qian-chu-rxjava-yi-ji-chu-pian.html)
- [给 Android 开发者的 RxJava 详解](http://gank.io/post/560e15be2dca930e00da1083)
- [ReactiveX文档中文翻译](https://mcxiaoke.gitbooks.io/rxdocs/content/Intro.html)