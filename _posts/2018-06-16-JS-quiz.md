<blockquote>JS学习日常所得</blockquote>

#### 查 [filter](https://juejin.im/post/5a5f3eaf518825733201a6a7) 的时候看到关于去重的三种方式

    var arrS = [1, 2, 3, 4, 5, 5, 5, 8, 8, 2, 3, 9, 4, 1, 2, 'a', 'a', 'c', 'd'];
    var resF = arrS.filter(function (item, idx, arr) {
        return arr.indexOf(item) === idx;
    });
    //这个去重明显是依赖indexOf,跟filter关系不大吧
    console.log(resF);
    //数组[]数字还是数字,字符串还是字符串
    var arrSet = new Set(arrS);
    console.log(arrSet);
    //set{}数字还是数字,字符串还是字符串
    var obj = {};
    arrS.forEach(function (item) { obj[item] = item });
    // keys 返回所有可枚举属性的字符串数组
    console.log(Object.keys(obj));
    //数组[]全部是字符串

filter结合map做数据筛选的方式,评论提到性能问题,有些认同,但没有测试

#### 一个很*&&……%￥的面试题

[can (a==1 && a==2 && a==3) ever evaluate to true](https://stackoverflow.com/questions/48270127/can-a-1-a-2-a-3-ever-evaluate-to-true)

有意思的是

    a = [1,2,3];
    a.join = a.shift;
    console.log(a == 1 && a == 2 && a == 3);

最高赞同解

    const a = {
      i: 1,
      toString: function () {
        return a.i++;
      }
    }

    if(a == 1 && a == 2 && a == 3) {
      console.log('Hello World!');
    }

这里`toString`等同于`valueOf`

由此看到[宽松相等](https://www.zhihu.com/question/46943112/answer/122096589)

    [] == ![]

解释如下:
1. 等号右边有!,优先级比==高,先计算右边结果;
2. []为非假值,false;
3. ==任意一边有boolean,,将其转为number,右边变成0;
4. ==两边为object和boolean,将object转number

    Number([].valueOf()) ==> 0

至此,等式成立.

#### 然后发现

    [] == []

返回false,以及

    null == 0 >> false

    null >= 0 >> true

    null > 0 >> false

首先`[]==[]`因为不是一个object,然后

因为`==`和`>=`,`>`也就是相等运算符和关系运算符并不是一类:

    关系运算符,在设计上,总是需要运算元尝试转为一个number,而相等运算符在设计上,则没有这方面的考虑

`==`的规则是这个样子的:

1、当两个运算数的类型不同时,将它们转换成相同的类型：

1)一个数字与一个字符串,字符串转换成数字之后,进行比较.

2)true转换为1、false转换为0,进行比较.

3)一个对象、数组、函数 与 一个数字或字符串,对象、数组、函数转换为原始类型的值,然后进行比较(先使用valueOf,如果不行就使用toString).

2、当两个运算数类型相同,或转换成相同类型后：

1)2个字符串：同一位置上的字符相等,2个字符串就相同.

2)2个数字：2个数字相同,就相同.如果一个是NaN,或两个都是NaN,则不相同.

3)2个都是true,或者2个都是false,则相同.

4)2个引用的是同一个对象、函数、数组,则它们相等,如果引用的不是同一个对象、函数、数组,则不相同,即使这2个对象、函数、数组可以转换成完全相等的原始值.

5)2个null,或者2个都是未定义的,那么它们相等.

有[表格版描述](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Equality_comparisons_and_sameness)如下

<table class="standard-table">
 <thead>
  <tr>
   <th scope="row">&nbsp;</th>
   <th colspan="7" style="text-align: center;" scope="col">被比较值 B</th>
  </tr>
 </thead>
 <tbody>
  <tr>
   <th scope="row">&nbsp;</th>
   <td>&nbsp;</td>
   <td style="text-align: center;">Undefined</td>
   <td style="text-align: center;">Null</td>
   <td style="text-align: center;">Number</td>
   <td style="text-align: center;">String</td>
   <td style="text-align: center;">Boolean</td>
   <td style="text-align: center;">Object</td>
  </tr>
  <tr>
   <th colspan="1" rowspan="6" scope="row">被比较值 A</th>
   <td>Undefined</td>
   <td style="text-align: center;"><code>true</code></td>
   <td style="text-align: center;"><code>true</code></td>
   <td style="text-align: center;"><code>false</code></td>
   <td style="text-align: center;"><code>false</code></td>
   <td style="text-align: center;"><code>false</code></td>
   <td style="text-align: center;"><code>IsFalsy(B)</code></td>
  </tr>
  <tr>
   <td>Null</td>
   <td style="text-align: center;"><code>true</code></td>
   <td style="text-align: center;"><code>true</code></td>
   <td style="text-align: center;"><code>false</code></td>
   <td style="text-align: center;"><code>false</code></td>
   <td style="text-align: center;"><code>false</code></td>
   <td style="text-align: center;"><code>IsFalsy(B)</code></td>
  </tr>
  <tr>
   <td>Number</td>
   <td style="text-align: center;"><code>false</code></td>
   <td style="text-align: center;"><code>false</code></td>
   <td style="text-align: center;"><code>A === B</code></td>
   <td style="text-align: center;"><code>A === ToNumber(B)</code></td>
   <td style="text-align: center;"><code>A=== ToNumber(B) </code></td>
   <td style="text-align: center;"><code>A=== ToPrimitive(B)&nbsp;</code></td>
  </tr>
  <tr>
   <td>String</td>
   <td style="text-align: center;"><code>false</code></td>
   <td style="text-align: center;"><code>false</code></td>
   <td style="text-align: center;"><code>ToNumber(A) === B</code></td>
   <td style="text-align: center;"><code>A === B</code></td>
   <td style="text-align: center;"><code>ToNumber(A) === ToNumber(B)</code></td>
   <td style="text-align: center;"><code>ToPrimitive(B) == A</code></td>
  </tr>
  <tr>
   <td>Boolean</td>
   <td style="text-align: center;"><code>false</code></td>
   <td style="text-align: center;"><code>false</code></td>
   <td style="text-align: center;"><code>ToNumber(A) === B</code></td>
   <td style="text-align: center;"><code>ToNumber(A) === ToNumber(B)</code></td>
   <td style="text-align: center;"><code>A === B</code></td>
   <td style="text-align: center;">ToNumber(A) == ToPrimitive(B)</td>
  </tr>
  <tr>
   <td>Object</td>
   <td style="text-align: center;"><font face="Consolas, Liberation Mono, Courier, monospace">false</font></td>
   <td style="text-align: center;"><font face="Consolas, Liberation Mono, Courier, monospace">false</font></td>
   <td style="text-align: center;"><code>ToPrimitive(A) == B</code></td>
   <td style="text-align: center;"><code>ToPrimitive(A) == B</code></td>
   <td style="text-align: center;">ToPrimitive(A) == ToNumber(B)</td>
   <td style="text-align: center;">
    <p><code>A === B</code></p>
   </td>
  </tr>
 </tbody>
