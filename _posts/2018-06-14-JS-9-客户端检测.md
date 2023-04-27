# 9 客户端检测

每种浏览器有各自的怪癖,客户端检测是一种行之有效的开发策略;

不到万不得已,还是应该使用最通用的方法,然后再考虑特定于浏览器的技术增强该方案;

# 9.1 能力检测

能力检测又称特性检测,就是识别浏览器的能力;

举例来说,IE5.0之前的版本不支持document.getElementById()这个DOM方法;

尽管可以用document.all属性实现相同的目的,于是;

```
function getElement(id) {
    if (document.getElementById) {
        return document.getElementById(id);
    } else if (document.all) {
        return document.all[id];
    } else {
        throw new Error("No way to retrieve element!");
    }
}
```

一个特性存在,不一定意味着另一个特性也存在;

例子:

```
function getWindowWidth() {
    if (document.all) { //假设是IE
        return document.documentElement.clientWidth; //错误的用法!!!
    } else {
        return window.innerWidth;
    }
}
```

这是一个错误使用能力检测的例子;

getWindowWidth()函数首先检查document.all是否存在,

如果是则返回document.documentElement.clientWidth;

第8章曾经讨论过,IE8及之前版本确实不支持window.innerWidth属性;

但问题是document.all存在也不一定表示浏览器就是IE;

实际上,也可能是Opera;

Opera支持document.all,也支持window.innerWidth;

## 更可靠的能力检测

检测对象是否存在有时并不可能,还应该检测这个对象是不是想要的类型;

```
//不要这样做!这不是能力检测——只检测了是否存在相应的方法
function isSortable(object) {
    return !!object.sort;
}

//这样更好:检查sort 是不是函数
function isSortable(object) {
    return typeof object.sort == "function";
}
```

这一节又是IE的主场了,因为IE跟别人不一样的种种特色;

在浏览器环境下测试任何对象的某个特性是否存在,要使用下面这个函数;

```
//作者:Peter Michaux
function isHostMethod(object, property) {
    var t = typeof object[property];
    return t == 'function' ||
        (!!(t == 'object' && object[property])) ||
        t == 'unknown';
}

result = isHostMethod(xhr, "open"); //true
result = isHostMethod(xhr, "foo"); //false
```

这里回顾一下`!!`是强转为Boolean类型;

然而各种特性的实现方式随时有可能变化,所以这个方法也不是一劳永逸的;

也不存在一劳永逸的方案;

## 能力检测不是浏览器检测

错误例子:

```
//错误!还不够具体
var isFirefox = !!(navigator.vendor && navigator.vendorSub);
//错误!假设过头了
var isIE = !!(document.all && document.uniqueID);
```

如果你知道自己的应用程序需要使用某些特定的浏览器特性,

那么最好是一次性检测所有相关特性,而不要分别检测,两个例子;

一个检测浏览器是否支持Netscapte风格的插件;

另一个检测浏览器是否具备DOM1级所规定的能力;

得到的布尔值可以在以后继续使用,从而节省重新检测能力的时间;

```
//确定浏览器是否支持Netscape 风格的插件
var hasNSPlugins = !!(navigator.plugins && navigator.plugins.length);
//确定浏览器是否具有DOM1 级规定的能力
var hasDOM1 = !!(document.getElementById && document.createElement &&
    document.getElementsByTagName);
```

# 9.2 怪癖检测

怪癖检测(quirks detection)的目标是识别浏览器的特殊行为;

与能力检测确认浏览器支持什么能力不同,

怪癖检测是想要知道浏览器存在什么缺陷,"怪癖"也就是bug;

例子:

```
var hasDontEnumQuirk = function () {
    var o = {
        toString: function () {
        }
    };
    for (var prop in o) {
        if (prop == "toString") {
            return false;
        }
    }
    return true;
}();
```

IE8及更早版本中存在一个bug,

即如果某个实例属性与[[Enumerable]]标记为false的某个原型属性同名,

那么该实例属性将不会出现在for-in循环当中;

第二个例子:

```
var hasEnumShadowsQuirk = function () {
    var o = {
        toString: function () {
        }
    };
    var count = 0;
    for (var prop in o) {
        if (prop == "toString") {
            count++;
        }
    }
    return (count > 1);
}();
```

Safari3以前版本会枚举被隐藏的属性;

一般来说,"怪癖"都是个别浏览器所独有的,而且通常被归为bug;

在相关浏览器的新版本中,这些问题可能会也可能不会被修复;

由于检测"怪癖"涉及运行代码,因此我们建议仅检测那些对你有直接影响的"怪癖";

而且最好在脚本一开始就执行此类检测,以便尽早解决问题;

# 9.3 用户代理检测

争议很大;

