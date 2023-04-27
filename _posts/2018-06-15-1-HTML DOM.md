## Document Object Model

JavaScript对大小写敏感;

<pre>
<code>getElementById;
getElementByTagName;</code>
</pre>


通过类名查找HTML元素在IE5,6,7,8中无效;

`document.write(Date());`

绝不要使用在文档加载之后使用 `document.write()`,这会覆盖该文档.

    document.getElementById(id).innerHTML = new HTML

改变内容

    document.getElementById(id).attribute = new value

改变属性

    document.getElementById(id).style.property=new style

改变样式

    <p id="p2"> Hello world!</p>
    <script>
      document.getElementById("p2").style.color = "blue";
    </script>


## 响应事件
### onclick

举例

    <!DOCTYPE html>
    <html>
    <head>
		<script>
		function changetext(id) {
			id.innerHTML="谢谢!";
		}
		</script>
	</head>
	<body>
		<h1 onclick="changetext(this)">请点击该文本</h1>
	</body>
	</html>

或者

	<script>
		document.getElementById("myBtn").onclick=function(){displayDate()};
	</script>

### onload和onunload事件
进入或离开页面时触发
onload 事件可用于检测访问者的浏览器类型和浏览器版本,并基于这些信息来加载网页的正确版本.
onload 和 onunload 事件可用于处理 cookie.

### onchange 事件
onchange 事件常结合对输入字段的验证来使用.
下面是一个如何使用 onchange 的例子,当用户改变输入字段的内容时,会调用 `upperCase()` 函数.
实例

	<input type="text" id="fname" onchange="upperCase()">

### onmouseover 和 onmouseout 事件
onmouseover 和 onmouseout 事件可用于在用户的鼠标移至 HTML 元素上方或移出元素时触发函数,
onmousedown、onmouseup 以及 onclick 事件构成了鼠标点击事件的所有部分
### onfocus事件

## 节点
举例

	<div id="div1">
		<p id="p1">这是一个段落</p>
		<p id="p2">这是另一个段落</p>
	</div>

	<script>
	var para=document.createElement("p");
	var node=document.createTextNode("这是新段落.");
	para.appendChild(node);

	var element=document.getElementById("div1");
	element.appendChild(para);
	</script>

如需删除 HTML 元素,您必须首先获得该元素的父元素:

实例

	<div id="div1">
		<p id="p1">这是一个段落.</p>
		<p id="p2">这是另一个段落.</p>
	</div>

	<script>
	var parent=document.getElementById("div1");
	var child=document.getElementById("p1");
	parent.removeChild(child);
	</script>

### 提示
如果能够在不引用父元素的情况下删除某个元素,就太好了.
不过很遗憾.DOM 需要清楚您需要删除的元素,以及它的父元素.
这是常用的解决方案：找到您希望删除的子元素,然后使用其 parentNode 属性来找到父元素.


