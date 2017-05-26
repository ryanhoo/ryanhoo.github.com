---
title: "Android大图片裁剪终极解决方案（下：拍照截图）"
date: 2014-06-03 22:34:51 +0800
comments: true
categories: [Android]
desc: 上一篇博客中，我们学习到了如何使用 Android 相册截图。在这篇博客中，我将向大家展示如何拍照截图。拍照截图有点儿特殊，要知道，现在的 Android 智能手机的摄像头都是几百万的像素，拍出来的图片都是非常大的。因此，我们不能像对待相册截图一样使用 Bitmap 小图，无论大图小图都统一使用 Uri 进行操作。
---

上一篇博客中，我们学习到了如何使用Android相册截图。在这篇博客中，我将向大家展示如何拍照截图。

拍照截图有点儿特殊，要知道，现在的Android智能手机的摄像头都是几百万的像素，拍出来的图片都是非常大的。因此，我们不能像对待相册截图一样使用Bitmap小图，无论大图小图都统一使用Uri进行操作。

#### 一、首先准备好需要使用到的Uri：

```java
private static final String IMAGE_FILE_LOCATION = "file:///sdcard/temp.jpg";//temp file
Uri imageUri = Uri.parse(IMAGE_FILE_LOCATION);//The Uri to store the big bitmap
```

#### 二、使用MediaStore.ACTION_IMAGE_CAPTURE可以轻松调用Camera程序进行拍照：

```java
Intent intent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);//action is capture
intent.putExtra(MediaStore.EXTRA_OUTPUT, imageUri);
startActivityForResult(intent, TAKE_BIG_PICTURE);//or TAKE_SMALL_PICTURE
```

#### 三、接下来就可以在 onActivityResult中拿到返回的数据（Uri），并将Uri传递给截图的程序。

```java
switch (requestCode) {
case TAKE_BIG_PICTURE:
	Log.d(TAG, "TAKE_BIG_PICTURE: data = " + data);//it seems to be null
	//TODO sent to crop
	cropImageUri(imageUri, 800, 400, CROP_BIG_PICTURE);
	
	break;
case TAKE_SMALL_PICTURE:
	Log.i(TAG, "TAKE_SMALL_PICTURE: data = " + data);
	//TODO sent to crop 
	cropImageUri(imageUri, 300, 150, CROP_SMALL_PICTURE);
	
	break;
}
```

可以看到，无论是拍大图片还是小图片，都是使用的Uri，只是尺寸不同而已。我们将这个操作封装在一个方法里面。

```java
private void cropImageUri(Uri uri, int outputX, int outputY, int requestCode){
	Intent intent = new Intent("com.android.camera.action.CROP");
	intent.setDataAndType(uri, "image/*");
	intent.putExtra("crop", "true");
	intent.putExtra("aspectX", 2);
	intent.putExtra("aspectY", 1);
	intent.putExtra("outputX", outputX);
	intent.putExtra("outputY", outputY);
	intent.putExtra("scale", true);
	intent.putExtra(MediaStore.EXTRA_OUTPUT, uri);
	intent.putExtra("return-data", false);
	intent.putExtra("outputFormat", Bitmap.CompressFormat.JPEG.toString());
	intent.putExtra("noFaceDetection", true); // no face detection
	startActivityForResult(intent, requestCode);
}
```

#### 四、最后一步，我们已经将数据传入裁剪图片程序，接下来要做的就是处理返回的数据了：

```java
switch (requestCode) {
case CROP_BIG_PICTURE://from crop_big_picture
	Log.d(TAG, "CROP_BIG_PICTURE: data = " + data);//it seems to be null
	if(imageUri != null){
		Bitmap bitmap = decodeUriAsBitmap(imageUri);
		imageView.setImageBitmap(bitmap);
	}
	break;
case CROP_SMALL_PICTURE:
	if(imageUri != null){
		Bitmap bitmap = decodeUriAsBitmap(imageUri);
		imageView.setImageBitmap(bitmap);
	}else{
		Log.e(TAG, "CROP_SMALL_PICTURE: data = " + data);
	}
	break;
default:
	break;
}
```

#### 效果图：

![截图演示][1]

> 代码托管于GitHub，会不定期更新：[https://github.com/ryanhoo/PhotoCropper][2]


[1]: /images/blog/android/184229_tlMc_245415.gif
[2]: https://github.com/ryanhoo/PhotoCropper
