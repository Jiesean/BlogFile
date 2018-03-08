---
title: 自定义圆形ImageView
date: 2016-08-18 21:16:04
tags: Android
---

使用很简单的一种实现方法，可能存在一些bug，后期不断进行完善。

１．实现CircleImageView继承ImageView，重写onDraw()方法。说明全在注释中。

<!-- more -->

```
/**
 * 自定义圆形imageview
 * Created by Jiesean on 16-8-17.
 */
public class CircleImageView extends ImageView {

    private Paint paint = new Paint();

    //constructor 1
    public CircleImageView(Context context) {
        super(context);
    }

    //constructor ２
    public CircleImageView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    //constructor ３
    public CircleImageView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }


    /**
     * 重写ImageView的控件绘制函数
     * @param canvas　
     */
    @Override
    protected void onDraw(Canvas canvas) {
        Drawable drawable = getDrawable();
        //判断是否设置了src
        if (drawable == null) {
            return;
        }
        //判断图片大小是否为０
        if (getWidth() == 0 || getHeight() == 0) {
            return;
        }
        //判断是都为图片类型
        if (!(drawable instanceof BitmapDrawable)) {
            return;
        }
        //转换成bitmap
        Bitmap b = ((BitmapDrawable) drawable).getBitmap();
        //creates a mutable copy of the image using the option specified "BitMap.Config.ARGB_8888"
        //BitMap.Config.ARGB_8888:Each pixel is stored on 4 bytes.
        Bitmap bitmap = b.copy(Bitmap.Config.ARGB_8888, true);
        Bitmap circleBitmap = getCircleBitmap(bitmap,getWidth()/2);

        //绘制图形
        canvas.drawBitmap(circleBitmap, 0, 0, null);
    }

    /**
     * 将制定的图形按照制定的半径压缩裁剪成圆形
     * @param radius xml文件中输入的width,制定的圆形的半径
     * @param bitmap 输入的图形
     * @return 按照制定的半径裁剪的圆形的bitmap
     */
    public Bitmap getCircleBitmap(Bitmap bitmap, int radius){
        Bitmap tempBitmap;
        Rect rect = new Rect(0, 0, 2*radius, 2*radius);

        //将照片裁剪成制定半径外切正方形大小
        if (bitmap.getHeight()!=(2*radius)||bitmap.getWidth()!=(2*radius)) {//图片原长宽不符合要求的矩形大小
            //按照要求大小裁剪图片
            tempBitmap = Bitmap.createScaledBitmap(bitmap, radius*2, radius*2, false);
        }
        else{//图片原尺寸符合要求
            tempBitmap = bitmap;
        }
        Bitmap show = Bitmap.createBitmap(tempBitmap.getWidth(), tempBitmap.getHeight(),
                Bitmap.Config.ARGB_8888);
        Canvas canvas = new Canvas(show);

        Paint paint = new Paint();
        //抗锯齿，防止图片周围产生锯齿
        paint.setAntiAlias(true);
        //用来对位图进行滤波处理,防止产生锯齿的一种方式
        paint.setFilterBitmap(true);
        //设置防抖
        paint.setDither(true);

        //黑色填充整个背景
        canvas.drawARGB(0, 0, 0, 0);
        //画一个圆形
        canvas.drawCircle(radius, radius, radius, paint);
        //这里设置了一个筛选器，以上面的图形为框架去裁剪下面的图形
        paint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.SRC_IN));
        canvas.drawBitmap(tempBitmap, rect, rect, paint);

        return show;
    }

}
```

２．使用自定义的控件。宽度为必要属性，决定了圆形的直径大小。
```
<com.jiesean.jieseanproject.customview.CircleImageView 
        android:id="@+id/password_logo_imageview"    
        android:layout_width="65dp"    
        android:layout_height="65dp"
```
３．实现效果

![效果.png](http://upload-images.jianshu.io/upload_images/1806858-b2f91ce362fc5ea7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)