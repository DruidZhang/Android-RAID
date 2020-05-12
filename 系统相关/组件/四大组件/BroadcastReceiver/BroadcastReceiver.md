# BroadcastReceiver

## 一. 简述
BroadcastReceiver用来接收广播,Broadcast发送广播,Intent在广播中用来携带信息.
## 二. 广播分类
1. 有序广播<br>
    根据Receiver的优先级先后收到的广播,优先级高的Receiver可以修改广播内容,也可以拦截广播不向下传递.
    
2. 无序广播<br>
    匹配的Receiver都能收到,没有先后顺序.<br>
  
3. 本地广播<br>
    有序广播和无序广播属于全局广播,发出的广播会被所有APP收到.本地广播只会被APP内的Receiver收到.
    
4. sticky广播<br>
    粘性广播.只要有符合条件的Receiver能接受广播,就会不停的发送广播,知道Receiver告诉它不要再发送广播.
## 三. BroadcastReceiver注册方式
1. 静态注册<br>
    在manifest文件中注册.<br>
    只要APP启动过一次,所注册的静态广播就会生效.无论app是在运行还是停止状态,有对应的广播发出时,系统会遍历所有的清单文件,通知对应的Receiver接受广播.如果想要APP的静态广播不被其他APP的广播干扰,可以在Receiver节点添加`android:exported="false"`属性.
```
<receiver android:name=".broadcastreceiverTest.MyStaticBroadcastReceiver">
        <intent-filter android:priority="2">
            <action android:name="my.broadcast.action"/>
        </intent-filter>
</receiver>
```

2. 动态注册<br>
```
val filter = IntentFilter()
filter.addAction(Intent.ACTION_AIRPLANE_MODE_CHANGED)
val dynamicFilter = IntentFilter().apply {
    addAction(mAction)
}
//设置优先级,int,越大优先级越高
dynamicFilter.priority = 1
with(context) {
    registerReceiver(myBroadcastReceiver, filter)
    registerReceiver(myDynamicBroadcastReceiver, dynamicFilter)
}
```
3. 注意<br>
    动态注册的Receiver优先级大于静态注册的Receiver.
## 四. 系统广播
常用系统广播

|字段|说明|
|:---:| --- |
|Intent.ACTION_AIRPLANE_MODE_CHANGED	|关闭或打开飞行模式时的广播|
|Intent.ACTION_BATTERY_LOW|表示电池电量低|
|Intent.ACTION_BATTERY_OKAY|表示电池电量充足，即从电池电量低变化到充足时会发出广播|
|Intent.ACTION_BOOT_COMPLETED|在系统启动完成后，这个动作被广播一次（只有一次）|
|Intent.ACTION_DATE_CHANGED|设备日期发生改变时会发出此广播|
|Intent.ACTION_HEADSET_PLUG|在耳机口上插入耳机时发出的广播|
|Intent.ACTION_INPUT_METHOD_CHANGED|改变输入法时发出的广播|

## 五. 发送自定义广播

```
val broadcast = Intent(mAction)
broadcast.putExtra("key", "test broadcast")
//无序广播
context.sendBroadcast(broadcast)
//有序广播
context.sendOrderedBroadcast(broadcast, null)
```
