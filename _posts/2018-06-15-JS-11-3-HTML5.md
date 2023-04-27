# 11.3 HTML5

对于传统HTML而言,HTML5是一个叛逆;

所有之前的版本对JavaScript接口的描述都不过三言两语;

主要篇幅都用于定义标记,与JavaScript相关的内容一概交由DOM规范去定义;

而HTML5规范则围绕如何使用新增标记定义了大量JavaScript API;

其中一些API与DOM重叠,定义了浏览器应该支持的DOM扩展;

## 与类相关的扩充

class使用率超高,所以新增了很多API简化CSS类的用法;

### getElementsByClassName()方法

通过既有的DOM功能实现的,而原生的实现具有极大的性能优势;

接收一个参数,即一个包含一或多个类名的字符串,返回带有指定类的所有元素的NodeList;

```
//取得所有类中包含"username"和"current"的元素,类名的先后顺序无所谓
var allCurrentUsernames = document.getElementsByClassName("username current");
//取得ID 为"myDiv"的元素中带有类名"selected"的所有元素
var selected = document.getElementById("myDiv").getElementsByClassName("selected");
```

### classList属性

classList是新集合DOMTokenList的实例,这个新类型定义的方法;

1.add(value):将给定的字符串值添加到列表中,如果值已经存在,就不添加了;

2.contains(value):表示列表中是否存在给定的值,如果存在则返回true,否则返回false;

3.remove(value):从列表中删除给定的字符串;

4.toggle(value):如果列表中已经存在给定的值,删除它;如果列表中没有给定的值,添加它;

### 焦点管理

HTML5也添加了辅助管理DOM焦点的功能;

首先就是document.activeElement属性,这个属性始终会引用DOM中当前获得了焦点的元素;

元素获得焦点的方式有页面加载,用户输入(通常是通过按Tab键)和在代码中调用focus()方法;

默认情况下,文档刚刚加载完成时,document.activeElement 中保存的是document.body元素的引用;

文档加载期间,document.activeElement的值为null;

document.hasFocus()确定文档是否获得了焦点;

通过检测文档是否获得了焦点,可以知道用户是不是正在与页面交互;

666666;

查询文档获知哪个元素获得了焦点,以及确定文档是否获得了焦点;

这两个功能最重要的用途是提高Web应用的无障碍性;

无障碍Web应用的一个主要标志就是恰当的焦点管理;

## HTMLDocument的变化

HTML5扩展了HTMLDocument,增加了新的功能;

### readyState属性

readyState可能有两个值:loading,complete,顾名思义;

```
if (document.readyState == "complete"){
//执行操作
}
```

### 兼容模式

IE赢了一局;

自从IE6开始区分渲染页面的模式是标准的还是混杂的,检测页面的兼容模式就成为浏览器的必要功能;

IE为此给document添加了一个名为compatMode的属性;

这个属性就是为了告诉开发人员浏览器采用了哪种渲染模式;

在标准模式下,document.compatMode的值等于"CSS1Compat";

而在混杂模式下,document.compatMode的值等于"BackCompat";

查了下MDN,备注说;

还有另外一种渲染模式, Gecko的"准标准模式";

该模式和标准规范模式的区别仅为表格单元内的图片布局方式不同;

且该模式的类型字符串仍为: "CSS1Compat";

### head属性

引用文档的`<head>`元素;

```
var head = document.head||document.getElementsByTagName("head")[0];
```

## 字符集属性

charset属性表示文档中实际使用的字符集,默认为"UTF16";

可以修改;

```
alert(document.charset); //"UTF-16"
document.charset = "UTF-8";
```

defaultCharset属性,表示根据默认浏览器及操作系统的设置,当前文档默认的字符集应该是什么;

如果文档没有默认字符集,那charset和defaultCharset的值可能会不一样;

## 自定义数据属性

HTML5规定可以为元素添加非标准的属性,但要添加前缀data-;

目的是为元素提供与渲染无关的信息,或者提供语义信息;

这些属性可以任意添加,随便命名,只要以data-开头即可;

## 插入标记

使用插入标记的技术,直接插入HTML字符串不仅更简单,速度也更快;

以下与插入标记相关的DOM扩展已经纳入了HTML5规范;

### innerHTML属性

在读模式下,innerHTML属性返回与调用元素的所有子节点(包括元素,注释和文本节点)对应的HTML标记;

在写模式下,innerHTML会根据指定的值创建新的DOM树,然后用这个DOM树完全替换调用元素原先的所有子节点;

不同浏览器返回的innerHTML是不同的;

```
div.innerHTML = "Hello & welcome, <b>\"reader\"!</b>";
<div id="content">Hello &amp; welcome, <b>&quot;reader&quot;!</b></div>
```

为innerHTML设置HTML字符串后,浏览器会将这个字符串解析为相应的DOM树;

因此设置了innerHTML之后,再从中读取HTML字符串,会得到与设置时不一样的结果;

原因在于返回的字符串是根据原始HTML字符串创建的DOM树经过序列化之后的结果;

限制:

