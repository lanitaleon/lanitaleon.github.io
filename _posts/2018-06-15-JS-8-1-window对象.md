# BOM

BOM的核心对象是window,表示浏览器的一个实例;

是ECMAScript规定的Global对象,是js访问浏览器窗口的接口;

## 全局作用域

定义全局变量与在window对象上直接定义属性还是有一点差别:

全局变量不能通过delete操作符删除,而直接在window对象上的定义的属性可以;

```
var age = 29;
window.color = "red";
//在IE < 9 时抛出错误,在其他所有浏览器中都返回false
delete window.age;
//在IE < 9 时抛出错误,在其他所有浏览器中都返回true
delete window.color; //returns true
alert(window.age); //29
alert(window.color); //undefined
```

使用var语句添加的window属性有一个名为[[Configurable]]的特性;

这个特性的值被设置为false,因此这样定义的属性不可以通过delete操作符删除

尝试访问未声明的变量会抛出错误,但是通过查询window对象;

可以知道某个可能未声明的变量是否存在;

```
//这里会抛出错误,因为oldValue 未定义
var newValue = oldValue;
//这里不会抛出错误,因为这是一次属性查询
//newValue 的值是undefined
var newValue = window.oldValue;
```

## 窗口关系及框架

每个框架都有自己的window对象,window对象的name属性是框架的名称;

也可以通过数值索引(从0开始,从左到右从上到下)在frames集合中访问相应的window对象;

```
<html>
<head>
    <title>Frameset Example</title>
</head>
<frameset rows="160,*">
    <frame src="frame.htm" name="topFrame">
    <frameset cols="50%,50%">
        <frame src="anotherframe.htm" name="leftFrame">
        <frame src="yetanotherframe.htm" name="rightFrame">
    </frameset>
</frameset>
</html>
```

可以通过window.frames[0]或者window.frames["topFrame"]来引用上方的框架;

最好使用top来引用这些框架top.frames[0];

然而frameset好像是deprecated html tag……emmm

与top相对的另一个window对象是parent;

parent(父)对象始终指向当前框架的直接上层框架;

在某些情况下,parent有可能等于top;

