# 11 DOM扩展

对DOM的两个主要的扩展是Selectors API(选择符API)和HTML5;

还有一个不那么引人瞩目的Element Traversal(元素遍历)规范,为DOM添加了一些属性;

# 11.1 选择符API

根据CSS选择符选择与某个模式匹配的DOM元素;

也就是jQuery做了的事情;

所有实现这一功能的JavaScript库都会写一个基础的CSS解析器;

然后再使用已有的DOM方法查询文档并找到匹配的节点;

尽管库开发人员在不知疲倦地改进这一过程的性能;


但到头来都只能通过运行JavaScript代码来完成查询操作;

而把这个功能变成原生API之后;

解析和树查询操作可以在浏览器内部通过编译后的代码来完成,极大地改善了性能;

querySelector()和querySelectorAll();

## querySelector()方法

接收一个CSS 选择符,返回与该模式匹配的第一个元素;

如果没有找到匹配的元素,返回null;

如果传入了不被支持的选择符,querySelector()会抛出错误;

## querySelectorAll()方法

接收一个CSS 选择符,这个方法返回的是一个NodeList的实例;

这个不是动态的,是一个快照;

例子:

```
//取得所有<p>元素中的所有<strong>元素
var strongs = document.querySelectorAll("p strong");

var i, len, strong;
for (i=0, len=strongs.length; i < len; i++) {
	strong = strongs[i]; //或者strongs.item(i)
	strong.className = "important";
}
```

同样的,如果传入了浏览器不支持的选择符或者选择符中有语法错误,querySelectorAll()会抛出错误;

## matchesSelector()方法

接收一个参数,即CSS选择符,如果调用元素与该选择符匹配,返回true;否则,返回false;

书上说截至2011年中,还没有浏览器支持该方法,我试了一下,现在是2018年6月1日,chrome也没有支持233;

但是呢,有一些代偿性的方法;

```
function matchesSelector(element, selector) {
    if (element.matchesSelector) {
        return element.matchesSelector(selector);
    } else if (element.msMatchesSelector) {
        return element.msMatchesSelector(selector);
    } else if (element.mozMatchesSelector) {
        return element.mozMatchesSelector(selector);
    } else if (element.webkitMatchesSelector) {
        return element.webkitMatchesSelector(selector);
    } else {
        throw new Error("Not supported.");
    }
}

if (matchesSelector(document.body, "body.page1")) {
//执行操作
}
```

# 11.2 元素遍历

对于元素间的空格,IE9及之前版本不会返回文本节点,而其他所有浏览器都会返回文本节点;

这样,就导致了在使用childNodes和firstChild等属性时的行为不一致;

为了弥补这一差异,而同时又保持DOM规范不变,Element Traversal规范新定义了一组属性;

IE NB;

1.childElementCount:返回子元素(不包括文本节点和注释)的个数;

2.firstElementChild:指向第一个子元素;firstChild的元素版;

3.lastElementChild:指向最后一个子元素;lastChild的元素版;

4.previousElementSibling:指向前一个同辈元素;previousSibling的元素版;

5.nextElementSibling:指向后一个同辈元素;nextSibling的元素版;

例子;

```
var i,
len,
child = element.firstElementChild;
while(child != element.lastElementChild) {
    processChild(child); //已知其是元素
    child = child.nextElementSibling;
}
```