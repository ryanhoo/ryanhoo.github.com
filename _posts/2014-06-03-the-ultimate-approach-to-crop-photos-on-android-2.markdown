---
title: Android 大图片裁剪终极解决方案（中：从相册截图）
date: 2014-06-03 22:21:35 +0800
comments: true
categories: [Android]
desc: 在这篇博客中将向大家展示如何从相册截图，让大家了解 Android  本身的限制，以及我们应当采取的实现方案。根据我们的分析与总结，图片的来源有拍照和相册，而可采取的操作有 1. 使用 Bitmap 并返回数据 2. 使用Uri不返回数据 前面我们了解到，使用 Bitmap 有可能会导致图片过大，而不能返回实际大小的图片，我将采用大图 Uri，小图 Bitmap 的数据存储方式。
---

在这篇博客中，我将向大家展示如何从相册截图。

上一篇博客中，我就拍照截图这一需求进行了详细的分析，试图让大家了解 Android 本身的限制，以及我们应当采取的实现方案。

根据我们的分析与总结，图片的来源有拍照和相册，而可采取的操作有

1. 使用Bitmap并返回数据
2. 使用Uri不返回数据

前面我们了解到，使用Bitmap有可能会导致图片过大，而不能返回实际大小的图片，我将采用大图Uri，小图Bitmap的数据存储方式。

我们将要使用到URI来保存拍照后的图片：

```java
static final String IMAGE_FILE_LOCATION = "file:///sdcard/temp.jpg";//temp file
Uri imageUri = Uri.parse(IMAGE_FILE_LOCATION);//The Uri to store the big bitmap
```

不难知道，我们从相册选取图片的**Action**为``Intent.ACTION_GET_CONTENT``。

根据我们上一篇博客的分析，我准备好了两个实例的Intent。

#### 一、从相册截大图：

```java
Intent intent = new Intent(Intent.ACTION_GET_CONTENT, null);
intent.setType("image/*");
intent.putExtra("crop", "true");
intent.putExtra("aspectX", 2);
intent.putExtra("aspectY", 1);
intent.putExtra("outputX", 600);
intent.putExtra("outputY", 300);
intent.putExtra("scale", true);
intent.putExtra("return-data", false);
intent.putExtra(MediaStore.EXTRA_OUTPUT, imageUri);
intent.putExtra("outputFormat", Bitmap.CompressFormat.JPEG.toString());
intent.putExtra("noFaceDetection", true); // no face detection
startActivityForResult(intent, CHOOSE_BIG_PICTURE);
```

#### 二、从相册截小图

```java
Intent intent = new Intent(Intent.ACTION_GET_CONTENT, null);
intent.setType("image/*");
intent.putExtra("crop", "true");
intent.putExtra("aspectX", 2);
intent.putExtra("aspectY", 1);
intent.putExtra("outputX", 200);
intent.putExtra("outputY", 100);
intent.putExtra("scale", true);
intent.putExtra("return-data", true);
intent.putExtra("outputFormat", Bitmap.CompressFormat.JPEG.toString());
intent.putExtra("noFaceDetection", true); // no face detection
startActivityForResult(intent, CHOOSE_SMALL_PICTURE);
```

#### 三、对应的onActivityResult可以这样处理返回的数据

```java
switch (requestCode) {
case CHOOSE_BIG_PICTURE:
	Log.d(TAG, "CHOOSE_BIG_PICTURE: data = " + data);//it seems to be null
	if(imageUri != null){
		Bitmap bitmap = decodeUriAsBitmap(imageUri);//decode bitmap
		imageView.setImageBitmap(bitmap);
	}
break;
case CHOOSE_SMALL_PICTURE:
	if(data != null){
		Bitmap bitmap = data.getParcelableExtra("data");
		imageView.setImageBitmap(bitmap);
	}else{
		Log.e(TAG, "CHOOSE_SMALL_PICTURE: data = " + data);
	}
break;
}
```

#### 效果图

![大图][1]

![小图][2]

[1]: /images/blog/android/183645_yuLJ_245415.gif
[2]: /images/blog/android/183707_DnNy_245415.gif