# ContentProvider
## 是什么
为了提供跨应用的数据共享方案.
## 怎么用
[参考文章1](https://www.jianshu.com/p/5e13d1fec9c9)<br>
[参考文章2](https://www.jianshu.com/p/94b8582d089a)<br>
1. 创建数据列表:继承SQLiteOpenHelper
2. 继承ContentProvider类,实现相关方法
3. manifest中声明provider以及定义相关权限
4. 通过ContentResolver根据URI进行增删改查,ContentObserver来监听provider变化