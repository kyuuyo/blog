---
title: DOM 渲染过程
categories: h5
date: 2021-12-03 10:22:00
---

**1. 浏览器接收到 HTML 文件并转换为 DOM 树**
> 当我们打开一个网页时，浏览器都会去请求对应的 `HTML` 文件。虽然平时我们写代码时都会分为 `JS`、`CSS`、`HTML` 文件，也就是字符串，但是计算机硬件是不理解这些字符串的，所以在网络中传输的内容其实都是 `0` 和 `1` 这些字节数据。当浏览器接收到这些字节数据以后，它会将这些字节数据转换为字符串，也就是我们写的代码。

![](/images/h5/domRender/desc_1.png)
> 当数据转换为字符串以后，浏览器会先将这些字符串通过**词法分析**转换为标记（`token`），这一过程在词法分析中叫做标记化（`tokenization`）

![](/images/h5/domRender/desc_2.png)
> 那么什么是标记呢？这其实属于编译原理这一块的内容了。简单来说，标记还是字符串，是构成代码的最小单位。这一过程会将代码分拆成一块块，并给这些内容打上标记，便于理解这些最小单位的代码是什么意思

![](/images/h5/domRender/desc_3.png)
> 当结束标记化后，这些标记会紧接着转换为 `Node`，最后这些 `Node` 会根据不同 `Node` 之前的联系构建为一颗 `DOM` 树

![](/images/h5/domRender/desc_4.png)
> 以上就是浏览器从网络中接收到 `HTML` 文件然后一系列的转换过程

![](/images/h5/domRender/desc_5.png)
> 当然，在解析 `HTML` 文件的时候，浏览器还会遇到 `CSS` 和 `JS` 文件，这时候浏览器也会去下载并解析这些文件，接下来就让我们先来学习浏览器如何解析 `CSS` 文件

**2. 将 CSS 文件转换为 CSSOM 树**
> 其实转换 `CSS` 到 `CSSOM` 树的过程和上一小节的过程是极其类似的

![](/images/h5/domRender/desc_6.png)

- 在这一过程中，浏览器会确定下每一个节点的样式到底是什么，并且这一过程其实是很消耗资源的。因为样式你可以自行设置给某个节点，也可以通过继承获得。在这一过程中，浏览器得递归 `CSSOM` 树，然后确定具体的元素到底是什么样式。

如果你有点不理解为什么会消耗资源的话，我这里举个例子
```
<div>
  <a> <span></span> </a>
</div>
<style>
  span {
    color: red;
  }
  div > a > span {
    color: red;
  }
</style>
```
> 对于第一种设置样式的方式来说，浏览器只需要找到页面中所有的 `span` 标签然后设置颜色，但是对于第二种设置样式的方式来说，浏览器首先需要找到所有的 `span` 标签，然后找到 `span` 标签上的 `a` 标签，最后再去找到 `div` 标签，然后给符合这种条件的 `span` 标签设置颜色，这样的递归过程就很复杂。所以我们应该尽可能的避免写过于具体的 `CSS` 选择器，然后对于 `HTML` 来说也尽量少的添加无意义标签，保证层级扁平

**3. 生成渲染树**
> 当我们生成 `DOM` 树和 `CSSOM` 树以后，就需要将这两棵树组合为渲染树

![](/images/h5/domRender/desc_7.png)

- 在这一过程中，不是简单的将两者合并就行了。渲染树只会包括需要显示的节点和这些节点的样式信息，如果某个节点是 `display: none` 的，那么就不会在渲染树中显示。
- 当浏览器生成渲染树以后，就会根据渲染树来进行布局（也可以叫做回流），然后调用 `GPU`绘制，合成图层，显示在屏幕上。对于这一部分的内容因为过于底层，还涉及到了硬件相关的知识，这里就不再继续展开内容了。



