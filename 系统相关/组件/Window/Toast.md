# Toast
## Toast设置gravity
```
Toast toast = Toast.makeText(this, "test", Toast.LENGTH_SHORT);
toast.setGravity(Gravity.BOTTOM|Gravity.RIGHT, 0, 0);
toast.show();

val toast = Toast.makeText(this,"test",Toast.LENGTH_SHORT)
toast.setGravity(Gravity.END.or(Gravity.TOP),0,0)
toast.show()
```

## 自定义Toast布局
```
Toast.setView(View view)
```

## 内存泄漏
toast使用的是Activity的context的话,在toast消失之前,toast持有Activity,可能导致Activity内存泄漏,所以推荐用Application的context.

## 显示数量限制
7.0的源码显示非系统的toast有数量显示,8.0没有这个限制.

## show()
```
public void show() {
    if (mNextView == null) {
        throw new RuntimeException("setView must have been called");
    }

    INotificationManager service = getService();
    String pkg = mContext.getOpPackageName();
    TN tn = mTN;
    tn.mNextView = mNextView;

    try {
        service.enqueueToast(pkg, tn, mDuration);
    } catch (RemoteException e) {
        // Empty
    }
}
```
生成一个tn然后入队.

## Toast展示消失的流程
从Toast.show开始,源码8.0,和7.0有部分差异,但是流程一样.
```
public void show() {
    if (mNextView == null) {
        throw new RuntimeException("setView must have been called");
    }

    INotificationManager service = getService();
    String pkg = mContext.getOpPackageName();
    TN tn = mTN;
    tn.mNextView = mNextView;

    try {
        service.enqueueToast(pkg, tn, mDuration);
    } catch (RemoteException e) {
        // Empty
    }
}
```
构造了一个tn:ITransientNotification.Stub对象,调用service放进队列里.

**NotificationManagerService.mService(IBinder).enqueueToast:**
```
...
ToastRecord record;
record = new ToastRecord(callingPid, pkg, callback, duration, token);
mToastQueue.add(record);
if (index == 0) {
    showNextToastLocked();
}
...
```
构造了一个ToastRecord,其中callback就是ITransientNotification,是tn的代理对象.token是service系统服务生成的,所以Toast可以显示在最上层,没有权限问题.

**showNextToastLocked():**
```
//展示
record.callback.show(record.token);
//消失
scheduleDurationReachedLocked(record);

```
### 展示
record.callback就是tn的代理对象,所以这里是用了RPC调用tn的的show方法,参数中的token就是service窗口的token.回到Toast的tn对象中.

**Toast.TN.show()**
```
mHandler.obtainMessage(SHOW, windowToken).sendToTarget();
```
发送了一个SHOW的消息,看下handler怎么处理的.

**Toast.TN.mHandler**
```
case SHOW: {
    IBinder token = (IBinder) msg.obj;
    handleShow(token);
    break;
}
```
调用了handleShow

**handleShow():**
```
...
mWM = (WindowManager)context.getSystemService(Context.WINDOW_SERVICE);
mParams.token = windowToken;
mWM.addView(mView, mParams);
...
```
用WindowManager和NotificationManagerService生成的token,添加mView.到这里就添加进window并展示了.

### 消失
**scheduleDurationReachedLocked(record):**
```
private void scheduleDurationReachedLocked(ToastRecord r)
{
    mHandler.removeCallbacksAndMessages(r);
    Message m = Message.obtain(mHandler, MESSAGE_DURATION_REACHED, r);
    long delay = r.duration == Toast.LENGTH_LONG ? LONG_DELAY : SHORT_DELAY;
    mHandler.sendMessageDelayed(m, delay);
}
```
发送了一个Message,what是MESSAGE_DURATION_REACHED,delay就是long和short,这里可以看出来toast展示时间只有两个选择.看下handler怎么处理的,注意这个handler是WorkHandler.

**NotificationManagerService.WorkHandler.handleMessage()**
```
...
case MESSAGE_DURATION_REACHED:
    handleDurationReached((ToastRecord)msg.obj);
    break;
...
```
这个方法又调用了cancelToastLocked()

**cancelToastLocked():**
```
ToastRecord record = mToastQueue.get(index);
record.callback.hide();
```
和展示一样,RPC调用了tn的hide方法,也是发送一个HIDE消息让handler调用HandleHide处理.

**Toast.TN.handleHide():**
```
if (mView.getParent() != null) {
    mWM.removeViewImmediate(mView)
}
```
调用windowManager去掉mView,这时候toast就消失了.
