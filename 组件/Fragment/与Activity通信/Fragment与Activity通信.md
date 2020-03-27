# Fragment与Activity通信
[Fragment参考文章](https://juejin.im/post/5cda8b81f265da034c704ab1)
## Activity<->Fragment
* 用接口,Activity实现接口,fragment调用
* 在构造fragment的时候,通过Fragment.setArguments(Bundle)来传入数据
* Activity调用Fragment的public方法
## Fragment<->Fragment
通过setTargetFragment,然后getTargetFragment().onActivityResult().接受的Fragment用onActivityResult接收.

