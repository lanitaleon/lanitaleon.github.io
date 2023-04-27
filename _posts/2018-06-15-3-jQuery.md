### JavaScript方式

	function myFunction() {
		var obj=document.getElementById("h01");
		obj.innerHTML="Hello jQuery";
	}
	onload=myFunction;

### jQuery 方式

	function myFunction() {
		$("#h01").html("Hello jQuery");
	}
	$(document).ready(myFunction);

document ready 函数是为了防止文档在完全加载(就绪)之前运行 jQuery 代码.

如果在文档没有完全加载之前就运行函数,操作可能失败.下面是两个具体的例子：

* 试图隐藏一个不存在的元素

* 获得未完全加载的图像的大小


## jQuery 属性选择器

jQuery 使用 XPath 表达式来选择带有给定属性的元素.

`$("[href]")` 选取所有带有 href 属性的元素.

`$("[href='#']")` 选取所有带有 href 值等于 "#" 的元素.

`$("[href!='#']")` 选取所有带有 href 值不等于 "#" 的元素.

`$("[href$='.jpg']")` 选取所有 href 值以 ".jpg" 结尾的元素.

## jQuery 名称冲突

jQuery 使用 $ 符号作为 jQuery 的简介方式.

某些其他 JavaScript 库中的函数(比如 Prototype)同样使用 $ 符号.

jQuery 使用名为 `noConflict()` 的方法来解决该问题.

`var jq=jQuery.noConflict()`,帮助您使用自己的名称(比如 jq)来代替 $ 符号.

这里注意到

	$.noConflict();
	jQuery(document).ready(function() {
	  jQuery("button").click(function(){
	    jQuery("p").text("jQuery 仍在运行！");
	  });
	});

是可行的,$被替换为jQuery

	var something = $.noConflict()

也是可行的

常用事件

	$(document).ready(function)	    //将函数绑定到文档的就绪事件(当文档完成加载时)
	$(selector).click(function)	    //触发或将函数绑定到被选元素的点击事件
	$(selector).dblclick(function)	//触发或将函数绑定到被选元素的双击事件
	$(selector).focus(function)	    //触发或将函数绑定到被选元素的获得焦点事件
	$(selector).mouseover(function)	//触发或将函数绑定到被选元素的鼠标悬停事件

	$(selector).hide(speed,callback);
	$(selector).show(speed,callback);

可选的 speed 参数规定隐藏/显示的速度,可以取以下值："slow"、"fast" 或毫秒.
可选的 callback 参数是隐藏或显示完成后所执行的函数名称.

	$("button").click(function(){
	  $("p").hide(1000);
	});

同理,toggle也可以设置速度

	$(selector).toggle(speed,callback);

可选的 speed 参数规定隐藏/显示的速度,可以取以下值："slow"、"fast" 或毫秒.

可选的 callback 参数是 `toggle()` 方法完成后所执行的函数名称.

同理

jQuery `fadeIn()` 用于淡入已隐藏的元素

jQuery `fadeOut()` 方法用于淡出可见元素

jQuery `fadeToggle()` 方法可以在 `fadeIn()` 与 `fadeOut()` 方法之间进行切换.

如果元素已淡出,则 `fadeToggle()` 会向元素添加淡入效果.

如果元素已淡入,则 `fadeToggle()` 会向元素添加淡出效果.

jQuery `fadeTo()` 方法允许渐变为给定的不透明度(值介于 0 与 1 之间).

jQuery `slideDown()` 方法用于向下滑动元素.

jQuery `slideUp()` 方法用于向上滑动元素

jQuery `slideToggle()`

## 动画

jQuery `animate()` 方法用于创建自定义动画.

	$(selector).animate({params},speed,callback);

必需的 params 参数定义形成动画的 CSS 属性.

可选的 speed 参数规定效果的时长.它可以取以下值："slow"、"fast" 或毫秒.

可选的 callback 参数是动画完成后所执行的函数名称.

举例

	<!DOCTYPE html>
	<html>
	<head>
	<script src="/jquery/jquery-1.11.1.min.js">
	</script>
	<script> 
	$(document).ready(function(){
	  $("button").click(function(){
	    $("div").animate({left:'250px'});
	  });
	});
	</script> 
	</head>
	 
	<body>
	<button>开始动画</button>
	<p>默认情况下,所有 HTML 元素的位置都是静态的,并且无法移动.如需对位置进行操作,记得首先把元素的 CSS position 属性设置为 relative、fixed 或 absolute.</p>
	<div style="background:#98bf21;height:100px;width:100px;position:absolute;">
	</div>
	
	</body>
	</html>

