---
title: web前端性能优化
categories: h5
date: 2021-12-23 20:00:00
---
<a name="eFFLu"></a>
# 一、加载优化
<a name="w4GSp"></a>
### 1、减少HTTP请求
一个完整的 HTTP 请求需要经历 DNS 查找，TCP 握手，浏览器发出 HTTP 请求，服务器接收请求，服务器处理请求并发回响应，浏览器接收响应等过程。接下来看一个具体的例子帮助理解 HTTP ：<br />![http.png](/images/h5/performance-optimization/http.png)<br />这是一个 HTTP 请求，请求的文件大小为 11.4KB<br />​

**名词解释：**

- **Queueing:** 在请求队列中的时间。
- **Stalled:** 从TCP 连接建立完成，到真正可以传输数据之间的时间差，此时间包括代理协商时间。
- **Initial Connection / Connecting:** 建立连接所花费的时间，包括TCP握手/重试和协商SSL
- **SSL:** 完成SSL握手所花费的时间。。
- **Request sent:** 发出网络请求所花费的时间，通常为一毫秒的时间。
- **Waiting(TFFB): **TFFB 是发出页面请求到接收到应答数据第一个字节的时间。
- **Con	tent Download: **接收响应数据所花费的时间。


<br />从这个例子可以看出，真正下载数据的时间占比为 0.84 / 39.63 = 2.12%，文件越小，这个比例越小，文件越大，比例就越高。这就是为什么要建议将多个小文件合并为一个大文件，从而减少 HTTP 请求次数的原因。<br />
<br />**优化建议：**

- 合并脚本文件或CSS文件。
- CSS Sprites利用CSS background相关元素进行背景图绝对定位,把多个图片合成一个图片。



<a name="XUP52"></a>
### 2、使用HTTP2
HTTP2 相比 HTTP1.1 有如下几个优点：

- **解析速度快**

服务器解析 HTTP1.1 的请求时，必须不断地读入字节，直到遇到分隔符 CRLF 为止。而解析 HTTP2 的请求就不用这么麻烦，因为 HTTP2 是基于帧的协议，每个帧都有表示帧长度的字段。

- **多路复用**

HTTP1.1 如果要同时发起多个请求，就得建立多个 TCP 连接，因为一个 TCP 连接同时只能处理一个 HTTP1.1 的请求。<br />在 HTTP2 上，多个请求可以共用一个 TCP 连接，这称为多路复用。同一个请求和响应用一个流来表示，并有唯一的流 ID 来标识。 多个请求和响应在 TCP 连接中可以乱序发送，到达目的地后再通过流 ID 重新组建。

- **头部压缩和二进制格式**

http1.x一直都是plain text,对此我只能想到一个优点，便于阅读和debug。但是，现在很多都走https，SSL也把plain text变成了二进制，那这个优点也没了。于是HTTP2搞了个HPACK压缩来压缩头部，减少报文大小(调试这样的协议将需要curl这样的工具，要进一步地分析网络数据流需要类似Wireshark的http2解析器)。

- **服务器推送**

这个功能通常被称作“缓存推送”。主要的思想是：当一个客户端请求资源X，而服务器知道它很可能也需要资源Z的情况下，服务器可以在客户端发送请求前，主动将资源Z推送给客户端。<br />这个功能帮助客户端将Z放进缓存以备将来之需。<br />​<br />
<a name="AFgXI"></a>
### 3、静态资源使用 CDN
内容分发网络（CDN）是一组分布在多个不同地理位置的 Web 服务器。我们都知道，当服务器离用户越远时，延迟越高。CDN 就是为了解决这一问题，在多个位置部署服务器，让用户离服务器更近，从而缩短请求时间。<br />​

**当用户访问一个网站时，如果没有 CDN，过程是这样的：**

1. 浏览器要将域名解析为 IP 地址，所以需要向本地 DNS 发出请求。
1. 本地 DNS 依次向根服务器、顶级域名服务器、权限服务器发出请求，得到网站服务器的 IP 地址。
1. 本地 DNS 将 IP 地址发回给浏览器，浏览器向网站服务器 IP 地址发出请求并得到资源。


<br />**如果用户访问的网站部署了 CDN，过程是这样的：**

1. 浏览器要将域名解析为 IP 地址，所以需要向本地 DNS 发出请求。
1. 本地 DNS 依次向根服务器、顶级域名服务器、权限服务器发出请求，得到全局负载均衡系统（GSLB）的 IP 地址。
1. 本地 DNS 再向 GSLB 发出请求，GSLB 的主要功能是根据本地 DNS 的 IP 地址判断用户的位置，筛选出距离用户较近的本地负载均衡系统（SLB），并将该 SLB 的 IP 地址作为结果返回给本地 DNS。
1. 本地 DNS 将 SLB 的 IP 地址发回给浏览器，浏览器向 SLB 发出请求。
1. SLB 根据浏览器请求的资源和地址，选出最优的缓存服务器发回给浏览器。
1. 浏览器再根据 SLB 发回的地址重定向到缓存服务器。
1. 如果缓存服务器有浏览器需要的资源，就将资源发回给浏览器。如果没有，就向源服务器请求资源，再发给浏览器并缓存在本地。