用户代理检测通过检测用户代理字符串来确定实际使用的浏览器;

在每一次HTTP请求过程中,用户代理字符串是作为响应首部发送的,

而且该字符串可以通过JavaScript的navigator.userAgent属性访问;

电子欺骗(spoofing):

就是指浏览器通过在自己的用户代理字符串加入一些错误或误导性信息,来达到欺骗服务器的目的;

接下来是一大段用户代理字符串的历史;

emmm真是充满商战意味的字符串;

### 识别呈现引擎

一个识别IE/Gecko/WebKit/KHTML/Opera呈现引擎的例子;

```
var client = function () {
    var engine = {
        //呈现引擎
        ie: 0,
        gecko: 0,
        webkit: 0,
        khtml: 0,
        opera: 0,
        //具体的版本号
        ver: null
    };
    //在此检测呈现引擎、平台和设备
    return {
        engine: engine
    };
}();

if (client.engine.ie) { //如果是IE,client.ie 的值应该大于0
    //针对IE 的代码
} else if (client.engine.gecko > 1.5) {
    if (client.engine.ver == "1.8.1") {
    //针对这个版本执行某些操作
    }
}
```

要正确地识别呈现引擎,关键是检测顺序要正确;

第一步是识别Opera,因为它的用户代理字符串有可能完全模仿其他浏览器,

我们不相信Opera,是因为(任何情况下)其用户代理字符串(都)不会将自己标识为Opera;

dbq,xswl;

直接上例子:

```
var ua = navigator.userAgent;
if (window.opera){
engine.ver = window.opera.version();
engine.opera = parseFloat(engine.ver);
} else if (/AppleWebKit\/(\S+)/.test(ua)){
engine.ver = RegExp["$1"];
engine.webkit = parseFloat(engine.ver);
} else if (/KHTML\/(\S+)/.test(ua)) {
engine.ver = RegExp["$1"];
engine.khtml = parseFloat(engine.ver);
} else if (/rv:([^\)]+)\) Gecko\/\d{8}/.test(ua)){
engine.ver = RegExp["$1"];
engine.gecko = parseFloat(engine.ver);
} else if (/MSIE ([^;]+)/.test(ua)){
engine.ver = RegExp["$1"];
engine.ie = parseFloat(engine.ver);
}
```

### 识别浏览器

添加属性browser保存每个浏览器的属性;

```
var client = function () {
    var engine = {
        //呈现引擎
        ie: 0,
        gecko: 0,
        webkit: 0,
        khtml: 0,
        opera: 0,
        //具体的版本
        ver: null
    };
    var browser = {
        //浏览器
        ie: 0,
        firefox: 0,
        safari: 0,
        konq: 0,
        opera: 0,
        chrome: 0,
        //具体的版本
        ver: null
    };
    //在此检测呈现引擎、平台和设备
    return {
        engine: engine,
        browser: browser
    };
}();

//检测呈现引擎及浏览器
var ua = navigator.userAgent;
if (window.opera) {
    engine.ver = browser.ver = window.opera.version();
    engine.opera = browser.opera = parseFloat(engine.ver);
} else if (/AppleWebKit\/(\S+)/.test(ua)) {
    engine.ver = RegExp["$1"];
    engine.webkit = parseFloat(engine.ver);
//确定是Chrome 还是Safari
    if (/Chrome\/(\S+)/.test(ua)) {
        browser.ver = RegExp["$1"];
        browser.chrome = parseFloat(browser.ver);
    } else if (/Version\/(\S+)/.test(ua)) {
        browser.ver = RegExp["$1"];
        browser.safari = parseFloat(browser.ver);
    } else {
//近似地确定版本号
        var safariVersion = 1;
        if (engine.webkit < 100) {
            safariVersion = 1;
        } else if (engine.webkit < 312) {
            safariVersion = 1.2;
        } else if (engine.webkit < 412) {
            safariVersion = 1.3;
        } else {
            safariVersion = 2;
        }
        browser.safari = browser.ver = safariVersion;
    }
} else if (/KHTML\/(\S+)/.test(ua) || /Konqueror\/([^;]+)/.test(ua)) {
    engine.ver = browser.ver = RegExp["$1"];
    engine.khtml = browser.konq = parseFloat(engine.ver);
} else if (/rv:([^\)]+)\) Gecko\/\d{8}/.test(ua)) {
    engine.ver = RegExp["$1"];
    engine.gecko = parseFloat(engine.ver);
//确定是不是Firefox
    if (/Firefox\/(\S+)/.test(ua)) {
        browser.ver = RegExp["$1"];
        browser.firefox = parseFloat(browser.ver);
    }
} else if (/MSIE ([^;]+)/.test(ua)) {
    engine.ver = browser.ver = RegExp["$1"];
    engine.ie = browser.ie = parseFloat(engine.ver);
}

if (client.engine.webkit) { //if it’s WebKit
    if (client.browser.chrome){
    //执行针对Chrome 的代码
    } else if (client.browser.safari){
    //执行针对Safari 的代码
    }
} else if (client.engine.gecko){
    if (client.browser.firefox){
    //执行针对Firefox 的代码
    } else {
    //执行针对其他Gecko 浏览器的代码
    }
}
```


