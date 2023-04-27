# 12 DOM2和DOM3

DOM1级主要定义的是HTML和XML文档的底层结构;

DOM2和DOM3级则在这个结构的基础上引入了更多的交互能力,也支持了更高级的XML特性;

DOM2和3分成许多模块:DOM2级核心,DOM2级视图,DOM2级事件,DOM2级样式,DOM2级遍历和范围,DOM2级HTML;

# 12.1 DOM变化

检测浏览器是否支持DOM2,3的模块;

```
var supportsDOM2Core = document.implementation.hasFeature("Core", "2.0");
var supportsDOM3Core = document.implementation.hasFeature("Core", "3.0");
var supportsDOM2HTML = document.implementation.hasFeature("HTML", "2.0");
var supportsDOM2Views = document.implementation.hasFeature("Views", "2.0");
var supportsDOM2XML = document.implementation.hasFeature("XML", "2.0");
```

本章只讨论那些已经有浏览器实现的部分,任何浏览器都没有实现的部分将不作讨论;

哦233

## 针对XML命名空间的变化

从技术上说,HTML不支持XML命名空间,但XHTML支持XML命名空间;

因此,本节给出的都是XHTML的示例;

```
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<title>Example XHTML page</title>
</head>
<body>
Hello world!
</body>
</html>
```

使用xmlns特性来指定,格式良好的XHTML页面中,应包裹在html元素中;

混用两种语言的话,命名空间的作用就凸显出来了;

```
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<title>Example XHTML page</title>
</head>
<body>
<svg xmlns="http://www.w3.org/2000/svg" version="1.1"
viewBox="0 0 100 100" style="width:100%; height:100%">
<rect x="0" y="0" width="100" height="100" style="fill:red"/>
</svg>
</body>
</html>
```

但是问题也来了,创建元素的时候,到底是创建哪个命名空间的元素呢;

"DOM2 级核心"通过为大多数DOM1级方法提供特定于命名空间的版本解决了这个问题;

### Node类型的变化

DOM2级中,Node类型包涵一下特定于命名空间的属性:

1.localName:不带命名空间前缀的节点名称;

2.namespaceURI:命名空间URI或者(未指定的情况下)null;

3.prefix:命名空间前缀或者(未指定的情况下)null;

```
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<title>Example XHTML page</title>
</head>
<body>
<s:svg xmlns:s="http://www.w3.org/2000/svg" version="1.1"
viewBox="0 0 100 100" style="width:100%; height:100%">
<s:rect x="0" y="0" width="100" height="100" style="fill:red"/>
</s:svg>
</body>
</html>
```

对于`<html>`元素来说,它的localName和tagName是"html",

namespaceURI 是"http://www.w3.org/1999/xhtml",而prefix是null;

对于`<s:svg>`元素而言,它的localName是"svg",tagName是"s:svg",

namespaceURI是"http://www.w3.org/2000/svg",而prefix是"s";

DOM3引入了更多的方法;

1.isDefaultNamespace(namespaceURI):在指定的namespaceURI是当前节点的默认命名空间的情况下返回true;

2.lookupNamespaceURI(prefix):返回给定prefix的命名空间;

3.lookupPrefix(namespaceURI):返回给定namespaceURI的前缀;

```
alert(document.body.isDefaultNamespace("http://www.w3.org/1999/xhtml"); //true
//假设svg 中包含着对<s:svg>的引用
alert(svg.lookupPrefix("http://www.w3.org/2000/svg")); //"s"
alert(svg.lookupNamespaceURI("s")); //"http://www.w3.org/2000/svg"
```

这样就获知了某个节点与文档其他元素之间的关系;

### Document类型的变化

DOM2级中,与命名空间有关的新方法;

1.createElementNS(namespaceURI, tagName):使用给定的tagName创建一个属于命名空间namespaceURI的新元素;

2.createAttributeNS(namespaceURI, attributeName):使用给定的attributeName创建一个属于命名空间namespaceURI的新特性;

3.getElementsByTagNameNS(namespaceURI, tagName):返回属于命名空间namespaceURI的tagName元素的NodeList;

```
//创建一个新的SVG元素
var svg = document.createElementNS("http://www.w3.org/2000/svg","svg");
//创建一个属于某个命名空间的新特性
var att = document.createAttributeNS("http://www.somewhere.com", "random");
//取得所有XHTML元素
var elems = document.getElementsByTagNameNS("http://www.w3.org/1999/xhtml", "*");
```

### Element类型的辩护

DOM2级核心中,操作特性的新方法;

1.getAttributeNS(namespaceURI,localName):

取得属于命名空间namespaceURI且名为localName的特性;

2.getAttributeNodeNS(namespaceURI,localName):

取得属于命名空间namespaceURI且名为localName的特性节点;

3.getElementsByTagNameNS(namespaceURI, tagName):

返回属于命名空间namespaceURI的tagName元素的NodeList;

4.hasAttributeNS(namespaceURI,localName):

确定当前元素是否有一个名为localName的特性,而且该特性的命名空间是namespaceURI;

注意,"DOM2 级核心"也增加了一个hasAttribute()方法,用于不考虑命名空间的情况;

5.removeAttriubteNS(namespaceURI,localName):

删除属于命名空间namespaceURI且名为localName的特性;

6.setAttributeNS(namespaceURI,qualifiedName,value):

设置属于命名空间namespace-URI且名为qualifiedName的特性的值为value;

7.setAttributeNodeNS(attNode):

设置属于命名空间namespaceURI的特性节点;

### NamedNodeMap类型的变化

新方法:

1.getNamedItemNS(namespaceURI,localName):

取得属于命名空间namespaceURI且名为localName的项;
 
