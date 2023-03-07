### 实现方法一：使用input file和FileReader

通过文件选择器<input type='fule' />来实现文件的选中

```html
<input type="file" id="fileInput"> 
	<!-- 添加multiple属性来选择多个文件
		<input type="file" id="fileInput" multiple>
	-->
```



```javascript
document
    .getElementById('fileInput')
    .addEventListener('change', function selectFile(){
    	console.log(this.files);
});
```



### 实现方法二：使用input file和FormData



