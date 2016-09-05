---
layout: post
title:  "一个基于MVP的Android标准化框架的制作过程"
date:   2016-09-05 17:38:22 +0800
categories: Android
---
# 一个基于MVP的Android标准化框架的制作过程
## MVP模式
`Model` `View` `Presenter` **MVP**
`Model` 模型，一切耗时的，复杂的操作（网络请求，算法等），都在这里进行。
`View` 视图，可以认为是UI，是activity而不是XML，它除了展现画面给用户一位，还是要监听用户操作，是交互。
`Presenter` 顾名思义，是这个模式的**主持人**，它是Model和View交流的中枢。 
`Model``View` 互相不知道彼此的动作，所以在开发，测试的时候可以并行，加快开发测试的速度。
`Presenter` 定义了View，Model的各种行为。
发起一个动作的过程
`View`-->`Presenter`-->`Model`-->`Presenter`-->`View`
`View`监听到事件，调用起`Presenter`相关的方法，该方法调用起`Model`相关的方法。
`Model`处理完事件后，把结果返回给`Presenter`，`Presenter`再控制`View`改变UI界面，完成交互。
这里的方法 均用`Interface`来实现， `View``Presenter``Model` 均实现相关的`Interface`来达到目的。
这就要求，在实现`View`，`Model`之前就要定义好他们所有的行为。
接下来 实现一个简单的例子：
先实现一个`View`即`Activity`
我们定义这个`View`的几个行为。

``` android
public interface DemoViewInterface {
    /*
    * 用Toast展示一段文字并显示他的长度
    * */
    void showToast(int length);
    /*
    * 用Dialog展示一段文字并显示它的长度
    * */
    void showDialog(int length);
    /*
    * 展示倒计时(演示耗时操作)
    * */
    void showCountDown(int count);

    /**
     * 展示网络请求得到的数据
     * @param msg 网络请求抓回来的数据
     */
    void showMessage(String msg);
}
```
接下来 在`Activity`中实现这个接口。
首先实现`XML`布局

