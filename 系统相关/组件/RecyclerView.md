# RecyclerView

## 四级缓存
RecyclerView有四级缓存:
* ArrayList<ViewHolder> mAttachedScrap
* ArrayList<ViewHolder> mCachedViews
* ViewCacheExtension mViewCacheExtension
* RecycledViewPool mRecyclerPool
其中ViewCacheExtension是留给自定义的缓存.

|缓存|作用|是否需要OnCreate|是否需要OnBind|
|:---:|:---:|---|---|
|mAttachedScrap|缓存屏幕中可见范围的ViewHolder|NO|NO|
|mCachedViews|缓存滑动时即将与RecyclerView分离的ViewHolder,按子View或id缓存,默认最多2个|NO|NO|
|mViewCacheExtension|自定义缓存|||
|mRecyclerPool|ViewHolder的缓存池,本质上是SparseArray,key是ViewType,value是ArrayList<ViewHolder>,默认每个ArrayList最多存放5个ViewHolder|NO|YES|

按照顺序获取缓存的ViewHolder,要是没有命中的话就从OnCreateViewHolder和OnBindViewHolder来获取.
## 自定义缓存
自定义缓存主要是通过集成ViewCacheExtension实现,这里是由Adapter来管理TYPE_SPECIAL类型的缓存.
```
inner class MyViewCacheExtension : RecyclerView.ViewCacheExtension() {
    override fun getViewForPositionAndType(recycler: RecyclerView.Recycler, pos: Int, viewType: Int): View? {
        return if (viewType == MyTestAdapter.TYPE_SPECIAL) {
            tcv("MyTestAdapter", "get from cache pos: $pos")
            adapter.viewCache.get(pos)
        } else {
            null
        }
    }

}
```
然后给recyclerView设置.
```
//让RecycleView的ViewPool不去缓存自己管理的类型
recycler.recycledViewPool.setMaxRecycledViews(MyTestAdapter.TYPE_SPECIAL, 0)
recycler.setViewCacheExtension(MyViewCacheExtension())
```
## 局部刷新
局部刷新是通过这个方法实现的,可以刷新指定位置的item.
```
public final void notifyItemChanged(int position, @Nullable Object payload) {
    mObservable.notifyItemRangeChanged(position, 1, payload);
}
```
在Adapter中实现这个.
```
@Override
public void onBindViewHolder(@NonNull MyTestViewHolder holder, int position, @NonNull List<Object> payloads) {
    super.onBindViewHolder(holder, position, payloads);
    if (payloads.isEmpty()) {
        ManyUtilsKt.tcv(TAG, "bind data without payloads");
    } else {
        ManyUtilsKt.tcv(TAG, "bind data with payloads");
        //payloads的size总是1
        holder.textView.setText(payloads.get(0) + "");
    }
}
```