多个属性调整举例

	<!DOCTYPE html>
	<html>
	<head>
	<script src="/jquery/jquery-1.11.1.min.js"></script>
	<script> 
		$(document).ready(function(){
		  $("button").click(function(){
		    $("div").animate({
		      left:'250px',
		      opacity:'0.5',
		      height:'150px',
		      width:'150px'
		    });
		  });
		});
	</script> 
	</head>
	<body>
		<button>开始动画</button>
		<p>默认情况下,所有 HTML 元素的位置都是静态的,并且无法移动.如需对位置进行操作,记得首先把元素的 CSS position 属性设置为 relative、fixed 或 absolute.</p>
		<div style="background:#98bf21;height:100px;width:100px;position:absolute;">
		</div>
	</body>
	</html>

### 说明

animate几乎可以操作所有css属性,但是

当使用 `animate()` 时,必须使用 Camel 标记法书写所有的属性名,

比如,必须使用 paddingLeft 而不是 padding-left,

使用 marginRight 而不是 margin-right,等等.

同时,色彩动画并不包含在核心 jQuery 库中.

如果需要生成颜色动画,您需要从 jQuery.com 下载 Color Animations 插件.

也可以定义相对值(该值相对于元素的当前值).需要在值的前面加上 += 或 -=：

实例

	$("button").click(function(){
	  $("div").animate({
	    left:'250px',
	    height:'+=150px',
	    width:'+=150px'
	  });
	});

甚至可以把属性的动画值设置为 "show"、"hide" 或 "toggle"：

实例

	$("button").click(function(){
	  $("div").animate({
	    height:'toggle'
	  });
	});

效果说明：点击隐藏,再点出现

更有意思的来了,队列

	<!DOCTYPE html>
	<html>
	<head>
	<script src="/jquery/jquery-1.11.1.min.js"></script>
	<script> 
		$(document).ready(function() {
		  $("button").click(function() {
		    var div=$("div");
		    div.animate({height:'300px',opacity:'0.4'},"slow");
		    div.animate({width:'300px',opacity:'0.8'},"slow");
		    div.animate({height:'100px',opacity:'0.4'},"slow");
		    div.animate({width:'100px',opacity:'0.8'},"slow");
		  });
		});
	</script> 
	</head>
	<body>
		<button>开始动画</button>
		<p>默认情况下,所有 HTML 元素的位置都是静态的,并且无法移动.如需对位置进行操作,记得首先把元素的 CSS position 属性	设置为 relative、fixed 或 absolute.</p>
		<div style="background:#98bf21;height:100px;width:100px;position:absolute;">
	</div>
	</body>
	</html>

效果描述：依次执行,感觉这样的话,可以直接写一个动画片出来了

jQuery `stop()` 方法用于停止动画或效果,在它们完成之前.

`stop()` 方法适用于所有 jQuery 效果函数,包括滑动、淡入淡出和自定义动画.

举例

	<!DOCTYPE html>
	<html>
	<head>
	<script src="/jquery/jquery-1.11.1.min.js"></script>
	<script> 
		$(document).ready(function(){
		  $("#flip").click(function(){
		    $("#panel").slideDown(5000);
		  });
		  $("#stop").click(function(){
		    $("#panel").stop();
		  });
		});
	</script>
	 
	<style type="text/css"> 
		#panel, #flip {
			padding:5px;
			text-align:center;
			background-color:#e5eecc;
			border:solid 1px #c3c3c3;
		}
		#panel {
			padding:50px;
			display:none;
		}
	</style>
	</head>
	<body> 
		<button id="stop">停止滑动</button>
		<div id="flip">点击这里,向下滑动面板</div>
		<div id="panel">Hello world!</div>
	</body>
	</html>

效果描述：点击开始滑动,点击停止,动画停止,再点击继续滑动,再点击停止,动画停止;
stopAll默认是false,即仅停止活动的动画
goToEnd可选,默认false
默认地,`stop()`会清除在被选元素上指定的动画,
但是我感觉这个效果只是暂停,不是清除啊

## Chaining链接

举例

	<!DOCTYPE html>
	<html>
	<head>
	<script src="/jquery/jquery-1.11.1.min.js"></script>
	<script>
		$(document).ready(function()
		  {
		  $("button").click(function(){
		    $("#p1").css("color","red").slideUp(2000).slideDown(2000);
		  });
		});
	</script>
	</head>
	<body>
		<p id="p1">jQuery 乐趣十足！</p>
		<button>点击这里</button>
	</body>
	</html>

效果描述：先变红色,然后向上滑动,再向下滑动;

没有使用animate,依然有动画的效果惹

因为jQuery的语法不严格,所以可以这样写

	$("#p1").css("color","red")
	  .slideUp(2000)
	  .slideDown(2000);

保持可读性

## jQuery HTML

`text()` - 设置或返回所选元素的文本内容

`html()` - 设置或返回所选元素的内容(包括 HTML 标记)

`val()` - 设置或返回表单字段的值