``` 

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="match_parent">

    <EditText
        android:id="@+id/et_input"
        android:hint="@string/hint"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
    <LinearLayout
        android:orientation="horizontal"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
        <Button
            android:id="@+id/btn_showtoast"
            android:text="@string/showtoast"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content" />
        <Button
            android:id="@+id/btn_showdialog"
            android:text="@string/showdialog"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content" />
    </LinearLayout>
    <LinearLayout
        android:orientation="horizontal"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
        <TextView
            android:id="@+id/tv_countdown_number"
            android:layout_gravity="center_vertical"
            android:text="10"
            android:textSize="30dp"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content" />
        <Button
            android:id="@+id/btn_countdown_start"
            android:text="@string/startcountdown"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content" />
        <Button
            android:id="@+id/btn_countdown_reset"
            android:text="@string/reset"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content" />
    </LinearLayout>
    <Button
        android:id="@+id/btn_request"
        android:text="@string/requert"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />
    <TextView
        android:id="@+id/tv_result"
        android:hint="@string/showresult"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />
</LinearLayout>
```
预览时这样子的：
![](http://ww3.sinaimg.cn/mw690/ad4f5c29gw1f7iwim7inpj20t6101aco.jpg)

在`Activity`中`Bind`所有控件，添加监听，实现接口后，我们就完成了这个`View`的第一步

``` android
public class DemoViewActivity extends Activity implements DemoViewInterface, View.OnClickListener {

    private Button btn_showToast,btn_showDialog,btn_countDownStart,btn_countDownReset,btn_request;
    private TextView tv_countDown_number,tv_showResult;
    private EditText et_input;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_demoview);
        bindView();
        bindEvent();
    }

    private void bindEvent() {
        btn_request.setOnClickListener(this);
        btn_countDownReset.setOnClickListener(this);
        btn_countDownStart.setOnClickListener(this);
        btn_showDialog.setOnClickListener(this);
        btn_showToast.setOnClickListener(this);
    }

    private void bindView() {
        et_input = (EditText) findViewById(R.id.et_input);
        btn_showToast = (Button) findViewById(R.id.btn_showtoast);
        btn_showDialog = (Button) findViewById(R.id.btn_showdialog);
        tv_countDown_number = (TextView) findViewById(R.id.tv_countdown_number);
        btn_countDownStart = (Button) findViewById(R.id.btn_countdown_start);
        btn_countDownReset = (Button) findViewById(R.id.btn_countdown_reset);
        btn_request = (Button) findViewById(R.id.btn_request);
        tv_showResult = (TextView) findViewById(R.id.tv_result);
    }


    @Override
    public void showToast(String str, int length) {
        String content = str + "&" + length;
        Toast.makeText(this,content,Toast.LENGTH_LONG).show();
    }

    @Override
    public void showDialog(String str, int length) {
        Dialog dialog = new Dialog(this);
        String content = str + "&" + length;
        TextView tv_content = new TextView(this);
        tv_content.setText(content);
        dialog.setContentView(tv_content);
        dialog.show();
    }

    @Override
    public void showCountDown(int count) {
        tv_countDown_number.setText(String.valueOf(count));
    }

    @Override
    public void showMessage(String msg) {
        tv_showResult.setText(msg);
    }

    @Override
    public void onClick(View v) {
        int id = v.getId();
        //ToDo 在每个case下实现presenter相应方法并调用
        switch (id){
            case R.id.btn_showtoast:
                break;
            case R.id.btn_showdialog:
                break;
            case R.id.btn_countdown_reset:
                break;
            case R.id.btn_countdown_start:
                break;
            case R.id.btn_request:
                break;
        }
    }
}
```
接下来， 实现`Presenter`和`View`一样，我们先定义它的所有行为。`View`的行为是在监听到用户事件后让`Presenter`调用的，而`Presenter`则是让`Model`在完成耗时操作或计算或网络请求的时候调用。

``` android
public interface DemoPresenterInterface {
    /**
     * 完成字符串长度的计算
     * @param str 字符串
     * @param length 字符串的长度
     */
    void completeStringCount(String str, int length);

    /**
     * 完成1秒的计数
     * @param count 当前的计数
     */
    void completeCountDownASecend(int count);

    /**
     * 完成网络请求
     * @param result 网络请求的结果
     */
    void completeRequest(String result);
}
```
Presenter的实现

``` android
public class DemoPresenter implements DemoPresenterInterface {
    /*
    * 在View 已经实现了的接口
    * */
    private DemoViewInterface demoViewInterface;

    // TODO: 实现DemoModel
    private DemoModel demoModel;
    /*
    * 判断是用吐司还是弹窗显示的变量
    * */
    private Boolean isToast;

    /**
     * 构造函数
     * @param demoViewInterface
     */
    DemoPresenter(DemoViewInterface demoViewInterface){
        this.demoViewInterface = demoViewInterface;
        isToast = true;//设置一个初始值
    }
    /*
    * 以下是创建供View调用的方法,具体的实现,要完成Model后才可以操作
    * */
    public void clickShowToast(){
        isToast = true;
        // TODO: 实现model对应方法并调用
    }
    public void clickShowDialog(){
        isToast = false;
        // TODO: 实现model对应方法并调用
    }
    public void clickCountDownStart(){
        // TODO: 实现model对应方法并调用
    }
    public void clickCountDownRestart(){
        // TODO: 实现model对应方法并调用
    }
    public void clickRequest(){
        // TODO: 实现model对应方法并调用
    }
    /*
    * 以下是对DemoPresenterInterface的实现
    * */
    @Override
    public void completeStringCount(String str, int length) {
        if (isToast) demoViewInterface.showToast(str,length);
        else demoViewInterface.showDialog(str,length);
    }

    @Override
    public void completeCountDownASecend(int count) {
        demoViewInterface.showCountDown(count);
    }

    @Override
    public void completeRequest(String result) {
        demoViewInterface.showMessage(result);
    }
}
```
最后，实现`Model`，这里不需要定义行为，因为它是`Presenter`私有成员变量，直接定义方法就可以让`Presenter`调用。

```android
public class DemoModel {

    private final int MAX_COUNTER = 10;
    private DemoPresenterInterface demoPresenterInterface;
    private int counter;
    public DemoModel(DemoPresenterInterface demoPresenterInterface){
        this.demoPresenterInterface = demoPresenterInterface;
        resetCounter();
    }

    private void resetCounter() {
        counter = MAX_COUNTER;
    }

    public void calculationStringLength(String str){

        demoPresenterInterface.completeStringCount(str,str.length());

    }
    /*
    * 子线程不能直接操作UI线程,通过handler解决
    * */
    private final android.os.Handler handler = new android.os.Handler() {
        @Override
        public void handleMessage(Message msg) {
            demoPresenterInterface.completeCountDownASecend(counter);
            super.handleMessage(msg);
        }
    };
    public void countDownASecond(){
        //耗时操作, 开个线程
        new Thread(){
            @Override
            public void run() {
                while (counter > 0)
                try {
                    Thread.sleep(1000);
                    counter --;
                    handler.sendEmptyMessage(0);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }.start();
    }

    public void countDownReset(){
        resetCounter();
        demoPresenterInterface.completeCountDownASecend(counter);
    }
    public void request(){
        //请求数据的URL
        String URL = "http://www.icityto.com/X_UserLogic/yesicity2015/nickname_Page/?Uid=yesicity2015&Field_YHID=559&Yesicity=1&Id=552";
        JsonObjectRequest request = new JsonObjectRequest(URL, null,
                new Response.Listener<JSONObject>() {
                    @Override
                    public void onResponse(JSONObject jsonObject) {
                        //请求成功回调
                        demoPresenterInterface.completeRequest(jsonObject.toString());
                    }
                },
                new Response.ErrorListener() {
                    @Override
                    public void onErrorResponse(VolleyError volleyError) {
                        //请求失败回调
                        demoPresenterInterface.completeRequest(volleyError.toString());
                    }
                });
        RequestQueue.getmQueue().add(request);
    }
}
```
下一步就是把之前的`todo`完成掉。

**`DemoViewActivity`中`onClick`里的`todo`**

```android
@Override
    public void onClick(View v) {
        int id = v.getId();
        switch (id){
            case R.id.btn_showtoast:
                demoPresenter.clickShowToast(et_input.getText().toString());
                break;
            case R.id.btn_showdialog:
                demoPresenter.clickShowDialog(et_input.getText().toString());
                break;
            case R.id.btn_countdown_reset:
                demoPresenter.clickCountDownRestart();
                break;
            case R.id.btn_countdown_start:
                demoPresenter.clickCountDownStart();
                break;
            case R.id.btn_request:
                demoPresenter.clickRequest();
                break;
        }
    }
```

**`DemoPresenter`中的`todo`**

```
	public void clickShowToast(String str){
        isToast = true;
        demoModel.calculationStringLength(str);
    }
    public void clickShowDialog(String str){
        isToast = false;
        demoModel.calculationStringLength(str);
    }
    public void clickCountDownStart(){
        demoModel.countDownASecond();
    }
    public void clickCountDownRestart(){
        demoModel.countDownReset();
    }
    public void clickRequest(){
        demoModel.request();
    }
```
至此`MVP`模式的基本实现完成