### 识别平台

添加system对象

```
var client = function () {
    var engine = {
//呈现引擎
        ie: 0,
        gecko: 0,
        webkit: 0,
        khtml: 0,
        opera: 0,
//具体的版本号
        ver: null
    };
    var browser = {
//浏览器
        ie: 0,
        firefox: 0,
        safari: 0,
        konq: 0,
        opera: 0,
        chrome: 0,
//具体的版本号
        ver: null
    };
    var system = {
        win: false,
        mac: false,
        x11: false
    };
//在此检测呈现引擎、平台和设备
    return {
        engine: engine,
        browser: browser,
        system: system
    };
}();

var p = navigator.platform;
system.win = p.indexOf("Win") == 0;
system.mac = p.indexOf("Mac") == 0;
system.x11 = (p.indexOf("X11") == 0) || (p.indexOf("Linux") == 0);
```

navigator.platform不像代理字符串那样魔幻,它可能的值有:

Win32,Win64,MacPPC,MacIntel,Xll,Linux i686;

这些值在不同的浏览器中都是一致的;

只检测Win,Mac,Xll,Linux是否存在是为了兼容以后出现其他变体;

### 识别windows操作系统

在用户代理字符串中获取操作系统的信息并不直观,因为各大浏览器标识的字符串不一样;

### 识别移动设备

是的,又要添加属性了;

```
var client = function () {
    var engine = {
//呈现引擎
        ie: 0,
        gecko: 0,
        webkit: 0,
        khtml: 0,
        opera: 0,
//具体的版本号
        ver: null
    };
    var browser = {
//浏览器
        ie: 0,
        firefox: 0,
        safari: 0,
        konq: 0,
        opera: 0,
        chrome: 0,
//具体的版本号
        ver: null
    };
    var system = {
        win: false,
        mac: false,
        x11: false,
//移动设备
        iphone: false,
        ipod: false,
        ipad: false,
        ios: false,
        android: false,
        nokiaN: false,
        winMobile: false
    };
//在此检测呈现引擎、平台和设备
    return {
        engine: engine,
        browser: browser,
        system: system
    };
}();

system.iphone = ua.indexOf("iPhone") > -1;
system.ipod = ua.indexOf("iPod") > -1;
system.ipod = ua.indexOf("iPad") > -1;

//检测iOS 版本
if (system.mac && ua.indexOf("Mobile") > -1) {
    if (/CPU (?:iPhone )?OS (\d+_\d+)/.test(ua)) {
        system.ios = parseFloat(RegExp.$1.replace("_", "."));
    } else {
        system.ios = 2; //不能真正检测出来,所以只能猜测
    }
}

//检测Android 版本
if (/Android (\d+\.\d+)/.test(ua)) {
    system.android = parseFloat(RegExp.$1);
}
```

### 识别游戏系统

这都能看见任天堂和PlayStation2333;

```
Opera/9.10 (Nintendo Wii;U; ; 1621; en)
Mozilla/5.0 (PLAYSTATION 3; 2.00)
```

添加属性;

