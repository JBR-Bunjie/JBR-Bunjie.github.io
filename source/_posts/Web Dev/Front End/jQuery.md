#什么是jQuery：
> jQuery 库是一个 JavaScript 文件，您可以使用 HTML 的<script></script>标签来引用它：

	```html
		<head>
			<script src="jquery-1.10.2.min.js"></script>
		</head>
	```
	/* JavaScript 是 HTML5 以及所有现代浏览器中的默认脚本语言！ */
	/* 所以没有必要再于<script>标签中写上：type="text/javascript" 了 */
##学习jQuery:
1. 查看当前jq版本：
在控制台中，使用$.fn.jquery命令，即可调用出当前页面的jq版本

2. jq语法：











/* 使用jq的代替方案————CDN(内容分发网络)，即，通过CDN来就近引用服务器中存有的jQuery */
/* 1. Staticfile CDN: */
	```html
		<head>
			<script src="https://cdn.staticfile.org/jquery/1.10.2/jquery.min.js"></script>
		</head>
	```
/* 2. 百度 CDN: */
	```html
		<script src="https://apps.bdimg.com/libs/jquery/2.1.4/jquery.min.js"></script>
	```
/* 3. 又拍云 CDN: */
	```html
		<script src="https://upcdn.b0.upaiyun.com/libs/jquery/jquery-2.0.2.min.js"></script>
	```
/* 4. 新浪 CDN: */
	```html
		<script src="https://lib.sinaapp.com/js/jquery/2.0.2/jquery-2.0.2.min.js"></script>
	```
/* 5. Google CDN: */
	```html
		<script src="https://ajax.googleapis.com/ajax/libs/jquery/1.10.2/jquery.min.js"></script>
	```
/* 6. Microsoft CDN: */
	```html
		<script src="https://ajax.aspnetcdn.com/ajax/jquery/jquery-1.9.0.min.js"></script>
	```
/* 使用CDN的两个优势 */
/* 1. 用户在访问其他站点时，已经从CDN处加载过 jQuery。所以结果是，当他们访问您的站点时，会从CDN的缓存中加载 jQuery，这样可以减少加载时间。 */
/* 2. 大多数 CDN 都可以确保当用户向其请求文件时，会从离用户最近的服务器上返回响应，这样也可以提高加载速度。 */