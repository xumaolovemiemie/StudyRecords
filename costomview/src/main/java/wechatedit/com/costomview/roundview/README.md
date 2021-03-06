#### 广告页右上角的“跳过”按钮

效果图：
####效果图:<br>![效果图](https://github.com/ZQiang94/StudyRecords/blob/master/costomview/src/main/res/mipmap-xhdpi/gif_00.gif "效果图")

一、“跳过”按钮
1.创建自定义View
```javascript
具体的自定义view
public class RoundProgressBar extends View {
    /**
     * 画笔对象的引用
     */
    private Paint paint;

    /**
     * 圆环的颜色
     */
    private int roundColor;

    /**
     * 圆环进度的颜色
     */
    private int roundProgressColor;

    /**
     * 中间进度百分比的字符串的颜色
     */
    private int textColor;

    /**
     * 中间进度百分比的字符串的字体
     */
    private float textSize;

    /**
     * 圆环的宽度
     */
    private float roundWidth;

    /**
     * 最大进度
     */
    private int max;

    /**
     * 当前进度
     */
    private int progress = 0;
    /**
     * 是否显示中间的进度
     */
    private boolean textIsDisplayable;

    /**
     * 进度的风格，实心或者空心
     */
    private int style;

    public static final int STROKE = 0;
    public static final int FILL = 1;

    private int interval = 8;  //第二个圆环距离大圆外边缘的距离，根据ui自己调整
    private String text;//中间要显示的文字
    private float lineWidth = 1;  //中间圆环的宽度

    public RoundProgressBar(Context context) {
        this(context, null);
    }

    public RoundProgressBar(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public RoundProgressBar(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
        paint = new Paint();

        TypedArray mTypedArray = context.obtainStyledAttributes(attrs,
                R.styleable.RoundProgressBar);

        //获取自定义属性和默认值
        roundColor = mTypedArray.getColor(R.styleable.RoundProgressBar_roundColor, Color.RED);
        roundProgressColor = mTypedArray.getColor(R.styleable.RoundProgressBar_roundProgressColor, Color.WHITE);
        textColor = mTypedArray.getColor(R.styleable.RoundProgressBar_textColor, Color.GREEN);
        textSize = mTypedArray.getDimension(R.styleable.RoundProgressBar_textSize, 15);
        text = mTypedArray.getString(R.styleable.RoundProgressBar_text);
        roundWidth = mTypedArray.getDimension(R.styleable.RoundProgressBar_roundWidth, 5);
        max = mTypedArray.getInteger(R.styleable.RoundProgressBar_max, 100);
        textIsDisplayable = mTypedArray.getBoolean(R.styleable.RoundProgressBar_textIsDisplayable, true);
        style = mTypedArray.getInt(R.styleable.RoundProgressBar_style, 0);

        mTypedArray.recycle();
    }


    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);

        int radius = (int) (getWidth() - roundWidth) / 2; //圆环的半径

        //最大圆
        int maxLayerX = getWidth() / 2;
        paint.setColor(getResources().getColor(R.color.black_translucent));
        paint.setStyle(Paint.Style.FILL);
        paint.setAntiAlias(true);
        canvas.drawCircle(maxLayerX, maxLayerX, maxLayerX, paint);

        //中间圆圈
        paint.setColor(getResources().getColor(R.color.withe));
        paint.setStyle(Paint.Style.STROKE);
        paint.setStrokeWidth(lineWidth); //暂时写死在这里，根据自己项目调整该值大小，如果有需要也可写入attr中
        canvas.drawCircle(maxLayerX, maxLayerX, maxLayerX - interval, paint); //画出圆环

        //中间的文字
        paint.setColor(getResources().getColor(R.color.withe));
        paint.setTextSize(textSize);
        paint.setStrokeWidth(0);
        paint.setTypeface(Typeface.DEFAULT);
        float textWidth = paint.measureText(text);
        Paint.FontMetricsInt fontMetrics = paint.getFontMetricsInt();
        int baseline = (getMeasuredHeight() - fontMetrics.bottom + fontMetrics.top) / 2 - fontMetrics.top;
        canvas.drawText(text, maxLayerX - textWidth / 2, baseline, paint);

        //设置进度是实心还是空心
        paint.setStrokeWidth(10); //设置圆环的宽度
        paint.setColor(getResources().getColor(R.color.withe));  //设置进度的颜色
        paint.setStyle(Paint.Style.FILL);
        float displacement = interval + lineWidth * 2;
        RectF oval = new RectF(maxLayerX - radius + displacement, maxLayerX - radius + displacement,
                maxLayerX + radius - displacement, maxLayerX + radius - displacement);  //用于定义的圆弧的形状和大小的界限

        switch (style) {
            case STROKE: {
                paint.setStyle(Paint.Style.STROKE);
                canvas.drawArc(oval, 0, 360 * progress / max, false, paint);  //根据进度画圆弧
                break;
            }
            case FILL: {
                paint.setStyle(Paint.Style.FILL_AND_STROKE);
                if (progress != 0)
                    canvas.drawArc(oval, 0, 360 * progress / max, true, paint);  //根据进度画圆弧
                break;
            }
        }

    }

    public void start() {
        new Thread(new Runnable() {

            @Override
            public void run() {
                while (progress <= 100) {
                    progress += 3;
                    setProgress(progress);
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }

            }
        }).start();
    }

    public void stop() {
        progress = 100;
    }


    public synchronized int getMax() {
        return max;
    }

    /**
     * 设置进度的最大值
     *
     * @param max
     */
    public synchronized void setMax(int max) {
        if (max < 0) {
            throw new IllegalArgumentException("max not less than 0");
        }
        this.max = max;
    }

    /**
     * 获取进度.需要同步
     *
     * @return
     */
    public synchronized int getProgress() {
        return progress;
    }

    /**
     * 设置进度，此为线程安全控件，由于考虑多线的问题，需要同步
     * 刷新界面调用postInvalidate()能在非UI线程刷新
     *
     * @param progress
     */
    public synchronized void setProgress(int progress) {
        if (progress < 0) {
            throw new IllegalArgumentException("progress not less than 0");
        }
        if (progress > max) {
            progress = max;
        }
        if (progress <= max) {
            this.progress = progress;
            postInvalidate();
        }

    }

}
```

