## Android 图片显示（Glide）

* [Glide引用](#glide引用)
* [基础](#基础)
    * [ImageResizing](#imageresizing)
    * [CenterCropping](#centercropping)
    * [ImageError](#imageerror)
    * [AsBitmap](#asbitmap)
    * [动画](#动画)
    * [缓存设置](#缓存设置)
    * [磁盘保存](#磁盘保存)
    * [加载缩略图](#加载缩略图)
    * [Transformations](#transformations)
    * [图片格式设置](#图片格式设置)
* [回调](#回调)
* [特殊](#特殊)



#### Glide引用
在项目中添加依赖非常简单：
```
dependencies {  
    compile 'com.github.bumptech.glide:glide:3.5.2' 
}
```
当然还需要给上网权限
```
<uses-permission android:name="android.permission.INTERNET"></uses-permission>
```


#### 基础
Glide和Picasso非常相似，Glide加载图片的方法和Picasso如出一辙。
    
    Glide.with(context).load(imageViewUrl).into(imageView); 
    Glide的with方法不光接受Context，还接受Activity 和 Fragment，Context会自动的从他们获取
    同时将Activity/Fragment作为with()参数的好处是：
    图片加载会和Activity/Fragment的生命周期保持一致，
    比如Paused状态在暂停加载，在Resumed的时候又自动重新加载。
    所以建议传参的时候传递Activity 和 Fragment给Glide，而不是Context。
    imageViewUrl可以是网络图片，本地图片，资源图片，uri，gif
```
Glide.with(context).load(imageViewUrl).into(imageView);  
```

###### ImageResizing
图片大小设置
```
Glide.override(300, 200);
```

###### CenterCropping
图片中心裁剪
```
Glide.centerCrop();
```

###### ImageError
设置占位图或者加载错误图：
```
Glide.placeholder(R.drawable.placeholder).error(R.drawable.imagenotfound)
```

###### AsBitmap
强制处理为bitmap
```
Glide.asBitmap();
```

###### 动画
Glide里面包含了一些显示动画
可以通过animate()添加，也可以直接调用
例如：crossFade()淡入显示
```
Glide.crossFade();
```
但是设置crossFade()必须去掉asBitmap()

###### 缓存设置
内存缓存设置,通过skipMemoryCache(boolean)来设置是否需要缓存到内存,默认是会缓存到内存的.
```
Glide.skipMemoryCache(true);
```

###### 磁盘保存
磁盘缓存,磁盘缓存通过diskCacheStrategy(DiskCacheStrategy)来设置,
DiskCacheStrategy一共有4种模式:
* DiskCacheStrategy.NONE:什么都不缓存
* DiskCacheStrategy.SOURCE:仅缓存原图(全分辨率的图片)
* DiskCacheStrategy.RESULT:仅缓存最终的图片,即修改了尺寸或者转换后的图片
* DiskCacheStrategy.ALL:缓存所有版本的图片,默认模式
```
Glide.diskCacheStrategy(DiskCacheStrategy.RESULT);
```

###### 加载缩略图
通过设置缩略图,我们可以在显示目标图片之前先展示一个第分辨率或者其他图片,当全分辨率的目标图片在后台加载完成后,
Glide会自动切换显示全像素的目标图片.  
设置缩略图有2种方式:
通过thumbnail(float)指定0.0f~1.0f的原始图像大小,例如全像素的大小是500500,如果设置为thumbnail为0.1f,即目标图片的10%,显示的缩略图大小就是5050;
```
Glide.thumbnail(0.1f);
```
通过thumbnail(DrawableRequestBuilder)方式来指定缩略图,该缩略图可以使用load的所有方式(网络,文件,uri,资源)加载.
```
//缩略图请求
    DrawableRequestBuilder<String> thumbnailRequest = Glide.with(this).load("缩略图url");
    Glide.thumbnail(thumbnailRequest);
```

###### Transformations
在显示目标图片之前,我们可以对目标图片的Bitmap进行相应的处理,例如::圆角图片,圆形图片,高斯模糊,旋转,灰度等等.
Bitmap处理可以在github上glide-transformations-master库


###### 图片格式设置
默认显示RGB_565，如果想替换其他格式
可以创建一个新的GlideModule将Bitmap格式转换到ARGB_8888：
```
public class GlideConfiguration implements GlideModule {  

    @Override  
    public void applyOptions(Context context, GlideBuilder builder) {  
        // Apply options to the builder here.  
        builder.setDecodeFormat(DecodeFormat.PREFER_ARGB_8888);  
    }  

    @Override  
    public void registerComponents(Context context, Glide glide) {  
        // register ModelLoaders here.  
    }  
}
```
同时在AndroidManifest.xml中将GlideModule定义为meta-data
```
<meta-data  
    android:name="com.inthecheesefactory.lab.glidepicasso.GlideConfiguration"  
    android:value="GlideModule"/>
```


#### 回调
SimpleTarget和ViewTarget
Target接口代表Glide中资源最终被加载到的地方并且可以毁掉Glide中的生命周期方法
如果我们想获取图片，但是不显示
```
private SimpleTarget target = new SimpleTarget<Bitmap>() {  
    @Override
    public void onResourceReady(Bitmap bitmap, GlideAnimation glideAnimation) {

       //在这里我们就可以获得加载的资源了，当然这里是一个bitmap。
    }
};

private void loadImageSimpleTarget() {  
    Glide
        .with( context )
        .load( imageUrl)
        .asBitmap()
        .into( target ); //使用Target。
}
```
要注意的是Glide的生命周期，图片利用如果图片没加载出来跳转到另一个页面利用，当然不会显示啦。
还可以指定SimpleTarget获得的资源的尺寸：
```
private SimpleTarget target2 = new SimpleTarget<Bitmap>( 250, 250 ) {  
    @Override
    public void onResourceReady(Bitmap bitmap, GlideAnimation glideAnimation) {
        imageView2.setImageBitmap( bitmap );
    }
};
```

**如果显示的控件不是ImageView时**
```
//定义一个ViewTarget，注意传入自定义View对象CustomView，还有ViewTarget的泛型。
    ViewTarget viewTarget = new ViewTarget<CustomView, GlideDrawable>( customView ) {
        @Override
        public void onResourceReady(GlideDrawable resource, GlideAnimation<? super GlideDrawable> glideAnimation) {
          //这里我们获得传入的自定义View，在这个自定义View中我们写了该View设置图片的方法。
            this.view.setImage( resource.getCurrent() );
        }
    };

    Glide
        .with( context.getApplicationContext() ) 
        .load( eatFoodyImages[2] )
        .into( viewTarget ); //使用Target。
```


#### 特殊
另外Glide还可以加载gif图片

    同时因为Glide和Activity/Fragment的生命周期是一致的，
    因此gif的动画也会自动的随着Activity/Fragment的状态暂停、重放。
    Glide 的缓存在gif这里也是一样，调整大小然后缓存。
    但是Glide 动画会消费太多的内存，因此谨慎使用。
除了gif动画之外，Glide还可以将任何的本地视频解码成一张静态图片。
Glide可以配置图片显示的动画。




参考网址：[http://ocnyang.com/tags/Glide](http://ocnyang.com/tags/Glide)

