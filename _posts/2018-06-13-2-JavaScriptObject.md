## 创建新对象有两种不同的方法：

1. 定义并创建对象的实例
2. 使用函数来定义对象,然后创建新的对象实例

举例

	person=new Object();
	person.firstname="Bill";
	person.lastname="Gates";
	person.age=56;
	person.eyecolor="blue";
或

	person = {firstname:"John",lastname:"Doe",age:50,eyecolor:"blue"};

或

	function person(firstname,lastname,age,eyecolor) {
		this.firstname=firstname;
		this.lastname=lastname;
		this.age=age;
		this.eyecolor=eyecolor;
	}
	var myFather=new person("Bill","Gates",56,"blue");
	var myMother=new person("Steve","Jobs",48,"green");

* JavaScript 是面向对象的语言,但 JavaScript 不使用类.

* 在 JavaScript 中,不会创建类,也不会通过类来创建对象(就像在其他面向对象的语言中那样).

* JavaScript 基于 prototype,而不是基于类的.

* JavaScript 不定义不同类型的数字,比如整数、短、长、浮点等等.

* JavaScript 中的所有数字都存储为根为 10 的 64 位(8 比特),浮点数.

* 整数(不使用小数点或指数计数法)最多为 15 位.

* 小数的最大位数是 17,但是浮点运算并不总是 100% 准确：

* 如果前缀为 0,则 JavaScript 会把数值常量解释为八进制数,如果前缀为 0 和 "x",则解释为十六进制数.

实例

	var y=0377;
	var z=0xFF;

## 数字属性和方法

### 属性

	MAX VALUE

	MIN VALUE

	NEGATIVE INFINITIVE

	POSITIVE INFINITIVE

	NaN

	prototype

	constructor

### 方法

	toExponential()

	toFixed()

	toPrecision()

	toString()

	valueOf()


## JS字符串

### 长度

	<html>
	<body>
	<script type="text/javascript">
		var txt="Hello World!"
		document.write(txt.length)
	</script>
	</body>
	</html>

### 样式

	<html>
	<body>
	
	<script type="text/javascript">
	
	var txt="Hello World!"
	
	document.write("<p>Big: " + txt.big() + "</p>")
	document.write("<p>Small: " + txt.small() + "</p>")
	
	document.write("<p>Bold: " + txt.bold() + "</p>")
	document.write("<p>Italic: " + txt.italics() + "</p>")
	
	document.write("<p>Blink: " + txt.blink() + " (does not work in IE)</p>")
	document.write("<p>Fixed: " + txt.fixed() + "</p>")
	document.write("<p>Strike: " + txt.strike() + "</p>")
	
	document.write("<p>Fontcolor: " + txt.fontcolor("Red") + "</p>")
	document.write("<p>Fontsize: " + txt.fontsize(16) + "</p>")
	
	document.write("<p>Lowercase: " + txt.toLowerCase() + "</p>")
	document.write("<p>Uppercase: " + txt.toUpperCase() + "</p>")
	
	document.write("<p>Subscript: " + txt.sub() + "</p>")
	document.write("<p>Superscript: " + txt.sup() + "</p>")
	
	document.write("<p>Link: " + txt.link("http://www.w3school.com.cn") + "</p>")
	</script>
	
	</body>
	</html>

### 定位位置0, -1, 6

	<html>
	<body>
	
	<script type="text/javascript">
	
	var str="Hello world!"
	document.write(str.indexOf("Hello") + "<br />")
	document.write(str.indexOf("World") + "<br />")
	document.write(str.indexOf("world"))
	
	</script>
	
	</body>
	</html>

### 匹配world, null, null, world!

	<html>
	<body>
	
	<script type="text/javascript">
	
	var str="Hello world!"
	document.write(str.match("world") + "<br />")
	document.write(str.match("World") + "<br />")
	document.write(str.match("worlld") + "<br />")
	document.write(str.match("world!"))
	
	</script>
	
	</body>
	</html>

### 替换

	<html>
	<body>
	
	<script type="text/javascript">
	
	var str="Visit Microsoft!"
	document.write(str.replace(/Microsoft/,"W3School"))
	
	</script>
	</body>
	</html>


### Date对象

	document.write(Date())

Tue Aug 22 2017 08:25:20 GMT+0800 (中国标准时间)

当前时间

	var d=new Date();
	document.write("从 1970/01/01 至今已过去 " + d.getTime() + " 毫秒");

从 1970/01/01 至今已过去 1503361464579 毫秒

	var d = new Date()
	d.setFullYear(1992,10,3)
	ocument.write(d)

Tue Nov 03 1992 08:24:49 GMT+0800 (中国标准时间)

	var d = new Date()
	document.write (d.toUTCString())

