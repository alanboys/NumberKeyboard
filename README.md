# NumberKeyboard
## 自定义键盘
>1、布局：
>
	(1)、宫格：我们可以将这个布局看成是宫格布局，然后需要计算出每个小宫格在屏幕中的位置（坐标），然后再用canvas画出相应的矩形即可。
	(2)、数字：我们需要计算出每个小宫格中的中心点位置（坐标），然后用canvas画数字上去，当然，我们知道最后一个是删除键，并不是数字，我们可以准备一张图片，将图片画到相应的位置。
>
>2、用户动作：
>
	(1)、按下：用户每一次按下的时候就表示这一次动作的开始，所以首先要将各种标识位（自定义所需要的标识位）设置成初始状态，然后需要记录按下的坐标，然后计算出用户按下的坐标与宫格中哪个点相对应，在记录相应数字。最后刷新布局
	(2)、抬起：用户抬起的时候需要刷新布局，然后将按下过程中记录下的数字或者删除键信息通过接口的形式回调给用户，最后恢复标识位
	(3)、取消：将所有的标识位恢复成初始状态。

好了，思路就讲到这里，我们来看看onDraw方法：

整体代码

import android.annotation.SuppressLint;
import android.annotation.TargetApi;
import android.content.Context;
import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.graphics.Canvas;
import android.graphics.Color;
import android.graphics.Paint;
import android.graphics.RectF;
import android.os.Build;
import android.util.AttributeSet;
import android.util.Log;
import android.view.MotionEvent;
import android.view.View;

import com.share.jack.numberkeyboard.R;

/**
 * Created by Jack on 16/10/13
 */
public class NumberKeyboardView extends View {

    private Paint mPaint;
    private Bitmap mBpDelete;
    private float clickX, clickY;   //点击时的x,y坐标
    private float mWidth, mHeight;   //屏幕的宽高
    private float mRectWidth, mRectHeight;   //单个按键的宽高
    private float mWidthOfBp, mHeightOfBp;
    private boolean isInit = false;   //view是否已经初始化
    private String number;//点击的数字
    private float[] xs = new float[6];//声明数组保存每一列的矩形中心的横坐标
    private float[] ys = new float[8];//声明数组保存每一排的矩形中心的纵坐标
    private OnNumberClickListener onNumberClickListener;
    private float x1, y1, x2, y2;  //按下的时候所处的矩形的左上和右下的坐标
    private boolean isBigWrite = true;

    /**
     * 判断刷新数据
     * -1 不进行数据刷新
     * 0  按下刷新
     * 1  弹起刷新
     */
    private int type = -1;

    public NumberKeyboardView(Context context) {
        super(context);
    }

    public NumberKeyboardView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public NumberKeyboardView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    public OnNumberClickListener getOnNumberClickListener() {
        return onNumberClickListener;
    }

    public void setOnNumberClickListener(OnNumberClickListener onNumberClickListener) {
        this.onNumberClickListener = onNumberClickListener;
    }

    @SuppressLint("DrawAllocation")
    @TargetApi(Build.VERSION_CODES.LOLLIPOP)
    @Override
    protected void onDraw(Canvas canvas) {
        if (!isInit) {
            initData();
        }
        //画键盘
        drawKeyboard(canvas);

        //判断是否点击数字
        if (clickX > 0 && clickY > 0) {
            if (type == 0) {  //按下刷新

                if ("delete".equals(number)) {
                    mPaint.setColor(Color.GRAY);
                    canvas.drawRoundRect(new RectF(60 + 5 * mRectWidth, mHeight / 2 + 10, 60 + 6 * mRectWidth, mHeight / 2 + 140 + mRectHeight), 10, 10, mPaint);
                    mPaint.setColor(Color.BLACK);
                    mPaint.setTextSize(30);// 设置字体大小
                    mPaint.setStrokeWidth(2);
                    canvas.drawText("删", xs[5], ys[0], mPaint);
                    canvas.drawText("除", xs[5], ys[1], mPaint);
                } else if ("big".equals(number)) {
                    mPaint.setColor(Color.GRAY);
                    canvas.drawRoundRect(new RectF(60 + 5 * mRectWidth, mHeight / 2 + 50 + 4 * mRectHeight, 60 + 6 * mRectWidth, mHeight / 2 + 180 + 5 * mRectHeight), 10, 10, mPaint);
                    mPaint.setColor(Color.BLACK);
                    mPaint.setTextSize(30);// 设置字体大小
                    mPaint.setStrokeWidth(2);
                    if(isBigWrite){
                        canvas.drawText("大", xs[5], ys[4], mPaint);
                        canvas.drawText("写", xs[5], ys[5], mPaint);
                    }else {
                        canvas.drawText("小", xs[5], ys[4], mPaint);
                        canvas.drawText("写", xs[5], ys[5], mPaint);
                    }
                } else if ("empty".equals(number)) {
                    mPaint.setColor(Color.GRAY);
                    canvas.drawRoundRect(new RectF(60 + 5 * mRectWidth, mHeight / 2 + 30 + 2 * mRectHeight, 60 + 6 * mRectWidth, mHeight / 2 + 160 + 3 * mRectHeight), 10, 10, mPaint);
                    mPaint.setColor(Color.BLACK);
                    mPaint.setTextSize(30);// 设置字体大小
                    mPaint.setStrokeWidth(2);
                    canvas.drawText("清", xs[5], ys[2], mPaint);
                    canvas.drawText("空", xs[5], ys[3], mPaint);
                } else {
                    mPaint.setColor(Color.GRAY);
                    canvas.drawRoundRect(new RectF(x1, y1, x2, y2), 10, 10, mPaint);
                    mPaint.setColor(Color.BLACK);
                    mPaint.setTextSize(60);// 设置字体大小
                    mPaint.setStrokeWidth(2);
                    canvas.drawText(number, clickX, clickY, mPaint);
                }
            } else if (type == 1) {  //抬起刷新
                if ("delete".equals(number)) {
                    mPaint.setColor(Color.WHITE);
                    canvas.drawRoundRect(new RectF(x1, y1, x2, y2), 10, 10, mPaint);
                    mPaint.setColor(Color.BLACK);
                    mPaint.setTextSize(30);// 设置字体大小
                    mPaint.setStrokeWidth(2);
                    canvas.drawText("删", xs[5], ys[0], mPaint);
                    canvas.drawText("除", xs[5], ys[1], mPaint);
//                    canvas.drawBitmap(mBpDelete, xs[2] - mWidthOfBp / 2 + 10, ys[3] - mHeightOfBp / 2 - 10, mPaint);
                } else if ("big".equals(number)) {
                    mPaint.setColor(Color.WHITE);
                    canvas.drawRoundRect(new RectF(x1, y1, x2, y2), 10, 10, mPaint);
                    mPaint.setColor(Color.BLACK);
                    mPaint.setTextSize(30);// 设置字体大小
                    mPaint.setStrokeWidth(2);
                    if(isBigWrite){
                        canvas.drawText("大", xs[5], ys[4], mPaint);
                        canvas.drawText("写", xs[5], ys[5], mPaint);
                    }else {
                        canvas.drawText("小", xs[5], ys[4], mPaint);
                        canvas.drawText("写", xs[5], ys[5], mPaint);
                    }
                } else if ("empty".equals(number)) {
                    mPaint.setColor(Color.WHITE);
                    canvas.drawRoundRect(new RectF(x1, y1, x2, y2), 10, 10, mPaint);
                    mPaint.setColor(Color.BLACK);
                    mPaint.setTextSize(30);// 设置字体大小
                    mPaint.setStrokeWidth(2);
                    canvas.drawText("清", xs[5], ys[2], mPaint);
                    canvas.drawText("空", xs[5], ys[3], mPaint);

                } else {
                    mPaint.setColor(Color.WHITE);
                    canvas.drawRoundRect(new RectF(x1, y1, x2, y2), 10, 10, mPaint);
                    mPaint.setColor(Color.BLACK);
                    mPaint.setTextSize(60);// 设置字体大小
                    mPaint.setStrokeWidth(2);
                    canvas.drawText(number, clickX, clickY, mPaint);
                }
            }
            //绘制完成后,重置
            clickX = 0;
            clickY = 0;
        }
    }

    private void initData() {
        mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        mWidth = getWidth();
        mHeight = getHeight();
        mBpDelete = BitmapFactory.decodeResource(getResources(), R.mipmap.keyboard_backspace);
        mWidthOfBp = mBpDelete.getWidth();
        mHeightOfBp = mBpDelete.getHeight();

        mRectWidth = (mWidth - 70) / 6;   //控制列数
        mRectHeight = (mHeight - 100) / 14;//控制排数 没加一排数字加2

        //一排有几列 每列的宽度
        xs[0] = mRectWidth / 2;
        xs[1] = (mRectWidth * 3) / 2 + 10;
        xs[2] = (mRectWidth * 5) / 2 + 20;
        xs[3] = (mRectWidth * 7) / 2 + 30;
        xs[4] = (mRectWidth * 9) / 2 + 40;
        xs[5] = (mRectWidth * 11) / 2 + 50;
         //一共有几排 每排的高度
        ys[0] = mRectHeight / 2 + 25 + mHeight / 2;
        ys[1] = (mRectHeight * 3) / 2 + 35 + mHeight / 2;
        ys[2] = (mRectHeight * 5) / 2 + 45 + mHeight / 2;
        ys[3] = (mRectHeight * 7) / 2 + 55 + mHeight / 2;
        ys[4] = (mRectHeight * 9) / 2 + 65 + mHeight / 2;
        ys[5] = (mRectHeight * 11) / 2 + 75 + mHeight / 2;
        ys[6] = (mRectHeight * 13) / 2 + 85 + mHeight / 2;

        isInit = true;
    }

    /**
     * drawRoundRect(float left, float top, float right, float bottom, float rx, float ry, Paint paint)这种方式在5.0以下的机器上会报错，
     * 需要换成drawRoundRect(RectF rect, float rx, float ry, Paint paint)
     *
     * @param canvas
     */

