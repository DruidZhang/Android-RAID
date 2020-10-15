# Dialog
## 默认dialog创建方式
通过AlertDialog.Builder,可以创建有确定取消的dialog,单选dialog,多选dialog,列表dialog.
## 自定义dialog
### 1. 通过builder
```
AlertDialog.Builder builder = new AlertDialog.Builder(context);
View view = LayoutInflater.from(context).inflate(R.layout.dialog_layout_test,null,false);
builder.setView(view);
final AlertDialog dialog = builder.create();
TextView okButton = view.findViewById(R.id.dialog_ok);
okButton.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View view) {
        ToastUtil.show("click ok");
        dialog.dismiss();
    }
});
dialog.show();
```
#### 2. 继承Dialog
继承Dialog然后定义逻辑.
## DialogFragment
Android推荐用DialogFragment.
1. 重写onCreateDialog方法,返回一个dialog,创建dialog的方法如上.
2. 也可以像普通Fragment重写onCreateView.