Tue, 22 Aug 2017 00:25:35 GMT

	var d=new Date()
	var weekday=new Array(7)
	weekday[0]="星期日"
	weekday[1]="星期一"
	weekday[2]="星期二"
	weekday[3]="星期三"
	weekday[4]="星期四"
	weekday[5]="星期五"
	weekday[6]="星期六"
	document.write("今天是" + weekday[d.getDay()])

配合运用输出今天是星期二

输出`d.getDay()`的结果是2

	<html>
	<head>
	<script type="text/javascript">
	function startTime()
	{
	var today=new Date()
	var h=today.getHours()
	var m=today.getMinutes()
	var s=today.getSeconds()
	// add a zero in front of numbers<10
	m=checkTime(m)
	s=checkTime(s)
	document.getElementById('txt').innerHTML=h+":"+m+":"+s
	t=setTimeout('startTime()',500)
	}
	
	function checkTime(i)
	{
	if (i<10) 
	  {i="0" + i}
	  return i
	}
	</script>
	</head>
	
	<body onload="startTime()">
	<div id="txt"></div>
	</body>
	</html>

在网页上显示一个动态的时间

`setTimeout()`方法用于在指定的毫秒数后调用函数或计算表达式.

表示月份的参数介于 0 到 11 之间.也就是说,如果希望把月设置为 8 月,则参数应该是 7.

Date 对象自动使用当前的日期和时间作为其初始值.

如果增加天数会改变月份或者年份,那么日期对象会自动完成这种转换.

### 比较日期

	if (myDate>today) {
		alert("Today is before 9th August 2008");
	} else {
		alert("Today is after 9th August 2008");
	}


## 数组对象

### 创建,赋值,for in输出

	var mycars = new Array()
	mycars[0] = "Saab"
	mycars[1] = "Volvo"
	mycars[2] = "BMW"
	for (x in mycars)
	{
	document.write(mycars[x] + "<br />")
	}

### 拼接concat

输出George,John,Thomas,James,Adrew,Martin

但是这并非扩展了arr

	var arr = new Array(3)
	arr[0] = "George"
	arr[1] = "John"
	arr[2] = "Thomas"
	
	var arr2 = new Array(3)
	arr2[0] = "James"
	arr2[1] = "Adrew"
	arr2[2] = "Martin"
	
	document.write(arr.concat(arr2))

输出George,John,Thomas,James,Adrew,Martinundefined

可见arr并没有被扩展

	var arr = new Array(3)
	arr[0] = "George"
	arr[1] = "John"
	arr[2] = "Thomas"
	
	var arr2 = new Array(3)
	arr2[0] = "James"
	arr2[1] = "Adrew"
	arr2[2] = "Martin"
	
	document.write(arr.concat(arr2))
	document.write(arr[3])

输出

George,John,Thomas

George.John.Thomas

将数组拼合成字符串

	var arr = new Array(3);
	arr[0] = "George"
	arr[1] = "John"
	arr[2] = "Thomas"
	
	document.write(arr.join());
	
	document.write("<br />");
	
	document.write(arr.join("."));

### 排序,默认是字典序

George,John,Thomas,James,Adrew,Martin

Adrew,George,James,John,Martin,Thomas

	var arr = new Array(6)
	arr[0] = "George"
	arr[1] = "John"
	arr[2] = "Thomas"
	arr[3] = "James"
	arr[4] = "Adrew"
	arr[5] = "Martin"
	
	document.write(arr + "<br />")
	document.write(arr.sort())

### 数字排序

注意到这个函数似曾相识,java中collection的排序好像允许加入一种排序的规范进行排序;

	function sortNumber(a, b) {
		return a - b
	}
	
	var arr = new Array(6)
	arr[0] = "10"
	arr[1] = "5"
	arr[2] = "40"
	arr[3] = "25"
	arr[4] = "1000"
	arr[5] = "1"
	
	document.write(arr + "<br />")
	document.write(arr.sort(sortNumber))

## 逻辑值

	<html>
	<body>
	
	<script type="text/javascript">
	var b1=new Boolean( 0)
	var b2=new Boolean(1)
	var b3=new Boolean("")
	var b4=new Boolean(null)
	var b5=new Boolean(NaN)
	var b6=new Boolean("false")
	
	document.write("0 是逻辑的 "+ b1 +"<br />")
	document.write("1 是逻辑的 "+ b2 +"<br />")
	document.write("空字符串是逻辑的 "+ b3 + "<br />")
	document.write("null 是逻辑的 "+ b4+ "<br />")
	document.write("NaN 是逻辑的 "+ b5 +"<br />")
	document.write("字符串 'false' 是逻辑的 "+ b6 +"<br />")
	</script>
	
	</body>
	</html>

