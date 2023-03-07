[FormData 对象的使用 - Web API 接口参考 | MDN (mozilla.org)](https://developer.mozilla.org/zh-CN/docs/Web/API/FormData/Using_FormData_Objects#使用formdata对象上传文件)

> FormData对象用以将数据编译成键值对，以便用`XMLHttpRequest`来发送数据。其主要用于发送表单数据，但亦可用于发送带键数据(keyed data)，而独立于表单使用。如果表单`enctype`属性设为multipart/form-data ，则会使用表单的[`submit()`](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLFormElement/submit)方法来发送数据，从而，发送数据具有同样形式。

## 从零开始创建FormData对象

你可以自己创建一个`FormData`对象，然后调用它的[`append()`](https://developer.mozilla.org/zh-CN/docs/Web/API/FormData/append)方法来添加字段，像这样：

```js
var formData = new FormData();

// 发送键值对
formData.append("username", "Groucho");
formData.append("accountnum", 123456); //数字123456会被立即转换成字符串 "123456"

// HTML 文件类型input，由用户选择
formData.append("userfile", fileInputElement.files[0]); // fileInputElement是 file类型的input标签

// JavaScript file-like 对象
var content = '<a id="a"><b id="b">hey!</b></a>'; // 新文件的正文
var blob = new Blob([content], { type: "text/xml"});

formData.append("webmasterfile", blob);

// 发送
var request = new XMLHttpRequest();
request.open("POST", "http://foo.com/submitform.php");
request.send(formData);
```

## 上传文件思路：

利用h5的`input`标签，将其`type`设置为`file`

[\<input type="file"\> - HTML（超文本标记语言） | MDN (mozilla.org)](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/Input/file)

此时，注意其一特殊的内置属性：`files`属性，它会以`FileList`对象的形式列出已选择的文件

[FileList - Web API 接口参考 | MDN (mozilla.org)](https://developer.mozilla.org/zh-CN/docs/Web/API/FileList)

值得注意的是，`Filelist`不是`formdata`，请不要尝试直接将`Filelist`对象发送给后端

同时，如需发送`formdata`，请先行创建一个`FormData`对象



`FormData`并非一定需要一个`form`来作为基础进行创建，但一定需要键值对

也可以利用

```js
formdata.append('file', document.getElementById("fileInput").files[0])
```

手动将传入的文件加入formdata对象中



发送请求时，需要特别设置其请求头：

```js
headers: {
      'Content-Type': "multipart/form-data",
}
```

防止所发送的文件被错误地转换了格式



### 循环获取formdata内容

[javascript - How to inspect FormData? - Stack Overflow](https://stackoverflow.com/questions/17066875/how-to-inspect-formdata)

```js
for (var [a, b] of formData.entries()) {
  console.log(a, b, '--------------');
}
```



```js
formdata.get(key)
```



> **注意**: 所有的输入元素都需要有**name**属性，否则无法访问到值。
>
> ——[FormData() - Web API 接口参考 | MDN (mozilla.org)](https://developer.mozilla.org/zh-CN/docs/Web/API/FormData/FormData)



## 上传文件完整攻略：

```html
<body>
<form id="form">
    <label>Choose File to Send: </label>
    <input type="file" multiple name="file" required/> <!-- 注意，用于输入的input一定要有name -->
    <br />
    <input type="submit" value="Send the file!"/>
</form>
<script>
    var form = document.getElementById('form');
    form.addEventListener('submit', function (ev) {
        var oData = new FormData(form);

        axios({
            method: "post",
            url: "upload",
            headers: {
                'Content-Type': "multipart/form-data",
            },
            data: oData,
            timeout: 3000,
        }).then(res => {
            console.log(res)
        })
        ev.preventDefault(); // 阻止对应组件的默认行为,此处为防止页面刷新导致努力ba
    }, false);
</script>
</body>
```

```python
# flask 后台
@app.route('/upload', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        print(request.files)

        print(request.files.getlist('file'))

        upload_files = request.files.getlist('file')

        for i in upload_files:
            i.save(i.filename + '.jpg')
        return "OK!"
    else:
        return "Wrong!"
```

