---
title: 20211207_Blob、File、Base64转换
date: 2021-12-06 18:33:47
banner_img: /images/posts/20211207_BlobFileBase64/banner.png
index_img: '/images/posts/20211207_BlobFileBase64/banner.png'
tags:
  - javascript
categories: 查缺补漏
---

-----

文件上传、图片压缩、文件下载、大文件断点续传，等等关于 js 来操作文件的需求。那么你真的了解文件类型之间的转换关系吗？如下：

- `Blob` --> `File`
- `File` --> `DataURL（base64）`
- `File` --> `BlobURL`
- `HTTPURL| DataURL | BlobURL` --> `Blob`

## 获取Blob对象

> `Blob` 类型是 `File` 文件类型的父类，它表示一个不可变、原始数据的类文件对象.

### 1. new Blob(array, option)

``` js
const blob = new Blob(['<h1>head1</h2>', {type: 'text/html'}]);
```

如上代码，就创建了一个 `blob` 对象，并声明了 `text/html` 类型 ，就像是创建一个 .html 文件。只不过它存在于浏览器的内存里。

### 2. `fetch(url)`

`fetch(blobURL|dataURL)`

- blobURL: 比如通过 `URL.createObjectURL()` 获得
  
  ``` js
  // blobURL 示例：
  blob:null/7025638d-c05f-4c75-87d6-470a427e9aa3
  ```

- dataURL: 如图片的 base64 格式，比如通过 `convasElement.toDataURL()` 获得
  
  ``` js
  // dataURL(base64) 黑色 1 像素示例：
  data:image/gif;base64,R0lGODlhAQABAIAAAAUEBAAAACwAAAAAAQABAAACAkQBADs=
  ```

`fetch(url, options)` 响应数据可被解析成：

- `res.arrayBuffer()`: `ArrayBuffer` 通用、固定长度的原始二进制数据缓冲区
- `res.blob()`: `Blob` 类型
- `res.formData()`: `FormData` 表单数据类型
- `res.json()`: `JSON` 格式
- `res.text()`: 文本格式
  
``` js
// 获取图片blob
// 通过 http、https 获取
fetch('http://XX')
.then(res => res.blob())
.then(blob => console.log(blob))
```

### 3. `canvasElement.toBlob(callback)`

> `canvas` 具有图像操作能力，支持将一个已有的图片作为图片源，来操作图像。

如下，通过 canvas 将图片资源转成 blob 对象

``` html
<body>
  <canvas width="100" height="100"></canvas>
</body>

<script>
  const $ = arg => document.querySelector(arg)
  let canvas = $('canvas')
  // async 自执行函数
  (async () => {
    let imgUrl = 'http://eg.com/to/path/someImg.png'
    let ctx = canvas.getContext('2d')
    let img = await fetchImg(imgUrl)
    // 向 canvas 画布上下文绘制图片
    ctx.drawImage(img, 0, 0)

    // 获取图片 blob 对象
    canvas.toBlob((blob) => {
      console.log('blob: ', blob)
    })

    // 获取图片 dataURL，也是 base64 格式
    let dataURL = canvas.toDataURL()
    console.log('dataURL: ', dataURL)
  })()

  // 获取图片资源，封装成 promise
  function fetchImg (url) {
    return new Promise((resolve, reject) => {
      let img = new Image()
      // 跨域图片处理
      img.crossOrigin = '*'
      img.src = url
      // 图片资源加载完成回调
      img.onload = () => {
        resolve(img)
      }
    })
  }
</script>
```

注：

- 如果图片没加载完，就调用 `drawImage`，`canvas` 绘制将失败，所以我们简单封装了 `fetchImg` 方法，确保图片资源加载完成后再开始绘制图片。
- 由于 `canvas` 中的图片可能来自一些第三方网站。在不做处理的情况下，使用跨域的图片绘制时会污染画布，这是出于安全考虑。在“被污染”的画布中调用 `toBlob()` `toDataURL()` `getImageData()` 会抛出安全警告。

  解决方法：

  ``` js
  let img = new Image()
  // 1. 增加 crossOrigin 属性，值为 anonymous
  // 含义：执行一个跨域请求，在请求头里加 origin 字段
  // 2. 后端要返回 Access-Control-Allow-Origin 响应头来允许跨域
  img.crossOrigin = 'anonymous'
  img.src = 'to/path'
  ```

  本质就是解决跨域问题，也可以使用 nginx 做个代理来解决

- `blob` 有 `slice(startIndex, endIndex)` 方法，复制 `blob` 对象某片段，与 js 数组的 `slice` 方法类似，文件的断点续传功能就是利用了改特性。

## 获取File对象

> File 包含文件的相关信息，可以通过 js 来访问其内容

### 1. `new File(bits, name[, options])`

``` js
// 1. 参数是字符串组成的数组
let hiFile = new File([`<h1>Hi gauseen!<h1>`], 'fileName', { type: 'text/html' })

// 2. blob 转 file 类型
let hiBlob = new Blob([`<h1>Hi gauseen!<h1>`], { type: 'text/html' })
let hiFile = new File([ hiBlob ], 'fileName', { type: 'text/html' })
```

如上代码，通过 `File` 构造函数，创建一个 `file` 对象，与上面的提到的 `blob` 类似。可以将 `blob` 转成 `file` 类型，这意味着上面获取的 `blob`，可以转成 `file` 类型。

### 2. `inputElement.files`

通过 `<input type="file">` 标签获取 `file` 对象

