---
layout: post
title:  Android学习
date:   2015-07-19 14:12:22
category: "学习"
---

当我们在写安卓的时候很多时候都需要异步加载网络上的内容，这个时候我们可以选择新建一个线程，然后将计算结果通过Handler传送给主线程，并修改UI，得到想要的结果。然而我们可以更方便的使用Asynctask达到我们的要求。相比来说Asynctask比Handler更加轻量级。

Asynctask的定义如下：

[java] view plaincopyprint?

    public abstract class AsyncTask<Params, Progress, Result> //抽象类  


三个泛型一次代表：“启动任务执行的输入参数”、“后台任务执行的进度”、“后台计算结果的类型”

使用Asynctask需要实现以下一些方法：

必须要的：

[java] view plaincopyprint?

    //在onPreExecute()完成后立即执行，用于执行较为费时的操作，一般我们在这里写代码，做操作  
    //此方法将接收输入参数和返回计算结果。在执行过程中可以调用publishProgress(Progress... values)来更新进度信息。  
      
    doInBackground(Params... params)  
      
    //当后台操作结束时，它将会被调用，计算结果将做为参数传递到此方法中，直接将结果显示到UI组件上，达到我们要的结果  
    onPostExecute(Result result)  
      
       
      
    //执行一个异步任务，需要我们在主线程中调用此方法，触发异步任务的执行  
    execute(Params... params)  



可选择实现的：

[java] view plaincopyprint?

    //在调用publishProgress(Progress... values)时，此方法被执行，直接将进度信息更新到UI组件上。  
      
    onProgressUpdate(Progress... values)  
      
    //被调用后立即执行，一般用来在执行后台任务前对UI做一些标记。  
      
    onPreExecute()  
      
    //用户调用取消时，要做的操作  
      
    onCancelled()  




在使用的时候，有几点需要格外注意：

1.Asynctask的实例必须在UI线程中创建。

2.execute(Params... params)方法必须在UI线程中调用。

3.不要手动调用onPreExecute()，doInBackground(Params... params)，onProgressUpdate(Progress... values)，onPostExecute(Result result)这几个方法。

4.不能在doInBackground(Params... params)中更改UI组件的信息，必须通过onPostExecute。

5.一个任务实例只能执行一次，如果执行第二次将会抛出异常。
