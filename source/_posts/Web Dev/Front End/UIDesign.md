响应式（Responsive）
响应式是什么呢？顾名思义，响应式，肯定会自动响应，响应什么？应用程序是在终端屏幕窗口中展示给用户，被用户访问的，那么就是响应屏幕的变化，不同终端屏幕尺寸大小不一致，需要针对不同尺寸屏幕进行不同的展示响应。

响应式（Responsive web design, RWD），是指一套应用程序用户界面（User Interface）能自动响应不同设备窗口或屏幕尺寸（screen size）并且内容，布局渲染表现良好。
自适应（Adaptive）
在响应式设计（RWD）之外，还有一种适配多设备屏幕的方式，自适应设计（Adaptive web design， AWD）。

自适应设计(AWD)，是指一个应用程序使用多版本用户界面，针对不同设备屏幕，服务器端返回不同版本用户界面，供用户访问。









## Design UI For Mobile Phone

网页自适应手机端的方式有两种：

1. PC端的设计与手机端相差不大，利用 @media媒体查询 调整百分比

2. PC端与手机端的设计分离，利用判断设备的方式做301跳转

下面就结合源代码讲一讲如何实际实现这两种方案：

### Method1

#### Step1

在html代码中添加如下代码

```html
<meta charset="utf-8" />
<meta content="width=device-width,initial-scale=1.0,maximum-scale=1.0,user-scalable=0" name="viewport">
<meta content="yes" name="apple-mobile-web-app-capable">
<meta content="yes" name="apple-touch-fullscreen">
<meta content="black" name="apple-mobile-web-app-status-bar-style">
<meta content="320" name="MobileOptimized">
```

 

#### Step2

设置PC端内容区域width:1100px，添加如下代码：

```css
.container{ width:1100px; max-width:100%; }
.container img{ max-width:100%; }
 
@media only screen and (max-width: 480px) {
           //此适配不同手机型号下文字，图片的大小
}
 
@media only screen and (max-width: 375px) {
 
            //此适配不同手机型号下文字，图片的大小
}
 
..//可继续
```

 

### Method2

在PC端公共Header中，添加如下JS

```html
<script type="text/javascript">
        if (window.location.toString().indexOf('pref=padindex') != -1) { } else {
            if (/AppleWebKit.*Mobile/i.test(navigator.userAgent) || (/MIDP|SymbianOS|NOKIA|SAMSUNG|LG|NEC|TCL|Alcatel|BIRD|DBTEL|Dopod|PHILIPS|HAIER|LENOVO|MOT-|Nokia|SonyEricsson|SIE-|Amoi|ZTE/.test(navigator.userAgent))) {
                if (window.location.href.indexOf("?mobile") < 0) {
                    try {
                        if (/Android|Windows Phone|webOS|iPhone|iPod|BlackBerry/i.test(navigator.userAgent)) {
                            window.location.href = "手机端链接";
                        } else if (/iPad/i.test(navigator.userAgent)) {
                        } else {
                        }
                    } catch (e) {
                    }
                }
            }
        }
</script>    
```