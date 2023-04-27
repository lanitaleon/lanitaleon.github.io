# 8.2 location对象

location提供与当前窗口加载的文档有关的信息&一些导航功能;

window.location和document.location引用的是用一个对象;

location将url解析为独立的片段:

<table>
    <tr>
        <th>属性名</th>
        <th>例子</th>
        <th>说明</th>
    </tr>
    <tr>
        <td>hash</td>
        <td>"#contents"</td>
        <td>返回URL中的hash(#号后跟零或多个字符),如果URL中不包含散列,则返回空字符串</td>
    </tr>
    <tr>
        <td>host</td>
        <td>"www.wrox.com:80"</td>
        <td>返回服务器名称和端口号(如果有)</td>
    </tr>
    <tr>
        <td>hostname</td>
        <td>"www.wrox.com"</td>
        <td>返回不带端口号的服务器名称</td>
    </tr>
    <tr>
        <td>href</td>
        <td>"http:/www.wrox.com"</td>
        <td>返回当前加载页面的完整URL,而location对象的toString()方法也返回这个值</td>
    </tr>
    <tr>
        <td>pathname</td>
        <td>"/WileyCDA/"</td>
        <td>返回URL中的目录和(或)文件名</td>
    </tr>
    <tr>
        <td>port</td>
        <td>"8080"</td>
        <td>返回URL中指定的端口号,如果URL中不包含端口号,则这个属性返回空字符串</td>
    </tr>
    <tr>
        <td>protocol</td>
        <td>"http:"</td>
        <td>返回页面使用的协议;通常是http:或https:</td>
    </tr>
    <tr>
        <td>search</td>
        <td>"?q=javascript"</td>
        <td>返回URL的查询字符串;这个字符串以问号开头</td>
    </tr>
</table>

## 查询字符串参数

例子:

```
function getQueryStringArgs() {
    //取得查询字符串并去掉开头的问号
    var qs = (location.search.length > 0 ? location.search.substring(1) : ""),
        //保存数据的对象
        args = {},
        //取得每一项
        items = qs.length ? qs.split("&") : [],
        item = null,
        name = null,
        value = null,
        //在for 循环中使用
        i,
        len = items.length;
    //逐个将每一项添加到args 对象中
    for (i = 0; i < len; i++) {
        item = items[i].split("=");
        name = decodeURIComponent(item[0]);
        value = decodeURIComponent(item[1]);
        if (name.length) {
            args[name] = value;
        }
    }
    return args;
}
```

## 位置操作

打开新的url:
```
	location.assign("http://www.wrox.com");
```

改变当前url:

```
//假设初始URL 为http://www.wrox.com/WileyCDA/
//将URL 修改为"http://www.wrox.com/WileyCDA/#section1"
location.hash = "#section1";
//将URL 修改为"http://www.wrox.com/WileyCDA/?q=javascript"
location.search = "?q=javascript";
//将URL 修改为"http://www.yahoo.com/WileyCDA/"
location.hostname = "www.yahoo.com";
//将URL 修改为"http://www.yahoo.com/mydir/"
location.pathname = "mydir";
//将URL 修改为"http://www.yahoo.com:8080/WileyCDA/"
location.port = 8080;
```

每次修改location的属性(hash除外),页面都会以新URL重新加载;

以上任意方式修改都会新增历史记录,可以通过后退按钮导航到前一个页面;

replace(),将位置改变的同时不会在历史记录中生成新记录,也就是不能后退;

```
location.replace("http://www.wrox.com/");
```
  

reload()重新加载当前页面;

```
location.reload(); //重新加载(有可能从缓存中加载)
location.reload(true); //重新加载(从服务器重新加载)
```  
  

# 8.3 navigator对象

虽然其他浏览器也通过其他方式提供了相同或相似的信息;

例如,IE中的window.clientInformation和Opera中的window.opera;

但navigator对象却是所有支持JavaScript的浏览器所共有的;

与其他BOM对象的情况一样,每个浏览器中的navigator对象也都有一套自己的属性;

然后是一大张表格,懒的搞了233

## 检测插件

非IE浏览器例子

```
//检测插件(在IE中无效)
function hasPlugin(name) {
    name = name.toLowerCase();
    for (var i = 0; i < navigator.plugins.length; i++) {
        if (navigator.plugins [i].name.toLowerCase().indexOf(name) > -1) {
            return true;
        }
    }
    return false;
}

//检测Flash
alert(hasPlugin("Flash"));
//检测QuickTime
alert(hasPlugin("QuickTime"));
```

name:插件的名字;

description:插件的描述;

filename:插件的文件名;

length:插件所处理的MIME类型数量;

IE中则突然麻烦;

IE中检测插件的唯一方式就是使用专有的ActiveXObject类型,并尝试创建一个特定插件的实例;

IE是以COM对象的方式实现插件,COM对象使用唯一标识符来标识;

也就是需要知道插件的COM标识符才可以检测这个插件;

```
//检测IE中的插件
function hasIEPlugin(name) {
    try {
        new ActiveXObject(name);
        return true;
    } catch (ex) {
        return false;
    }
}

//检测Flash
alert(hasIEPlugin("ShockwaveFlash.ShockwaveFlash"));
//检测QuickTime
alert(hasIEPlugin("QuickTime.QuickTime"));
```

因为以上方法差别很大,所以推荐针对每个插件写一个检测函数;

```
//检测所有浏览器中的Flash
function hasFlash() {
    var result = hasPlugin("Flash");
    if (!result) {
        result = hasIEPlugin("ShockwaveFlash.ShockwaveFlash");
    }
    return result;
}

//检测所有浏览器中的QuickTime
function hasQuickTime() {
    var result = hasPlugin("QuickTime");
    if (!result) {
        result = hasIEPlugin("QuickTime.QuickTime");
    }
    return result;
}

//检测Flash
alert(hasFlash());
//检测QuickTime
alert(hasQuickTime());
```

plugins集合有一个名叫refresh()的方法,用于刷新plugins以反映最新安装的插件;

这个方法接收一个参数:表示是否应该重新加载页面的一个布尔值;

如果将这个值设置为true,则会重新加载包含插件的所有页面;

否则,只更新plugins集合,不重新加载页面;


## 注册处理程序

Firefox2为navigator对象新增了registerContentHandler()和registerProtocolHandler()方法;

例子:

```
//将一个站点注册为处理RSS源的处理程序
navigator.registerContentHandler("application/rss+xml", "http://www.somereader.com?feed=%s", "Some Reader");
```

registerContentHandler()方法接收三个参数:

要处理的MIME类型/可以处理该MIME类型的页面的URL以及应用程序的名称;

第一个参数是RSS源的MIME类型;

第二个参数是应该接收RSS源URL的URL,其中的%s表示RSS源URL,由浏览器自动插入;

当下一次请求RSS源时,浏览器就会打开指定的URL,而相应的Web应用程序将以适当方式来处理该请求;

registerProtocolHandler()也接收三个参数:

要处理的协议(例如,mailto或ftp),处理该协议的页面的URL和应用程序的名称;

```
// 将一个应用程序注册为默认的邮件客户
navigator.registerProtocolHandler("mailto", "http://www.somemailclient.com?cmd=%s", "Some Mail Client");
```

Firefox2虽然实现了registerProtocolHandler(),但该方法还不能用;

Firefox3完整实现这个方法;


# 8.4 screen对象

Screen对象包含有关客户端显示屏幕的信息;

注释:没有应用于screen对象的公开标准,不过所有浏览器都支持该对象;

JavaScript中有几个对象在编程中用处不大,而screen对象就是其中之一;

233这么直接的吗

# 8.5 history对象

history对象保存着用户上网的历史记录,从窗口被打开的那一刻算起;

开发人员无法得知用户浏览过的url,但是可以随便地前进后退;

```
//后退一页,类似于单击浏览器的"后退"按钮
history.go(-1);
//前进一页,类似于单击浏览器的"前进"按钮
history.go(1);
//前进两页
history.go(2);
//跳转到最近的wrox.com 页面
history.go("wrox.com");
//跳转到最近的nczonline.net 页面
history.go("nczonline.net");
```

给go()方法传递一个字符串参数,浏览器会跳转到历史记录中包含该字符串的第一个位置;

可能后退,也可能前进,具体要看哪个位置最近;

如果历史记录中不包含该字符串,那么这个方法什么也不做;

前进后退的简写;

```
//后退一页
history.back();
//前进一页
history.forward();
```

length属性保存着历史记录的数量;

对于加载到窗口/标签页/框架中的第一个页面而言,length等于0;

```
if (history.length == 0) {
//这应该是用户打开窗口后的第一个页面
}
```

在IE8及更高版本/Opera/Firefox/Safari3及更高版本以及Chrome中,

这里所说的改变包括URL中hash的变化;

因此,设置location.hash会在这些浏览器中生成一条新的历史记录;