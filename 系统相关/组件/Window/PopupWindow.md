# PopupWindow

## 创建
```
View view = LayoutInflater.from(context).inflate(R.layout.dialog_layout_test, null, false);
PopupWindow popupWindow = new PopupWindow(view, ManyUtilsKt.dp2px(context, 200f), ManyUtilsKt.dp2px(context, 200f), true);
```
## 显示
```
popupWindow.showAsDropDown(attachView);
popupWindow.showAsDropDown(attachView, ManyUtilsKt.dp2px(context, 20f), 0);
popupWindow.showAtLocation(attachView,Gravity.TOP,0,0);
```
showAtLocation是根据屏幕来确定位置的,和attachView位置没关系.
## 动画
```
popupWindow.setAnimationStyle(R.style.anim_popupWindow);

<style name="anim_popupWindow">
    <item name="android:windowEnterAnimation">@anim/popupwindow_in</item>
    <item name="android:windowExitAnimation">@anim/popupwindow_out</item>
</style>
```
## 注意
`popupWindow.setOutsideTouchable(true);`有时候会失效,得设置backgroundDrawable<br>
`popupWindow.setBackgroundDrawable(new ColorDrawable(0x00000000));`

## 和Dialog的区别
* dialog创建的时候可以不传入宽高,popupwindow需要.
* popupwindow默认不响应BACK,除非设置setFocusable(true).
* dialog显示的时候回添加蒙层.
* popuowindow是阻塞式对话框,显示的时候不会响应其他操作,Dialog点击背景会响应,是非阻塞式.