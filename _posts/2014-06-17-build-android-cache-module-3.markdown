---
title: 【构建Android缓存模块】（三）Controller & 异步图片加载
date: 2014-06-17 15:48:38 +0800
comments: true
categories: [Android]
desc: 上一篇博客我们学习了缓存模块的实现， 缓存分做两份：Memory Cache 和 File Cache。方法也很简单，分别是：1. 存储文件 2. 按唯一key值索引文件 3. 清空缓存 区别在于内存缓存读取优先，因为它读写的速度更快。但是考虑到内存限制，退而选用文件存储，分担内存缓存的压力。原理非常简单，在第一课中已经详细分析了。那么要怎么才能将这个缓存模块与UI模块的显示关联起来呢？在这里我们需要一个控制器，掌管数据流向和读写，同时控制 UI 的显示。
---

上一篇博客我们学习了缓存模块的实现， 缓存分做两份：``Memory Cache``和``File Cache``。方法也很简单，分别是：

- 存储文件
- 按唯一key值索引文件
- 清空缓存

区别在于内存缓存读取优先，因为它读写的速度更快。但是考虑到内存限制，退而选用文件存储，分担内存缓存的压力。

原理非常简单，在第一课中已经详细分析了。那么要怎么才能将这个缓存模块与UI模块的显示关联起来呢？在这里我们需要一个控制器，掌管数据流向和读写，同时控制UI的显示。

那么这个控制器需要以下的元素：

- 内存缓存
- 硬盘缓存
- 异步任务处理
- 控制UI显示

```java
//caches
private MemoryCache memoryCache;
private FileCache fileCache;
//Asynchronous task
private static AsyncImageLoader imageLoader;
```

``Memory Cache``和``File Cache``在上一课中有具体的实现，这里有一个异步的任务处理器—— ``AsyncImageDownloader``，它用来在后台下载数据，完成下载后存储数据到缓存中，并更新UI的显示 。让我们来看看它是如何实现的：

```java
class AsyncImageDownloader extends AsyncTask<Void, Void, Bitmap>{
	private ImageView imageView;
	private String fileName;
	
	public AsyncImageDownloader(ImageView imageView, String fileName){
		this.imageView = imageView;
		this.fileName = fileName;
	}
	
	@Override
	protected void onPreExecute() {
		super.onPreExecute();
		imageView.setImageResource(R.drawable.placeholder);
	}
	
	@Override
	protected Bitmap doInBackground(Void... arg0) {
		String url = Utils.getRealUrlOfPicture(fileName);
		HttpResponse response = new HttpRetriever().requestGet(url, null);
		Log.i(TAG, "url: " + url);
		Log.i(TAG, "respone: " + response);
		InputStream in = null;
		try {
			if(response != null && response.getEntity() != null)
				in = response.getEntity().getContent();
		} catch (IllegalStateException e) {
			e.printStackTrace();
			return null;
		} catch (IOException e) {
			e.printStackTrace();
			return null;
		}
		
		//TODO to be optimized: adjust the size of bitmap
		return BitmapFactory.decodeStream(in);
	}
	
	@Override
	protected void onPostExecute(Bitmap result) {
		super.onPostExecute(result);
		if(result != null && imageView != null)
			imageView.setImageBitmap(result);
		
		//TODO cache the bitmap both in sdcard & memory
		memoryCache.put(fileName, result);// key is a unique token, value is the bitmap
		
		fileCache.put(fileName, result);
	}
}
```

可以看到这个类的构造函数需要两个参数，分别是文件名和对应要显示的``ImageView``，那么在任务开始的时候，可以为该``ImageView``设置未下载状态的图片，然后下载完成后更新UI。

> 注：需要提醒的是，这里的唯一key值，我使用的是文件名，因为我接收到的文件名是唯一的。猿媛们也可以根据自己的需求，设计自己的唯一key值算法。

接下来，我们需要读用``key``值索引相应的``Bitmap``：