</table>


### debounce&&throttling

原文[debounce&throttling实例](http://www.css88.com/archives/7010)

感觉就是对settimeout和setinterval的封装

`debounce` 合并一段时间内的重复连续操作,如连续快速点击鼠标

`throttle` 只允许一个函数在x毫秒内执行一次(x毫秒内至少执行一次)

`immediate` debounce的执行在一连串操作之后,immediate(leading)发生在一连串操作刚开始

现有封装好的库underscore/Lodash,Lodash支持自定义需要的函数生成压缩库


#### Promise例子

    new Promise(function (resolve, reject) {
        log('start new Promise...');
        var timeOut = Math.random() * 2;
        log('set timeout to: ' + timeOut + ' seconds.');
        setTimeout(function () {
            if (timeOut < 1) {
                log('call resolve()...');
                resolve('200 OK');
            }
            else {
                log('call reject()...');
                reject('timeout in ' + timeOut + ' seconds.');
            }
        }, timeOut * 1000);
    }).then(function (r) {
        log('Done: ' + r);
    }).catch(function (reason) {
        log('Failed: ' + reason);
    });

就像ajax一样分离处理结果,然后还可以

    job1.then(job2).then(job3).catch(handleError);

避免顺序执行的嵌套,然后针对容错`race`

    var p1 = new Promise(function (resolve, reject) {
        setTimeout(resolve, 500, 'P1');
    });
    var p2 = new Promise(function (resolve, reject) {
        setTimeout(resolve, 600, 'P2');
    });
    Promise.race([p1, p2]).then(function (result) {
        console.log(result); // 'P1'
    });

p1执行得快,p2虽然还在执行,但是结果将被抛弃

### [奇怪的面试题](https://www.liayal.com/article/5abde53da6cf4e67bc05c9ea)

1. 引用和赋值
<pre>
<code>
var foo = {n: 1}; // foo n1
var bar = foo;  // bar === foo
// 0. foo,bar > x0A 赋值已经完成
foo.x = foo = {n: 2};
// 先计算左边 再计算右边 再赋值
// 1. foo.x > foo = x0A > foo n1 x
// 先计算左边 给之前的foo也就是foo-old一个x属性
// * 这里为新的x属性分配了内存空间x1C?但是并没有对x赋值
// 2. foo > x1B foo.x > x1B
// * 接着计算右边 foo-new被赋值x1B 于是x属性也就是x1C指向x1B 该行计算结束
// 3. bar指向 x0A没有改变过
// * x0A 该地址内是 n1 x:n2 x1B地址内是 n2 x2C地址内是 n2
console.log(foo.x);  //undefined
console.log(bar.x);  //{n:2}
</code>
</pre>

2. 形参和匿名函数

<pre>
<code>
(function(x, f = () => x) { 
// 首先这里给参数f默认赋值了一个匿名函数,由于作用域的关系,函数f是不能访问到函数内的x的
    var x;
    // 只进行了声明而没有赋值,所以在作用域链还会找到形参x
    // 这里试了一下如果x有赋值比如5,y会被赋值成5
    var y = x; // 这里y的值取的还是形参x的值
    x = 2; 
    // 这里对上面的var x进行赋值而形参x的值是不受影响的console.log(arguments[0]试一下,所以f()返回是1,此时作用域链上会先找到函数内声明的x
    return [x, y, f()]; // [2, 1, 1]
})(1);
(function(x, f = () => x) {
    var y = x; // 这里只声明了y, x 还是形参x
    x = 2; // 这里改变了形参x的值,所以 f() 返回是 2
    return [x, y, f()]; // [2, 1, 2]
})(1)
</code>
</pre>
默认参数可用于后面的默认参数,所以f可以获取前面的x形参但是不能访问到函数内部的x

### [1]+[2]-[3] === 9

[1]+[2] //"12"

-[3] //-3

"12"-3 //9

以及发现

"123"*1 //123

### Date

使用Date.parse()时传入的字符串使用实际月份01~12,转换为Date对象后getMonth()获取的月份值为0~11

JavaScript的Date对象月份值从0开始,牢记0=1月,1=2月,2=3月,……,11=12月.