# Android performance optimization 性能优化

## UI优化

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

## 字符串优化:尽量用StringBuilder代替Stirng<br>  
 速度快慢为：StringBuilder > StringBuffer > String<br>
 更具体可以看[博客园：酥风](http://www.cnblogs.com/su-feng/p/6659064.html)<br>
 
[baidu]:http://www.baidu.com/img/bdlogo.gif "百度Logo" 
 #### String最慢的原因：<br>
　　String为字符串常量，而StringBuilder和StringBuffer均为字符串变量，即String对象一旦创建之后该对象是不可更改的，但后两者的对象是变量，是可以更改的。<br>
 上代码:
 ```
 public class MainActivity extends AppCompatActivity implements View.OnClickListener {
    private  int rowlength=20;
    private  int length=300;
    private int[][] matrix=new int [rowlength][length];
    private Random ran=new Random();//随机数
    private Button btn_1;
    private Button btn_2;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        initView();
        for (int i=0;i<rowlength;i++){
            for (int j=0;j<length;j++){
                matrix[i][j]=ran.nextInt();
            }
        }
    }

    private void initView() {
        btn_1 = (Button) findViewById(R.id.btn_1);
        btn_2 = (Button) findViewById(R.id.btn_2);

        btn_1.setOnClickListener(this);
        btn_2.setOnClickListener(this);
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()) {
            case R.id.btn_1:
                fun_string();
                break;
            case R.id.btn_2:
                fun_stringBuilder();
                break;
        }
    }


    //String拼接，打印起始时间和终止时间
    private void fun_string() {
        String rowStr="";
        System.out.println("doString start:");
        for (int i=0;i<rowlength;i++){
           for (int j=0;j<length;j++){
               rowStr=rowStr+matrix[i][j];
               rowStr=rowStr+".";
           }
           System.out.println("do String row"+i);
       }
        System.out.println("do String rowStr:"+rowStr.toString().length());
    }
    //StringBuilder拼接，打印起始时间和终止时间
    private void fun_stringBuilder() {
        StringBuilder rowStr=new StringBuilder();
        System.out.println("doBuilder start");
        for (int i=0;i<rowlength;i++){
            for (int j=0;j<length;j++){
                rowStr.append(matrix[i][j]);
                rowStr.append(".");
            }
            System.out.println("doBuilder row"+i);
        }
        System.out.println("doBuilder rowStr:"+rowStr.toString().length());
    }
}
 ```
 让我们看看效果
## String
![图3](https://github.com/forevermistery/UI-Optimization/blob/master/screen_shoot/fun_string.png)<br>
从上图我们可以看出，开始时间为`14:16:23.537` 结束时间为`14:16:28.153`花了5秒多
## StringBuilder
![图4](https://github.com/forevermistery/UI-Optimization/blob/master/screen_shoot/fun_stringBuilder.png)<br>
开始时间为`14:16:38.736` 结束时间为`14:16:38.743`飞快的结束任务
#### 由此验证StringBuilder确实效率比string高，性能快



 
## Bitmap 图片优化
#### 主要有三种方法
* 通过缩放图片 ``
* 通过设置格式 `Bitmap.Config`
* 加载大图时 `bitmapRegionDecoder.decodeRegion`
```public class MainActivity extends AppCompatActivity {

    private int height;
    private int width;
    private ImageView img_splah;
    private String url="/mnt/shared/Image/icon.jpg";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        WindowManager wm= (WindowManager) getSystemService(WINDOW_SERVICE);
         width = wm.getDefaultDisplay().getWidth();
         height = wm.getDefaultDisplay().getHeight();
         img_splah=findViewById(R.id.img_splash);

//         img_splah.setLayerType(View.LAYER_TYPE_SOFTWARE, null);//使用后加载大图的黑屏变白屏

        ChangeRGB();
        //load_Part_img();



    }
    //
    private void load_Part_img() {
        File file=new File("/mnt/shared/Image/icon.jpg");
        try {
            FileInputStream inputStream=new FileInputStream(file);
            BitmapRegionDecoder bitmapRegionDecoder=BitmapRegionDecoder.newInstance(inputStream,false);
            //获取图片的宽高
            BitmapFactory.Options tmpOptions=new BitmapFactory.Options();

            int width=tmpOptions.outWidth;
            int height=tmpOptions.outHeight;
            //设置显示图片的中心区域
            BitmapFactory.Options options=new BitmapFactory.Options();
            options.inPreferredConfig=Bitmap.Config.ARGB_4444;
            Bitmap bitmap=bitmapRegionDecoder.decodeRegion(
                    new Rect(width/4-200,height/4-200,width/4+200,height/4+200),options);
            System.out.println("bitmap的大小:"+bitmap.getByteCount());
            img_splah.setImageBitmap(bitmap);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }

    }

    //通过设置解码格式
    private void ChangeRGB() {
        /** A->透明度 R->红  G->绿   B->蓝
         * Bitmap.Config.RGB_565      16位=2个字节   没有A通道，每个像素占用2个字节，图片失真小，但是没有透明度；
         * Bitmap.Config.ARGB_4444    16位=2个字节   四个通道都是4位，每个像素占用2个字节，图片的失真比较严重；
         * Bitmap.Config.ARGB_8888   32位=4个字节    四个通道都是8位，每个像素占用4个字节，图片质量是最高的，但是占用的内存也是最大的；
         * Bitmap.Config.ALPHA_8    8位 =1个字节     只有A通道，每个像素占用1个字节大大小，只有透明度，没有颜色值。
         */
        //[2]图片大小  图片的宽*高*字节数
        BitmapFactory.Options options=new BitmapFactory.Options();
        options.inJustDecodeBounds=true;//不去真真解析位图，但是还能够获取到图片的宽高
        options.inPreferredConfig=Bitmap.Config.ARGB_4444;
        BitmapFactory.decodeFile(url,options);
//        System.out.println("~~~~~~~~~~"+options);
        //[3]获取图片宽 和高
        int imgWidth=options.outWidth;
        int imgHeight=options.outHeight;
        System.out.println("图片的分辨率为"+imgWidth+"*"+imgHeight+"~~~系统的宽高为"+width+"*"+height);

        //[4]计算缩放比 按照大的去缩放
        int scale=1;//定义一个变量 缩放比
        int scaleX=imgWidth/width; //有点怪的是这用int去相除，一般也不会是整数，导致scale初始值(不变)
        int scaleY=imgHeight/height;//
        if (scaleX>=scaleY && scaleY>scale){
                    scale=scaleX;

        }
        if (scaleY>scaleX && scaleY>scale){
            scale=scaleY;

        }
        System.out.println("~~~~~~~~缩放比为:"+scale);
        //[5]按照我们计算出来的缩放比例进行显示图片
        options.inSampleSize=scale;

        //[6]开始真正洗的解析位图
        options.inJustDecodeBounds=false;//去真正解析位图，但是还能获取到图片宽和高
        Bitmap bitmap=BitmapFactory.decodeFile(url,options);
        System.out.println("ARGB处理后```图片大小为"+bitmap.getByteCount());
        img_splah.setImageBitmap(bitmap);//最后一步，把bitmap显示到ImageView上

    }


}
```

    