但在没有框架的情况下,parent一定等于top(此时它们都等于window;

```
<html>
<head>
    <title>Frameset Example</title>
</head>
<frameset rows="100,*">
    <frame src="frame.htm" name="topFrame">
    <frameset cols="50%,50%">
        <frame src="anotherframe.htm" name="leftFrame">
        <frame src="anotherframeset.htm" name="rightFrame">
    </frameset>
</frameset>
</html>


<html>
<head>
    <title>Frameset Example</title>
</head>
<frameset cols="50%,50%">
    <frame src="red.htm" name="redFrame">
    <frame src="blue.htm" name="blueFrame">
</frameset>
</html>
```


除非最高层窗口是通过window.open()打开的,否则其window对象的name属性不会包含任何值;

self对象始终指向window;实际上,self和window对象可以互换使用;

引入self对象的目的只是为了与top和parent对象对应起来,因此它不格外包含其他值;

可以window.parent,window.top,也可以window.parent.parent.frames[0];

在使用框架的情况下,浏览器中会存在多个Global对象;

在每个框架中定义的全局变量会自动成为框架中window对象的属性;

由于每个window对象都包含原生类型的构造函数,因此每个框架都有一套自己的构造函数;

这些构造函数一一对应,但并不相等;

例如,top.Object并不等于top.frames[0].Object;

这个问题会影响到对跨框架传递的对象使用instanceof操作符

## 窗口位置

跨浏览器获取窗口左边和上边的位置;

```
var leftPos = (typeof window.screenLeft == "number") ?
    window.screenLeft : window.screenX;
var topPos = (typeof window.screenTop == "number") ?
    window.screenTop : window.screenY;
```

如果是双显示器,可能会显示成负数,比如我的left:-1920;

因为各个浏览器对此的实现不统一,跨浏览器的条件下无法取得窗口左边和上班的精确坐标值;

但是,使用moveTo()和moveBy()方法有可能将窗口精确移动到一个新位置;

```
//将窗口移动到屏幕左上角
window.moveTo(0,0);
//将窗向下移动100 像素
window.moveBy(0,100);
//将窗口移动到(200,300)
window.moveTo(200,300);
//将窗口向左移动50 像素
window.moveBy(-50,0);
```

但是这两个方法可能会被浏览器禁用;

在Opera和IE7(及更高版本)中默认就是禁用的;

这两个方法都不适用于框架,只能对最外层的window 对象使用;


## 窗口大小

因为各个浏览器的实现不统一,无法确定浏览器窗口本身的大小,但是可以获得页面视口的大小;

```
var pageWidth = window.innerWidth,
    pageHeight = window.innerHeight;
if (typeof pageWidth != "number"){
    if (document.compatMode == "CSS1Compat"){
        pageWidth = document.documentElement.clientWidth;
        pageHeight = document.documentElement.clientHeight;
    } else {
        pageWidth = document.body.clientWidth;
        pageHeight = document.body.clientHeight;
    }
}
```

document.compatMode可以确定页面是否处于标准模式;

移动设备中,window.innerWidth和window.innerHeight保存着可见视口,也就是屏幕上可见页面区域的大小;

移动IE通过document.documentElement.client-Width和document.documentElement.clientHeihgt提供了相同的信息;

在其他移动浏览器中,document.documentElement度量的是布局视口;

移动IE浏览器通过document.body.clientWidth 和document.body.clientHeight提供了相同的信息;

IE就是ze样不一样的烟火了;

resizeTo()和resizeBy()方法可以调整浏览器窗口的大小;

```
//调整到100×100
window.resizeTo(100, 100);
//调整到200×150
window.resizeBy(100, 50);
//调整到 300×300
window.resizeTo(300, 300);
```

resizeTo()接收浏览器窗口的新宽度和新高度;

resizeBy()接收新窗口与原窗口的宽度和高度之差;

但是这两个方法可能会被浏览器禁用;

在Opera和IE7(及更高版本)中默认就是禁用的;

这两个方法都不适用于框架,只能对最外层的window对象使用;

是不是突然眼熟(

## 导航和打开窗口

window.open()接收4个参数:

* 要加载的URL,
* 窗口目标,
* 一个特性字符串,
* 一个表示新页面是否取代浏览器历史记录中当前加载页面的布尔值;

通常只须传递第一个参数,最后一个参数只在不打开新窗口的情况下使用;

```
//等同于<a href="http://www.wrox.com" target="topFrame"></a>
window.open("http://www.wrox.com/", "topFrame");
```

如果给window.open()传递的第二个参数并不是一个已经存在的窗口或框架;

那么该方法就会根据在第三个参数位置上传入的字符串创建一个新窗口或新标签页;

如果没有传入第三个参数,那么就会打开一个带有全部默认设置(工具栏/地址栏和状态栏等)的新浏览器窗口或者打开一个新标签页——根据浏览器设置;

在不打开新窗口的情况下,会忽略第三个参数;

第三个参数是一个逗号分隔的设置字符串,表示在新窗口中都显示哪些特性;

> 整个特性字符串中不允许出现空格

```
window.open("http://www.wrox.com/", "wroxWindow",
    "height=400,width=400,top=10,left=10,resizable=yes");
```

通过window.open()可以打开一个新窗口,返回一个指向新窗口的引用;

有些浏览器不允许调整主浏览器窗口的大小或位置,但是可以通过以上方式操作新打开的窗口;

主窗口在不经用户同意的情况下也不能关闭,新打开的窗口可以通过top.close()关闭自己;

关闭后引用还在,但是除了检测closed属性外没有其他用处;

```
    wroxWin.close();
    alert(wroxWin.closed);//true
```

新创建的window对象有一个opener属性,保存着打开它的原始窗口对象;

```
var wroxWin = window.open("http://www.wrox.com", "wroxWin", "height=400,width=400,top=10,left=10,resizable=yes");
alert(wroxWin.opener == window);
```

但是原始浏览器并不会记录它们打开的窗口,甚至有些浏览器比如ie/chrome会在独立的进程中运行每个标签页;

chrome将新创建的标签页的opener属性设置为null,也就是新创建的标签页不需要与打开它的标签页通信;

标签页之间的联系一旦切断,将无法恢复;

### 安全限制

曾经广告商肆无忌惮地使用伪装成系统对话框的弹出窗口,引诱用户点击;

所以各大浏览器对弹出窗口进行了限制,当弹出窗口被阻止时,window.open()将抛出一个错误;

```
var blocked = false;
try {
    var wroxWin = window.open("http://www.wrox.com", "_blank");
    if (wroxWin == null) {
        blocked = true;
    }
} catch (ex) {
    blocked = true;
}
if (blocked) {
    alert("The popup was blocked!");
}
```

## 间歇调用和超时调用

setTimeout()

```
//不建议传递字符串!
setTimeout("alert('Hello world!') ", 1000);
//推荐的调用方式
setTimeout(function () {
    alert("Hello world!");
}, 1000);
```

传递字符串可能损失性能;

经过第二个参数设定的时间,指定的代码也不一定执行;

js的单线程序的解释器,因此就需要一个任务队列;

setTimeout()的第二个参数是告诉js经过多久把这个任务添加到队列里;

如果队列中已经有任务了,就不会立即执行;

调用setTimeout()之后,该方法会返回一个数值ID,表示超时调用;

这个超时调用ID是计划执行代码的唯一标识符,可以通过它来取消超时调用;

```
//设置超时调用
var timeoutId = setTimeout(function () {
    alert("Hello world!");
}, 1000);
//注意:把它取消
clearTimeout(timeoutId);
```

超时调用的代码都是在全局作用域中执行的,

因此函数中this的值在非严格模式下指向window对象,

在严格模式下是undefined;

间歇调用的例子:

```
var num = 0;
var max = 10;
var intervalId = null;

function incrementNumber() {
    num++;
//如果执行次数达到了max设定的值,则取消后续尚未执行的调用
    if (num == max) {
        clearInterval(intervalId);
        alert("Done");
    }
}

intervalId = setInterval(incrementNumber, 500);
```

这个例子可以用setTimeout实现如下:

```
var num = 0;
var max = 10;

function incrementNumber() {
    num++;
//如果执行次数未达到max设定的值,则设置另一次超时调用
    if (num < max) {
        setTimeout(incrementNumber, 500);
    } else {
        alert("Done");
    }
}

setTimeout(incrementNumber, 500);
```

可见,在使用超时调用时,没有必要跟踪超时调用ID,

因为每次执行代码之后,如果不再设置另一次超时调用,调用就会自行停止;

一般认为,使用超时调用来模拟间歇调用的是一种最佳模式;

在开发环境下,很少使用真正的间歇调用,

原因是后一个间歇调用可能会在前一个间歇调用结束之前启动;

而像前面示例中那样使用超时调用,则完全可以避免这一点;

所以,最好不要使用间歇调用;

……我码了这么多字你就跟我说这个

## 系统对话框

alert(),confirm(),prompt(),print(),find()

```
if (confirm("Are you sure?")) {
    alert("I'm so glad you're sure! ");
} else {
    alert("I'm sorry to hear you're not sure. ");
}

var result = prompt("What is your name? ", "");
if (result !== null) {
    alert("Welcome, " + result);
}

//显示"打印"对话框
window.print();
//显示"查找"对话框
window.find();
```

除了前三种以外,chrome引入了一种新特性;

如果当前脚本在执行过程中会打开两个或多个对话框,那么从第二个对话框开始,

每个对话框中都会显示一个复选框,让用户选择是否屏蔽后续的对话框显示;

除非用户刷新页面,chrome这么做之后,firefox和ie9也这么做了2333

剩下两个,print()和find()是异步显示的,虽然给的消息很少,一般不会用到;

但是既然是异步显示的,那chrome的对话框计数器就不会把它计算在内,也就是不会被禁用;