<a name="Gpg8F"></a>
### 4、优化资源加载

- 优化资源加载位置。更改资源加载时机,使尽可能快地展示出页面内容,尽可能快地使功能可用。
   - CSS文件放在head中，先外联css，后内联css；
   - JS文件放在body底部，先外联css，后内联js；
   - 处理页面、处理页面布局的JS文件放在head中；
   - body中间尽量不写style标签和script标签。
   - 使用defer和async异步加载script脚本。使用defer异步加载,在HTML解析完成后执行,defer的实际效果与将代码放在body底部类似。使用async进行异步加载,加载完成后立即执行。
- 使用资源预加载preload和资源预读取prefetch。
   - preload让浏览器提前加载指定资源,需要执行时再执行,可以加速本页面的加载速度。
   - prefetch告诉浏览器加载下一页面可能会用到的资源,可以加速下一个页面的加载速度。
   - preconnect使浏览器能够预先建立一个连接，等真正需要加载资源的时候就能够直接请求了。
- 不需要自动播放的视频文件，真实url存放data-src，通过播放事件动态设置src属性
- 延迟加载，优先加载首屏内容,非首屏内容延迟
- 使用字体图标 iconfont 代替图片图标
- 使用 fontmin-webpack 插件对字体文件进行压缩

![image.png](/images/h5/performance-optimization/image.png)

- 使用骨架屏或loading动画优化体验
<a name="mKJOL"></a>
# 二、图像优化
<a name="SliP8"></a>
### 1、图片延迟加载
在页面中，先不给图片设置路径，只有当图片出现在浏览器的可视区域时，才去加载真正的图片，这就是延迟加载。对于图片很多的网站来说，一次性加载全部图片，会对用户体验造成很大的影响，所以需要使用图片延迟加载。<br />首先可以将图片这样设置，在页面不可见时图片不会加载：
```html
<img data-src="https://avatars0.githubusercontent.com/u/22117876?s=460&u=7bd8f32788df6988833da6bd155c3cfbebc68006&v=4">
```
等页面可见时，使用 JS 加载图片：
```javascript
const img = document.querySelector('img')
img.src = img.dataset.src
```
<a name="LEK4s"></a>
### 2、响应式图片
响应式图片的优点是浏览器能够根据屏幕大小自动加载合适的图片。

- 通过 picture 实现
```javascript
<picture>
	<source srcset="banner_w1000.jpg" media="(min-width: 801px)">
	<source srcset="banner_w800.jpg" media="(max-width: 800px)">
	<img src="banner_w800.jpg" alt="">
</picture>
```
通过 @media 实现
```javascript
@media (min-width: 769px) {
	.bg {
		background-image: url(bg1080.jpg);
	}
}
@media (max-width: 768px) {
	.bg {
		background-image: url(bg768.jpg);
	}
}
```
<a name="Nm71f"></a>
### 3、利用缩略图
例如，你有一个 1920 * 1080 大小的图片，用缩略图的方式展示给用户，并且当用户鼠标悬停在上面时才展示全图。如果用户从未真正将鼠标悬停在缩略图上，则浪费了下载图片的时间。<br />所以，我们可以用两张图片来实行优化。一开始，只加载缩略图，当用户悬停在图片上时，才加载大图。还有一种办法，即对大图进行延迟加载，在所有元素都加载完成后手动更改大图的 src 进行下载。<br />​<br />
<a name="TyukC"></a>
### 4、尽可能利用 CSS3 效果代替图片
有很多图片使用 CSS 效果（渐变、阴影等）就能画出来，这种情况选择 CSS3 效果更好。因为代码大小通常是图片大小的几分之一甚至几十分之一。<br />​<br />
<a name="J4L7b"></a>
### 5、选择合适格式的图片
在大多数的web页面中，图片占到了页面大小的60%-70%。因此在web开发中，不同的场景使用合适的图片格式对web页面的性能和体验是很重要的。<br />![image.png](/images/h5/performance-optimization/image2.png)<br />注：可以使用svg格式图片来实现品牌logo、控件图标等结构简单的几何图形。减少svg标签里面不必要的信息，如xml命名、图层及注释信息等可以有效减小svg的大小。
<a name="HLy2p"></a>
# 三、构建优化
<a name="StjZM"></a>
### 1、写高性能的代码
**css部分：**

