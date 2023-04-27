# 10.2 DOM操作技术

因为浏览器的坑,有的时候DOM的操作并不简明;

## 动态脚本

创建script节点;

```
function loadScript(url){
	var script = document.createElement("script");
	script.type = "text/javascript";
	script.src = url;
	document.body.appendChild(script);
}
```

在IE中`<script>`是一种特殊元素,不允许DOM访问子节点;

```
// 其他浏览器可以
var script = document.createElement("script");
script.type = "text/javascript";
script.appendChild(document.createTextNode("function sayHi(){alert('hi');}"));
document.body.appendChild(script);

// 兼容IE
var script = document.createElement("script");
script.type = "text/javascript";
script.text = "function sayHi(){alert('hi');}";
document.body.appendChild(script);

// 兼容早期Safari和IE
var script = document.createElement("script");
script.type = "text/javascript";
var code = "function sayHi(){alert('hi');}";
try {
	script.appendChild(document.createTextNode("code"));
} catch (ex){
	script.text = "code";
}
document.body.appendChild(script);
```

Safari不能支持text但是可以使用文本节点,所以try;

以这种方式加载的代码会在全局作用域中执行,而且当脚本执行后将立即可用;

实际上,这样执行代码与在全局作用域中把相同的字符串传递给eval()是一样的;

## 动态样式

同样的,css也可以;

```
function loadStyles(url){
	var link = document.createElement("link");
	link.rel = "stylesheet";
	link.type = "text/css";
	link.href = url;
	var head = document.getElementsByTagName("head")[0];
	head.appendChild(link);
}
```

加载外部样式文件的过程是异步的,也就是加载样式与执行JavaScript代码的过程没有固定的次序;

一般来说,知不知道样式已经加载完成并不重要;

第13章将讨论利用事件来检测这个过程是否完成;

同样的,IE来了;

```
var style = document.createElement("style");
style.type = "text/css";
try{
	// 普通浏览器
	style.appendChild(document.createTextNode("body{background-color:red}"));
} catch (ex){
	// IE浏览器
	style.styleSheet.cssText = "body{background-color:red}";
}
var head = document.getElementsByTagName("head")[0];
head.appendChild(style);
```

这种方式会实时地向页面中添加样式,因此能够马上看到变化;

如果专门针对IE编写代码,务必小心使用styleSheet.cssText属性;

在重用同一个`<style>`元素并再次设置这个属性时,有可能会导致浏览器崩溃;

同样,将cssText属性设置为空字符串也可能导致浏览器崩溃;

IE666;

## 操作表格

显然,使用之前介绍的方式创建一个表格要写死人;

HTML DOM不想你死,于是就有了;

为`<table>`元素添加的属性和方法如下;

1.caption:保存着对`<caption>`元素(如果有)的指针;

2.tBodies:是一个`<tbody>`元素的HTMLCollection;

3.tFoot:保存着对`<tfoot>`元素(如果有)的指针;

4.tHead:保存着对`<thead>`元素(如果有)的指针;

5.rows:是一个表格中所有行的HTMLCollection;

6.createTHead():创建`<thead>`元素,将其放到表格中,返回引用;

7.createTFoot():创建`<tfoot>`元素,将其放到表格中,返回引用;

8.createCaption():创建`<caption>`元素,将其放到表格中,返回引用;

9.deleteTHead():删除`<thead>`元素;

10.deleteTFoot():删除`<tfoot>`元素;

11.deleteCaption():删除`<caption>`元素;

12.deleteRow(pos):删除指定位置的行;

13.insertRow(pos):向rows集合中的指定位置插入一行;

为`<tbody>`元素添加的属性和方法如下;

1.rows:保存着`<tbody>`元素中行的HTMLCollection;

2.deleteRow(pos):删除指定位置的行;

3.insertRow(pos):向rows集合中的指定位置插入一行,返回对新插入行的引用;

为`<tr>`元素添加的属性和方法如下;

1.cells:保存着`<tr>`元素中单元格的HTMLCollection;

2.deleteCell(pos):删除指定位置的单元格;

3.insertCell(pos):向cells集合中的指定位置插入一个单元格,返回对新插入单元格的引用;

例子:

```
//创建table
var table = document.createElement("table");
table.border = 1;
table.width = "100%";
//创建tbody
var tbody = document.createElement("tbody");
table.appendChild(tbody);
//创建第一行
tbody.insertRow(0);
tbody.rows[0].insertCell(0);
tbody.rows[0].cells[0].appendChild(document.createTextNode("Cell 1,1"));
tbody.rows[0].insertCell(1);
tbody.rows[0].cells[1].appendChild(document.createTextNode("Cell 2,1"));
//创建第二行
tbody.insertRow(1);
tbody.rows[1].insertCell(0);
tbody.rows[1].cells[0].appendChild(document.createTextNode("Cell 1,2"));
tbody.rows[1].insertCell(1);
tbody.rows[1].cells[1].appendChild(document.createTextNode("Cell 2,2"));
//将表格添加到文档主体中
document.body.appendChild(table);
```

这里文本是cell 列,行;

## 使用NodeList

回顾一下动态集合三兄弟:NodeList,NamedNodeMap,HTMLCollection;

动态的,实时的,忘记的话可能会写出死循环;

```
var divs = document.getElementsByTagName("div"),
i,
div;
for (i=0; i < divs.length; i++){
div = document.createElement("div");
document.body.appendChild(div);
}
```

解决方案当然是暂存一下初始length;

```
var divs = document.getElementsByTagName("div"),
i,
len,
div;
for (i=0, len=divs.length; i < len; i++){
	div = document.createElement("div");
	document.body.appendChild(div);
}
```

# 10.3 小结

DOM操作往往是JavaScript程序中开销最大的部分,而因访问NodeList导致的问题为最多;

NodeList对象都是"动态的",这就意味着每次访问NodeList对象,都会运行一次查询;

有鉴于此,最好的办法就是尽量减少DOM操作;