IE8及更早版本是唯一能在这种情况下执行脚本的浏览器,但必须满足一些条件;

一是必须为`<script>`元素指定defer属性,

二是`<script>`元素必须位于(微软所谓的)"有作用域的元素"(scoped element)之后;

`<script>`元素被认为是"无作用域的元素"(NoScope element),

也就是在页面中看不到的元素,与`<style>`元素或注释类似;

如果通过innerHTML插入的字符串开头就是一个"无作用域的元素",

那么IE会在解析这个字符串前先删除该元素;

```
div.innerHTML = "<script defer>alert('hi');<\/script>"; //无效
// 可以运行
div.innerHTML = "_<script defer>alert('hi');<\/script>";
div.innerHTML = "<div>&nbsp;</div><script defer>alert('hi');<\/script>";
div.innerHTML = "<input type=\"hidden\"><script defer>alert('hi');<\/script>";
```

不支持innerHTML的元素:

`<col>`,`<colgroup>`,`<frameset>`,`<head>`,`<html>`,`<style>`,`<table>`,`<tbody>`,`<tfoot>`,`<thead>`,`<tr>`;

在IE8及更早版本中,`<title>`元素也没有innerHTML属性;

### outerHTML属性

在读模式下,outerHTML返回调用它的元素及所有子节点的HTML标签;

在写模式下,outerHTML会根据指定的HTML字符串创建新的DOM子树,然后用这个DOM子树完全替换调用元素;

不同浏览器返回的outerHTML也是不同的;

```
div.outerHTML = "<p>This is a paragraph.</p>";
// 等价于
var p = document.createElement("p");
p.appendChild(document.createTextNode("This is a paragraph."));
div.parentNode.replaceChild(p, div);
```

### insertAdjacentHTML()

接收两个参数:插入位置和要插入的HTML文本;

第一个参数:

1."beforebegin",在当前元素之前插入一个紧邻的同辈元素;

2."afterbegin",在当前元素之下插入一个新的子元素或在第一个子元素之前再插入新的子元素;

3."beforeend",在当前元素之下插入一个新的子元素或在最后一个子元素之后再插入新的子元素;

4."afterend",在当前元素之后插入一个紧邻的同辈元素;

```
//作为前一个同辈元素插入
element.insertAdjacentHTML("beforebegin", "<p>Hello world!</p>");
//作为第一个子元素插入
element.insertAdjacentHTML("afterbegin", "<p>Hello world!</p>");
//作为最后一个子元素插入
element.insertAdjacentHTML("beforeend", "<p>Hello world!</p>");
//作为后一个同辈元素插入
element.insertAdjacentHTML("afterend", "<p>Hello world!</p>");
```

### 内存与性能问题

使用本节介绍的方法替换子节点可能会导致浏览器的内存占用问题,

尤其是在IE中,问题更加明显;

在删除带有事件处理程序或引用了其他JavaScript对象子树时,就有可能导致内存占用问题;

假设某个元素有一个事件处理程序(或者引用了一个JavaScript 对象作为属性),

在使用前述某个属性将该元素从文档树中删除后,

元素与事件处理程序(或JavaScript对象)之间的绑定关系在内存中并没有一并删除;

如果这种情况频繁出现,页面占用的内存数量就会明显增加;

因此,在使用innerHTML,outerHTML属性和insertAdjacentHTML()方法时,

最好先手工删除要被替换的元素的所有事件处理程序和JavaScript对象属性;

一般来说,在插入大量新HTML 标记时,

使用innerHTML属性与通过多次DOM操作先创建节点再指定它们之间的关系相比,

效率要高得多;

这是因为在设置innerHTML或outerHTML时,就会创建一个HTML解析器;

这个解析器是在浏览器级别的代码(通常是C++编写的)基础上运行的,

因此比执行JavaScript快得多;

不可避免地,创建和销毁HTML解析器也会带来性能损失,

所以最好能够将设置innerHTML或outerHTML的次数控制在合理的范围内;

错误例子:

```
for (var i=0, len=values.length; i < len; i++){
    ul.innerHTML += "<li>" + values[i] + "</li>"; //要避免这种频繁操作！！
}
```

正确例子:

```
for (var i=0, len=values.length; i < len; i++){
    itemsHtml += "<li>" + values[i] + "</li>";
}
ul.innerHTML = itemsHtml;
```

## scrollIntoView()方法

控制页面滚动;

scrollIntoView()可以在所有HTML元素上调用,

通过滚动浏览器窗口或某个容器元素,调用元素就可以出现在视口中;

如果给这个方法传入true作为参数,或者不传入任何参数;

那么窗口滚动之后会让调用元素的顶部与视口顶部尽可能平齐;

如果传入false作为参数,调用元素会尽可能全部出现在视口中;

可能的话,调用元素的底部会与视口顶部平齐,不过顶部不一定平齐;

```
//让元素可见
document.forms[0].scrollIntoView();
```

支持scrollIntoView()方法的浏览器有IE,Firefox,Safari和Opera;

8102年了,chrome依然没有支持;