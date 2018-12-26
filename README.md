# UI-Optimization
### UI优化

总而言之，就是布局结构越复杂，系统需要分配CPU/GPU渲染的内存越多,UI界面体验越不良好，因此UI优化就是尽可能的减少布局层次
* 1 编写布局时尽量用RelativeLayout 在复杂布局时候有可能减少嵌套层数

* 2![图1](https://github.com/forevermistery/UI-Optimization/blob/master/screen_shoot/1.png)
如上图，我们可以清晰的看出最外层是一个相对布局包裹起来的结构,一般来说，当我们创建布局文件时,如图2
![图2](https://github.com/forevermistery/UI-Optimization/blob/master/screen_shoot/2.png)，布局层级是由最外层DecorView包着LinerLayout，下一层则是一个TitleBar(4.0后的叫法,4.0之前叫Viewstub)和一个id为content的framLayout组成 (ps:这也是为什么创建activity为什么方法叫setcontentView()而不是setView()),frameLayout下面就是我们的布局代码能看到的东西了
* 3 为了优化UI减少层级 可以使用merge标签和viewstub和include(最常用)<br>
首先用得最多的应该是include，按照官方的意思，include就是为了解决重复定义相同布局的问题。例如你有五个界面，这五个界面的顶部都有布局一模一样的一个返回按钮和一个文本控件，在不使用include的情况下你在每个界面都需要重新在xml里面写同样的返回按钮和文本控件的顶部栏，这样的重复工作会相当的恶心。使用include标签，我们只需要把这个会被多次使用的顶部栏独立成一个xml文件，然后在需要使用的地方通过include标签引入即可。
* 4值得一提的事
```Java
btn = findViewById(R.id.btn);
btn.imageView.setVisibility(View.VISIBLE);√
    imageView.setVisibility(View.GONE);//使用GONE会改变组件大小，不推荐使用(性能差
    imageView.setVisibility(View.INVISIBLE);√
```
在android虚拟机运行时，当界面大小发生变化时，系统会重新测量组件布局，需要重新绘制，会消耗一定的内存,所以不推荐使用
* 5当使用fragment时 使用系统自带的fragment而不要使用v4包里的,v4包里的fragment使用时会多一些层级，而且没啥用

    