### 输出

0 是逻辑的 false

1 是逻辑的 true

空字符串是逻辑的 false

null 是逻辑的 false

NaN 是逻辑的 false

字符串 'false' 是逻辑的 true

Boolean(逻辑)对象用于将非逻辑值转换为逻辑值(true 或者 false)

## 算数对象

	round()

最接近的整数

	random()

随机0-1之间

	max()

两个数中较大的

	min()

两个数中最小的

JavaScript 提供 8 种可被 Math 对象访问的算数值：

* 常数

* 圆周率

* 2 的平方根

* 1/2 的平方根

* 2 的自然对数

* 10 的自然对数

* 以 2 为底的 e 的对数

* 以 10 为底的 e 的对数


这是在 Javascript 中使用这些值的方法：(与上面的算数值一一对应)

	Math.E
	Math.PI
	Math.SQRT2
	Math.SQRT1_2
	Math.LN2
	Math.LN10
	Math.LOG2E
	Math.LOG10E

floor()

## RegExp 对象用于规定在文本中检索的内容

	var patt1 = new RegExp("e");
	document.write(patt1.test("The best things in life are free")); 

由于该字符串中存在字母 "e",以上代码的输出将是：

	true

`test()`方法检索字符串中的指定值.返回值是 true 或 false.

	var patt1 = new RegExp("e");

	document.write(patt1.exec("The best things in life are free"));
 
由于该字符串中存在字母 "e",以上代码的输出将是：

	e

`exec()`方法检索字符串中的指定值.返回值是被找到的值.如果没有发现匹配,则返回 null.

在使用 "g" 参数时,`exec()`的工作原理如下：

找到第一个 "e",并存储其位置

如果再次运行 `exec()`,则从存储的位置开始检索,并找到下一个 "e",并存储其位置

	var patt1 = new RegExp("e","g");
	do {
		result = patt1.exec("The best things in life are free");
		document.write(result);
	}
	while (result != null) 

由于这个字符串中 6 个 "e" 字母,代码的输出将是：

	eeeeeenull

g参数是global,可以找到某个字符的所有存在

	var patt1=new RegExp("e");
	
	document.write(patt1.test("The best things in life are free"));
	
	patt1.compile("d");
	
	document.write(patt1.test("The best things in life are free"));

由于字符串中存在 "e",而没有 "d",以上代码的输出是：

	truefalse

`compile()`方法用于改变 RegExp.

`compile()`既可以改变检索模式,也可以添加或删除第二个参数.

## window浏览器对象

浏览器对象模型(Browser Object Model)尚无正式标准

### Window 尺寸

有三种方法能够确定浏览器窗口的尺寸(浏览器的视口,不包括工具栏和滚动条).
对于Internet Explorer、Chrome、Firefox、Opera 以及 Safari：

	window.innerHeight - 浏览器窗口的内部高度
	window.innerWidth - 浏览器窗口的内部宽度

对于 Internet Explorer 8、7、6、5：

	document.documentElement.clientHeight
	document.documentElement.clientWidth

或者

	document.body.clientHeight
	document.body.clientWidth

实用的 JavaScript 方案(涵盖所有浏览器)：

实例

	var w=window.innerWidth
	|| document.documentElement.clientWidth
	|| document.body.clientWidth;
	
	var h=window.innerHeight
	|| document.documentElement.clientHeight
	|| document.body.clientHeight;

	window.open()       //打开新窗口
	window.close()      //关闭当前窗口
	window.moveTo() 	//移动当前窗口
	window.resizeTo() 	//调整当前窗口的尺寸
	
	screen.availWidth   //属性返回访问者屏幕的宽度,以像素计,减去界面特性,比如窗口任务栏.
	screen.availHeight  //属性返回访问者屏幕的高度,以像素计,减去界面特性,比如窗口任务栏.
	
	window.location     //对象在编写时可不使用 window 这个前缀.

一些例子：

	location.hostname   //返回 web 主机的域名
	location.pathname   //返回当前页面的路径和文件名
	location.port       //返回 web 主机的端口 (80 或 443)
	location.protocol   //返回所使用的 web 协议(http:// 或 https://)
	location.href       //属性返回当前页面的 URL.

	<html>
	<head>
	<script>
	function newDoc() {
		window.location.assign("http://www.w3school.com.cn")
	}
	</script>
	</head>
	<body>
		<input type="button" value="加载新文档" onclick="newDoc()">
	</body>
	</html>

`window.history` 对象在编写时可不使用 window 这个前缀.

为了保护用户隐私,对 JavaScript 访问该对象的方法做出了限制.

一些方法：

	history.back()     //与在浏览器点击后退按钮相同
	
	history.forward()  //与在浏览器中点击按钮向前相同
	
	window.navigator   //对象包含有关访问者浏览器的信息.