```java
public Bitmap getBitmap(String key){
	Bitmap bitmap = null;
	//1. search memory
	bitmap = memoryCache.get(key);
	
	//2. search sdcard
	if(bitmap == null){
		File file = fileCache.getFile(key);
		if(file != null)
			bitmap = BitmapHelper.decodeFile(file, null);
	}
	
	return bitmap;
}
```
到这里，一个简单的缓存框架就搭建成功了。它简洁有效，但是非常单薄，似乎不够强大，需要你们根据自己的需求进行修改。另外它本来的目的就是用于演示，理解这个以后，我们再来看``Google``的``BitmapFun``。

不过，我将它应用在一个小项目中，性能还不错。对于小项目的需求，应该是够的。

最后，附上使用方法，以及整个类的代码。

## 使用

```java
AsyncImageLoader imageLoader = AsyncImageLoader.getInstance(this);、
imageLoader.displayBitmap(imageView, fileName);
```

## AsyncImageLoader

```java
public class AsyncImageLoader {

	private static final String TAG = "AsyncImageLoader";
	
	//caches
	private MemoryCache memoryCache;
	private FileCache fileCache;
	//Asynchronous task
	private static AsyncImageLoader imageLoader;

	class AsyncImageDownloader extends AsyncTask<Void, Void, Bitmap>{
		private ImageView imageView;
		private String fileName;
		
		public AsyncImageDownloader(ImageView imageView, String fileName){
			this.imageView = imageView;
			this.fileName = fileName;
		}
		
		@Override
		protected void onPreExecute() {
			super.onPreExecute();
			imageView.setImageResource(R.drawable.placeholder);
		}
		
		@Override
		protected Bitmap doInBackground(Void... arg0) {
			String url = Utils.getRealUrlOfPicture(fileName);
			HttpResponse response = new HttpRetriever().requestGet(url, null);
			Log.i(TAG, "url: " + url);
			Log.i(TAG, "respone: " + response);
			InputStream in = null;
			try {
				if(response != null && response.getEntity() != null)
					in = response.getEntity().getContent();
			} catch (IllegalStateException e) {
				e.printStackTrace();
				return null;
			} catch (IOException e) {
				e.printStackTrace();
				return null;
			}
			
			//TODO to be optimized: adjust the size of bitmap
			return BitmapFactory.decodeStream(in);
		}
		
		@Override
		protected void onPostExecute(Bitmap result) {
			super.onPostExecute(result);
			if(result != null && imageView != null)
				imageView.setImageBitmap(result);
			
			//TODO cache the bitmap both in sdcard & memory
			memoryCache.put(fileName, result);// key is a unique token, value is the bitmap
			
			fileCache.put(fileName, result);
		}
	}
	
	private AsyncImageLoader(Context context){
		this.memoryCache 		= 	new MemoryCache();
		this.fileCache			= 	new FileCache(context);
	}
	
	public static AsyncImageLoader getInstance(Context context){
		if(imageLoader == null)
			imageLoader = new AsyncImageLoader(context);
		
		return imageLoader;
	}
	
	public void displayBitmap(ImageView imageView, String fileName){
		//no pic for this item
		if(fileName == null || "".equals(fileName))
			return;
		
		Bitmap bitmap = getBitmap(fileName);
		//search in cache, if there is no such bitmap, launch downloads
		if(bitmap != null){
			imageView.setImageBitmap(bitmap);
		}
		else{
			Log.w(TAG, "Can't find the file you required.");
			new AsyncImageDownloader(imageView, fileName).execute();
		}
	}
	
	public Bitmap getBitmap(String key){
		Bitmap bitmap = null;
		//1. search memory
		bitmap = memoryCache.get(key);
		
		//2. search sdcard
		if(bitmap == null){
			File file = fileCache.getFile(key);
			if(file != null)
				bitmap = BitmapHelper.decodeFile(file, null);
		}
		
		return bitmap;
	}
	
	public void clearCache(){
		if(memoryCache != null)
			memoryCache.clear();
		if(fileCache != null)
			fileCache.clear();
	}
}
```

## 源码

附上源码，不过服务器的源码暂时还没有放出来，先看看客户端的吧。

[https://github.com/ryanhoo/SoftRead][1]

[1]: https://github.com/ryanhoo/SoftRead