自定义属性
```javascript
<declare-styleable name="RoundProgressBar">
        <attr name="roundColor" format="color"/>
        <attr name="roundProgressColor" format="color"/>
        <attr name="roundWidth" format="dimension"></attr>
        <attr name="textColor" format="color"/>
        <attr name="textSize" format="dimension"/>
        <attr name="text" format="string"/>
        <attr name="max" format="integer"></attr>
        <attr name="textIsDisplayable" format="boolean"></attr>
        <attr name="style">
            <enum name="STROKE" value="0"></enum>
            <enum name="FILL" value="1"></enum>
        </attr>
    </declare-styleable>
```
在布局文件中使用
```javascript
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="wechatedit.com.costomview.MainActivity">

    <wechatedit.com.costomview.roundview.RoundProgressBar
        android:id="@+id/roundbar"
        android:layout_width="80dp"
        android:layout_height="80dp"
        android:layout_alignParentRight="true"
        app:layout_constraintBottom_toBottomOf="@+id/activity_main"
        app:layout_constraintLeft_toLeftOf="@+id/activity_main"
        app:layout_constraintRight_toRightOf="@+id/activity_main"
        app:layout_constraintTop_toTopOf="@+id/activity_main"
        app:text="跳过"
        app:textSize="18sp"/>

</android.support.constraint.ConstraintLayout>
```

最后是在代码中使用该控件：
```javascript
    @butterknife.Bind(R.id.roundbar)
    RoundProgressBar roundbar;
    
     roundbar.start();

     roundbar.setOnClickListener(new View.OnClickListener() {
         @Override
         public void onClick(View view) {
             roundbar.stop();
             Toast.makeText(MainActivity.this, "This button is clicked.", Toast.LENGTH_SHORT).show();
         }
     });
```

二、“圆形图片”进度条  基本同“跳过”按钮
1.自定义属性
```javascript
 <!--圆形Img进度条-->
    <declare-styleable name="AroundCircleView">
        <attr name="edgeColor" format="color"/>
        <attr name="edgeSize" format="dimension"/>
        <attr name="edgeBgColor" format="color"/>
    </declare-styleable>
```

2.自定义View[点击查看](https://github.com/ZQiang94/StudyRecords/blob/master/costomview/src/main/java/wechatedit/com/costomview/roundview/AroundCircleView.java)

3.布局文中使用[查看具体代码](https://github.com/ZQiang94/StudyRecords/blob/master/costomview/src/main/res/layout/activity_main.xml)
```javascript
<wechatedit.com.costomview.roundview.AroundCircleView
        android:id="@+id/acv_icon"
        android:layout_width="120dp"
        android:layout_height="120dp"
        android:layout_marginTop="10dp"
        android:src="@mipmap/github"
        app:edgeBgColor="@color/withe"
        app:edgeColor="#9cff0000"
        app:edgeSize="8dp"/>
```
4.代码中使用[具体代码](https://github.com/ZQiang94/StudyRecords/blob/master/costomview/src/main/java/wechatedit/com/costomview/MainActivity.java)
```javascript
 //----------圆形图片进度条 初始化----------
    @butterknife.Bind(R.id.acv_icon)
    AroundCircleView acvIcon;
    private int progress;
    private boolean falg = true;
    Handler weakHandler = new Handler(new Handler.Callback() {
        @Override
        public boolean handleMessage(Message msg) {

            if (msg.what == 1) {
                acvIcon.setProgress(progress);
            }
            if (progress == 100) {
                Toast.makeText(MainActivity.this, "end", Toast.LENGTH_SHORT).show();
            }
            return false;
        }
    });


//start 开启
 private void startImgProgressBar() {
        progress = 0;
        acvIcon.setProgress(progress);
        new Thread(new Runnable() {
            @Override
            public void run() {
                while (falg && progress <= 100) {
                    SystemClock.sleep(100);
                    progress += 1;
                    weakHandler.sendEmptyMessage(1);
                    if (progress > 100) {
                        weakHandler.sendEmptyMessage(2);
                    }
                }
            }
        }).start();

    }

```
