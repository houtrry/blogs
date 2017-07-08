Android的TextView有默认的paddingTop和paddingBottom, 导致跟UI的设计有出入. 
最后在网上找到几种方法
1. 设置marginTop和marginBottom为负值, 比如marginTop="-3dp"
这种方法无法确定这个负值具体是多少, 实际项目中很少用到.
2. 设置android:includeFontPadding="false"
3. 使用TextViewWithoutPaddings
```
public class NoPaddingTextView extends AppCompatTextView {

    private int mAdditionalPadding;

    public NoPaddingTextView(Context context) {
        super(context);
        init();
    }

    public NoPaddingTextView(Context context, AttributeSet attrs) {
        super(context, attrs);
        init();
    }

    private void init() {
        setIncludeFontPadding(false);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        int yOff = -mAdditionalPadding / 6;
        canvas.translate(0, yOff);
        super.onDraw(canvas);
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        getAdditionalPadding();

        int mode = MeasureSpec.getMode(heightMeasureSpec);
        if (mode != MeasureSpec.EXACTLY) {
            int measureHeight = measureHeight(getText().toString(), widthMeasureSpec);

            int height = measureHeight - mAdditionalPadding;
            height += getPaddingTop() + getPaddingBottom();
            heightMeasureSpec = MeasureSpec.makeMeasureSpec(height, MeasureSpec.EXACTLY);
        }
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    }

    private int measureHeight(String text, int widthMeasureSpec) {
        float textSize = getTextSize();

        TextView textView = new TextView(getContext());
        textView.setTextSize(TypedValue.COMPLEX_UNIT_PX, textSize);
        textView.setText(text);
        textView.measure(widthMeasureSpec, 0);
        return textView.getMeasuredHeight();
    }

    private int getAdditionalPadding() {
        float textSize = getTextSize();

        TextView textView = new TextView(getContext());
        textView.setTextSize(TypedValue.COMPLEX_UNIT_PX, textSize);
        textView.setLines(1);
        textView.measure(0, 0);
        int measuredHeight = textView.getMeasuredHeight();
        if (measuredHeight - textSize > 0) {
            mAdditionalPadding = (int) (measuredHeight - textSize);
            Log.v("NoPaddingTextView", "onMeasure: height=" + measuredHeight + " textSize=" + textSize + " mAdditionalPadding=" + mAdditionalPadding);
        }
        return mAdditionalPadding;
    }
}
```
**4. 使用NoPaddingTextView**
```
public class NoPaddingTextView extends AppCompatTextView {

    private int mAdditionalPadding;

    public NoPaddingTextView(Context context) {
        super(context);
        init();
    }

    public NoPaddingTextView(Context context, AttributeSet attrs) {
        super(context, attrs);
        init();
    }

    private void init() {
        setIncludeFontPadding(false);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        int yOff = -mAdditionalPadding / 6;
        canvas.translate(0, yOff);
        super.onDraw(canvas);
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        getAdditionalPadding();

        int mode = MeasureSpec.getMode(heightMeasureSpec);
        if (mode != MeasureSpec.EXACTLY) {
            int measureHeight = measureHeight(getText().toString(), widthMeasureSpec);

            int height = measureHeight - mAdditionalPadding;
            height += getPaddingTop() + getPaddingBottom();
            heightMeasureSpec = MeasureSpec.makeMeasureSpec(height, MeasureSpec.EXACTLY);
        }
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    }

    private int measureHeight(String text, int widthMeasureSpec) {
        float textSize = getTextSize();

        TextView textView = new TextView(getContext());
        textView.setTextSize(TypedValue.COMPLEX_UNIT_PX, textSize);
        textView.setText(text);
        textView.measure(widthMeasureSpec, 0);
        return textView.getMeasuredHeight();
    }

    private int getAdditionalPadding() {
        float textSize = getTextSize();

        TextView textView = new TextView(getContext());
        textView.setTextSize(TypedValue.COMPLEX_UNIT_PX, textSize);
        textView.setLines(1);
        textView.measure(0, 0);
        int measuredHeight = textView.getMeasuredHeight();
        if (measuredHeight - textSize > 0) {
            mAdditionalPadding = (int) (measuredHeight - textSize);
            Log.v("NoPaddingTextView", "onMeasure: height=" + measuredHeight + " textSize=" + textSize + " mAdditionalPadding=" + mAdditionalPadding);
        }
        return mAdditionalPadding;
    }
}
```
下面, 对方法2/3/4进行测试, 测试结果如下, 

![](http://i.imgur.com/rMz4VtT.jpg)

其中第一行没有做任何处理, 第二三四行分别对应方法2/3/4


由结果可知, **方法4最接近想要的效果, 方法2对TextView有一定程度的优化, 但离目标还挺远.方法3文字的底部有一部分未完整显示**.

测试布局文件如下
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:orientation="vertical"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:gravity="center"
    android:background="#AA00FF00">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="我们中国人"
        android:textSize="60sp"
        android:textAllCaps="false"
        android:layout_marginTop="10dp"
        android:textColor="@color/colorBlack"
        android:background="@color/blue_text_color"/>

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="我们中国人"
        android:textSize="60sp"
        android:textAllCaps="false"
        android:layout_marginBottom="10dp"
        android:layout_marginTop="10dp"
        android:includeFontPadding="false"
        android:textColor="@color/colorBlack"
        android:background="@color/blue_text_color"/>

    <com.het.myroatedemo.ui.widget.TextViewWithoutPaddings
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="我们中国人"
        android:textSize="60sp"
        android:textAllCaps="false"
        android:textColor="@color/colorBlack"
        android:background="@color/blue_text_color"/>

    <com.het.myroatedemo.ui.widget.NoPaddingTextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="我们中国人"
        android:textSize="60sp"
        android:textAllCaps="false"
        android:layout_marginTop="10dp"
        android:textColor="@color/colorBlack"
        android:background="@color/blue_text_color"/>
</LinearLayout>
```


参考链接:

https://stackoverflow.com/questions/6593885/how-to-remove-the-top-and-bottom-space-on-textview-of-android
https://stackoverflow.com/questions/4768738/android-textview-remove-spacing-and-padding-on-top-and-bottom
    

