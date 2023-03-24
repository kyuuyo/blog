---
layout:
  - post
title: 实现简单的富文本功能
categories: h5
date: 2021-12-17 16:07:51
tags:
---
#### 参考WEB API以及部分WangEditor源码逻辑
布局如下
![](/images/h5/simple-editor-demo/001.png)
```
<div class="system-set-page">
  <!-- 操作按钮 -->
  <ul class="btn-list">
    <li>
      <button @click="changeWeight">加粗</button>
    </li>
    <li>
      <button @click="changeColor">颜色</button>
    </li>
    <li>
      <button @click="changeSize">字号</button>
    </li>
  </ul>
  <!-- 编辑区域 -->
  <div class="eidtor-cotainer" @mouseup="mouseup" contentEditable="true" v-html="contentText"></div>
</div>
```
- 全局属性 contenteditable  是一个枚举属性，表示元素是否可被用户编辑。 如果可以，浏览器会修改元素的部件以允许编辑。
---
- <div class="eidtor-cotainer" contentEditable="true">
    <p>
      可编辑内容
    </p>
    <p>
      可编辑内容
    </p>
  </div>
---
### 一、要做到内容编辑，首先要保存选中的文本、段落或其他必要数据。
简单说就是保存被拖蓝的数据
![](/images/h5/simple-editor-demo/002.png)
但是选中后，是不可以发生点击事件的，不然就会取消拖蓝。
所以需要在选中结束后，直接保存被选中的数据，目前找到最好的方式是 mouseup 事件
```
methods: {
  mouseup() {
    // 鼠标抬起，获取当前选中范围
    this.getSelectRange()
  },
  getSelectRange() {
    let selection = window.getSelection()
    <!-- 方法一 -->
    // 选中开始节点
    this.startNode = selection.anchorNode
    // 选中结束节点
    this.endNode = selection.focusNode
    // 选中的 开始节点内的 起点的偏移量数字
    this.startOffset = selection.anchorOffset
    // 选中的 结束节点内的 终点的偏移量数字
    this.endOffset = selection.focusOffset

    <!-- 方法二 -->
    this.rangeAt = selection.getRangeAt(0)
  }
}
```
- window.getSelection()
返回一个 Selection 对象，表示用户选择的文本范围或光标的当前位置。
包含选中节点、位置等必要参数
selection.toString()可将选中范围内数据转为字符串
————————————————————————————————————————
目前有两种方式获取到具体的选中范围，各有优劣，需根据不同情况使用

- ***方法一***
```
selection.anchorNode      返回选区开始位置所属的节点
selection.focusNode       返回所选内容的结束位置部分所属的节点
selection.anchorOffset    返回选区的锚节点（ Selection.anchorNode）起点偏移量的数字
selection.focusOffset     返回选区终点（鼠标松开瞬间所记录的那个点）在焦点（Selection.focusNode）中的偏移量
```
根据这些数据，就可以获取到选中的范围
- ***方法二***
```
selection.getRangeAt(index)    将返回 range 对象, index 为多个选中的下标
```
在部分浏览器中，按住Ctrl可以选中不连续的多个文本、段落，所以需要根据下标获取指定数据
**大多数情况下，可直接使用方法二，简单便捷。但是如果涉及到特定操作，只能根据选中节点、位置来获取数据**

————————————————————————————————————————
### 二、点击相关按钮，选中被保存的范围
此时触发了点击事件，拖蓝取消，所以需要重新选中点击前 被选中的范围
```
  <!-- 重新设置选中范围前，必须首先取消所有已选中状态 -->
  window.getSelection().removeAllRanges()
```
根据上一步不同的保存方式，同样有两种方法

***方法一***
```
const newRange = document.createRange()
if (this.startOffset < this.endOffset) {
  // 设置开始位置（开始节点，起点的偏移量）
  newRange.setStart(this.startNode, this.startOffset)
  // 设置结束位置（结束节点，终点的偏移量）
  newRange.setEnd(this.endNode, this.endOffset)
} else {
  // 设置开始位置（开始节点，起点的偏移量）
  newRange.setStart(this.endNode, this.endOffset)
  // 设置结束位置（结束节点，终点的偏移量）
  newRange.setEnd(this.startNode, this.startOffset)
}
```
- document.createRange()
设置一个range对象，并设置其开始结束范围
**需注意，因实际拖蓝操作可能是 从左到右（与文档方向相同）选择文本或从右到左（与文档方向相反）选择文本，所以需要注意开始结束的节点、偏移量变化。以上仅为最简单的初步判断，无法应对复杂环境。**

***方法二***
```
window.getSelection().addRange(this.rangeAt)
```
- addRange()
将一个区域（Range）对象 增加到选区（Selection）当中。

此时，保存的选中范围将会被重新选中。

### 三、改变数据
使用 execCommand 方法，可操纵大部分可编辑元素

```
document.execCommand(aCommandName, aShowDefaultUI, aValueArgument)
aCommandName     一个 DOMString ，命令的名称。可用命令列表请参阅 命令 。
aShowDefaultUI   一个 Boolean， 是否展示用户界面，一般为 false。Mozilla 没有实现。
aValueArgument   一些命令（例如insertImage）需要额外的参数（insertImage需要提供插入image的url），默认为null。 
```
以下仅简单举例
- 文本加粗
```
  document.execCommand('bold', false, null)
  <!-- 开启或关闭选中文字或插入点的粗体字效果。IE浏览器使用 <strong>标签，而不是<b>标签。 -->
```
- 改变颜色
```
  document.execCommand('foreColor', false, 'red')
  <!-- 在插入点或者选中文字部分修改字体颜色. 需要提供一个颜色值字符串作为参数。 -->
```
- 改变字号
```
  document.execCommand('fontSize', false, '5')
  <!-- 在插入点或者选中文字部分修改字体大小. 需要提供一个HTML字体尺寸 (1-7) 作为参数。 -->
```

### 四、重新保存选中范围
**数据在被修改后，因内容改变，必须重新保存选中范围
否则会导致重新设置数据失败**



