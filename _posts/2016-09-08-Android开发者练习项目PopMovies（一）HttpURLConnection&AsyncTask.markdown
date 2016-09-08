---
layout: post
title:  "Android开发者练习项目PopMovies（一）HttpURLConnection&AsyncTask"
date:   2016-09-08 10:09:56 +0800
categories: Android
---

# 前言

最近参加了一个Google官方的远程教育课程，在课程中要完成一些项目。借此，我写成Blog，在自己巩固已经掌握的API的同时，希望分享给大家。

# AsyncTask

这是一个轻量级的异步任务类。与Handler的作用一样。
优点：使用方便，结构简单
缺点：只能处理单个异步任务，在多个异步任务并发的处理上不如Handler

```
// URL 是doInBackground的入参类型
// Integer 是onProgressUpdate的入参类型
// Long 是doInBackground 的出参类型 已经 onPostExecute的入参类型
// 当然， 这些类型都是你自定义的， 这里只是个例子，如果为空，则设为Void
private class DownloadFilesTask extends AsyncTask<URL, Integer, Long> {
		// doInBackground 在子线程中进行 
		// 这里还有一个 方法 叫void onPreExecute () 是在doInBackground前被执行的一些初始化的
		// 一些初始化的工作可以在 onPreExecute 进行。
     protected Long doInBackground(URL... urls) {
         int count = urls.length;
         long totalSize = 0;
         for (int i = 0; i < count; i++) {
             totalSize += Downloader.downloadFile(urls[i]);
             // 这个publishProgress 会调用onProgressUpdate，
             // 把你计算的进度结果告诉onProgressUpdate
             publishProgress((int) ((i / (float) count) * 100));
             // Escape early if cancel() is called
             // AsyncTask.cancel() 可以取消任务。
             // 所以当任务有可能被取消的时候， 就需要做一些处理，防止出错。
             if (isCancelled()) break;
         }
         return totalSize;
     }
		// onProgressUpdate 在UI线程的方法。告诉你 任务进行到百分之几了
		// 在UI上可以告诉用户这个百分比
     protected void onProgressUpdate(Integer... progress) {
         setProgressPercent(progress[0]);
     }
		// onPostExecute 同样在UI线程的方法 任务完成后的一些处理
		// result 就是网络加载的结果了
     protected void onPostExecute(Long result) {
         showDialog("Downloaded " + result + " bytes");
     }
     // 顺带提一句 onPreExecute 也是在UI线程中的
 }
```
开始任务 就一句话

```
//这里的execute 参数就是 doInBackground的入参，最终会转化成数组，所以，放几个都可以。
new DownloadFilesTask().execute(url1, url2);
```
**注:上诉代码来自developer.android.com中文注释为博主添加**

# HttpURLConnection

`对于上手就是OKHttp，Volley，等网络请求框架的人来说（没错就是我）HttpURLConnection 确实需要学习一下。哈哈！如果你已经很熟了，翻篇吧。`

写来写去，发现CSDN有个大牛，写的特别清楚特别全面就贴个[链接](http://blog.csdn.net/woxueliuyun/article/details/43267365)吧，膜拜膜拜。

说到了网络请求， 这里就提一下我使用的 图片加载管理吧-->Picasso。
如果你用 Androidstudio 的话 添加和使用都很方便。
首先你在build.gradle(Module:app)中 添加Picasso依赖

```
//简单粗暴一看就会
android {
    ...
    repositories {
        mavenCentral()
    }
}
dependencies {
    ...
    compile 'com.squareup.picasso:picasso:2.5.2'
}
```

然后就可以食用Picasso啦。

```
//最粗暴的用法
Picasso.with(context).load(url).into(imageview);
```
想看 Picasso 详细点的， 具体点的， 丰富的的，[戳这里](http://square.github.io/picasso/)<http://square.github.io/picasso/>

# 数据来源

说了这么多，异步任务，网络请求，图片加载缓存的控件，我们的数据从那里拿呢
www.themoviedb.org 由这个TA提供，你需要注册一个帐户，然后添加一个APP，并填写一下信息（反正我是瞎写的）然后就会得到一个API_KEY，这是你请求数据的关键哟。
之后你想要啥数据<http://docs.themoviedb.apiary.io/#/>参见API文档吧。

# 结束

网络请求，已经接口相关的准备工作完成。



