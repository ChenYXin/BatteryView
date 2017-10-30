# BatteryView
一个Android开发中可能会使用到电池控件view，有水平和垂直两个方向，同时根据电池电量更改电池中的电量颜色
<br/>

**前言**：本文提供一个Android开发中可能会使用到电池控件view，有水平和垂直两个方向，同时根据电池电量更改电池中的电量颜色。首先看下效果图：
![](http://upload-images.jianshu.io/upload_images/3550596-d8eb509bffbcf0d3.gif?imageMogr2/auto-orient/strip)
以下为实现过程：

1.在values目录下新建attrs,添加所需要的名字啊，包括可以更改的电池排列方向，电池颜色，电池电量。
```
<declare-styleable name="Battery">
    <attr name="batteryOrientation">
        <enum name="horizontal" value="0"/>
        <enum name="vertical" value="1"/>
    </attr>
    <attr name="batteryColor" format="color"/>
    <attr name="batteryPower" format="integer"/>
</declare-styleable>
```
2.创建BatteryView文件继承View，在View的构造方法中，获取我们需要的自定义样式。重写onMesure，onDraw方法。
```
package com.donkor.demo;

import android.content.Context;
import android.content.res.TypedArray;
import android.graphics.Canvas;
import android.graphics.Color;
import android.graphics.Paint;
import android.graphics.RectF;
import android.util.AttributeSet;
import android.view.View;

/**
 * @author donkor
 * 自定义水平\垂直电池控件
 */
public class BatteryView extends View {
    private int mPower = 100;
    private int orientation;
    private int width;
    private int height;
    private int mColor;

    public BatteryView(Context context) {
        super(context);
    }

    public BatteryView(Context context, AttributeSet attrs) {
        super(context, attrs);
        TypedArray typedArray = context.obtainStyledAttributes(attrs, R.styleable.Battery);
        mColor = typedArray.getColor(R.styleable.Battery_batteryColor, 0xFFFFFFFF);
        orientation = typedArray.getInt(R.styleable.Battery_batteryOrientation, 0);
        mPower = typedArray.getInt(R.styleable.Battery_batteryPower, 100);
        width = getMeasuredWidth();
        height = getMeasuredHeight();
        /**
         * recycle() :官方的解释是：回收TypedArray，以便后面重用。在调用这个函数后，你就不能再使用这个TypedArray。
         * 在TypedArray后调用recycle主要是为了缓存。当recycle被调用后，这就说明这个对象从现在可以被重用了。
         *TypedArray 内部持有部分数组，它们缓存在Resources类中的静态字段中，这样就不用每次使用前都需要分配内存。
         */
        typedArray.recycle();
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        //对View上的內容进行测量后得到的View內容占据的宽度
        width = getMeasuredWidth();
        //对View上的內容进行测量后得到的View內容占据的高度
        height = getMeasuredHeight();
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        //判断电池方向    horizontal: 0   vertical: 1
        if (orientation == 0) {
            drawHorizontalBattery(canvas);
        } else {
            drawVerticalBattery(canvas);
        }
    }

    /**
     * 绘制水平电池
     *
     * @param canvas
     */
    private void drawHorizontalBattery(Canvas canvas) {
        Paint paint = new Paint();
        paint.setColor(mColor);
        paint.setStyle(Paint.Style.STROKE);
        float strokeWidth = width / 20.f;
        float strokeWidth_2 = strokeWidth / 2;
        paint.setStrokeWidth(strokeWidth);
        RectF r1 = new RectF(strokeWidth_2, strokeWidth_2, width - strokeWidth - strokeWidth_2, height - strokeWidth_2);
        //设置外边框颜色为黑色
        paint.setColor(Color.BLACK);
        canvas.drawRect(r1, paint);
        paint.setStrokeWidth(0);
        paint.setStyle(Paint.Style.FILL);
        //画电池内矩形电量
        float offset = (width - strokeWidth * 2) * mPower / 100.f;
        RectF r2 = new RectF(strokeWidth, strokeWidth, offset, height - strokeWidth);
        //根据电池电量决定电池内矩形电量颜色
        if (mPower < 30) {
            paint.setColor(Color.RED);
        }
        if (mPower >= 30 && mPower < 50) {
            paint.setColor(Color.BLUE);
        }
        if (mPower >= 50) {
            paint.setColor(Color.GREEN);
        }
        canvas.drawRect(r2, paint);
        //画电池头
        RectF r3 = new RectF(width - strokeWidth, height * 0.25f, width, height * 0.75f);
        //设置电池头颜色为黑色
        paint.setColor(Color.BLACK);
        canvas.drawRect(r3, paint);
    }

    /**
     * 绘制垂直电池
     *
     * @param canvas
     */
    private void drawVerticalBattery(Canvas canvas) {
        Paint paint = new Paint();
        paint.setColor(mColor);
        paint.setStyle(Paint.Style.STROKE);
        float strokeWidth = height / 20.0f;
        float strokeWidth2 = strokeWidth / 2;
        paint.setStrokeWidth(strokeWidth);
        int headHeight = (int) (strokeWidth + 0.5f);
        RectF rect = new RectF(strokeWidth2, headHeight + strokeWidth2, width - strokeWidth2, height - strokeWidth2);
        canvas.drawRect(rect, paint);
        paint.setStrokeWidth(0);
        float topOffset = (height - headHeight - strokeWidth) * (100 - mPower) / 100.0f;
        RectF rect2 = new RectF(strokeWidth, headHeight + strokeWidth + topOffset, width - strokeWidth, height - strokeWidth);
        paint.setStyle(Paint.Style.FILL);
        canvas.drawRect(rect2, paint);
        RectF headRect = new RectF(width / 4.0f, 0, width * 0.75f, headHeight);
        canvas.drawRect(headRect, paint);
    }

    /**
     * 设置电池电量
     *
     * @param power
     */
    public void setPower(int power) {
        this.mPower = power;
        if (mPower < 0) {
            mPower = 100;
        }
        invalidate();//刷新VIEW
    }

    /**
     * 设置电池颜色
     *
     * @param color
     */
    public void setColor(int color) {
        this.mColor = color;
        invalidate();
    }

    /**
     * 获取电池电量
     *
     * @return
     */
    public int getPower() {
        return mPower;
    }
}
```
3.在布局文件中声明我们的VIEW
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center"
    android:orientation="vertical"
   >
    <com.donkor.demo.BatteryView
        android:id="@+id/horizontalBattery"
        android:layout_width="30dp"
        android:layout_marginBottom="30dp"
        android:layout_height="14dp"
        android:layout_marginTop="5dp"
        android:background="#fff"
        android:gravity="center"
        app:batteryPower="70"
        app:batteryColor="@android:color/black"
        app:batteryOrientation="horizontal"
        />
    <com.donkor.demo.BatteryView
        android:id="@+id/verticalBattery"
        android:layout_width="14dp"
        android:layout_height="25dp"
        android:layout_marginTop="5dp"
        android:background="#fff"
        android:gravity="center"
        app:batteryOrientation="vertical"
        />
</LinearLayout>
```
4.在Activity中加入一个线程，使电池无限循环，营造出一个正在充电的状态，这样就大功告成。

```
package com.donkor.demo;

import android.app.Activity;
import android.graphics.Color;
import android.os.Bundle;
import android.os.Handler;
import android.os.Message;

import java.util.Timer;
import java.util.TimerTask;

/**
 * @author donkor
 */
public class MainActivity extends Activity {

    private BatteryView horizontalBattery, verticalBattery;
    private int power;
    private Handler mHandler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
            switch (msg.what) {
                case 0:
                    horizontalBattery.setPower(power += 5);
                    if (power == 100) {
                        power = 0;
                    }
                    break;
                default:
                    break;
            }
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        horizontalBattery = (BatteryView) findViewById(R.id.horizontalBattery);
        verticalBattery = (BatteryView) findViewById(R.id.verticalBattery);

        verticalBattery.setColor(Color.BLACK);
        verticalBattery.setPower(85);

        new Timer().schedule(new TimerTask() {
            @Override
            public void run() {
                mHandler.sendEmptyMessage(0);
            }
        }, 0, 100);
    }
}
```

**文章地址：http://blog.csdn.net/donkor_/article/details/53175903**

# About me
❤ 我觉得大家一起学习和交流才会更有意思，如果您觉得我的文章还不错，或是对您有过帮助，欢迎Follow、Fork、Star .咱们大家一起学习，一起交流~~❤

Email ：donkor@yeah.net

csdn_blog: http://blog.csdn.net/donkor_

jianshu_blog: http://www.jianshu.com/u/10f35b1d7e12