2.removeNamedItemNS(namespaceURI,localName):

移除属于命名空间namespaceURI且名为localName的项;

4.setNamedItemNS(node):

添加node,这个节点已经事先指定了命名空间信息;

由于一般都是通过元素访问特性,所以这些方法很少使用;

## 其他方面的变化

确保API可靠性完整性的变化;

### DocumentType类型的变化

新增三个属性:publicId,systemId,internalSubset;

```
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"
"http://www.w3.org/TR/html4/strict.dtd">
```

publicId是"-//W3C//DTD HTML 4.01//EN",

systemId是"http://www.w3.org/TR/html4/strict.dtd";

```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
"http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd"
[<!ELEMENT name (#PCDATA)>] >
```

document.doctype.internalSubset 将得到"<!ELEMENT name (#PCDATA)>";

极少访问这些信息;

### Document类型的变化

importNode()方法,接收两个参数,要复制的节点和一个表示是否复制子节点的布尔值;

返回的结果是原来节点的副本;

```
var newNode = document.importNode(oldNode, true); //导入节点及其所有子节点
document.body.appendChild(newNode);
```

defaultView属性,保存了一个指向拥有给定文档的窗口的指针;

除了IE外所以浏览器都支持这个属性,在IE中,等价属性叫parentWindow;

"DOM2级核心"为document.implementation对象规定了两个新方法:createDocumentType()和createDocument();

前者接收3个参数:文档类型名称,publicId,systemId;

```
var doctype = document.implementation.createDocumentType("html",
"-//W3C//DTD HTML 4.01//EN",
"http://www.w3.org/TR/html4/strict.dtd");
```

createDocument()接受3个参数:

针对文档中元素的namespaceURI,文档元素的标签名,新文档的文档类型;

```
	// 创建一个没有命名空间的新文档,文档元素为<root>,没有指定文档类型;
var doc = document.implementation.createDocument("", "root", null);

// 创建一个XHTML文档
var doctype = document.implementation.createDocumentType("html",
" -//W3C//DTD XHTML 1.0 Strict//EN",
"http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd");
var doc = document.implementation.createDocument("http://www.w3.org/1999/xhtml",
"html", doctype);
```

"DOM2级HTML"模块也为document.implementation新增了一个方法,名叫createHTMLDocument();

这个方法的用途是创建一个完整的HTML文档;

包括`<html>`,`<head>`,`<title>`和`<body>`元素;

这个方法只接受一个参数,即新创建文档的标题(放在`<title>`元素中的字符串);

返回新的HTML文档;

通过调用createHTMLDocument()创建的这个文档,是HTMLDocument类型的实例;

因而具有该类型的所有属性和方法,包括title和body属性;

### Node类型的变化

isSupported(),接收两个参数:特性名和特性版本号;

```
if (document.body.isSupported("HTML", "2.0")){
//执行只有"DOM2 级HTML"才支持的操作
}
```

DOM3级引入了两个辅助比较节点的方法:isSameNode()和isEqualNode();

这两个方法都接受一个节点参数,并在传入节点与引用的节点相同或相等时返回true;

所谓相同,指的是两个节点引用的是同一个对象;

所谓相等,指的是两个节点是相同的类型,具有相等的属性(nodeName,nodeValue,等等),

而且它们的attributes和childNodes属性也相等(相同位置包含相同的值);

```
var div1 = document.createElement("div");
div1.setAttribute("class", "box");
var div2 = document.createElement("div");
div2.setAttribute("class", "box");
alert(div1.isSameNode(div1)); //true
alert(div1.isEqualNode(div2)); //true
alert(div1.isSameNode(div2)); //false
```

DOM3级还针对为DOM节点添加额外数据引入了新方法;

setUserData()方法会将数据指定给节点,接受3个参数:

要设置的键,实际的数据(可以是任何数据类型)和处理函数;

```
document.body.setUserData("name", "Nicholas", function(){});
```

getUserData()并传入相同的键,就可以取得该数据;

```
var value = document.body.getUserData("name");
```

传入setUserData()中的处理函数会在带有数据的节点被复制,删除,重命名或引入一个文档时调用;

因而你可以事先决定在上述操作发生时如何处理用户数据;

处理函数接受5个参数:

1.表示操作类型的数值(1 表示复制,2 表示导入,3 表示删除,4 表示重命名);

2.数据键;

3.数据值;

4.源节点;

5.目标节点;

在删除节点时,源节点是null;除在复制节点时,目标节点均为null;

在函数内部,你可以决定如何存储数据;

```
var div = document.createElement("div");
div.setUserData("name", "Nicholas", function (operation, key, value, src, dest) {
    if (operation == 1) {
        dest.setUserData(key, value, function () {
        });
    }
});
var newDiv = div.cloneNode(true);
alert(newDiv.getUserData("name")); //"Nicholas"
```

这里,先创建了一个<div>元素,然后又为它添加了一些数据(用户数据);

在使用cloneNode()复制这个元素时,就会调用处理函数,

从而将数据自动复制到了副本节点;

结果在通过副本节点调用getUserData()时,就会返回与原始节点中包含的相同的值;

### 框架的变化

框架和内嵌框架分别用HTMLFrameElement 和HTMLIFrameElement表示,

它们在DOM2级中都有了一个新属性,名叫contentDocument;

这个属性包含一个指针,指向表示框架内容的文档对象;

```
var iframe = document.getElementById("myIframe");
var iframeDoc = iframe.contentDocument; //在IE8 以前的版本中无效
```

IE8之前不支持框架中的contentDocument属性,但支持一个名叫contentWindow的属性;

所以;

```
var iframe = document.getElementById("myIframe");
var iframeDoc = iframe.contentDocument || iframe.contentWindow.document;
```