    @TargetApi(Build.VERSION_CODES.LOLLIPOP)
    private void drawKeyboard(Canvas canvas) {

        mPaint.setColor(Color.WHITE);
        //画宫格
        //第一排
        canvas.drawRoundRect(new RectF(10, mHeight / 2 + 10, 10 + mRectWidth, mHeight / 2 + 10 + mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(20 + mRectWidth, mHeight / 2 + 10, 20 + 2 * mRectWidth, mHeight / 2 + 10 + mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(30 + 2 * mRectWidth, mHeight / 2 + 10, 30 + 3 * mRectWidth, mHeight / 2 + 10 + mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(40 + 3 * mRectWidth, mHeight / 2 + 10, 40 + 4 * mRectWidth, mHeight / 2 + 10 + mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(50 + 4 * mRectWidth, mHeight / 2 + 10, 50 + 5 * mRectWidth, mHeight / 2 + 10 + mRectHeight), 10, 10, mPaint);


        //第二排
        canvas.drawRoundRect(new RectF(10, mHeight / 2 + 20 + mRectHeight, 10 + mRectWidth, mHeight / 2 + 20 + 2 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(20 + mRectWidth, mHeight / 2 + 20 + mRectHeight, 20 + 2 * mRectWidth, mHeight / 2 + 20 + 2 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(30 + 2 * mRectWidth, mHeight / 2 + 20 + mRectHeight, 30 + 3 * mRectWidth, mHeight / 2 + 20 + 2 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(40 + 3 * mRectWidth, mHeight / 2 + 20 + mRectHeight, 40 + 4 * mRectWidth, mHeight / 2 + 20 + 2 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(50 + 4 * mRectWidth, mHeight / 2 + 20 + mRectHeight, 50 + 5 * mRectWidth, mHeight / 2 + 20 + 2 * mRectHeight), 10, 10, mPaint);


        //第三排
        canvas.drawRoundRect(new RectF(10, mHeight / 2 + 30 + 2 * mRectHeight, 10 + mRectWidth, mHeight / 2 + 30 + 3 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(20 + mRectWidth, mHeight / 2 + 30 + 2 * mRectHeight, 20 + 2 * mRectWidth, mHeight / 2 + 30 + 3 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(30 + 2 * mRectWidth, mHeight / 2 + 30 + 2 * mRectHeight, 30 + 3 * mRectWidth, mHeight / 2 + 30 + 3 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(40 + 3 * mRectWidth, mHeight / 2 + 30 + 2 * mRectHeight, 40 + 4 * mRectWidth, mHeight / 2 + 30 + 3 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(50 + 4 * mRectWidth, mHeight / 2 + 30 + 2 * mRectHeight, 50 + 5 * mRectWidth, mHeight / 2 + 30 + 3 * mRectHeight), 10, 10, mPaint);

        //第四排
        canvas.drawRoundRect(new RectF(10, mHeight / 2 + 40 + 3 * mRectHeight, 10 + mRectWidth, mHeight / 2 + 40 + 4 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(20 + mRectWidth, mHeight / 2 + 40 + 3 * mRectHeight, 20 + 2 * mRectWidth, mHeight / 2 + 40 + 4 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(30 + 2 * mRectWidth, mHeight / 2 + 40 + 3 * mRectHeight, 30 + 3 * mRectWidth, mHeight / 2 + 40 + 4 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(40 + 3 * mRectWidth, mHeight / 2 + 40 + 3 * mRectHeight, 40 + 4 * mRectWidth, mHeight / 2 + 40 + 4 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(50 + 4 * mRectWidth, mHeight / 2 + 40 + 3 * mRectHeight, 50 + 5 * mRectWidth, mHeight / 2 + 40 + 4 * mRectHeight), 10, 10, mPaint);

        //第五排
        canvas.drawRoundRect(new RectF(10, mHeight / 2 + 50 + 4 * mRectHeight, 10 + mRectWidth, mHeight / 2 + 50 + 5 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(20 + mRectWidth, mHeight / 2 + 50 + 4 * mRectHeight, 20 + 2 * mRectWidth, mHeight / 2 + 50 + 5 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(30 + 2 * mRectWidth, mHeight / 2 + 50 + 4 * mRectHeight, 30 + 3 * mRectWidth, mHeight / 2 + 50 + 5 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(40 + 3 * mRectWidth, mHeight / 2 + 50 + 4 * mRectHeight, 40 + 4 * mRectWidth, mHeight / 2 + 50 + 5 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(50 + 4 * mRectWidth, mHeight / 2 + 50 + 4 * mRectHeight, 50 + 5 * mRectWidth, mHeight / 2 + 50 + 5 * mRectHeight), 10, 10, mPaint);

        //第六排
        canvas.drawRoundRect(new RectF(10, mHeight / 2 + 60 + 5 * mRectHeight, 10 + mRectWidth, mHeight / 2 + 60 + 6 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(20 + mRectWidth, mHeight / 2 + 60 + 5 * mRectHeight, 20 + 2 * mRectWidth, mHeight / 2 + 60 + 6 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(30 + 2 * mRectWidth, mHeight / 2 + 60 + 5 * mRectHeight, 30 + 3 * mRectWidth, mHeight / 2 + 60 + 6 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(40 + 3 * mRectWidth, mHeight / 2 + 60 + 5 * mRectHeight, 40 + 4 * mRectWidth, mHeight / 2 + 60 + 6 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(50 + 4 * mRectWidth, mHeight / 2 + 60 + 5 * mRectHeight, 50 + 5 * mRectWidth, mHeight / 2 + 60 + 6 * mRectHeight), 10, 10, mPaint);


        //第七排
        canvas.drawRoundRect(new RectF(10, mHeight / 2 + 70 + 6 * mRectHeight, 10 + mRectWidth, mHeight / 2 + 70 + 7 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(20 + mRectWidth, mHeight / 2 + 70 + 6 * mRectHeight, 20 + 2 * mRectWidth, mHeight / 2 + 70 + 7 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(30 + 2 * mRectWidth, mHeight / 2 + 70 + 6 * mRectHeight, 30 + 3 * mRectWidth, mHeight / 2 + 70 + 7 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(40 + 3 * mRectWidth, mHeight / 2 + 70 + 6 * mRectHeight, 40 + 4 * mRectWidth, mHeight / 2 + 70 + 7 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(50 + 4 * mRectWidth, mHeight / 2 + 70 + 6 * mRectHeight, 50 + 5 * mRectWidth, mHeight / 2 + 70 + 7 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(60 + 5 * mRectWidth, mHeight / 2 + 70 + 6 * mRectHeight, 60 + 6 * mRectWidth, mHeight / 2 + 70 + 7 * mRectHeight), 10, 10, mPaint);

        /*删除 清空 大写 */
        canvas.drawRoundRect(new RectF(60 + 5 * mRectWidth, mHeight / 2 + 10, 60 + 6 * mRectWidth, mHeight / 2 + 140 + mRectHeight), 10, 10, mPaint);
//        canvas.drawRoundRect(new RectF(60 + 5 * mRectWidth, mHeight / 2 + 20 + mRectHeight, 60 + 6 * mRectWidth, mHeight / 2 + 20 + 2 * mRectHeight), 10, 10, mPaint2);

        canvas.drawRoundRect(new RectF(60 + 5 * mRectWidth, mHeight / 2 + 30 + 2 * mRectHeight, 60 + 6 * mRectWidth, mHeight / 2 + 160 + 3 * mRectHeight), 10, 10, mPaint);
//        canvas.drawRoundRect(new RectF(60 + 5 * mRectWidth, mHeight / 2 + 40 + 3 * mRectHeight, 60 + 6 * mRectWidth, mHeight / 2 + 40 + 4 * mRectHeight), 10, 10, mPaint2);
        canvas.drawRoundRect(new RectF(60 + 5 * mRectWidth, mHeight / 2 + 50 + 4 * mRectHeight, 60 + 6 * mRectWidth, mHeight / 2 + 180 + 5 * mRectHeight), 10, 10, mPaint);
//        canvas.drawRoundRect(new RectF(60 + 5 * mRectWidth, mHeight / 2 + 60 + 5 * mRectHeight, 60 + 6 * mRectWidth, mHeight / 2 + 60 + 6 * mRectHeight), 10, 10, mPaint2);

        mPaint.setColor(Color.BLACK);
        mPaint.setTextSize(60);// 设置字体大小
        mPaint.setStrokeWidth(2);
        //画数字
        //第一排
        canvas.drawText("1", xs[0], ys[0], mPaint);
        canvas.drawText("2", xs[1], ys[0], mPaint);
        canvas.drawText("3", xs[2], ys[0], mPaint);
        canvas.drawText("4", xs[3], ys[0], mPaint);
        canvas.drawText("5", xs[4], ys[0], mPaint);
        //第二排
        canvas.drawText("6", xs[0], ys[1], mPaint);
        canvas.drawText("7", xs[1], ys[1], mPaint);
        canvas.drawText("8", xs[2], ys[1], mPaint);
        canvas.drawText("9", xs[3], ys[1], mPaint);
        canvas.drawText("0", xs[4], ys[1], mPaint);
        //第三排
        canvas.drawText(isBigWrite ? "a" : "A", xs[0], ys[2], mPaint);
        canvas.drawText(isBigWrite ? "b" : "B", xs[1], ys[2], mPaint);
        canvas.drawText(isBigWrite ? "c" : "C", xs[2], ys[2], mPaint);
        canvas.drawText(isBigWrite ? "d" : "D", xs[3], ys[2], mPaint);
        canvas.drawText(isBigWrite ? "e" : "E", xs[4], ys[2], mPaint);
        //第四排
        canvas.drawText(isBigWrite ? "f" : "F", xs[0], ys[3], mPaint);
        canvas.drawText(isBigWrite ? "g" : "G", xs[1], ys[3], mPaint);
        canvas.drawText(isBigWrite ? "h" : "H", xs[2], ys[3], mPaint);
        canvas.drawText(isBigWrite ? "i" : "I", xs[3], ys[3], mPaint);
        canvas.drawText(isBigWrite ? "j" : "J", xs[4], ys[3], mPaint);
        //第五排
        canvas.drawText(isBigWrite ? "k" : "K", xs[0], ys[4], mPaint);
        canvas.drawText(isBigWrite ? "l" : "L", xs[1], ys[4], mPaint);
        canvas.drawText(isBigWrite ? "m" : "M", xs[2], ys[4], mPaint);
        canvas.drawText(isBigWrite ? "n" : "N", xs[3], ys[4], mPaint);
        canvas.drawText(isBigWrite ? "o" : "O", xs[4], ys[4], mPaint);
        //第六排
        canvas.drawText(isBigWrite ? "p" : "P", xs[0], ys[5], mPaint);
        canvas.drawText(isBigWrite ? "q" : "Q", xs[1], ys[5], mPaint);
        canvas.drawText(isBigWrite ? "r" : "R", xs[2], ys[5], mPaint);
        canvas.drawText(isBigWrite ? "s" : "S", xs[3], ys[5], mPaint);
        canvas.drawText(isBigWrite ? "t" : "T", xs[4], ys[5], mPaint);
        //第七排
        canvas.drawText(isBigWrite ? "u" : "U", xs[0], ys[6], mPaint);
        canvas.drawText(isBigWrite ? "v" : "V", xs[1], ys[6], mPaint);
        canvas.drawText(isBigWrite ? "w" : "W", xs[2], ys[6], mPaint);
        canvas.drawText(isBigWrite ? "x" : "X", xs[3], ys[6], mPaint);
        canvas.drawText(isBigWrite ? "y" : "Y", xs[4], ys[6], mPaint);
        canvas.drawText(isBigWrite ? "z" : "Z", xs[5], ys[6], mPaint);

        mPaint.setColor(Color.BLACK);
        mPaint.setTextSize(30);// 设置字体大小
        mPaint.setStrokeWidth(2);
	//其实是将两个字放到两个宫格的位置
        canvas.drawText("删", xs[5], ys[0], mPaint);
        canvas.drawText("除", xs[5], ys[1], mPaint);
        canvas.drawText("清", xs[5], ys[2], mPaint);
        canvas.drawText("空", xs[5], ys[3], mPaint);
	//控制大小写切换
        if(isBigWrite){
            canvas.drawText("大", xs[5], ys[4], mPaint);
            canvas.drawText("写", xs[5], ys[5], mPaint);
        }else {
            canvas.drawText("小", xs[5], ys[4], mPaint);
            canvas.drawText("写", xs[5], ys[5], mPaint);
        }



    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        float x = event.getX();
        float y = event.getY();
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN: //按下
                setDefault();
                handleDown(x, y);
                return true;
            case MotionEvent.ACTION_UP: //弹起
                type = 1;//弹起刷新
                invalidate();//刷新界面
                //一次按下结束,返回点击的数字
                if (onNumberClickListener != null) {
                    if (number != null) {
                        if (number.equals("delete")) {
                            onNumberClickListener.onNumberDelete();
                        } else if (number.equals("empty")) {
                            onNumberClickListener.onNumberEmpty();
                        } else if (number.equals("big")) {
                            if (isBigWrite) {
                                isBigWrite = false;
                            } else {
                                isBigWrite = true;
                            }
                            invalidate();//大小写切换时刷新页面
                        } else {
                            onNumberClickListener.onNumberReturn(number);
                        }
                    }
                }
                //恢复默认
                setDefault();
                return true;
            case MotionEvent.ACTION_CANCEL:  //取消
                //恢复默认值
                setDefault();
                return true;
            default:
                break;
        }
        return false;
    }

    private void setDefault() {
        clickX = 0;
        clickY = 0;
        number = null;
        type = -1;
    }

   //将每个位置赋值为要输入的数字、字母
    private void handleDown(float x, float y) {
        if (y < mHeight / 2) {
            return;
        }
        if (x >= 10 && x <= 10 + mRectWidth) {   //第一列
            clickX = xs[0];
            if (y >= mHeight / 2 + 10 && y <= mHeight / 2 + 10 + mRectHeight) {  //第一排(1)
                clickY = ys[0];
                x1 = 10;
                y1 = mHeight / 2 + 10;
                x2 = 10 + mRectWidth;
                y2 = mHeight / 2 + 10 + mRectHeight;
                number = "1";
            } else if (y >= mHeight / 2 + 20 + mRectHeight && y <= mHeight / 2 + 20 + 2 * mRectHeight) {  //第二排(4)
                x1 = 10;
                y1 = mHeight / 2 + 20 + mRectHeight;
                x2 = 10 + mRectWidth;
                y2 = mHeight / 2 + 20 + 2 * mRectHeight;
                clickY = ys[1];
                number = "6";
            } else if (y >= mHeight / 2 + 30 + 2 * mRectHeight && y <= mHeight / 2 + 30 + 3 * mRectHeight) {  //第三排(7)
                x1 = 10;
                y1 = mHeight / 2 + 30 + 2 * mRectHeight;
                x2 = 10 + mRectWidth;
                y2 = mHeight / 2 + 30 + 3 * mRectHeight;
                clickY = ys[2];
                number = isBigWrite ? "a" : "A";
            } else if (y >= mHeight / 2 + 40 + 3 * mRectHeight && y <= mHeight / 2 + 40 + 4 * mRectHeight) { //第四排(0)
                x1 = 10;
                y1 = mHeight / 2 + 40 + 3 * mRectHeight;
                x2 = 10 + mRectWidth;
                y2 = mHeight / 2 + 40 + 4 * mRectHeight;
                clickY = ys[3];
                number = isBigWrite ? "f" : "F";
            } else if (y >= mHeight / 2 + 50 + 4 * mRectHeight && y <= mHeight / 2 + 50 + 5 * mRectHeight) { //第四排(0)
                x1 = 10;
                y1 = mHeight / 2 + 50 + 4 * mRectHeight;
                x2 = 10 + mRectWidth;
                y2 = mHeight / 2 + 50 + 5 * mRectHeight;
                clickY = ys[4];
                number = isBigWrite ? "k" : "K";
            } else if (y >= mHeight / 2 + 60 + 5 * mRectHeight && y <= mHeight / 2 + 60 + 6 * mRectHeight) { //第四排(0)
                x1 = 10;
                y1 = mHeight / 2 + 60 + 5 * mRectHeight;
                x2 = 10 + mRectWidth;
                y2 = mHeight / 2 + 60 + 6 * mRectHeight;
                clickY = ys[5];
                number = isBigWrite ? "p" : "P";
            } else if (y >= mHeight / 2 + 70 + 6 * mRectHeight && y <= mHeight / 2 + 70 + 7 * mRectHeight) { //第四排(0)
                x1 = 10;
                y1 = mHeight / 2 + 70 + 6 * mRectHeight;
                x2 = 10 + mRectWidth;
                y2 = mHeight / 2 + 70 + 7 * mRectHeight;
                clickY = ys[6];
                number = isBigWrite ? "u" : "U";
            }
        } else if (x >= 20 + mRectWidth && x <= 20 + 2 * mRectWidth) {  //第二列
            clickX = xs[1];
            if (y >= mHeight / 2 + 10 && y <= mHeight / 2 + 10 + mRectHeight) {  //第一排(2)
                x1 = 20 + mRectWidth;
                y1 = mHeight / 2 + 10;
                x2 = 20 + 2 * mRectWidth;
                y2 = mHeight / 2 + 10 + mRectHeight;
                clickY = ys[0];
                number = "2";
            } else if (y >= mHeight / 2 + 20 + mRectHeight && y <= mHeight / 2 + 20 + 2 * mRectHeight) {  //第二排(5)
                x1 = 20 + mRectWidth;
                y1 = mHeight / 2 + 20 + mRectHeight;
                x2 = 20 + 2 * mRectWidth;
                y2 = mHeight / 2 + 20 + 2 * mRectHeight;
                clickY = ys[1];
                number = "7";
            } else if (y >= mHeight / 2 + 30 + 2 * mRectHeight && y <= mHeight / 2 + 30 + 3 * mRectHeight) {  //第三排(8)
                x1 = 20 + mRectWidth;
                y1 = mHeight / 2 + 30 + 2 * mRectHeight;
                x2 = 20 + 2 * mRectWidth;
                y2 = mHeight / 2 + 30 + 3 * mRectHeight;
                clickY = ys[2];
                number = isBigWrite ? "b" : "B";
            } else if (y >= mHeight / 2 + 40 + 3 * mRectHeight && y <= mHeight / 2 + 40 + 4 * mRectHeight) { //第四排(0)
                x1 = 20 + mRectWidth;
                y1 = mHeight / 2 + 40 + 3 * mRectHeight;
                x2 = 20 + 2 * mRectWidth;
                y2 = mHeight / 2 + 40 + 4 * mRectHeight;
                clickY = ys[3];
                number = isBigWrite ? "g" : "G";
            } else if (y >= mHeight / 2 + 50 + 4 * mRectHeight && y <= mHeight / 2 + 50 + 5 * mRectHeight) { //第五排(0)
                x1 = 20 + mRectWidth;
                y1 = mHeight / 2 + 50 + 4 * mRectHeight;
                x2 = 20 + 2 * mRectWidth;
                y2 = mHeight / 2 + 50 + 5 * mRectHeight;
                clickY = ys[4];
                number = isBigWrite ? "l" : "L";
            } else if (y >= mHeight / 2 + 60 + 5 * mRectHeight && y <= mHeight / 2 + 60 + 6 * mRectHeight) { //第六排(0)
                x1 = 20 + mRectWidth;
                y1 = mHeight / 2 + 60 + 5 * mRectHeight;
                x2 = 20 + 2 * mRectWidth;
                y2 = mHeight / 2 + 60 + 6 * mRectHeight;
                clickY = ys[5];
                number = isBigWrite ? "q" : "Q";
            } else if (y >= mHeight / 2 + 70 + 6 * mRectHeight && y <= mHeight / 2 + 70 + 7 * mRectHeight) { //第七排(0)
                x1 = 20 + mRectWidth;
                y1 = mHeight / 2 + 70 + 6 * mRectHeight;
                x2 = 20 + 2 * mRectWidth;
                y2 = mHeight / 2 + 70 + 7 * mRectHeight;
                clickY = ys[6];
                number = isBigWrite ? "v" : "V";
            }
        } else if (x >= 30 + 2 * mRectWidth && x <= 30 + 3 * mRectWidth) {   //第三列
            clickX = xs[2];
            if (y >= mHeight / 2 + 10 && y <= mHeight / 2 + 10 + mRectHeight) {  //第一排(3)
                x1 = 30 + 2 * mRectWidth;
                y1 = mHeight / 2 + 10;
                x2 = 30 + 3 * mRectWidth;
                y2 = mHeight / 2 + 10 + mRectHeight;
                clickY = ys[0];
                number = "3";
            } else if (y >= mHeight / 2 + 20 + mRectHeight && y <= mHeight / 2 + 20 + 2 * mRectHeight) {  //第二排(6)
                x1 = 30 + 2 * mRectWidth;
                y1 = mHeight / 2 + 20 + mRectHeight;
                x2 = 30 + 3 * mRectWidth;
                y2 = mHeight / 2 + 20 + 2 * mRectHeight;
                clickY = ys[1];
                number = "8";
            } else if (y >= mHeight / 2 + 30 + 2 * mRectHeight && y <= mHeight / 2 + 30 + 3 * mRectHeight) {  //第三排(9)
                x1 = 30 + 2 * mRectWidth;
                y1 = mHeight / 2 + 30 + 2 * mRectHeight;
                x2 = 30 + 3 * mRectWidth;
                y2 = mHeight / 2 + 30 + 3 * mRectHeight;
                clickY = ys[2];
                number = isBigWrite ? "c" : "C";
            } else if (y >= mHeight / 2 + 40 + 3 * mRectHeight && y <= mHeight / 2 + 40 + 4 * mRectHeight) { //第四排()
                x1 = 30 + 2 * mRectWidth;
                y1 = mHeight / 2 + 40 + 3 * mRectHeight;
                x2 = 30 + 3 * mRectWidth;
                y2 = mHeight / 2 + 40 + 4 * mRectHeight;
                clickY = ys[3];
                number = isBigWrite ? "h" : "H";
            } else if (y >= mHeight / 2 + 50 + 4 * mRectHeight && y <= mHeight / 2 + 50 + 5 * mRectHeight) { //第5排()
                x1 = 30 + 2 * mRectWidth;
                y1 = mHeight / 2 + 50 + 4 * mRectHeight;
                x2 = 30 + 3 * mRectWidth;
                y2 = mHeight / 2 + 50 + 5 * mRectHeight;
                clickY = ys[4];
                number = isBigWrite ? "m" : "M";
            } else if (y >= mHeight / 2 + 60 + 5 * mRectHeight && y <= mHeight / 2 + 60 + 6 * mRectHeight) { //第6排()
                x1 = 30 + 2 * mRectWidth;
                y1 = mHeight / 2 + 60 + 5 * mRectHeight;
                x2 = 30 + 3 * mRectWidth;
                y2 = mHeight / 2 + 60 + 6 * mRectHeight;
                clickY = ys[5];
                number = isBigWrite ? "r" : "R";
            } else if (y >= mHeight / 2 + 70 + 6 * mRectHeight && y <= mHeight / 2 + 70 + 7 * mRectHeight) { //第7排()
                x1 = 30 + 2 * mRectWidth;
                y1 = mHeight / 2 + 70 + 6 * mRectHeight;
                x2 = 30 + 3 * mRectWidth;
                y2 = mHeight / 2 + 70 + 7 * mRectHeight;
                clickY = ys[6];
                number = isBigWrite ? "w" : "W";
            }
        } else if (x >= 40 + 3 * mRectWidth && x <= 40 + 4 * mRectWidth) {   //第四列
            clickX = xs[3];
            if (y >= mHeight / 2 + 10 && y <= mHeight / 2 + 10 + mRectHeight) {  //第一排(3)
                x1 = 40 + 3 * mRectWidth;
                y1 = mHeight / 2 + 10;
                x2 = 40 + 4 * mRectWidth;
                y2 = mHeight / 2 + 10 + mRectHeight;
                clickY = ys[0];
                number = "4";
            } else if (y >= mHeight / 2 + 20 + mRectHeight && y <= mHeight / 2 + 20 + 2 * mRectHeight) {  //第二排(6)
                x1 = 40 + 3 * mRectWidth;
                y1 = mHeight / 2 + 20 + mRectHeight;
                x2 = 40 + 4 * mRectWidth;
                y2 = mHeight / 2 + 20 + 2 * mRectHeight;
                clickY = ys[1];
                number = "9";
            } else if (y >= mHeight / 2 + 30 + 2 * mRectHeight && y <= mHeight / 2 + 30 + 3 * mRectHeight) {  //第三排(9)
                x1 = 40 + 3 * mRectWidth;
                y1 = mHeight / 2 + 30 + 2 * mRectHeight;
                x2 = 40 + 4 * mRectWidth;
                y2 = mHeight / 2 + 30 + 3 * mRectHeight;
                clickY = ys[2];
                number = isBigWrite ? "d" : "D";
            } else if (y >= mHeight / 2 + 40 + 3 * mRectHeight && y <= mHeight / 2 + 40 + 4 * mRectHeight) { //第四排()
                x1 = 40 + 3 * mRectWidth;
                y1 = mHeight / 2 + 40 + 3 * mRectHeight;
                x2 = 40 + 4 * mRectWidth;
                y2 = mHeight / 2 + 40 + 4 * mRectHeight;
                clickY = ys[3];
                number = isBigWrite ? "i" : "I";
            } else if (y >= mHeight / 2 + 50 + 4 * mRectHeight && y <= mHeight / 2 + 50 + 5 * mRectHeight) { //第5排()
                x1 = 40 + 3 * mRectWidth;
                y1 = mHeight / 2 + 50 + 4 * mRectHeight;
                x2 = 40 + 4 * mRectWidth;
                y2 = mHeight / 2 + 50 + 5 * mRectHeight;
                clickY = ys[4];
                number = isBigWrite ? "n" : "N";
            } else if (y >= mHeight / 2 + 60 + 5 * mRectHeight && y <= mHeight / 2 + 60 + 6 * mRectHeight) { //第6排()
                x1 = 40 + 3 * mRectWidth;
                y1 = mHeight / 2 + 60 + 5 * mRectHeight;
                x2 = 40 + 4 * mRectWidth;
                y2 = mHeight / 2 + 60 + 6 * mRectHeight;
                clickY = ys[5];
                number = isBigWrite ? "s" : "S";
            } else if (y >= mHeight / 2 + 70 + 6 * mRectHeight && y <= mHeight / 2 + 70 + 7 * mRectHeight) { //第7排()
                x1 = 40 + 3 * mRectWidth;
                y1 = mHeight / 2 + 70 + 6 * mRectHeight;
                x2 = 40 + 4 * mRectWidth;
                y2 = mHeight / 2 + 70 + 7 * mRectHeight;
                clickY = ys[6];
                number = isBigWrite ? "x" : "X";
            }
        } else if (x >= 50 + 4 * mRectWidth && x <= 50 + 5 * mRectWidth) {   //第五列
            clickX = xs[4];
            if (y >= mHeight / 2 + 10 && y <= mHeight / 2 + 10 + mRectHeight) {  //第一排(3)
                x1 = 50 + 4 * mRectWidth;
                y1 = mHeight / 2 + 10;
                x2 = 50 + 5 * mRectWidth;
                y2 = mHeight / 2 + 10 + mRectHeight;
                clickY = ys[0];
                number = "5";
            } else if (y >= mHeight / 2 + 20 + mRectHeight && y <= mHeight / 2 + 20 + 2 * mRectHeight) {  //第二排(6)
                x1 = 50 + 4 * mRectWidth;
                y1 = mHeight / 2 + 20 + mRectHeight;
                x2 = 50 + 5 * mRectWidth;
                y2 = mHeight / 2 + 20 + 2 * mRectHeight;
                clickY = ys[1];
                number = "0";
            } else if (y >= mHeight / 2 + 30 + 2 * mRectHeight && y <= mHeight / 2 + 30 + 3 * mRectHeight) {  //第三排(9)
                x1 = 50 + 4 * mRectWidth;
                y1 = mHeight / 2 + 30 + 2 * mRectHeight;
                x2 = 50 + 5 * mRectWidth;
                y2 = mHeight / 2 + 30 + 3 * mRectHeight;
                clickY = ys[2];
                number = isBigWrite ? "e" : "E";
            } else if (y >= mHeight / 2 + 40 + 3 * mRectHeight && y <= mHeight / 2 + 40 + 4 * mRectHeight) { //第四排()
                x1 = 50 + 4 * mRectWidth;
                y1 = mHeight / 2 + 40 + 3 * mRectHeight;
                x2 = 50 + 5 * mRectWidth;
                y2 = mHeight / 2 + 40 + 4 * mRectHeight;
                clickY = ys[3];
                number = isBigWrite ? "j" : "J";
            } else if (y >= mHeight / 2 + 50 + 4 * mRectHeight && y <= mHeight / 2 + 50 + 5 * mRectHeight) { //第五排()
                x1 = 50 + 4 * mRectWidth;
                y1 = mHeight / 2 + 50 + 4 * mRectHeight;
                x2 = 50 + 5 * mRectWidth;
                y2 = mHeight / 2 + 50 + 5 * mRectHeight;
                clickY = ys[4];
                number = isBigWrite ? "o" : "O";
            } else if (y >= mHeight / 2 + 60 + 5 * mRectHeight && y <= mHeight / 2 + 60 + 6 * mRectHeight) { //第六排()
                x1 = 50 + 4 * mRectWidth;
                y1 = mHeight / 2 + 60 + 5 * mRectHeight;
                x2 = 50 + 5 * mRectWidth;
                y2 = mHeight / 2 + 60 + 6 * mRectHeight;
                clickY = ys[5];
                number = isBigWrite ? "t" : "T";
            } else if (y >= mHeight / 2 + 70 + 6 * mRectHeight && y <= mHeight / 2 + 70 + 7 * mRectHeight) { //第七排()
                x1 = 50 + 4 * mRectWidth;
                y1 = mHeight / 2 + 70 + 6 * mRectHeight;
                x2 = 50 + 5 * mRectWidth;
                y2 = mHeight / 2 + 70 + 7 * mRectHeight;
                clickY = ys[6];
                number = isBigWrite ? "y" : "Y";
            } else if (y >= mHeight / 2 + 80 + 7 * mRectHeight && y <= mHeight / 2 + 80 + 8 * mRectHeight) { //第八排()
                x1 = 50 + 4 * mRectWidth;
                y1 = mHeight / 2 + 80 + 7 * mRectHeight;
                x2 = 50 + 5 * mRectWidth;
                y2 = mHeight / 2 + 80 + 8 * mRectHeight;
                clickY = ys[6];
                number = isBigWrite ? "z" : "Z";
            }
        } else if (x >= 60 + 5 * mRectWidth && x <= 70 + 6 * mRectWidth) {   //第六列
            clickX = xs[5];
            if (y >= mHeight / 2 + 10 && y <= mHeight / 2 + 140 + mRectHeight) {  //第一排(3)
                x1 = 60 + 5 * mRectWidth;
                y1 = mHeight / 2 + 10;
                x2 = 60 + 6 * mRectWidth;
                y2 = mHeight / 2 + 10 + mRectHeight;
                clickY = ys[0];
                number = "delete";
            } else if (y >= mHeight / 2 + 30 + 2 * mRectHeight && y <= mHeight / 2 + 160 + 3 * mRectHeight) {  //第三排(9)
                x1 = 60 + 5 * mRectWidth;
                y1 = mHeight / 2 + 30 + 2 * mRectHeight;
                x2 = 60 + 6 * mRectWidth;
                y2 = mHeight / 2 + 30 + 3 * mRectHeight;
                clickY = ys[2];
                number = "empty";
            } else if (y >= mHeight / 2 + 50 + 4 * mRectHeight && y <= mHeight / 2 + 180 + 5 * mRectHeight) { //第五排()
                x1 = 60 + 5 * mRectWidth;
                y1 = mHeight / 2 + 50 + 4 * mRectHeight;
                x2 = 60 + 6 * mRectWidth;
                y2 = mHeight / 2 + 50 + 5 * mRectHeight;
                clickY = ys[4];
                number = "big";
            } else if (y >= mHeight / 2 + 70 + 6 * mRectHeight && y <= mHeight / 2 + 70 + 7 * mRectHeight) { //第七排()
                x1 = 60 + 5 * mRectWidth;
                y1 = mHeight / 2 + 70 + 6 * mRectHeight;
                x2 = 60 + 6 * mRectWidth;
                y2 = mHeight / 2 + 70 + 7 * mRectHeight;
                clickY = ys[6];
                number = "z";
            }
        }
        type = 0;   //按下刷新
        invalidate();
    }

    public interface OnNumberClickListener {
        //回调点击的数字
        void onNumberReturn(String number);

        //的回调
        void onNumberDelete();

        //的回调
        void onNumberEmpty();
    }
}
package com.share.jack.numberkeyboard.widget;

import android.annotation.SuppressLint;
import android.annotation.TargetApi;
import android.content.Context;
import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.graphics.Canvas;
import android.graphics.Color;
import android.graphics.Paint;
import android.graphics.RectF;
import android.os.Build;
import android.util.AttributeSet;
import android.util.Log;
import android.view.MotionEvent;
import android.view.View;

import com.share.jack.numberkeyboard.R;

/**
 * Created by Jack on 16/10/13
 */
public class NumberKeyboardView extends View {

    private Paint mPaint;
    private Bitmap mBpDelete;
    private float clickX, clickY;   //点击时的x,y坐标
    private float mWidth, mHeight;   //屏幕的宽高
    private float mRectWidth, mRectHeight;   //单个按键的宽高
    private float mWidthOfBp, mHeightOfBp;
    private boolean isInit = false;   //view是否已经初始化
    private String number;//点击的数字
    private float[] xs = new float[6];//声明数组保存每一列的矩形中心的横坐标
    private float[] ys = new float[8];//声明数组保存每一排的矩形中心的纵坐标
    private OnNumberClickListener onNumberClickListener;
    private float x1, y1, x2, y2;  //按下的时候所处的矩形的左上和右下的坐标
    private boolean isBigWrite = true;

    /**
     * 判断刷新数据
     * -1 不进行数据刷新
     * 0  按下刷新
     * 1  弹起刷新
     */
    private int type = -1;

    public NumberKeyboardView(Context context) {
        super(context);
    }

    public NumberKeyboardView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public NumberKeyboardView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    public OnNumberClickListener getOnNumberClickListener() {
        return onNumberClickListener;
    }

    public void setOnNumberClickListener(OnNumberClickListener onNumberClickListener) {
        this.onNumberClickListener = onNumberClickListener;
    }

    @SuppressLint("DrawAllocation")
    @TargetApi(Build.VERSION_CODES.LOLLIPOP)
    @Override
    protected void onDraw(Canvas canvas) {
        if (!isInit) {
            initData();
        }
        //画键盘
        drawKeyboard(canvas);

        //判断是否点击数字
        if (clickX > 0 && clickY > 0) {
            if (type == 0) {  //按下刷新

                if ("delete".equals(number)) {
                    mPaint.setColor(Color.GRAY);
                    canvas.drawRoundRect(new RectF(60 + 5 * mRectWidth, mHeight / 2 + 10, 60 + 6 * mRectWidth, mHeight / 2 + 140 + mRectHeight), 10, 10, mPaint);
                    mPaint.setColor(Color.BLACK);
                    mPaint.setTextSize(30);// 设置字体大小
                    mPaint.setStrokeWidth(2);
                    canvas.drawText("删", xs[5], ys[0], mPaint);
                    canvas.drawText("除", xs[5], ys[1], mPaint);
                } else if ("big".equals(number)) {
                    mPaint.setColor(Color.GRAY);
                    canvas.drawRoundRect(new RectF(60 + 5 * mRectWidth, mHeight / 2 + 50 + 4 * mRectHeight, 60 + 6 * mRectWidth, mHeight / 2 + 180 + 5 * mRectHeight), 10, 10, mPaint);
                    mPaint.setColor(Color.BLACK);
                    mPaint.setTextSize(30);// 设置字体大小
                    mPaint.setStrokeWidth(2);
                    if(isBigWrite){
                        canvas.drawText("大", xs[5], ys[4], mPaint);
                        canvas.drawText("写", xs[5], ys[5], mPaint);
                    }else {
                        canvas.drawText("小", xs[5], ys[4], mPaint);
                        canvas.drawText("写", xs[5], ys[5], mPaint);
                    }
                } else if ("empty".equals(number)) {
                    mPaint.setColor(Color.GRAY);
                    canvas.drawRoundRect(new RectF(60 + 5 * mRectWidth, mHeight / 2 + 30 + 2 * mRectHeight, 60 + 6 * mRectWidth, mHeight / 2 + 160 + 3 * mRectHeight), 10, 10, mPaint);
                    mPaint.setColor(Color.BLACK);
                    mPaint.setTextSize(30);// 设置字体大小
                    mPaint.setStrokeWidth(2);
                    canvas.drawText("清", xs[5], ys[2], mPaint);
                    canvas.drawText("空", xs[5], ys[3], mPaint);
                } else {
                    mPaint.setColor(Color.GRAY);
                    canvas.drawRoundRect(new RectF(x1, y1, x2, y2), 10, 10, mPaint);
                    mPaint.setColor(Color.BLACK);
                    mPaint.setTextSize(60);// 设置字体大小
                    mPaint.setStrokeWidth(2);
                    canvas.drawText(number, clickX, clickY, mPaint);
                }
            } else if (type == 1) {  //抬起刷新
                if ("delete".equals(number)) {
                    mPaint.setColor(Color.WHITE);
                    canvas.drawRoundRect(new RectF(x1, y1, x2, y2), 10, 10, mPaint);
                    mPaint.setColor(Color.BLACK);
                    mPaint.setTextSize(30);// 设置字体大小
                    mPaint.setStrokeWidth(2);
                    canvas.drawText("删", xs[5], ys[0], mPaint);
                    canvas.drawText("除", xs[5], ys[1], mPaint);
//                    canvas.drawBitmap(mBpDelete, xs[2] - mWidthOfBp / 2 + 10, ys[3] - mHeightOfBp / 2 - 10, mPaint);
                } else if ("big".equals(number)) {
                    mPaint.setColor(Color.WHITE);
                    canvas.drawRoundRect(new RectF(x1, y1, x2, y2), 10, 10, mPaint);
                    mPaint.setColor(Color.BLACK);
                    mPaint.setTextSize(30);// 设置字体大小
                    mPaint.setStrokeWidth(2);
                    if(isBigWrite){
                        canvas.drawText("大", xs[5], ys[4], mPaint);
                        canvas.drawText("写", xs[5], ys[5], mPaint);
                    }else {
                        canvas.drawText("小", xs[5], ys[4], mPaint);
                        canvas.drawText("写", xs[5], ys[5], mPaint);
                    }
                } else if ("empty".equals(number)) {
                    mPaint.setColor(Color.WHITE);
                    canvas.drawRoundRect(new RectF(x1, y1, x2, y2), 10, 10, mPaint);
                    mPaint.setColor(Color.BLACK);
                    mPaint.setTextSize(30);// 设置字体大小
                    mPaint.setStrokeWidth(2);
                    canvas.drawText("清", xs[5], ys[2], mPaint);
                    canvas.drawText("空", xs[5], ys[3], mPaint);

                } else {
                    mPaint.setColor(Color.WHITE);
                    canvas.drawRoundRect(new RectF(x1, y1, x2, y2), 10, 10, mPaint);
                    mPaint.setColor(Color.BLACK);
                    mPaint.setTextSize(60);// 设置字体大小
                    mPaint.setStrokeWidth(2);
                    canvas.drawText(number, clickX, clickY, mPaint);
                }
            }
            //绘制完成后,重置
            clickX = 0;
            clickY = 0;
        }
    }

    private void initData() {
        mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        mWidth = getWidth();
        mHeight = getHeight();
        mBpDelete = BitmapFactory.decodeResource(getResources(), R.mipmap.keyboard_backspace);
        mWidthOfBp = mBpDelete.getWidth();
        mHeightOfBp = mBpDelete.getHeight();

        mRectWidth = (mWidth - 70) / 6;   //每个按键左右间距10
        mRectHeight = (mHeight - 100) / 14;//每个按键上下间距10

        xs[0] = mRectWidth / 2;
        xs[1] = (mRectWidth * 3) / 2 + 10;
        xs[2] = (mRectWidth * 5) / 2 + 20;
        xs[3] = (mRectWidth * 7) / 2 + 30;
        xs[4] = (mRectWidth * 9) / 2 + 40;
        xs[5] = (mRectWidth * 11) / 2 + 50;

        ys[0] = mRectHeight / 2 + 25 + mHeight / 2;
        ys[1] = (mRectHeight * 3) / 2 + 35 + mHeight / 2;
        ys[2] = (mRectHeight * 5) / 2 + 45 + mHeight / 2;
        ys[3] = (mRectHeight * 7) / 2 + 55 + mHeight / 2;
        ys[4] = (mRectHeight * 9) / 2 + 65 + mHeight / 2;
        ys[5] = (mRectHeight * 11) / 2 + 75 + mHeight / 2;
        ys[6] = (mRectHeight * 13) / 2 + 85 + mHeight / 2;

        isInit = true;
    }

    /**
     * drawRoundRect(float left, float top, float right, float bottom, float rx, float ry, Paint paint)这种方式在5.0以下的机器上会报错，
     * 需要换成drawRoundRect(RectF rect, float rx, float ry, Paint paint)
     *
     * @param canvas
     */

    @TargetApi(Build.VERSION_CODES.LOLLIPOP)
    private void drawKeyboard(Canvas canvas) {

        mPaint.setColor(Color.WHITE);
        //画宫格
        //第一排
        canvas.drawRoundRect(new RectF(10, mHeight / 2 + 10, 10 + mRectWidth, mHeight / 2 + 10 + mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(20 + mRectWidth, mHeight / 2 + 10, 20 + 2 * mRectWidth, mHeight / 2 + 10 + mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(30 + 2 * mRectWidth, mHeight / 2 + 10, 30 + 3 * mRectWidth, mHeight / 2 + 10 + mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(40 + 3 * mRectWidth, mHeight / 2 + 10, 40 + 4 * mRectWidth, mHeight / 2 + 10 + mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(50 + 4 * mRectWidth, mHeight / 2 + 10, 50 + 5 * mRectWidth, mHeight / 2 + 10 + mRectHeight), 10, 10, mPaint);


        //第二排
        canvas.drawRoundRect(new RectF(10, mHeight / 2 + 20 + mRectHeight, 10 + mRectWidth, mHeight / 2 + 20 + 2 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(20 + mRectWidth, mHeight / 2 + 20 + mRectHeight, 20 + 2 * mRectWidth, mHeight / 2 + 20 + 2 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(30 + 2 * mRectWidth, mHeight / 2 + 20 + mRectHeight, 30 + 3 * mRectWidth, mHeight / 2 + 20 + 2 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(40 + 3 * mRectWidth, mHeight / 2 + 20 + mRectHeight, 40 + 4 * mRectWidth, mHeight / 2 + 20 + 2 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(50 + 4 * mRectWidth, mHeight / 2 + 20 + mRectHeight, 50 + 5 * mRectWidth, mHeight / 2 + 20 + 2 * mRectHeight), 10, 10, mPaint);


        //第三排
        canvas.drawRoundRect(new RectF(10, mHeight / 2 + 30 + 2 * mRectHeight, 10 + mRectWidth, mHeight / 2 + 30 + 3 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(20 + mRectWidth, mHeight / 2 + 30 + 2 * mRectHeight, 20 + 2 * mRectWidth, mHeight / 2 + 30 + 3 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(30 + 2 * mRectWidth, mHeight / 2 + 30 + 2 * mRectHeight, 30 + 3 * mRectWidth, mHeight / 2 + 30 + 3 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(40 + 3 * mRectWidth, mHeight / 2 + 30 + 2 * mRectHeight, 40 + 4 * mRectWidth, mHeight / 2 + 30 + 3 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(50 + 4 * mRectWidth, mHeight / 2 + 30 + 2 * mRectHeight, 50 + 5 * mRectWidth, mHeight / 2 + 30 + 3 * mRectHeight), 10, 10, mPaint);

        //第四排
        canvas.drawRoundRect(new RectF(10, mHeight / 2 + 40 + 3 * mRectHeight, 10 + mRectWidth, mHeight / 2 + 40 + 4 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(20 + mRectWidth, mHeight / 2 + 40 + 3 * mRectHeight, 20 + 2 * mRectWidth, mHeight / 2 + 40 + 4 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(30 + 2 * mRectWidth, mHeight / 2 + 40 + 3 * mRectHeight, 30 + 3 * mRectWidth, mHeight / 2 + 40 + 4 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(40 + 3 * mRectWidth, mHeight / 2 + 40 + 3 * mRectHeight, 40 + 4 * mRectWidth, mHeight / 2 + 40 + 4 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(50 + 4 * mRectWidth, mHeight / 2 + 40 + 3 * mRectHeight, 50 + 5 * mRectWidth, mHeight / 2 + 40 + 4 * mRectHeight), 10, 10, mPaint);

        //第五排
        canvas.drawRoundRect(new RectF(10, mHeight / 2 + 50 + 4 * mRectHeight, 10 + mRectWidth, mHeight / 2 + 50 + 5 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(20 + mRectWidth, mHeight / 2 + 50 + 4 * mRectHeight, 20 + 2 * mRectWidth, mHeight / 2 + 50 + 5 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(30 + 2 * mRectWidth, mHeight / 2 + 50 + 4 * mRectHeight, 30 + 3 * mRectWidth, mHeight / 2 + 50 + 5 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(40 + 3 * mRectWidth, mHeight / 2 + 50 + 4 * mRectHeight, 40 + 4 * mRectWidth, mHeight / 2 + 50 + 5 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(50 + 4 * mRectWidth, mHeight / 2 + 50 + 4 * mRectHeight, 50 + 5 * mRectWidth, mHeight / 2 + 50 + 5 * mRectHeight), 10, 10, mPaint);

        //第六排
        canvas.drawRoundRect(new RectF(10, mHeight / 2 + 60 + 5 * mRectHeight, 10 + mRectWidth, mHeight / 2 + 60 + 6 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(20 + mRectWidth, mHeight / 2 + 60 + 5 * mRectHeight, 20 + 2 * mRectWidth, mHeight / 2 + 60 + 6 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(30 + 2 * mRectWidth, mHeight / 2 + 60 + 5 * mRectHeight, 30 + 3 * mRectWidth, mHeight / 2 + 60 + 6 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(40 + 3 * mRectWidth, mHeight / 2 + 60 + 5 * mRectHeight, 40 + 4 * mRectWidth, mHeight / 2 + 60 + 6 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(50 + 4 * mRectWidth, mHeight / 2 + 60 + 5 * mRectHeight, 50 + 5 * mRectWidth, mHeight / 2 + 60 + 6 * mRectHeight), 10, 10, mPaint);


        //第七排
        canvas.drawRoundRect(new RectF(10, mHeight / 2 + 70 + 6 * mRectHeight, 10 + mRectWidth, mHeight / 2 + 70 + 7 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(20 + mRectWidth, mHeight / 2 + 70 + 6 * mRectHeight, 20 + 2 * mRectWidth, mHeight / 2 + 70 + 7 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(30 + 2 * mRectWidth, mHeight / 2 + 70 + 6 * mRectHeight, 30 + 3 * mRectWidth, mHeight / 2 + 70 + 7 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(40 + 3 * mRectWidth, mHeight / 2 + 70 + 6 * mRectHeight, 40 + 4 * mRectWidth, mHeight / 2 + 70 + 7 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(50 + 4 * mRectWidth, mHeight / 2 + 70 + 6 * mRectHeight, 50 + 5 * mRectWidth, mHeight / 2 + 70 + 7 * mRectHeight), 10, 10, mPaint);
        canvas.drawRoundRect(new RectF(60 + 5 * mRectWidth, mHeight / 2 + 70 + 6 * mRectHeight, 60 + 6 * mRectWidth, mHeight / 2 + 70 + 7 * mRectHeight), 10, 10, mPaint);

        /*删除 清空 大写 */
        canvas.drawRoundRect(new RectF(60 + 5 * mRectWidth, mHeight / 2 + 10, 60 + 6 * mRectWidth, mHeight / 2 + 140 + mRectHeight), 10, 10, mPaint);
//        canvas.drawRoundRect(new RectF(60 + 5 * mRectWidth, mHeight / 2 + 20 + mRectHeight, 60 + 6 * mRectWidth, mHeight / 2 + 20 + 2 * mRectHeight), 10, 10, mPaint2);

        canvas.drawRoundRect(new RectF(60 + 5 * mRectWidth, mHeight / 2 + 30 + 2 * mRectHeight, 60 + 6 * mRectWidth, mHeight / 2 + 160 + 3 * mRectHeight), 10, 10, mPaint);
//        canvas.drawRoundRect(new RectF(60 + 5 * mRectWidth, mHeight / 2 + 40 + 3 * mRectHeight, 60 + 6 * mRectWidth, mHeight / 2 + 40 + 4 * mRectHeight), 10, 10, mPaint2);
        canvas.drawRoundRect(new RectF(60 + 5 * mRectWidth, mHeight / 2 + 50 + 4 * mRectHeight, 60 + 6 * mRectWidth, mHeight / 2 + 180 + 5 * mRectHeight), 10, 10, mPaint);
//        canvas.drawRoundRect(new RectF(60 + 5 * mRectWidth, mHeight / 2 + 60 + 5 * mRectHeight, 60 + 6 * mRectWidth, mHeight / 2 + 60 + 6 * mRectHeight), 10, 10, mPaint2);

        mPaint.setColor(Color.BLACK);
        mPaint.setTextSize(60);// 设置字体大小
        mPaint.setStrokeWidth(2);
        //画数字
        //第一排
        canvas.drawText("1", xs[0], ys[0], mPaint);
        canvas.drawText("2", xs[1], ys[0], mPaint);
        canvas.drawText("3", xs[2], ys[0], mPaint);
        canvas.drawText("4", xs[3], ys[0], mPaint);
        canvas.drawText("5", xs[4], ys[0], mPaint);
        //第二排
        canvas.drawText("6", xs[0], ys[1], mPaint);
        canvas.drawText("7", xs[1], ys[1], mPaint);
        canvas.drawText("8", xs[2], ys[1], mPaint);
        canvas.drawText("9", xs[3], ys[1], mPaint);
        canvas.drawText("0", xs[4], ys[1], mPaint);
        //第三排
        canvas.drawText(isBigWrite ? "a" : "A", xs[0], ys[2], mPaint);
        canvas.drawText(isBigWrite ? "b" : "B", xs[1], ys[2], mPaint);
        canvas.drawText(isBigWrite ? "c" : "C", xs[2], ys[2], mPaint);
        canvas.drawText(isBigWrite ? "d" : "D", xs[3], ys[2], mPaint);
        canvas.drawText(isBigWrite ? "e" : "E", xs[4], ys[2], mPaint);
        //第四排
        canvas.drawText(isBigWrite ? "f" : "F", xs[0], ys[3], mPaint);
        canvas.drawText(isBigWrite ? "g" : "G", xs[1], ys[3], mPaint);
        canvas.drawText(isBigWrite ? "h" : "H", xs[2], ys[3], mPaint);
        canvas.drawText(isBigWrite ? "i" : "I", xs[3], ys[3], mPaint);
        canvas.drawText(isBigWrite ? "j" : "J", xs[4], ys[3], mPaint);
        //第五排
        canvas.drawText(isBigWrite ? "k" : "K", xs[0], ys[4], mPaint);
        canvas.drawText(isBigWrite ? "l" : "L", xs[1], ys[4], mPaint);
        canvas.drawText(isBigWrite ? "m" : "M", xs[2], ys[4], mPaint);
        canvas.drawText(isBigWrite ? "n" : "N", xs[3], ys[4], mPaint);
        canvas.drawText(isBigWrite ? "o" : "O", xs[4], ys[4], mPaint);
        //第六排
        canvas.drawText(isBigWrite ? "p" : "P", xs[0], ys[5], mPaint);
        canvas.drawText(isBigWrite ? "q" : "Q", xs[1], ys[5], mPaint);
        canvas.drawText(isBigWrite ? "r" : "R", xs[2], ys[5], mPaint);
        canvas.drawText(isBigWrite ? "s" : "S", xs[3], ys[5], mPaint);
        canvas.drawText(isBigWrite ? "t" : "T", xs[4], ys[5], mPaint);
        //第七排
        canvas.drawText(isBigWrite ? "u" : "U", xs[0], ys[6], mPaint);
        canvas.drawText(isBigWrite ? "v" : "V", xs[1], ys[6], mPaint);
        canvas.drawText(isBigWrite ? "w" : "W", xs[2], ys[6], mPaint);
        canvas.drawText(isBigWrite ? "x" : "X", xs[3], ys[6], mPaint);
        canvas.drawText(isBigWrite ? "y" : "Y", xs[4], ys[6], mPaint);
        canvas.drawText(isBigWrite ? "z" : "Z", xs[5], ys[6], mPaint);

        mPaint.setColor(Color.BLACK);
        mPaint.setTextSize(30);// 设置字体大小
        mPaint.setStrokeWidth(2);
        canvas.drawText("删", xs[5], ys[0], mPaint);
        canvas.drawText("除", xs[5], ys[1], mPaint);
        canvas.drawText("清", xs[5], ys[2], mPaint);
        canvas.drawText("空", xs[5], ys[3], mPaint);
        if(isBigWrite){
            canvas.drawText("大", xs[5], ys[4], mPaint);
            canvas.drawText("写", xs[5], ys[5], mPaint);
        }else {
            canvas.drawText("小", xs[5], ys[4], mPaint);
            canvas.drawText("写", xs[5], ys[5], mPaint);
        }



    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        float x = event.getX();
        float y = event.getY();
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN: //按下
                setDefault();
                handleDown(x, y);
                return true;
            case MotionEvent.ACTION_UP: //弹起
                type = 1;//弹起刷新
                invalidate();//刷新界面
                //一次按下结束,返回点击的数字
                if (onNumberClickListener != null) {
                    if (number != null) {
                        if (number.equals("delete")) {
                            onNumberClickListener.onNumberDelete();
                        } else if (number.equals("empty")) {
                            onNumberClickListener.onNumberEmpty();
                        } else if (number.equals("big")) {
                            if (isBigWrite) {
                                isBigWrite = false;
                            } else {
                                isBigWrite = true;
                            }
                            invalidate();
                        } else {
                            onNumberClickListener.onNumberReturn(number);
                        }
                    }
                }
                //恢复默认
                setDefault();
                return true;
            case MotionEvent.ACTION_CANCEL:  //取消
                //恢复默认值
                setDefault();
                return true;
            default:
                break;
        }
        return false;
    }

    private void setDefault() {
        clickX = 0;
        clickY = 0;
        number = null;
        type = -1;
    }

    private void handleDown(float x, float y) {
        if (y < mHeight / 2) {
            return;
        }
        if (x >= 10 && x <= 10 + mRectWidth) {   //第一列
            clickX = xs[0];
            if (y >= mHeight / 2 + 10 && y <= mHeight / 2 + 10 + mRectHeight) {  //第一排(1)
                clickY = ys[0];
                x1 = 10;
                y1 = mHeight / 2 + 10;
                x2 = 10 + mRectWidth;
                y2 = mHeight / 2 + 10 + mRectHeight;
                number = "1";
            } else if (y >= mHeight / 2 + 20 + mRectHeight && y <= mHeight / 2 + 20 + 2 * mRectHeight) {  //第二排(4)
                x1 = 10;
                y1 = mHeight / 2 + 20 + mRectHeight;
                x2 = 10 + mRectWidth;
                y2 = mHeight / 2 + 20 + 2 * mRectHeight;
                clickY = ys[1];
                number = "6";
            } else if (y >= mHeight / 2 + 30 + 2 * mRectHeight && y <= mHeight / 2 + 30 + 3 * mRectHeight) {  //第三排(7)
                x1 = 10;
                y1 = mHeight / 2 + 30 + 2 * mRectHeight;
                x2 = 10 + mRectWidth;
                y2 = mHeight / 2 + 30 + 3 * mRectHeight;
                clickY = ys[2];
                number = isBigWrite ? "a" : "A";
            } else if (y >= mHeight / 2 + 40 + 3 * mRectHeight && y <= mHeight / 2 + 40 + 4 * mRectHeight) { //第四排(0)
                x1 = 10;
                y1 = mHeight / 2 + 40 + 3 * mRectHeight;
                x2 = 10 + mRectWidth;
                y2 = mHeight / 2 + 40 + 4 * mRectHeight;
                clickY = ys[3];
                number = isBigWrite ? "f" : "F";
            } else if (y >= mHeight / 2 + 50 + 4 * mRectHeight && y <= mHeight / 2 + 50 + 5 * mRectHeight) { //第四排(0)
                x1 = 10;
                y1 = mHeight / 2 + 50 + 4 * mRectHeight;
                x2 = 10 + mRectWidth;
                y2 = mHeight / 2 + 50 + 5 * mRectHeight;
                clickY = ys[4];
                number = isBigWrite ? "k" : "K";
            } else if (y >= mHeight / 2 + 60 + 5 * mRectHeight && y <= mHeight / 2 + 60 + 6 * mRectHeight) { //第四排(0)
                x1 = 10;
                y1 = mHeight / 2 + 60 + 5 * mRectHeight;
                x2 = 10 + mRectWidth;
                y2 = mHeight / 2 + 60 + 6 * mRectHeight;
                clickY = ys[5];
                number = isBigWrite ? "p" : "P";
            } else if (y >= mHeight / 2 + 70 + 6 * mRectHeight && y <= mHeight / 2 + 70 + 7 * mRectHeight) { //第四排(0)
                x1 = 10;
                y1 = mHeight / 2 + 70 + 6 * mRectHeight;
                x2 = 10 + mRectWidth;
                y2 = mHeight / 2 + 70 + 7 * mRectHeight;
                clickY = ys[6];
                number = isBigWrite ? "u" : "U";
            }
        } else if (x >= 20 + mRectWidth && x <= 20 + 2 * mRectWidth) {  //第二列
            clickX = xs[1];
            if (y >= mHeight / 2 + 10 && y <= mHeight / 2 + 10 + mRectHeight) {  //第一排(2)
                x1 = 20 + mRectWidth;
                y1 = mHeight / 2 + 10;
                x2 = 20 + 2 * mRectWidth;
                y2 = mHeight / 2 + 10 + mRectHeight;
                clickY = ys[0];
                number = "2";
            } else if (y >= mHeight / 2 + 20 + mRectHeight && y <= mHeight / 2 + 20 + 2 * mRectHeight) {  //第二排(5)
                x1 = 20 + mRectWidth;
                y1 = mHeight / 2 + 20 + mRectHeight;
                x2 = 20 + 2 * mRectWidth;
                y2 = mHeight / 2 + 20 + 2 * mRectHeight;
                clickY = ys[1];
                number = "7";
            } else if (y >= mHeight / 2 + 30 + 2 * mRectHeight && y <= mHeight / 2 + 30 + 3 * mRectHeight) {  //第三排(8)
                x1 = 20 + mRectWidth;
                y1 = mHeight / 2 + 30 + 2 * mRectHeight;
                x2 = 20 + 2 * mRectWidth;
                y2 = mHeight / 2 + 30 + 3 * mRectHeight;
                clickY = ys[2];
                number = isBigWrite ? "b" : "B";
            } else if (y >= mHeight / 2 + 40 + 3 * mRectHeight && y <= mHeight / 2 + 40 + 4 * mRectHeight) { //第四排(0)
                x1 = 20 + mRectWidth;
                y1 = mHeight / 2 + 40 + 3 * mRectHeight;
                x2 = 20 + 2 * mRectWidth;
                y2 = mHeight / 2 + 40 + 4 * mRectHeight;
                clickY = ys[3];
                number = isBigWrite ? "g" : "G";
            } else if (y >= mHeight / 2 + 50 + 4 * mRectHeight && y <= mHeight / 2 + 50 + 5 * mRectHeight) { //第五排(0)
                x1 = 20 + mRectWidth;
                y1 = mHeight / 2 + 50 + 4 * mRectHeight;
                x2 = 20 + 2 * mRectWidth;
                y2 = mHeight / 2 + 50 + 5 * mRectHeight;
                clickY = ys[4];
                number = isBigWrite ? "l" : "L";
            } else if (y >= mHeight / 2 + 60 + 5 * mRectHeight && y <= mHeight / 2 + 60 + 6 * mRectHeight) { //第六排(0)
                x1 = 20 + mRectWidth;
                y1 = mHeight / 2 + 60 + 5 * mRectHeight;
                x2 = 20 + 2 * mRectWidth;
                y2 = mHeight / 2 + 60 + 6 * mRectHeight;
                clickY = ys[5];
                number = isBigWrite ? "q" : "Q";
            } else if (y >= mHeight / 2 + 70 + 6 * mRectHeight && y <= mHeight / 2 + 70 + 7 * mRectHeight) { //第七排(0)
                x1 = 20 + mRectWidth;
                y1 = mHeight / 2 + 70 + 6 * mRectHeight;
                x2 = 20 + 2 * mRectWidth;
                y2 = mHeight / 2 + 70 + 7 * mRectHeight;
                clickY = ys[6];
                number = isBigWrite ? "v" : "V";
            }
        } else if (x >= 30 + 2 * mRectWidth && x <= 30 + 3 * mRectWidth) {   //第三列
            clickX = xs[2];
            if (y >= mHeight / 2 + 10 && y <= mHeight / 2 + 10 + mRectHeight) {  //第一排(3)
                x1 = 30 + 2 * mRectWidth;
                y1 = mHeight / 2 + 10;
                x2 = 30 + 3 * mRectWidth;
                y2 = mHeight / 2 + 10 + mRectHeight;
                clickY = ys[0];
                number = "3";
            } else if (y >= mHeight / 2 + 20 + mRectHeight && y <= mHeight / 2 + 20 + 2 * mRectHeight) {  //第二排(6)
                x1 = 30 + 2 * mRectWidth;
                y1 = mHeight / 2 + 20 + mRectHeight;
                x2 = 30 + 3 * mRectWidth;
                y2 = mHeight / 2 + 20 + 2 * mRectHeight;
                clickY = ys[1];
                number = "8";
            } else if (y >= mHeight / 2 + 30 + 2 * mRectHeight && y <= mHeight / 2 + 30 + 3 * mRectHeight) {  //第三排(9)
                x1 = 30 + 2 * mRectWidth;
                y1 = mHeight / 2 + 30 + 2 * mRectHeight;
                x2 = 30 + 3 * mRectWidth;
                y2 = mHeight / 2 + 30 + 3 * mRectHeight;
                clickY = ys[2];
                number = isBigWrite ? "c" : "C";
            } else if (y >= mHeight / 2 + 40 + 3 * mRectHeight && y <= mHeight / 2 + 40 + 4 * mRectHeight) { //第四排()
                x1 = 30 + 2 * mRectWidth;
                y1 = mHeight / 2 + 40 + 3 * mRectHeight;
                x2 = 30 + 3 * mRectWidth;
                y2 = mHeight / 2 + 40 + 4 * mRectHeight;
                clickY = ys[3];
                number = isBigWrite ? "h" : "H";
            } else if (y >= mHeight / 2 + 50 + 4 * mRectHeight && y <= mHeight / 2 + 50 + 5 * mRectHeight) { //第四排()
                x1 = 30 + 2 * mRectWidth;
                y1 = mHeight / 2 + 50 + 4 * mRectHeight;
                x2 = 30 + 3 * mRectWidth;
                y2 = mHeight / 2 + 50 + 5 * mRectHeight;
                clickY = ys[4];
                number = isBigWrite ? "m" : "M";
            } else if (y >= mHeight / 2 + 60 + 5 * mRectHeight && y <= mHeight / 2 + 60 + 6 * mRectHeight) { //第四排()
                x1 = 30 + 2 * mRectWidth;
                y1 = mHeight / 2 + 60 + 5 * mRectHeight;
                x2 = 30 + 3 * mRectWidth;
                y2 = mHeight / 2 + 60 + 6 * mRectHeight;
                clickY = ys[5];
                number = isBigWrite ? "r" : "R";
            } else if (y >= mHeight / 2 + 70 + 6 * mRectHeight && y <= mHeight / 2 + 70 + 7 * mRectHeight) { //第四排()
                x1 = 30 + 2 * mRectWidth;
                y1 = mHeight / 2 + 70 + 6 * mRectHeight;
                x2 = 30 + 3 * mRectWidth;
                y2 = mHeight / 2 + 70 + 7 * mRectHeight;
                clickY = ys[6];
                number = isBigWrite ? "w" : "W";
            }
        } else if (x >= 40 + 3 * mRectWidth && x <= 40 + 4 * mRectWidth) {   //第四列
            clickX = xs[3];
            if (y >= mHeight / 2 + 10 && y <= mHeight / 2 + 10 + mRectHeight) {  //第一排(3)
                x1 = 40 + 3 * mRectWidth;
                y1 = mHeight / 2 + 10;
                x2 = 40 + 4 * mRectWidth;
                y2 = mHeight / 2 + 10 + mRectHeight;
                clickY = ys[0];
                number = "4";
            } else if (y >= mHeight / 2 + 20 + mRectHeight && y <= mHeight / 2 + 20 + 2 * mRectHeight) {  //第二排(6)
                x1 = 40 + 3 * mRectWidth;
                y1 = mHeight / 2 + 20 + mRectHeight;
                x2 = 40 + 4 * mRectWidth;
                y2 = mHeight / 2 + 20 + 2 * mRectHeight;
                clickY = ys[1];
                number = "9";
            } else if (y >= mHeight / 2 + 30 + 2 * mRectHeight && y <= mHeight / 2 + 30 + 3 * mRectHeight) {  //第三排(9)
                x1 = 40 + 3 * mRectWidth;
                y1 = mHeight / 2 + 30 + 2 * mRectHeight;
                x2 = 40 + 4 * mRectWidth;
                y2 = mHeight / 2 + 30 + 3 * mRectHeight;
                clickY = ys[2];
                number = isBigWrite ? "d" : "D";
            } else if (y >= mHeight / 2 + 40 + 3 * mRectHeight && y <= mHeight / 2 + 40 + 4 * mRectHeight) { //第四排()
                x1 = 40 + 3 * mRectWidth;
                y1 = mHeight / 2 + 40 + 3 * mRectHeight;
                x2 = 40 + 4 * mRectWidth;
                y2 = mHeight / 2 + 40 + 4 * mRectHeight;
                clickY = ys[3];
                number = isBigWrite ? "i" : "I";
            } else if (y >= mHeight / 2 + 50 + 4 * mRectHeight && y <= mHeight / 2 + 50 + 5 * mRectHeight) { //第四排()
                x1 = 40 + 3 * mRectWidth;
                y1 = mHeight / 2 + 50 + 4 * mRectHeight;
                x2 = 40 + 4 * mRectWidth;
                y2 = mHeight / 2 + 50 + 5 * mRectHeight;
                clickY = ys[4];
                number = isBigWrite ? "n" : "N";
            } else if (y >= mHeight / 2 + 60 + 5 * mRectHeight && y <= mHeight / 2 + 60 + 6 * mRectHeight) { //第四排()
                x1 = 40 + 3 * mRectWidth;
                y1 = mHeight / 2 + 60 + 5 * mRectHeight;
                x2 = 40 + 4 * mRectWidth;
                y2 = mHeight / 2 + 60 + 6 * mRectHeight;
                clickY = ys[5];
                number = isBigWrite ? "s" : "S";
            } else if (y >= mHeight / 2 + 70 + 6 * mRectHeight && y <= mHeight / 2 + 70 + 7 * mRectHeight) { //第四排()
                x1 = 40 + 3 * mRectWidth;
                y1 = mHeight / 2 + 70 + 6 * mRectHeight;
                x2 = 40 + 4 * mRectWidth;
                y2 = mHeight / 2 + 70 + 7 * mRectHeight;
                clickY = ys[6];
                number = isBigWrite ? "x" : "X";
            }
        } else if (x >= 50 + 4 * mRectWidth && x <= 50 + 5 * mRectWidth) {   //第五列
            clickX = xs[4];
            if (y >= mHeight / 2 + 10 && y <= mHeight / 2 + 10 + mRectHeight) {  //第一排(3)
                x1 = 50 + 4 * mRectWidth;
                y1 = mHeight / 2 + 10;
                x2 = 50 + 5 * mRectWidth;
                y2 = mHeight / 2 + 10 + mRectHeight;
                clickY = ys[0];
                number = "5";
            } else if (y >= mHeight / 2 + 20 + mRectHeight && y <= mHeight / 2 + 20 + 2 * mRectHeight) {  //第二排(6)
                x1 = 50 + 4 * mRectWidth;
                y1 = mHeight / 2 + 20 + mRectHeight;
                x2 = 50 + 5 * mRectWidth;
                y2 = mHeight / 2 + 20 + 2 * mRectHeight;
                clickY = ys[1];
                number = "0";
            } else if (y >= mHeight / 2 + 30 + 2 * mRectHeight && y <= mHeight / 2 + 30 + 3 * mRectHeight) {  //第三排(9)
                x1 = 50 + 4 * mRectWidth;
                y1 = mHeight / 2 + 30 + 2 * mRectHeight;
                x2 = 50 + 5 * mRectWidth;
                y2 = mHeight / 2 + 30 + 3 * mRectHeight;
                clickY = ys[2];
                number = isBigWrite ? "e" : "E";
            } else if (y >= mHeight / 2 + 40 + 3 * mRectHeight && y <= mHeight / 2 + 40 + 4 * mRectHeight) { //第四排()
                x1 = 50 + 4 * mRectWidth;
                y1 = mHeight / 2 + 40 + 3 * mRectHeight;
                x2 = 50 + 5 * mRectWidth;
                y2 = mHeight / 2 + 40 + 4 * mRectHeight;
                clickY = ys[3];
                number = isBigWrite ? "j" : "J";
            } else if (y >= mHeight / 2 + 50 + 4 * mRectHeight && y <= mHeight / 2 + 50 + 5 * mRectHeight) { //第五排()
                x1 = 50 + 4 * mRectWidth;
                y1 = mHeight / 2 + 50 + 4 * mRectHeight;
                x2 = 50 + 5 * mRectWidth;
                y2 = mHeight / 2 + 50 + 5 * mRectHeight;
                clickY = ys[4];
                number = isBigWrite ? "o" : "O";
            } else if (y >= mHeight / 2 + 60 + 5 * mRectHeight && y <= mHeight / 2 + 60 + 6 * mRectHeight) { //第六排()
                x1 = 50 + 4 * mRectWidth;
                y1 = mHeight / 2 + 60 + 5 * mRectHeight;
                x2 = 50 + 5 * mRectWidth;
                y2 = mHeight / 2 + 60 + 6 * mRectHeight;
                clickY = ys[5];
                number = isBigWrite ? "t" : "T";
            } else if (y >= mHeight / 2 + 70 + 6 * mRectHeight && y <= mHeight / 2 + 70 + 7 * mRectHeight) { //第七排()
                x1 = 50 + 4 * mRectWidth;
                y1 = mHeight / 2 + 70 + 6 * mRectHeight;
                x2 = 50 + 5 * mRectWidth;
                y2 = mHeight / 2 + 70 + 7 * mRectHeight;
                clickY = ys[6];
                number = isBigWrite ? "y" : "Y";
            } else if (y >= mHeight / 2 + 80 + 7 * mRectHeight && y <= mHeight / 2 + 80 + 8 * mRectHeight) { //第八排()
                x1 = 50 + 4 * mRectWidth;
                y1 = mHeight / 2 + 80 + 7 * mRectHeight;
                x2 = 50 + 5 * mRectWidth;
                y2 = mHeight / 2 + 80 + 8 * mRectHeight;
                clickY = ys[6];
                number = isBigWrite ? "z" : "Z";
            }
        } else if (x >= 60 + 5 * mRectWidth && x <= 70 + 6 * mRectWidth) {   //第六列
            clickX = xs[5];
            if (y >= mHeight / 2 + 10 && y <= mHeight / 2 + 140 + mRectHeight) {  //第一排(3)
                x1 = 60 + 5 * mRectWidth;
                y1 = mHeight / 2 + 10;
                x2 = 60 + 6 * mRectWidth;
                y2 = mHeight / 2 + 10 + mRectHeight;
                clickY = ys[0];
                number = "delete";
            } else if (y >= mHeight / 2 + 30 + 2 * mRectHeight && y <= mHeight / 2 + 160 + 3 * mRectHeight) {  //第三排(9)
                x1 = 60 + 5 * mRectWidth;
                y1 = mHeight / 2 + 30 + 2 * mRectHeight;
                x2 = 60 + 6 * mRectWidth;
                y2 = mHeight / 2 + 30 + 3 * mRectHeight;
                clickY = ys[2];
                number = "empty";
            } else if (y >= mHeight / 2 + 50 + 4 * mRectHeight && y <= mHeight / 2 + 180 + 5 * mRectHeight) { //第五排()
                x1 = 60 + 5 * mRectWidth;
                y1 = mHeight / 2 + 50 + 4 * mRectHeight;
                x2 = 60 + 6 * mRectWidth;
                y2 = mHeight / 2 + 50 + 5 * mRectHeight;
                clickY = ys[4];
                number = "big";
            } else if (y >= mHeight / 2 + 70 + 6 * mRectHeight && y <= mHeight / 2 + 70 + 7 * mRectHeight) { //第七排()
                x1 = 60 + 5 * mRectWidth;
                y1 = mHeight / 2 + 70 + 6 * mRectHeight;
                x2 = 60 + 6 * mRectWidth;
                y2 = mHeight / 2 + 70 + 7 * mRectHeight;
                clickY = ys[6];
                number = "z";
            }
        }
        type = 0;   //按下刷新
        invalidate();
    }

    public interface OnNumberClickListener {
        //回调点击的数字
        void onNumberReturn(String number);

        //的回调
        void onNumberDelete();

        //的回调
        void onNumberEmpty();
    }
}

接下来再来看看效果吧：


```
drawRoundRect(float left, float top, float right, float bottom, float rx, float ry, Paint paint)这种方式在5.0以下的机器上会报错，
需要换成drawRoundRect(RectF rect, float rx, float ry, Paint paint)
```