警告：来自 navigator 对象的信息具有误导性,不应该被用于检测浏览器版本,这是因为：

navigator 数据可被浏览器使用者更改

浏览器无法报告晚于浏览器发布的新操作系统

	<div id="example"></div>
	<script>
		txt = "<p>Browser CodeName: " + navigator.appCodeName + "</p>";
		txt+= "<p>Browser Name: " + navigator.appName + "</p>";
		txt+= "<p>Browser Version: " + navigator.appVersion + "</p>";
		txt+= "<p>Cookies Enabled: " + navigator.cookieEnabled + "</p>";
		txt+= "<p>Platform: " + navigator.platform + "</p>";
		txt+= "<p>User-agent header: " + navigator.userAgent + "</p>";
		txt+= "<p>User-agent language: " + navigator.systemLanguage + "</p>";
		document.getElementById("example").innerHTML=txt;
	</script>

由于 navigator 可误导浏览器检测,使用对象检测可用来嗅探不同的浏览器.

由于不同的浏览器支持不同的对象,您可以使用对象来检测浏览器.

例如,由于只有 Opera 支持属性 "window.opera",您可以据此识别出 Opera.

例子：

	if (window.opera) {...some action...}

	<html>
	<head>
	<script type="text/javascript">
	function disp_prompt()
	  {
	  var name=prompt("请输入您的名字","Bill Gates")
	  if (name!=null && name!="")
	    {
	    document.write("你好！" + name + " 今天过得怎么样？")
	    }
	  }
	</script>
	</head>
	<body>
	
	<input type="button" onclick="disp_prompt()" value="显示提示框" />
	
	</body>
	</html>

### 提示框

	<html>
	<head>
	<script type="text/javascript">
	function show_confirm() {
		var r=confirm("Press a button!");
		if (r==true) {
		 	alert("You pressed OK!");
		} else {
		  	alert("You pressed Cancel!");
		}
	}
	</script>
	</head>
	<body>
		<input type="button" onclick="show_confirm()" value="Show a confirm box" />
	</body>

### 确认框

	alert("再次向您问好！在这里,我们向您演示" + '\n' + "如何向警告框添加折行.")

警告框,可以加入`\n`支持换行

### 计时事件

举例

	<html>
	<head>
	<script type="text/javascript">
	var c=0
	var t
	function timedCount() {
		document.getElementById('txt').value=c
		c=c+1
		t=setTimeout("timedCount()",1000)
	}
	
	function stopCount() {
		c=0;
		setTimeout("document.getElementById('txt').value=0",0);
		clearTimeout(t);
	}
	</script>
	</head>
	
	<body>
	<form>
		<input type="button" value="开始计时！" onClick="timedCount()">
		<input type="text" id="txt">
		<input type="button" value="停止计时！" onClick="stopCount()">
	</form>
	
	<p>请点击上面的"开始计时"按钮来启动计时器.输入框会一直进行计时,从 0 开始.点击"停止计时"按钮可以终止计时,并将计数重置为 0.</p>
	</body>
	</html>

`setTimeout()`
未来的某时执行代码

`clearTimeout()`
取消`setTimeout()`

## cookie对象

	<html>
	<head>
	<script type="text/javascript">
	function getCookie(c_name) {
		if (document.cookie.length>0) { 
			c_start=document.cookie.indexOf(c_name + "=")
			if (c_start != -1) { 
				c_start = c_start + c_name.length+1 
				c_end = document.cookie.indexOf(";",c_start)
				if (c_end == -1) 
					c_end = document.cookie.length
				return unescape(document.cookie.substring(c_start, c_end))
			} 
		}
		return ""
	}
	
	function setCookie(c_name,value,expiredays) {
		var exdate=new Date()
		exdate.setDate(exdate.getDate()+expiredays)
		document.cookie = c_name + "=" + escape(value) +
		    ((expiredays==null) ? "" : "; expires="+exdate.toGMTString())
	}
	
	function checkCookie() {
		username=getCookie('username')
		if (username!=null && username!="") {
			alert('Welcome again '+username+'!')
		} else {
		  	username=prompt('Please enter your name:',"")
		  	if (username!=null && username!="") {
		    	setCookie('username',username,365)
		  	}
		}
	}
	</script>
	</head>
	<body onLoad="checkCookie()">
	</body>
	</html>

这里`escape()`函数可对字符串进行编码,这样就可以在所有的计算机上读取该字符串.

注意到getCookie中,indexof,第二个参数是可选参数,指从哪里开始检索

注意到用;结尾,可知每条cookie使用分号结尾

差不多JavaScript就到这里了,下面的jQuery会单开
