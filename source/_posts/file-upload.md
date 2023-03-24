---
layout:
  - post
title: 文件上传
categories: h5
date: 2021-11-18 20:31:51
tags:
---
# 一. 获取文件
```
<input type="file" />
```

## 如需修改样式，可使用如下方式
```
<button>
  <label>
    点击上传
    <input type="file" style="display: none;" />
  </label>
</button>
```
- <button>
    <label>
      点击上传
      <input type="file" style="display: none;" />
    </label>
  </button>

# 二. 文件限制条件判断
判断文件大小，格式等是否符合条件
多种方式，可互相结合使用
### *1、使用input标签的属性进行条件限制*
- 是否支持多选
```
<!-- 支持多选 -->
<input type="file" multiple="true" />
<!-- 不支持多选 -->
<input type="file" multiple="false" />
```
- 文件类型限制
```
<!-- 仅限图片类型 -->
<input type="file" accept="image/*" />

<!-- 仅限视频类型 -->
<input type="file" accept="video/*" />

<!-- 仅限音频类型 -->
<input type="file" accept="audio/*" />
```

### *2、文件选中后，根据相关属性进行限制条件判断*

```
<button>
  <label>
    点击上传
    <input type="file" id="fileSelect" style="display: none;" />
  </label>
</button>

<script>
  $('#fileSelect').on('change', function(e) {
    <!-- 选中文件列表 -->
    let fileSelectEle = e.target
    const selectList = fileSelectEle.files
    for (let i = 0; i < selectList.length; i++) {
      <!-- 当前文件 -->
      const selectItem = selectList[i]
    }
  })
</script>
```

- 是否支持多选
```
if(selectList.length > n) {
  alert('最多选择n个文件')
}
```

- 文件类型限制
```
if(selectItem.type !== 'image/png') {
  alert('仅支持png图片类型')
}

if(selectItem.type !== 'video/mp4') {
  alert('仅支持mp4视频类型')
}
```

- 文件大小限制
```
if(selectItem.size > 1024 * 1024 * n) {
  alert('文件大小不能超过 n M')
}
```
## 如果文件条件不符合
```
if(selectItem.size > 1024 * 1024 * n) {
  fileSelectEle.value = ''
  alert('文件大小不能超过 n M')
  return
}
```

# 三. 符合限制条件后，根据后端接口所需数据类型进行文件数据转换
```
let itemData = selectItem
```
- base64
```
if (window.FileReader) {
  var reader = new FileReader()
  reader.readAsDataURL(fileItem)
  //监听文件读取结束后事件
  reader.onloadend = function (e) {
    itemData = e.target.result
  }
}
```

- blob
```
let parts = base64String.split(';base64,')
let contentType = parts[0].split(':')[1]
let raw = window.atob(parts[1])
let rawLength = raw.length
let uInt8Array = new Uint8Array(rawLength)
for (let i = 0; i < rawLength; ++i) {
  uInt8Array[i] = raw.charCodeAt(i)
}
itemData = new Blob([uInt8Array], { type: contentType })
```

# 四. 换取upload_id
## 使用统一接口，换取upload_id
详见文档：http://yapi.chinacici.com/project/588/interface/api/16768
```
let upload_id = res.data.upload_id
```

# 五. 使用upload_id上传文件
```
let formData = new FormData()
formData.append('content', itemData)
formData.append('upload_id', upload_id)
formData.append('is_private', 1=隐私 0=非隐私)
```
详见文档：http://yapi.chinacici.com/project/588/interface/api/cat_1760

# 六. 上传成功
## 使用成功返回，渲染页面预览
## 数据回显同理

```
<ul class="preview-list">
  <li class="preview-item">
    <img src="" />
    <p></p>
    <a href=""></a>
  </li>
</ul>

<script>
  let priviewList.push(res.data.file_url)
</script>
```

# 七. 删除已上传文件

```
  priviewList.splice(n, 1)
  alert('已删除第n个文件')
```