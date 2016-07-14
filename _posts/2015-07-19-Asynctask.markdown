---
layout: post
title:  Asynctask使用小结 
date:   2015-07-19 14:12:22
category: "学习"
---

当我们在写安卓的时候很多时候都需要异步加载网络上的内容，这个时候我们可以选择新建一个线程，然后将计算结果通过Handler传送给主线程，并修改UI，得到想要的结果。然而我们可以更方便的使用Asynctask达到我们的要求。相比来说Asynctask比Handler更加轻量级。

Asynctask的定义如下：
```java
public abstract class AsyncTask<Params, Progress, Result> //抽象类  
```

三个泛型一次代表：“启动任务执行的输入参数”、“后台任务执行的进度”、“后台计算结果的类型”

使用Asynctask需要实现以下一些方法：

必须要的：

```java
//在onPreExecute()完成后立即执行，用于执行较为费时的操作，一般我们在这里写代码，做操作  
//此方法将接收输入参数和返回计算结果。在执行过程中可以调用publishProgress(Progress... values)来更新进度信息。  

doInBackground(Params... params)  

//当后台操作结束时，它将会被调用，计算结果将做为参数传递到此方法中，直接将结果显示到UI组件上，达到我们要的结果  
onPostExecute(Result result)  

//执行一个异步任务，需要我们在主线程中调用此方法，触发异步任务的执行  
execute(Params... params)  
```

可选择实现的：

```java
//在调用publishProgress(Progress... values)时，此方法被执行，直接将进度信息更新到UI组件上。  

onProgressUpdate(Progress... values)  

//被调用后立即执行，一般用来在执行后台任务前对UI做一些标记。  

onPreExecute()  

//用户调用取消时，要做的操作  

onCancelled()  
```

在使用的时候，有几点需要格外注意：

1.Asynctask的实例必须在UI线程中创建。

2.execute(Params... params)方法必须在UI线程中调用。

3.不要手动调用onPreExecute()，doInBackground(Params... params)，onProgressUpdate(Progress... values)，onPostExecute(Result result)这几个方法。

4.不能在doInBackground(Params... params)中更改UI组件的信息，必须通过onPostExecute。

5.一个任务实例只能执行一次，如果执行第二次将会抛出异常。

###AsyncTask的原理
先来看他的构造函数：

```java
	public AsyncTask() {
        mWorker = new WorkerRunnable<Params, Result>() {
            public Result call() throws Exception {
                mTaskInvoked.set(true);//原子布尔变量置true
                
                Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);//设置优先级为后线程
                //noinspection unchecked
                Result result = doInBackground(mParams);//调用doInBackground
                Binder.flushPendingCommands();//神奇的刷新
                return postResult(result);
            }
        };

        mFuture = new FutureTask<Result>(mWorker) {
            @Override
            protected void done() {
                try {
                    postResultIfNotInvoked(get());
                } catch (InterruptedException e) {
                    android.util.Log.w(LOG_TAG, e);
                } catch (ExecutionException e) {
                    throw new RuntimeException("An error occurred while executing doInBackground()",
                            e.getCause());
                } catch (CancellationException e) {
                    postResultIfNotInvoked(null);
                }
            }
        };
    }
```

构造方法里面先初始化一个WorkerRunnable,他是一个Callable，主要是传入的数据和返回的数据进行封装，然后初始化一个FutureTask。WorkerRunnable源码如下：

```java
	private static abstract class WorkerRunnable<Params, Result> implements Callable<Result> {
        Params[] mParams;
    }
```

在call方法里面最后调用了postResult，他主要是获取一个通过InternalHandler获得重用消息，然后将消息辗转反侧发送到消息队列，等待Handler处理，postResult源码如下：

```java
	private Result postResult(Result result) {
        @SuppressWarnings("unchecked")
        Message message = getHandler().obtainMessage(MESSAGE_POST_RESULT,
                new AsyncTaskResult<Result>(this, result));//放入AsyncTaskResult到msg.obj中
        message.sendToTarget();//发送到MessageQueue，等待被取出然后通过handleMessage处理
        return result;
    }
```

上面用到了getHandler，继续跟踪源码看到如下代码，线程安全的返回了一个InternalHandler：

```java
	private static Handler getHandler() {
        synchronized (AsyncTask.class) {
            if (sHandler == null) {//单例
                sHandler = new InternalHandler();
            }
            return sHandler;
        }
    }
```

然后InternalHandler是什么呢？当时一个Handler啦，他是继承了Handler的一个静态内部类(为什么是静态的？防止持有外部类引用导致内存泄漏？)，如下：

```java
	private static class InternalHandler extends Handler {
        public InternalHandler() {
        	//在ActivityThread.main()已经调用了prepareMainLooper()初始化了一个Looper了，直接取就行
            super(Looper.getMainLooper());
        }
        @SuppressWarnings({"unchecked", "RawUseOfParameterizedType"})
        @Override
        public void handleMessage(Message msg) {
            AsyncTaskResult<?> result = (AsyncTaskResult<?>) msg.obj;//obtainMessage的时候赋值的
            switch (msg.what) {
                case MESSAGE_POST_RESULT:
                    // There is only one result
                    result.mTask.finish(result.mData[0]);//判断已经结束，调用finish
                    break;
                case MESSAGE_POST_PROGRESS:
                    result.mTask.onProgressUpdate(result.mData);//更新进度的消息，更新进度
                    break;
            }
        }
    }
```