- **css尽量使用简写**
```css
.box {
	margin-top: 0px;
	margin-right: 5px;
  margin-bottom: 0px;
	margin-right: 5px;
}
```
将上述代码简写为：
```css
.box {
	margin: 0 5px;
}
```

- **避免使用CSS表达式**

CSS表达式可以动态的设置CSS属性，在IE5-IE8中支持，其他浏览器中表达式会被忽略。例如下面表达式在不同时间设置不同的背景颜色。
```css
background-color: expression( (new Date()).getHours()%2 ? "#B8D4FF" : "#F08A00" );
```
CSS表达式的问题在于它被重新计算的次数远比我们想象的要多，不仅在网页绘制或大小改变时计算，即使我们滚动屏幕或者移动鼠标的时候也在计算，因此我们还是尽量避免使用它来防止使用不当而造成的性能损耗。<br />如果想达到类似的效果我们可以通过简单的脚本做到
```javascript
var currentTime = new Date().getHours();
if (currentTime % 2) {
    if (document.body) {
        document.body.style.background = "#B8D4FF";
    }
}
else {
    if (document.body) {
        document.body.style.background = "#F08A00";
    }
}
```

- **降低 CSS 选择器的复杂性**

浏览器读取选择器，遵循的原则是从选择器的右边到左边读取。<br />看个示例：
```css
#block .text p {
	color: red;
}
```

1. 查找所有 P 元素。
1. 查找结果 1 中的元素是否有类名为 text 的父元素
1. 查找结果 2 中的元素是否有 id 为 block 的父元素


<br />**CSS 选择器优先级：内联 > ID选择器 > 类选择器 > 标签选择器**<br />根据以上两个信息可以得出结论：

1. 选择器越短越好。
1. 尽量使用高优先级的选择器，例如 ID 和类选择器。
1. 避免使用通配符 *。



- **避免使用@import**

避免使用@import的原因很简单，因为它相当于将css放在网页内容底部，会覆盖之前的样式。

- **将样式表置顶**

经样式表(css)放在网页的HEAD中会让网页显得加载速度更快，因为这样做可以使浏览器逐步加载已将下载的网页内容。这对内容比较多的网页尤其重要，用户不用一直等待在一个白屏上，而是可以先看已经下载的内容。<br />如果将样式表放在底部，浏览器会拒绝渲染已经下载的网页，因为大多数浏览器在实现时都努力避免重绘，样式表中的内容是绘制网页的关键信息，没有下载下来之前只好对不起观众了。<br />​

**js部分：**

- **减少条件判断**

当判断条件数量越来越多时，越倾向于使用 switch 而不是 if-else
```javascript
if (color == 'blue') {

} else if (color == 'yellow') {

} else if (color == 'white') {

} else if (color == 'black') {

} else if (color == 'green') {

} else if (color == 'orange') {

} else if (color == 'pink') {

}

switch (color) {
    case 'blue':

        break
    case 'yellow':

        break
    case 'white':

        break
    case 'black':

        break
    case 'green':

        break
    case 'orange':

        break
    case 'pink':

        break
}
```
像上面的这种情况，从可读性来说，使用 switch 是比较好的（js 的 switch 语句不是基于哈希实现，而是循环判断，所以说 if-else、switch 从性能上来说是一样的，但是代码量却少了很多）。<br />​

当条件语句特别多时，使用 switch 和 if-else 不是最佳的选择，这时不妨试一下查找表。查找表可以使用数组和对象来构建。
```javascript
switch (index) {
    case '0':
        return result0
    case '1':
        return result1
    case '2':
        return result2
    case '3':
        return result3
    case '4':
        return result4
    case '5':
        return result5
    case '6':
        return result6
    case '7':
        return result7
    case '8':
        return result8
    case '9':
        return result9
    case '10':
        return result10
    case '11':
        return result11
}
```
可以将这个 switch 语句转换为查找表
```javascript
const results = [result0,result1,result2,result3,result4,result5,result6,result7,result8,result9,result10,result11]

return results[index]
```

- **使用局部变量而非全局变量**
- **不要覆盖原生方法**

无论你的 JavaScript 代码如何优化，都比不上原生方法。因为原生方法是用低级语言写的（C/C++），并且被编译成机器码，成为浏览器的一部分。当原生方法可用时，尽量使用它们，特别是数学运算和 DOM 操作。

- **去除重复脚本**

重复的脚本不仅浪费浏览器的下载时间，而且浪费解析和执行时间。一般用来避免引入重复脚本的做法是使用统一的脚本管理模块，这样不仅可以避免重复脚本引入，还可以兼顾脚本依赖管理和版本管理。

- **减少DOM操作**

通过Javascript访问DOM元素没有我们想象中快，元素多的网页尤其慢

- **将脚本置底**