### 3. `DragEvent.dataTransfer.files`

通过拖、放获取 file 对象

``` html
<body>
  <div id="output">
     将文件拖放到这里~
  </div>
</body>

<script>
  const $ = arg => document.querySelector(arg)
  let outputEle = $('#output')
  // ondragover 事件规定在何处放置被拖动的数据
  outputEle.addEventListener('dragover', dragEvent => {
    dragEvent.preventDefault()
  })
  // ondrop 事件放置文件时触发
  outputEle.addEventListener('drop', dragEvent => {
    dragEvent.preventDefault()
    // DataEvent.dataTransfer 属性保存着拖拽操作中的数据
    let files = dragEvent.dataTransfer.files
    console.log('drag files: ', files)
  })
</script>
```

### 4. `canvasElement.mozGetAsFile()`

注： 截止 2020-01-13，该方法仅支持火狐浏览器

``` js
let file = canvasElement.mozGetAsFile('imgName')
```

## DataURL（base64）

DataURL，前缀为 data: 协议的 URL，可以存储一些小型数据

语法：

```js
data:[<mediatype>][;base64],<data>
```

如下

``` js
data:image/gif;base64,R0lGODlhAQABAIAAAAUEBAAAACwAAAAAAQABAAACAkQBADs=
```

上面提到的 Blob File 类型，如何“消费”它们呢？接着向下看

### 1. FileReader

允许 Web 应用程序异步读取存储在用户计算机上的文件（`blob` 或 `file`）。

``` js
// 将 blob 或 file 转成 DataURL（base64） 形式
fileReader(someFile).then(base64 => {
  console.log('base64: ', base64)
})

function fileReader (blob) {
  return new Promise((resolve, reject) => {
    const reader = new FileReader()
    reader.onload = (e) => {
      resolve(e.target.result)
    }
    reader.readAsDataURL(blob)
  })
}
```

### 2. `canvasElement.toDataURL()`

## BlobURL(ObjectURL)

> `BlobURL` 也叫 `ObjectURL`，它可以让**只支持 URL 协议的 Api**（如：`<a> <link> <img> <script>`） 访问 `file` 或 `blob` 对象。

> 需要注意的是，即使是同样的二进制数据，每调用一次`URL.createObjectURL`方法，就会得到一个不一样的URL。

> 这个URL的存在时间，等同于网页的存在时间，一旦网页刷新或卸载，这个URL就失效。除此之外，也可以手动调用`URL.revokeObjectURL`方法，使URL失效。

如下，生成 `blobURL`，`createObjectURL` 方法创建从 `URL` 到 `Blob` 的映射关系。
如：`'blob:http://eg.com/550e8400-e29b-41d4-a716-446655440000'`

``` js
// object 创建 URL 的 File 对象、Blob 对象或者 MediaSource 对象
let blobURL = URL.createObjectURL(object)
```

如下，`revokeObjectURL` 方法撤消 `blobURL` 与 `Blob` 的映射关系，有助于浏览器垃圾回收，提示性能。

``` js
URL.revokeObjectURL(blobURL)
```

## 形成闭环

``` bash
blob --> file --> dataURL(base64) | blobURL --> blob
```

- blob => dataURL(base64)
  
  ``` js
  // use FileReader
  function blob2base64(blob) {
    return new Promise(function(resolve, reject) {
      const reader = new FileReader();
      reader.onload = function(e) {
        resolve(e.target.result);
      }
      reader.readAsDataURL(blob);
    })
  }

  // use canvasElement

  ```

- blob => blobUrl(objectUrl)

  ``` js
  URL.createObjectURL(blob);
  ```

- dataURL|blobURL => blob
  
  ``` js
  // 利用fetch
  fetch(url)
  .then(res => res.blob())
  .then(blob => {
    console.log('blob: ', blob)
  })

  // 基本
  function dataURLtoBlob(dataurl) {
    const arr = dataurl.split(','), 
    type = arr[0].match(/:(.*?);/)[1],
    bstr = atob(arr[1]),
    n = bstr.length,
    u8arr = new Uint8Array(n);
    while (n--) {
      u8arr[n] = bstr.charCodeAt(n);
    }
    return new Blob([u8arr], { type });
  }
  ```

### canvas

1. 创建canvas

    ``` js
    const canvas = document.createElementByTag('canvas');
    ```

2. 创建img
  
    ``` js
    const img = new Image();
    ```

3. 加载img并绘制canvas

    ``` js
    const context = canvas.getContext('2d');
    img.onload = function() {
      ctx.drawImage(img, 0, 0); // 绘制图片
    }
    ```

4. 生成 base64/blob

    ``` js
    canvas.toDataURL();
    canvas.toBlob(blob => {
      ...
    })
    ```

demo:

``` html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  <canvas id="canvas" width="200px" height="200px"></canvas>
  <img id="img" width="200px" height="200px" />
  <input type="file" onchange="upload(this)" />
  <script>
    const canvas = document.getElementById("canvas");
    const img = document.getElementById("img");
    function upload(e) {
      [file] = e.files;
      const blobUrl = URL.createObjectURL(file); // blobUrl
      img.setAttribute('src', blobUrl);
      img.onload = function() {
        var ctx = canvas.getContext('2d');
        ctx.drawImage(img, 0, 0); // 绘图
        console.log(canvas.toDataURL()) // base64
        canvas.toBlob(blob => { // 转blob
          console.log(blob)
        })
      }
    }
  </script>
</body>
</html>
```
