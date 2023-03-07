mipmap-mdpi：108 * 108

mipmap-hdpi：162 * 162

mipmap-xhdpi：216* 216

mipmap-xxhdpi：324 * 324

mipmap-xxxhdpi：432 * 432



mipmap-mdpi 48
ic_launcher.png/ic_launcher_round.png

mipmap-hdpi 72
ic_launcher.png/ic_launcher_round.png

mipmap-xhdpi 96
ic_launcher.png/ic_launcher_round.png

mipmap-xxhdpi 144
ic_launcher.png/ic_launcher_round.png

mipmap-xxxhdpi 192
ic_launcher.png/ic_launcher_round.png



DPI:每英寸像素数

**简单的屏幕分辨率计算方法：**

DisplayMetrics metrics = this.getResources().getDisplayMetrics();
float density = metrics.density;
int dpi = metrics.densityDpi;
int heightPixels = metrics.heightPixels;
int widthPixels = metrics.widthPixels;
Log.e("---metrics---", "比例:"+density+"dpi:"+dpi+"高像素:"+heightPixels+"宽像素:"+widthPixels);

**图片大小适应不同屏幕：**

```java
img.post(new Runnable() {
   @Override
	public void run() {
	int spec = View.MeasureSpec.makeMeasureSpec(0, View.MeasureSpec.UNSPECIFIED);
	img.measure(spec,spec);
	int measuredWidth=img.getMeasuredWidth();int measuredHeight=img.getMeasuredHeight();//原始大小
	if (measuredWidth==0){
		return;
  	}
 
    int width=img.getWidth();int height=img.getHeight();//真实大小
    LinearLayout.LayoutParams lp=(LinearLayout.LayoutParams)img.getLayoutParams();
	lp.width=width;
	lp.height=width*(measuredHeight / measuredWidth);
	img.setLayoutParams(lp);//设置大小
	}
});
```

**dp与px计算图（mdpi  1dp=1px）：**

 

ldpi:1dp=0.75px  mdpi:1dp=1px  hdpi:1dp=1.5px  xhdpi:1dp=2px  xxhdpi:1dp=3px  xxxhdpi:1dp=4px

Android手机屏幕标准            对应图标尺寸标准    屏幕密度     比例

xxxhdpi 3840*2160              192*192       640      16

xxhdpi 1920*1080               144*144       480      12

xhdpi  1280*720               96*96        320      8

hdpi  480*800               72*72        240      6

mdpi  480*320               48*48        160      4

ldpi  320*240               36*36        120      3

注：Android studio mipmap文件夹只存放启动图标icon

http://blog.csdn.net/a704755096/article/details/46342689

**屏幕横竖屏布局切换：**

1)单个布局xml直接横竖屏切换，不重新加载数据：android:configChanges="orientation|keyboardHidden|screenSize"

2)layout-land和layout-port布局横竖屏切换，不重新加载数据：FragmentActivity重写onRetainCustomNonConfigurationInstance()

Activity 重写onRetainNonConfigurationInstance()保存数据，在onCreate()时判断getLastNonConfigurationInstance()是否null:

Java代码 ![收藏代码](https://imgconvert.csdnimg.cn/aHR0cDovL2lwam1jLml0ZXllLmNvbS9pbWFnZXMvaWNvbl9zdGFyLnBuZw)

1. @Override 
2. **public** **void** onCreate(Bundle savedInstanceState) { 
3.   **super**.onCreate(savedInstanceState); 
4.   setContentView(R.layout.main); 
5.  
6.   Object data = getLastNonConfigurationInstance(); 
7.   **if** (data == **null**) { 
8. ​    findviewbyidLoadMyData(); 
9.   } 
10.   ... 
11. }  

*更多：安卓图片动画（http://www.open-open.com/lib/view/open1335777066015.html）、点九图工具：*

*1.打开Android 工程包 SDK文件，tools文件，双击draw9patch.bat*

*2.弹出的窗口点击 File，点击要编辑的图片open 9patch* 

*3.编辑。鼠标左键：划线   /   shift + 鼠标左键：删除划线*

*4.保存。点击save 9patch*