[**html字符串生成Nodes的过程就是个 AST 过程，点击查看 AST 知识**](https://www.yuque.com/qingtian2633/xe5co0/vbsryh)


# 文章2


## 1，前言
常考的一个题，就是从输入URL到显示页面的过程是什么。
浏览器渲染就是其中的一个环节，就是当文件资源返回给浏览器之后，浏览器是怎么渲染出页面的。
## 2，浏览器主要组成与浏览器线程
### 浏览器的组成

- **页面控件** - 包括地址栏、前进后退按钮、书签菜单等窗口上除了网页显示区域以外的部分
- **浏览器引擎** - 查询与操作渲染引擎的接口
- **渲染引擎** - 浏览器内核 ，负责显示请求的内容。比如请求到HTML, 它会负责解析HTML、CSS并将结果显示到窗口中
- **网络** - 用于网络请求, 如HTTP请求。它包括平台无关的接口和各平台独立的实现
- **UI后端** - 绘制基础元件，如组合框与窗口。它提供平台无关的接口，内部使用操作系统的相应实现
- **JS解释器** - 用于解析执行JavaScript代码
- **数据存储持久层** - 浏览器需要把所有数据存到硬盘上，如cookies。新的HTML5规范规定了一个完整（虽然轻量级）的浏览器中的数据库`web database`



`注意：线程和进程其他文章介绍了，可以理解为打开一个软件就是一个进程，多个线程辅助这一个进程。`
要注意的是，打开浏览器软件是一个进程，然后每打开一个Tab网页，也开启了一个进程！
### 浏览器的进程与线程
这里只研究一个Tab页，一个浏览器进程，所需要协助的各种线程

- GUI 渲染线程
- JS 引擎线程
- 定时触发器线程
- 事件处理线程
- 异步 http 请求线程
#### **GUI渲染线程**
GUI-图形用户界面，那GUI渲染线程也就是负责渲染浏览器界面上显示的内容。
即负责解析HTML标签和CSS样式。


#### **事件处理线程**
点击事件等


#### 定时触发器线程
setTimeout等
**​**

#### **异步HTTP请求线程**
异步HTTP请求线程主要用于处理**XMLHttpRequest**请求。
当我们的程序需要向服务器发起请求时，就会交给该线程处理。
 当请求得到响应后，如果有需要执行的回调函数，会将回调放入JS的任务队列，后续在由JS引擎线程处理。


#### 简单流程
![](/images/h5/domRender/desc_8.png)




## 3，渲染过程


HTTP 请求回来的文件是一堆字符串，如下
![](/images/h5/domRender/desc_9.png)


**大概的流程：**

1. 浏览器将获取的HTML文档解析成DOM树
1. 处理CSS标记，构成层叠样式表模型CSSOM树(CSS Object Model)
1. 将DOM和CSSOM合并为渲染树(Render树)将会被创建，代表一系列将被渲染的对象
1. 渲染树的每个元素包含的内容都是计算过的，它被称之为布局layout。浏览器使用一种流式处理的方法，只需要一次pass绘制操作就可以布局所有的元素
1. 将渲染树的各个节点绘制到屏幕上，这一步被称为绘制painting



图形说明：

![](/images/h5/domRender/desc_10.png)


需要注意的是，以上五个步骤并不一定一次性顺序完成，比如DOM或CSSOM被修改时，亦或是哪个过程会重复执行，这样才能计算出哪些像素需要在屏幕上进行重新渲染。而在实际情况中，JavaScript和CSS的某些操作往往会多次修改DOM或者CSSOM。

![](/images/h5/domRender/desc_11.png)

### 具体流程
### 1，构建DOM树
假设先不考虑链接的 js 和 css，只单纯的 html ，DOM 树的构建：
（1）浏览器接收到后端返回的HTML数据，传输过程中是二进制字节流，浏览器先将**字节**转为**html字符串 , **如 "<html><head></head></html>"
（2）解析html字符串，转化为 Tokens 数组，类似AST那个文章，Tokens 数组类似这样 

| [{tagName: 'html', type: 'DOCTYPE'}, {tagName: 'head', type: 'startTag'}, {tagName: 'head', type: 'endTag'}, {tagName: 'html', type: 'endTag'}] |
| --- |

怎么将字符串转为数组的，可以参考 AST 文章，词法分析之转换tokens的简单代码实现（当然那里是通过空格进行分割的，这里可以通过<>分割，或者类似babel那种通过不断移动字符串光标指针判断什么情况下结束一个token項的读取）
（3）将上面的 tokens 数组转为 Nodes，每个 Node 都添加特定的属性（或属性访问器），通过指针能够确定 Node 的父、子、兄弟关系和所属 treeScope（例如：iframe 的 treeScope 与外层页面的 treeScope 不同）
![](/images/h5/domRender/desc_12.png)

（4）**构建 DOM 树（Nodes -> DOM Tree）**—— 最重要的工作是建立起每个结点的父子兄弟关系


总体：
![](/images/h5/domRender/desc_13.png)

注意：
① DOM树在构建的过程中可能会被CSS和JS的加载而执行阻塞（后面有讲）。
② display:none 的元素也会在DOM树中。
③ 注释也会在DOM树中
④ script标签会在DOM树中
### 2，CSSOM 树的构建
与 DOM 树的构建过程相似，CSSOM 的构建也要经历以下过程：
![](/images/h5/domRender/desc_14.png)

最终构建的 CSSOM 树大致如下：
![](/images/h5/domRender/desc_15.png)
​

浏览器会解析CSS文件并生成CSS规则树，在过程中，每个CSS文件都会被分析成StyleSheet对象，每个对象都包括CSS规则，CSS规则对象包括对应的选择器和声明对象以及其他对象。
CSS解释器会**将复合的CSS规则拆分成多个声明对象**（可以参考下图中border的解析）
CSS规则树的构建和DOM树的构建是同步进行的

### 3，渲染树的构建  Render树

 

1. DOM 树与 CSSOM 树融合成渲染树
1. 渲染树只包括渲染页面需要的节点




![](/images/h5/domRender/desc_16.png)

通过DOM树和CSS规则树，浏览器就可以通过这两个构建渲染树了。浏览器会先从DOM树的根节点开始遍历每个可见节点，然后对每个可见节点找到适配的CSS样式规则并应用。具体的规则有以下几点需要
注意：

1. Render Tree和DOM Tree不完全对应 （可以看到上图中，Render Tree 中是不包含head、link等这些不可见的元素的）
1. display: none的元素不在Render Tree中（如上图中的 span）
1. visibility: hidden的元素在Render Tree中

总结：
Render Tree 是基于DOM树和CSS规则树构建的，是文档的**可视化**表示。
它的作用是为了后续能将元素进行正确的布局和绘制。

1. 一些非可视化的元素不会插入到呈现树中，比如**head**元素、**display为none**的元素。
1. 呈现树构建时会计算每一个可视化元素的样式属性。
1. 样式属性计算就是为每一个元素查找匹配的CSS规则，这个就是根据我们写的CSS选择器进行匹配的。



另外，renderer与DOM元素的位置也可能是不一样的。那些添加了`float`或者`position:absolute`的元素，因为它们脱离了正常的文档流，构造Render树的时候会针对它们实际的位置进行构造。
### 4，布局**（Layout）**
上面确定了renderer的样式规则后，然后就是重要的显示元素布局了。当renderer构造出来并添加到Render树上之后，它并没有**位置跟大小信息**，为它确定这些信息的过程，接下来是布局(layout)。
​

浏览器进行页面布局基本过程是以浏览器可见区域为画布，**左上角为**`**(0,0)**`**基础坐标，从左到右，从上到下从DOM的根节点开始画**，首先确定显示元素的大小跟位置，此过程是通过浏览器计算出来的，用户CSS中定义的量未必就是浏览器实际采用的量。如果显示元素有子元素得先去确定子元素的显示信息。
​

布局阶段输出的结果称为box盒模型（width,height,margin,padding,border,left,top,…），盒模型精确表示了每一个元素的位置和大小，并且所有相对度量单位此时都转化为了绝对单位。
​

即计算元素的位置和大小
​

### 5，**绘制（Paint）**
在绘制(painting)阶段，渲染引擎会遍历Render树，并调用renderer的 paint() 方法，将renderer的内容显示在屏幕上。绘制工作是使用UI后端组件完成的。
​

## 4，阻塞
上面的流程构建，DOM树和CSS树，都是基于单独去分析，下面将整体看一下相互间的作用
首先看一下这个文章
文章的结论：


- css的加载**不会**阻塞DOM的**解析**
- css的加载**会**阻塞DOM的**渲染**
- JS的**加载和执行**会阻塞DOM的**解析**
- JS的**加载和执行**会阻塞DOM的**渲染**
- CSS的加载阻塞JS的运行，不阻塞JS的加载
- CSS的加载与JS的加载之间是否有影响？不考虑，浏览器自身会偷看并预先下载
- 遇到script标签会触发渲染，以便获得最新的样式给JS代码

注意： 有些文章可能会提到，css的加载会阻塞DOM的解析，那是因为，如果不存在js的话，不阻塞；如果存在js的话，css的加载阻塞js的运行，js的运行阻塞了DOM的解析，所以css间接的影响了DOM（所以通常会把css放在头部，js放在body尾）


加载：文件下载
解析：生成树（DOM、CSS）
渲染：Render 树及以后
## 5，最后的步骤
当HTML文档解析过程完毕后，
（1）浏览器继续进行<script>为defer模式的脚本加载，并且按照顺序运行
（2）defer都运行完了，触发 DOMContentLoaded 事件
（3）运行 <script> 为 async 的代码，都运行完了后，触发 load事件（load事件是全部的资源加载完成，包括图片之类的）。
​

async、defer与DOMContentLoaded的执行先后关系、load事件执行时期


## 6，回流和重绘
HTML默认是流式布局的，但CSS和JS会打破这种布局，改变DOM的外观样式以及大小和位置。因此我们就需要知道两个概念:
　　① reflow（回流）：当浏览器发现某个部分发生了变化从而影响了布局，这个时候就需要**倒回去重新渲染**，大家称这个回退的过程叫 reflow。
　　常见的reflow是一些会影响页面布局的操作，诸如Tab，隐藏等。reflow 会从 html 这个 root frame 开始递归往下，依次计算所有的结点几何尺寸和位置，以确认是渲染树的一部分发生变化还是整个渲染树。reflow几乎是无法避免的，因为只要用户进行交互操作，就势必会发生页面的一部分的重新渲染，且通常我们也无法预估浏览器到底会reflow哪一部分的代码，因为他们会相互影响。
　　② repaint（重绘）： repaint则是当我们改变某个元素的背景色、文字颜色、边框颜色等等不影响它周围或内部布局的属性时，屏幕的一部分要重画，但是元素的几何尺寸和位置没有发生改变。
需要注意的是，display:none 会触发 reflow，而visibility: hidden属性则并不算是不可见属性，它的语义是隐藏元素，但元素仍然占据着布局空间，它会被渲染成一个空框，这在我们上面有提到过。所以 visibility:hidden 只会触发 repaint，因为没有发生位置变化。


我们不能避免reflow，但还是能通过一些操作来减少回流：
　　① 用transform做形变和位移
　　② 通过绝对位移来脱离当前层叠上下文，形成新的Render Layer
有些情况下，比如修改了元素的样式，浏览器并不会立刻reflow 或 repaint 一次，而是会把这样的操作积攒一批，然后做一次 reflow，这又叫异步 reflow 或增量异步 reflow。但是在有些情况下，比如 resize 窗口，改变了页面默认的字体等。对于这些操作，浏览器会马上进行 reflow。


注：回流必将引起重绘，而重绘不一定会引起回流。


### 何时发生重绘和回流


- 页面第一次加载的时候
- 改变字体，改变元素尺寸（如果只改变字体颜色，元素背景颜色那么只触发重绘）
- 改变元素里面内容的时候（比如在input里面输入文字）
- 添加/删除可见DOM元素（注意：如果元素本身就display:none的元素不会发生重排；visibility:hidden的元素显示或隐藏不影响重排）
- fiexd定位的元素，在拖动滚动条的时候会一直发生回流。
- 调整窗口大小的时候
- 计算offertWidth 和 offsetHeight属性的时候
### 如何减少重绘和回流


- 使用tranform代替top
- 使用visibility 代替 display:none（前者只会重绘，后者会回流）
- 避免使用table布局（可能一个很小的改动会造成整个table的重新布局）
- 尽可能在dom树末端改变class（回流是不可避免的，但可以减少其影响。尽可能在DOM树末端改变class，可以限制回流的范围，使其影响尽可能少的节点）
- 避免设置多层内联样式（比如 div > span > a，因为浏览器要去递归寻找，比较复杂。）
- 将动画应用到position属性为absolute或者fixed的元素上（避免影响其他元素布局，这样只是重绘，而不是回流，同时控制动画可以选择requestAnimationFrame）
- 避免使用CSS表达式
- 避免频繁的操作dom（你可以js创建一个元素，完成所有dom操作在添加到页面当中）
- 避免频繁的读取会引发回流/重绘的属性（如果要多次使用可以保存起来 const width = element.offertWidth）

（1）cssText 多个属性
原来

| const el = document.getElementById('test'); el.style.padding = '5px'; el.style.borderLeft = '1px'; el.style.borderRight = '2px'; |
| --- |

例子中，有三个样式属性被修改了，每一个都会影响元素的几何结构，引起回流。当然，大部分现代浏览器都对其做了优化，因此，只会触发一次重排。但是如果在旧版的浏览器或者在上面代码执行的时候，有其他代码访问了布局信息(上文中的会触发回流的布局信息)，那么就会导致三次重排。
因此，我们可以合并所有的改变然后依次处理，比如我们可以采取以下的方式：

| 使用cssText const el = document.getElementById('test'); el.style.cssText += 'border-left: 1px; border-right: 2px; padding: 5px;'; 修改CSS的class const el = document.getElementById('test'); el.className += ' active'; |
| --- |

（2）批量修改DOM
先隐藏元素，然后对元素进行一系列的操作，最后再显示元素

| var list = document.getElementById("list"); list.style.display = 'none’; appendDataToElement(list,data); list.style.display = 'block'; |
| --- |



（3）利用 class 替换样式
不要单独一个属性一个属性的修改，直接替换一个class


（4）cloneNode （将原始元素拷贝到一个脱离文档的节点中,修改副本,完成后再替换回去； ）

| var old = document.getElementById("list"); var clone = old.cloneNode(true); appendDataToElement(clone,data); old.parentNode.replaceChild(clone,old); |
| --- |



（5）document.createDocumentFragment()；（使用文档片段（document fragment）在当前DOM之外构建一个子树，再插回去）
不仅仅是一个节点了，直接创建一个文档片段


（6）使用trsansform
CSS的最终表现分为以下四步：Recalculate Style -> Layout -> Paint Setup and Paint -> Composite Layers
按照中文的意思大致是 查找并计算样式 -> 排布 -> 绘制 -> 组合层。
由于transform是位于Composite Layers层，而width、left、margin等则是位于Layout层，在Layout层发生的改变必定导致Paint Setup and Paint -> Composite Layers，
所以相对而言使用transform实现的动画效果肯定比使用改变位置(margin-left等)这些更加流畅。


## 7，优化渲染性能
太多了，看这个文章的吧


## 8， 什么是GPU加速，如何使用GPU加速，GPU加速的缺点

- **优点**：使用transform、opacity、filters等属性时，会直接在GPU中完成处理，这些属性的变化不会引起回流重绘
- **缺点**：GPU渲染字体会导致字体模糊，过多的GPU处理会导致内存问题


![](/images/h5/domRender/desc_17.png)
