---
title: 【构建Android缓存模块】（二）Memory Cache & File Cache
date: 2014-06-05 23:34:36 +0800
comments: true
categories: [Android]
desc: 内存缓存的存取速度非常惊人，远远快于文件读取，如果没有内存限制，当然首选这种方式。遗憾的是我们有着 16M 的限制（当然大多数设备限制要高于 Android 官方说的这个数字），这也正是大 Bitmap 容易引起 OOM 的原因。Memory Cache 将使用 WeakHashMap 作为缓存的中枢，当程序内存告急时，它会主动清理部分弱引用（因此，当引用指向为 null ，我们必须转向硬盘缓存读取数据，如果硬盘也没有，那还是重新下载吧）。
---

* Contents
{:toc}

上一篇博客我们讲到普通应用缓存Bitmap的实现分析，根据 MVC 的实现原理，我将这个简单的缓存实现单独写成一个模块，这样可以方便以后的使用，对于任意的需求，都属于一个可插拔式的功能。

之前提到，这个缓存模块主要有两个子部件：

## Memory Cache

内存缓存的存取速度非常惊人，远远快于文件读取，如果没有内存限制，当然首选这种方式。遗憾的是我们有着**16M**的限制（当然大多数设备限制要高于 Android 官方说的这个数字），这也正是大 Bitmap容易引起 OOM 的原因。Memory Cache 将使用 ``WeakHashMap`` 作为缓存的中枢，当程序内存告急时，它会主动清理部分弱引用（因此，当引用指向为 null ，我们必须转向硬盘缓存读取数据，如果硬盘也没有，那还是重新下载吧）。

能力越大，责任越大？人家只是跑得快了点儿，总得让人家休息，我们一定不希望让内存成为第一位跑完马拉松的 Pheidippides ，一次以后就挂了吧？作为精打细算的猿媛，我们只能将有限的内存分配给 Memory Cache ，将更繁重的任务托付给任劳任怨的 SDCard 。

## File Cache

硬盘读取速度当然不如内存，但是为了珍惜宝贵的流量，不让你的用户在月底没有流量时嚎叫着要删掉你开发的“流量杀手”，最好是避免重复下载。在第一次下载以后，将数据保存在本地即可。

文件读写的技术并不是很新颖的技术，Java Core 那点儿就够你用了。不过要记得我们可是将Bitmap写入文件啊，怎么写入呢？不用着急，Android 的 Bitmap 本身就具备将数据写入 OutputStream 的能力。我将这些额外的方法写在一个帮助类中：BitmapHelper

```java
public static boolean saveBitmap(File file, Bitmap bitmap){
	if(file == null || bitmap == null)
		return false;
	try {
		BufferedOutputStream out = new BufferedOutputStream(new FileOutputStream(file));
		return bitmap.compress(CompressFormat.JPEG, 100, out);
	} catch (FileNotFoundException e) {
		e.printStackTrace();
		return false;
	}
}
```

最后附上Memory Cache和File Cache的具体代码，非常简单。

```java
public class MemoryCache {
	private static final String TAG = "MemoryCache";
	
	//WeakReference Map: key=string, value=Bitmap
    private WeakHashMap<String, Bitmap> cache = new WeakHashMap<String, Bitmap>();
    
    /**
     * Search the memory cache by a unique key. 
     * @param key Should be unique. 
     * @return The Bitmap object in memory cache corresponding to specific key.
     * */
    public Bitmap get(String key){
        if(key != null)
        	return cache.get(key);
        return null;
    }
    
    /**
     * Put a bitmap into cache with a unique key.
     * @param key Should be unique.
     * @param value A bitmap.
     * */
    public void put(String key, Bitmap value){
    	if(key != null && !"".equals(key) && value != null){
    		cache.put(key, value);
    		//Log.i(TAG, "cache bitmap: " + key);
    		Log.d(TAG, "size of memory cache: " + cache.size());
    	}
    }

    /**
     * clear the memory cache.
     * */
    public void clear() {
        cache.clear();
    }
}
```

```java
public class FileCache {
	
	private static final String TAG = "MemoryCache";
	
	private File cacheDir;	//the directory to save images

	/**
	 * Constructor
	 * @param context The context related to this cache.
	 * */
	public FileCache(Context context) {
		// Find the directory to save cached images
		if (android.os.Environment.getExternalStorageState()
				.equals(android.os.Environment.MEDIA_MOUNTED))
			cacheDir = new File(
					android.os.Environment.getExternalStorageDirectory(),
					Config.CACHE_DIR);
		else
			cacheDir = context.getCacheDir();
		if (!cacheDir.exists())
			cacheDir.mkdirs();
		
		Log.d(TAG, "cache dir: " + cacheDir.getAbsolutePath());
	}

	/**
	 * Search the specific image file with a unique key.
	 * @param key Should be unique.
	 * @return Returns the image file corresponding to the key.
	 * */
	public File getFile(String key) {
		File f = new File(cacheDir, key);
		if (f.exists()){
			Log.i(TAG, "the file you wanted exists " + f.getAbsolutePath());
			return f;
		}else{
			Log.w(TAG, "the file you wanted does not exists: " + f.getAbsolutePath());
		}

		return null;
	}

	/**
	 * Put a bitmap into cache with a unique key.
	 * @param key Should be unique.
	 * @param value A bitmap.
	 * */
	public void put(String key, Bitmap value){
		File f = new File(cacheDir, key);
		if(!f.exists())
			try {
				f.createNewFile();
			} catch (IOException e) {
				e.printStackTrace();
			}
		//Use the util's function to save the bitmap.
		if(BitmapHelper.saveBitmap(f, value))
			Log.d(TAG, "Save file to sdcard successfully!");
		else
			Log.w(TAG, "Save file to sdcard failed!");
		
	}
	
	/**
	 * Clear the cache directory on sdcard.
	 * */
	public void clear() {
		File[] files = cacheDir.listFiles();
		for (File f : files)
			f.delete();
	}
}
```

 没什么难的地方，直接贴代码。下一篇博客我将讲解如何使用异步任务下载数据，以及使用Controller操作Model，控制View的显示。 