讲道理,text()我几乎就没成功拿出来过

	$("#w3s").attr("href","http://www.w3school.com.cn/jquery");

## 设置属性

	$("button").click(function(){
	  $("#w3s").attr({
	    "href" : "http://www.w3school.com.cn/jquery",
	    "title" : "W3School jQuery Tutorial"
	  });
	});

设置多个属性

`append()` - 在被选元素的结尾插入内容

`prepend()` - 在被选元素的开头插入内容

`after()` - 在被选元素之后插入内容

`before()` - 在被选元素之前插入内容

举例

	<!DOCTYPE html>
	<html>
	<head>
	<script src="/jquery/jquery-1.11.1.min.js"></script>
	<script>
	function appendText() {
		var txt1="<p>Text.</p>";              // 以 HTML 创建新元素
		var txt2=$("<p></p>").text("Text.");  // 以 jQuery 创建新元素
		var txt3=document.createElement("p");
		txt3.innerHTML="Text.";               // 通过 DOM 来创建文本
		$("body").append(txt1,txt2,txt3);        // 追加新元素
	}
	</script>
	</head>
	<body>
		<p>This is a paragraph.</p>
		<button onclick="appendText()">追加文本</button>
	</body>
	</html>

可以追加多个,用多种方式都可以创建

和after加以区别的是,append添加在被选元素的结尾,也就是

	<span>aaaaaaaaaaaaaaa<a href="#">ddddd</a></span>

还是内部

而after是

	<span>aaaaaaaaaaaaaaa</span>

	<a href="#">ddddd</a>

同样另一对也是这样

`remove()` - 删除被选元素(及其子元素)

`empty()` - 从被选元素中删除子元素

`remove()`支持接收参数对被删元素进行过滤

举例

	$("p").remove(".italic");

`addClass()` - 向被选元素添加一个或多个类

`removeClass()` - 从被选元素删除一个或多个类

`toggleClass() `- 对被选元素进行添加/删除类的切换操作

`css()`方法设置或返回被选元素的一个或多个样式属性.

举例

    $("p").css({"background-color":"yellow","font-size":"200%"});

## jQuery 提供多个处理尺寸的重要方法

`width()` 方法设置或返回元素的宽度(不包括内边距、边框或外边距).

`height()` 方法设置或返回元素的高度(不包括内边距、边框或外边距).

`outerWidth()` 方法返回元素的宽度(包括内边距和边框).

`outerHeight()` 方法返回元素的高度(包括内边距和边框).

`innerWidth()` 方法返回元素的宽度(包括内边距).

`innerHeight()` 方法返回元素的高度(包括内边距).

注意是padding和border,不是margin

## jQuery遍历

`parent()` 方法返回被选元素的直接父元素

`parents()` 方法返回被选元素的所有祖先元素,它一路向上直到文档的根元素 (`<html>`)

### 支持参数筛选

	$(document).ready(function(){
	  $("span").parents("ul");
	});

`parentsUntil()` 方法返回介于两个给定元素之间的所有祖先元素

	$(document).ready(function(){
	  $("span").parentsUntil("div");
	});

`children()` 方法返回被选元素的所有直接子元素.

### 支持筛选

	$(document).ready(function(){
	  $("div").children("p.1");
	});

`find()` 方法返回被选元素的后代元素,一路向下直到最后一个后代.

举例

	$(document).ready(function(){
	  $("div").find("span");
	});

举例2

	$(document).ready(function(){
	  $("div").find("*");
	});

例1中效果描述,往下一直找,不论辈分是儿子孙子还是曾孙子,凡是符合都算数

例2效果描述,返回全部的后代

`siblings()` 方法返回被选元素的所有同胞元素.

`next()` 方法返回被选元素的下一个同胞元素.

`nextAll()` 方法返回被选元素的所有跟随的同胞元素.

`nextUntil()` 方法返回介于两个给定参数之间的所有跟随的同胞元素.

`prev()`, `prevAll()` 以及 `prevUntil()` 方法的工作方式与上面的方法类似,只不过方向相反而已：

它们返回的是前面的同胞元素(在 DOM 树中沿着同胞元素向后遍历,而不是向前).

`first()` 方法返回被选元素的首个元素.

`last()` 方法返回被选元素的最后一个元素.

`eq()` 方法返回被选元素中带有指定索引号的元素.

举例
	
	$(document).ready(function(){
	  $("p").eq(1);
	});

注意到是索引号,0开始

`filter()` 方法允许您规定一个标准.不匹配这个标准的元素会被从集合中删除,匹配的元素会被返回.

举例

	$(document).ready(function(){
	  $("p").filter(".intro");
	});

`not()` 方法返回不匹配标准的所有元素.

提示：`not()` 方法与 `filter()` 相反.


下一章AJAX另开