HTTP/1.1官方文档建议浏览器每个主机名下并行下载的文件数不要超过两个，如果图片来自多个主机名，并行下载的数量就可以超过两个。如果脚本正在下载，浏览器就不开始任何其它下载任务，即使是在不同主机名下的。因为浏览器要在脚本下载之后依次解析和执行。<br />因此对于脚本提速，我们可以考虑以下方式，

- 把脚本置底，这样可以让网页渲染所需要的内容尽快加载显示给用户。
- 现在主流浏览器都支持defer关键字，可以指定脚本在文档加载后执行。
- HTML5中新加了async关键字，可以让脚本异步执行。
<a name="Nx5p4"></a>
### 2、代码压缩

- **gzip压缩**

HTTP协议上的gzip编码是一种用来改进web应用程序性能的技术，web服务器和客户端（浏览器）必须共同支持gzip。目前主流的浏览器都支持该协议。<br />客户端http请求头Accept-Encoding声明浏览器支持的压缩方式，服务端配置启用压缩，压缩的文件类型，压缩方式。当客户端请求到服务端的时候，服务器解析请求头，如果客户端支持gzip压缩，响应时对请求的资源进行处理并返回给客户端，浏览器按照自己的方式解析，在http响应头，我们可以看到**content-encoding:gzip**，这是指服务端使用了gzip的压缩方式，返回的是gzip文件。<br />​

**在 webpack 可以使用如下插件进行压缩：**

- JavaScript：UglifyPlugin
- CSS ：MiniCssExtractPlugin
- HTML：HtmlWebpackPlugin


<br />其实，我们还可以做得更好。那就是使用 gzip 压缩。可以通过向 HTTP 请求头中的 Accept-Encoding 头添加 gzip 标识来开启这一功能。当然，服务器也得支持这一功能。<br />gzip 是目前最流行和最有效的压缩方法。举个例子，我用 Vue 开发的项目构建后生成的 app.js 文件大小为 1.4MB，使用 gzip 压缩后只有 573KB，体积减少了将近 60%。<br />附上 webpack 和 node 配置 gzip 的使用方法。<br />​

**下载插件**
```javascript
npm install compression-webpack-plugin --save-dev
npm install compression
```
**webpack 配置**
```javascript
const CompressionPlugin = require('compression-webpack-plugin');

module.exports = {
  plugins: [new CompressionPlugin()],
}
```
**node 配置**
```javascript
const compression = require('compression')
// 在其他中间件前使用
app.use(compression())
```
<a name="zz5IT"></a>
### 3、事件处理

- **使用事件委托**

事件委托利用了事件冒泡，只指定一个事件处理程序，就可以管理某一类型的所有事件。所有用到按钮的事件（多数鼠标事件和键盘事件）都适合采用事件委托技术， 使用事件委托可以节省内存。
```javascript
<ul>
  <li>苹果</li>
  <li>香蕉</li>
  <li>凤梨</li>
</ul>

// good
document.querySelector('ul').onclick = (event) => {
  const target = event.target
  if (target.nodeName === 'LI') {
    console.log(target.innerHTML)
  }
}

// bad
document.querySelectorAll('li').forEach((e) => {
  e.onclick = function() {
    console.log(this.innerHTML)
  }
}) 
```

- **事件节流和防抖**

频繁触发相关事件会引发大量的计算，进而引发页面抖动或者卡顿<br />**事件节流：**<br />规定在一个单位时间内，只能触发一次函数。如果这个单位时间内触发多次函数，只有一次生效。
```javascript

  function throttle(fun, delay) {
        let last, deferTimer
        return function (args) {
            let that = this
            let _args = arguments
            let now = +new Date()
            if (last && now < last + delay) {
                clearTimeout(deferTimer)
                deferTimer = setTimeout(function () {
                    last = now
                    fun.apply(that, _args)
                }, delay)
            }else {
                last = now
                fun.apply(that,_args)
            }
        }
    }

    let throttleAjax = throttle(ajax, 1000)

    let inputc = document.getElementById('throttle')
    inputc.addEventListener('keyup', function(e) {
        throttleAjax(e.target.value)
    })
```
**事件防抖：**<br />在事件被触发n秒后再执行回调，如果在这n秒内又被触发，则重新计时。
```javascript
//模拟一段ajax请求
function ajax(content) {
  console.log('ajax request ' + content)
}

function debounce(fun, delay) {
    return function (args) {
        let that = this
        let _args = args
        clearTimeout(fun.id)
        fun.id = setTimeout(function () {
            fun.call(that, _args)
        }, delay)
    }
}
    
let inputb = document.getElementById('debounce')

let debounceAjax = debounce(ajax, 500)

inputb.addEventListener('keyup', function (e) {
        debounceAjax(e.target.value)
    })
```

<br />

