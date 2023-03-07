## THE DEFFERENCE BETWEEN SVG ELEMENT(CODE) AND SVG FILE

- reference: 

[How can I save svg code as a .svg image? - Stack Overflow](https://stackoverflow.com/questions/12628968/how-can-i-save-svg-code-as-a-svg-image)

- conclusions:

"Just had to make sure it has attribute: xmlns and versions, and then save it in a text file and give it a .svg extension. Also make sure to add xml tag a the top of the svg file just before pasting the svg tag \<?xml version="1.0" encoding="utf-8"?\>"

## SAVE (RATHER: DOWNLOAD) THE SVG ELEMENT:

- refenece: [javascript - How do I save/export an SVG file after creating an SVG with D3.js (IE, safari and chrome)? - Stack Overflow](https://stackoverflow.com/questions/23218174/how-do-i-save-export-an-svg-file-after-creating-an-svg-with-d3-js-ie-safari-an)

- code (which was  used in NeatSankey) :

```js
function save_as_svg(){
        var svg_data = document.getElementById("svg").innerHTML //put id of your svg element here

        var head = '<svg title="graph" version="1.1" xmlns="http://www.w3.org/2000/svg">'

        //if you have some additional styling like graph edges put them inside <style> tag

        var style = '<style></style>'

        var full_svg = head +  style + svg_data + "</svg>"
        var blob = new Blob([full_svg], {type: "image/svg+xml"});  
        saveAs(blob, "graph.svg");
};
```

## METHOD NOT EXAMINED:

- reference: [URL.createObjectURL() - Web API 接口参考 | MDN (mozilla.org)](https://developer.mozilla.org/zh-CN/docs/Web/API/URL/createObjectURL)

- code:

```js
var svgData = $("#figureSvg")[0].outerHTML;
var svgBlob = new Blob([svgData], {type:"image/svg+xml;charset=utf-8"});
var svgUrl = URL.createObjectURL(svgBlob);
var downloadLink = document.createElement("a");

downloadLink.href = svgUrl;
downloadLink.download = "newesttree.svg";
document.body.appendChild(downloadLink);
downloadLink.click();
document.body.removeChild(downloadLink);
```



## MORE RESOURCE

- [eligrey/FileSaver.js: An HTML5 saveAs() FileSaver implementation (github.com)](https://github.com/eligrey/FileSaver.js)

- [jimmywarting/StreamSaver.js: StreamSaver writes stream to the filesystem directly asynchronous (github.com)](https://github.com/jimmywarting/StreamSaver.js)

- [FileSaver.js 介绍 - yunser_blog - 博客园 (cnblogs.com)](https://www.cnblogs.com/yunser/p/7629399.html)



- [Working with files - Mozilla | MDN](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/Working_with_files)