dbq不贴代码了,突然看到下一节是完整代码,仿佛在逗我(

## 完整识别的代码

包含识别引擎,平台,Windows,移动设备和游戏系统;

```
var client = function () {
    //呈现引擎
    var engine = {
        ie: 0,
        gecko: 0,
        webkit: 0,
        khtml: 0,
        opera: 0,
        //完整的版本号
        ver: null
    };
    //浏览器
    var browser = {
        //主要浏览器
        ie: 0,
        firefox: 0,
        safari: 0,
        konq: 0,
        opera: 0,
        chrome: 0,
        //具体的版本号
        ver: null
    };
    //平台、设备和操作系统
    var system = {
        win: false,
        mac: false,
        x11: false,
        //移动设备
        iphone: false,
        ipod: false,
        ipad: false,
        ios: false,
        android: false,
        nokiaN: false,
        winMobile: false,
        //游戏系统
        wii: false,
        ps: false
    };
    //检测呈现引擎和浏览器
    var ua = navigator.userAgent;
    if (window.opera) {
        engine.ver = browser.ver = window.opera.version();
        engine.opera = browser.opera = parseFloat(engine.ver);
    } else if (/AppleWebKit\/(\S+)/.test(ua)) {
        engine.ver = RegExp["$1"];
        engine.webkit = parseFloat(engine.ver);
        //确定是Chrome 还是Safari
        if (/Chrome\/(\S+)/.test(ua)) {
            browser.ver = RegExp["$1"];
            browser.chrome = parseFloat(browser.ver);
        } else if (/Version\/(\S+)/.test(ua)) {
            browser.ver = RegExp["$1"];
            browser.safari = parseFloat(browser.ver);
        } else {
            //近似地确定版本号
            var safariVersion = 1;
            if (engine.webkit < 100) {
                safariVersion = 1;
            } else if (engine.webkit < 312) {
                safariVersion = 1.2;
            } else if (engine.webkit < 412) {
                safariVersion = 1.3;
            } else {
                safariVersion = 2;
            }
            browser.safari = browser.ver = safariVersion;
        }
    } else if (/KHTML\/(\S+)/.test(ua) || /Konqueror\/([^;]+)/.test(ua)) {
        engine.ver = browser.ver = RegExp["$1"];
        engine.khtml = browser.konq = parseFloat(engine.ver);
    } else if (/rv:([^\)]+)\) Gecko\/\d{8}/.test(ua)) {
        engine.ver = RegExp["$1"];
        engine.gecko = parseFloat(engine.ver);
        //确定是不是Firefox
        if (/Firefox\/(\S+)/.test(ua)) {
            browser.ver = RegExp["$1"];
            browser.firefox = parseFloat(browser.ver);
        }
    } else if (/MSIE ([^;]+)/.test(ua)) {
        engine.ver = browser.ver = RegExp["$1"];
        engine.ie = browser.ie = parseFloat(engine.ver);
    }
    //检测浏览器
    browser.ie = engine.ie;
    browser.opera = engine.opera;
    //检测平台
    var p = navigator.platform;
    system.win = p.indexOf("Win") == 0;
    system.mac = p.indexOf("Mac") == 0;
    system.x11 = (p == "X11") || (p.indexOf("Linux") == 0);
    //检测Windows 操作系统
    if (system.win) {
        if (/Win(?:dows )?([^do]{2})\s?(\d+\.\d+)?/.test(ua)) {
            if (RegExp["$1"] == "NT") {
                switch (RegExp["$2"]) {
                    case "5.0":
                        system.win = "2000";
                        break;
                    case "5.1":
                        system.win = "XP";
                        break;
                    case "6.0":
                        system.win = "Vista";
                        break;
                    case "6.1":
                        system.win = "7";
                        break;
                    default:
                        system.win = "NT";
                        break;
                }
            } else if (RegExp["$1"] == "9x") {
                system.win = "ME";
            } else {
                system.win = RegExp["$1"];
            }
        }
    }
    //移动设备
    system.iphone = ua.indexOf("iPhone") > -1;
    system.ipod = ua.indexOf("iPod") > -1;
    system.ipad = ua.indexOf("iPad") > -1;
    system.nokiaN = ua.indexOf("NokiaN") > -1;
    //windows mobile
    if (system.win == "CE") {
        system.winMobile = system.win;
    } else if (system.win == "Ph") {
        if (/Windows Phone OS (\d+.\d+)/.test(ua)) {
            ;
            system.win = "Phone";
            system.winMobile = parseFloat(RegExp["$1"]);
        }
    }
    //检测iOS 版本
    if (system.mac && ua.indexOf("Mobile") > -1) {
        if (/CPU (?:iPhone )?OS (\d+_\d+)/.test(ua)) {
            system.ios = parseFloat(RegExp.$1.replace("_", "."));
        } else {
            system.ios = 2; //不能真正检测出来,所以只能猜测
        }
    }
    //检测Android 版本
    if (/Android (\d+\.\d+)/.test(ua)) {
        system.android = parseFloat(RegExp.$1);
    }
    //游戏系统
    system.wii = ua.indexOf("Wii") > -1;
    system.ps = /playstation/i.test(ua);
    //返回这些对象
    return {
        engine: engine,
        browser: browser,
        system: system
    };
}();
```

## 使用方法

用户代理检测是客户端检测的最后一个选择;

只要可能,都应该优先采用能力检测和怪癖检测;

用户代理检测一般适用于下列情形:

1.不能直接准确地使用能力检测或怪癖检测;

例如,某些浏览器实现了为将来功能预留的存根(stub)函数;

在这种情况下,仅测试相应的函数是否存在还得不到足够的信息;

2.同一款浏览器在不同平台下具备不同的能力;

这时候,可能就有必要确定浏览器位于哪个平台下;

3.为了跟踪分析等目的需要知道确切的浏览器;

