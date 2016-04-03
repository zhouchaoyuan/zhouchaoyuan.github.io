听导师说项目中用了很多的Rxjava，然后就去看了看，果真的写了一坨坨看不懂的代码，所以趁这会有时间把Rxjava粗糙的学习一下（至少要看得懂代码吧）。看到别人的介绍，有人说“RxJava 真是太好用了”，而有人说“RxJava 真是太难用了”，果然自己真的成了懵逼了（什么鬼），但是我还是硬着头皮往下看了。好吧，其实我就是把Rxjava的用法整理一下。

###Before first

先导入使用的库：

	compile 'io.reactivex:rxjava:1.0.14' 
	compile 'io.reactivex:rxandroid:1.0.1'
	
他们在github的链接如下：

- [Rxjava](https://github.com/ReactiveX/RxJava)
- [RxAndroid](https://github.com/ReactiveX/RxAndroid)

###First

Rxjava中两个核心的东西，Observables（被观察者，事件源）和Subscribers（观察者），而Observables发出一系列事件，Subscribers处理这些事件，且每一个Subscriber会根据Observable发出的事件回调`onNext()`,`onComplete()`,`onError()`这三个方法，他们的含义大致如下：

- `onNext()`: 每一发出一个事件，会回调这个方法。
- `onCompleted()`: 事件队列完结。Rxjava把每个事件单独处理，更把它们看做一个队列。并规定当不会再有新的`onNext()`发出时，需要触发`onCompleted()`方法作为标志。
- `onError()`: 事件队列异常。在事件处理过程中出异常时，`onError()`会被触发，其他所有事件都被终止，并且所以得异常都有走到这个方法来，我们在这里处理异常非常方便。

>**Attention**:`onCompleted()`和`onError()`二者也是互斥的，即在队列中调用了其中一个，就不应该再调用另一个。

**一、**通常情况下我们这样使用一个subscriber订阅一个Observable（反过来了是吗？没错，就是这样子的），如下：

	observable.subscribe(subscriber);

接下来，我们是这么创建一个Observable的：

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
observable.subscribe(onNextAction);
observable.subscribe(onNextAction, onErrorAction);
observable.subscribe(onNextAction, onErrorAction, onCompletedAction);
```

**三、**我们还可以使用[retrolambda](https://github.com/evant/gradle-retrolambda)进一步简化

;

http://mushuichuan.com/2015/12/11/rxjava-operator-1/

http://www.pythonnote.com/archives/shen-ru-qian-chu-rxjava-yi-ji-chu-pian.html

http://gank.io/post/560e15be2dca930e00da1083#toc_1