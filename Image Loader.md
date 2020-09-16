# Watermelon Framework

## Image loader

### Dir
```
java
└ [packages]
  └ domain
    └ [domain_name]
      └ view
        └ activity
          └ [any_activity]
        └ adapter
          └ [any_adapter]
        └ fragment
          └ [any_view]
        ...
      ...
    ...
  ...
```

### Image manage

Just call the GlideImgManager (It can be changed name)

```java
GlideImgManager.getInstance()
    .setImages(this, 
        targetImgView,
        new ImageSource("{URL}", 
            ImageView.ScaleType.CENTER_CROP, 
            ImageSource.SourceType.DRAWABLE));
```

The GlideImgManager and ImageSource declarations

```java
// (GlideImgManager.java)
// ...
public void setImages(Context context, ImageView imageView, ImageSource imageSource) // ...
public void setImages(Context context, ImageView imageView, ImageSource imageSource, boolean placeholderOn) // ...
public void setImages(Context context, ImageView imageView, ImageSource imageSource, int refreshImageId) // ...
public void setImages(Context context, ImageView imageView, ImageSource imageSource, int refreshImageId, boolean placeholderOn) // ...
public void setImages(Context context, ImageView imageView, ImageSource imageSource, int refreshImageId, int placeholderImageId) // ...
```

```java
// (ImageSource.java)
// ...
public ImageSource(Drawable drawable) //...
public ImageSource(Drawable drawable, ScaleType scaleType) //...
public ImageSource(Drawable drawable, ScaleType scaleType, ImageSource.SourceType sourceType) //...
public ImageSource(String url) //...
public ImageSource(String url, ScaleType scaleType) //...
public ImageSource(String url, ScaleType scaleType, ImageSource.SourceType sourceType) //...
```


So you can call any of the functions or constructors it will be work.