上面判断结束了调用了finish，他的源码如下：

```java
	private void finish(Result result) {
        if (isCancelled()) {
            onCancelled(result);//取消
        } else {
            onPostExecute(result);//回调onPostExecute进行主线程工作
        }
        mStatus = Status.FINISHED;//更新状态
    }
```

然后你会发现AsyncTaskResult在上面已经出现了几次了，其实这也是一个静态的内部类，封装了外部类的引用和一些数据，源码如下：

```java
	private static class AsyncTaskResult<Data> {
        final AsyncTask mTask;
        final Data[] mData;
        AsyncTaskResult(AsyncTask task, Data... data) {
            mTask = task;
            mData = data;
        }
    }
```


接着呢回到构造函数里面初始化的第二个变量mFuture，他是一个FutureTask，传入之前初始化的mWorker，即一个Callable，然后复写了done方法，默认的done方法是空的。Callable会在FutureTask里面执行，执行之后调用`FutureTask`里面的done方法，在done方法里面调用get方法可以得到Callable的call方法返回的结果。另外，在call没有执行结束时get是被阻塞的。

这里还用到了postResultIfNotInvoked(get());这个方法，他的源码如下：

```java
	private void postResultIfNotInvoked(Result result) {
        final boolean wasTaskInvoked = mTaskInvoked.get();//获得原子变量的真实值
        if (!wasTaskInvoked) {
            postResult(result);
        }
    }
```
代码很简单，根据原子性变量mTaskInvoked判断是否发送结构到消息队列被处理。至此，构造方法所做的工作差不多分析完了，接下来是execute方法。

```java
	public final AsyncTask<Params, Progress, Result> execute(Params... params) {
        return executeOnExecutor(sDefaultExecutor, params);
    }
```

直接调用到了executeOnExecutor，executeOnExecutor的源码如下：

```java
	public final AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec,
            Params... params) {
        if (mStatus != Status.PENDING) {
            switch (mStatus) {
                case RUNNING:
                    throw new IllegalStateException("Cannot execute task:"
                            + " the task is already running.");
                case FINISHED:
                    throw new IllegalStateException("Cannot execute task:"
                            + " the task has already been executed "
                            + "(a task can be executed only once)");
            }
        }
        mStatus = Status.RUNNING;//更新状态为运行
        onPreExecute();//一些前期初始化工作
        mWorker.mParams = params;//将数据引用放到在构造方法第一个初始化的WorkerRunnable中
        exec.execute(mFuture);//执行mFuture，这个参数是构造方法里面第二个初始化的变量
        return this;
    }
```

关键代码就是`exec.execute(mFuture);`了，这用到了变量exec，其实就是成员变量sDefaultExecutor，他也属于一个Executor，被SerialExecutor初始化：

```java
	private static class SerialExecutor implements Executor {
        final ArrayDeque<Runnable> mTasks = new ArrayDeque<Runnable>();//双端队列
        Runnable mActive;
        public synchronized void execute(final Runnable r) {//重写execute
        	//在双端队列加入一个Runnable，其run方法调用了调用了execute的Runnable的run方法
            mTasks.offer(new Runnable() {
                public void run() {
                    try {
                        r.run();
                    } finally {
                        scheduleNext();//最后都会执行
                    }
                }
            });
            if (mActive == null) {
                scheduleNext();
            }
        }
        protected synchronized void scheduleNext() {
            if ((mActive = mTasks.poll()) != null) {
            	//取出队首的Runnable，交给线程池THREAD_POOL_EXECUTOR之行                  
                THREAD_POOL_EXECUTOR.execute(mActive);
            }
        }
    }
```

到这里就感觉比较清晰了，最后是通过`THREAD_POOL_EXECUTOR`执行的，他的参数是mActive，从双端队列取出来的一个Runnable，这个Runnable会调用FutureTask型的mFuture，二在FutureTask中又会执行WorkerRunnable型的mWorker，最终调用到doInBackground去处理我们想要做的事情。我们也可以看到其实AsyncTask就是Handler＋Thread的良好封装。


###注意事项

- 有大量线程执行任务的时候，避免使用AsyncTask，因为他的内部线程池的容量是有限的
- 没有与主线程的交互尽量不要使用AsyncTask，直接使用Thread
- AsyncTask不会会随着Activity的销毁而销毁，他会一直执行doInBackground()方法直到方法执行结束，然后根据不同情况进行不同的回调，直到结束
- 内存泄漏：在Activity中使用非静态匿名内部AsyncTask类，AsyncTask内部类会持有外部类的隐式引用。由于AsyncTask的生命周期可能比Activity的长，当Activity进行销毁AsyncTask还在执行时，由于AsyncTask持有Activity的引用，导致Activity对象无法回收，进而产生内存泄露
