---
title: 肯定有一款适合你的下载方案
date: 2019-05-11 08:30:00
tags:
- JavaScript
- Blob
---

当你看到文章的标题后以为在文章里找到了可以直接使用的下载方（代）案（码），读完文章可能会使你失望，但是本文的价值完成胜于找到“插入即用”的下载方案。让你从根源上了解各种下载姿势的原理，让我们一起来探索吧！


起初在网页应用中下载文件，会向服务器发送一个 HTTP 请求，服务器返回文件内容。例如图片文件，我们知道在浏览页面时，如果页面中发送图片文件的请求，服务器将图片文件返回给客户端，然后图片会展示在页面中。浏览器要怎么知道是展示文件还在下载文件呢？


##### Content-Disposition: attachment

在常规的HTTP应答中，`Content-Disposition` 响应头指示了HTTP body的内容以什么形式展示。是以内联的形式（`Content-Disposition: inline`，即网页或者页面的一部分），还是以附件的形式（`Content-Disposition: attachment`）下载并保存到本地。Content-Disposition 头还可以带上一个 `filename` 参数，filename 的值将预填为下载后的文件名。这个方案需要后端开发人员的配合才可行。



##### download

HTML `<a>` 元素可以创建通向其他网页、文件或同一页面内的位置超链接，在 HTML5 中，添加了一个新的属性 —— download。即使资源的响应头不是 `Content-Disposition: attachment`， 此属性也能指示浏览器下载 URL 指向的资源而不是导航到它， 点击带有 download 属性的链接将提示用户保存为本地文件。在使用这个属性的时候，需要注意几点。

1. download 属性仅适用于同源 URL。

要使 download 属性能正常下载 URL 指向的资源，URL 必须是同源的。对于不同源的资源，如果浏览器能够直接打开，例如图片，音频，视频等等，点击链接会正常跳转导航，链接的资源展示在浏览器中。特殊情况，如果是浏览器不能直接打开资源，例如PPT， zip等等，文件就会被浏览器下载。

2. URL 可以是 blob: URL 和 data: URL

URL 也可以是 `blob: URL` 或者 `data: URL`， 有了这个特性，再结合 Blob API，用户就可以下载非同源资源（前提是这个资源允许跨域）和用户自己生产的数据了，这个在后面再做详细讨论。

3. 资源响应头包含 Content-Disposition

如果 HTTP 头中包含 `Content-Disposition:  attachment`，且属性赋予了一个不同于 download 属性的 `filename` ，那么 HTTP 头属性优先级更高。



如果这时你已经被 Content-Disposition 和 download 属性绕晕了头，到底什么时候浏览器会执行下载而不是显示。我在下面的表格中总结了 Content-Disposition ，download 以及是否跨域的不同情况下，浏览器是如何处理资源的，它可以帮助大家来捋清思路。



|                  | Content-Disposition:inline               | Content-Disposition: attachment                              |
| ---------------- | ---------------------------------------- | ------------------------------------------------------------ |
| 同域(download)   | 行为: 下载资源， 文件名: download 属性值 | 行为: 下载资源， 文件名: Content-Disposition 头的filename 属性值 |
| 同域(无download) | 行为: 浏览器打开资源                     | 行为: 下载资源， 文件名: Content-Disposition 头的filename 属性值 |
| 跨域(download)   | 行为: 浏览器打开资源                     | 行为: 下载资源， 文件名: Content-Disposition 头的filename 属性值 |
| 跨域(无download) | 行为: 浏览器打开资源                     | 行为: 下载资源， 文件名: Content-Disposition 头的filename 属性值 |



如果对于非同源的资源，但是又想把它们下载到本地，我们要怎么处理呢？得利于HTML5 和一些新的 Web API ，我们就可以实现这个想法了。其中的大概原理就是将我们想下载的资源的二进制数据转换为 blob: URL 或者 data: URL，然后利用添加 download 属性的 HTML `<a>` 元素，将 blob: URL 或者 data: URL 赋值给 href 属性，这样资源就被可以下载了，接下来我一起来看看。



##### 利用 Canvas API 下载图片

这个方案可以下载同源和允许跨域的非同源的图片

1. 创建一个 Image 对象并赋值 url
2. 图片下载完成后绘制一个隐藏的 canvas 画布
3. 通过 HTMLCanvasElement 的 `toDataURL` 返回获取图片展示的 data:URL
   或者通过 `toBlob` 方法获取图片展示的 blob:URL
4. 创建一个隐藏的 a 标签， 给 a 标签添加 download 属性，
5. 将这个 data:URL / blob:URL 赋值给 a 标签的 href 属性

```
function downloadImageByCanvas (imgsrc, fileName= 'default') {
  let image = new Image();
  image.setAttribute('crossOrigin', 'anonymous'); //允许跨域

  image.onload = function () {
    let canvas = document.createElement('canvas');
    let { width, height } = image;
    canvas.width = width;
    canvas.height = height;
    canvas.getContext('2d').drawImage(image, 0, 0, width, height);

    let _dataURL = canvas.toDataURL('image/png'); // 得到图片的base64编码数据

    let a = document.createElement('a');
    a.download = fileName;
    a.href = _dataURL;
    a.click();
    a = null;
  };
  image.src = imgsrc;
}
```

##### 利用 Blob API 下载资源

* 无请求下载

Blob API 使无请求下载成为可能，比如网页应用中有一个文本输入框，用户完成内容输入之后，需要提供一个快捷的下载功能，将用户输入的内容下载下来。下面我们来实现这个简易的下载功能：

1. 获取用户输入的内容;
2. 使用 Blob API 将输入的内容转换为 Blob 数据；
3. 创建一个隐藏的 a 标签， 给 a 标签添加 download 属性；
4. 使用 `URL.createObjectURL` 方法将 Blob 数据转化为 blob: URL；
5. 将这个blob: URL 赋值给 a 标签的 href 属性。

按照上面的流程就可以成功下载了, show you code !


```html
  <textarea id="textArea"></textarea>
  <button id="btn">下载</button>
```

```javascript
  let btnEle = document.querySelector('#btn');
  let txt = document.querySelector('#textArea');

  btnEle.addEventListener('click', function () {
    downloadTxt(txt.value);
  });

  function downloadTxt (txt, name = 'default') {
    var blob = new Blob([txt], { type: 'text/plain' }); // 指定被放入到blob中的数组内容的MIME类型
    createAnchor(blob);
  }
```

```javascript
  function createAnchor (blob) {
    let a = document.createElement('a');
    let ObjectUrl = URL.createObjectURL(blob);

    a.download = 'default';
    a.href = ObjectUrl;

    setTimeout(() => {
      URL.revokeObjectURL(ObjectUrl);
    }, 3000);

    a.click();
    a = null;
  }
```

* 下载远程资源

如果我们想下载一些非同源的资源,  对于那些浏览器不能直接打开的资源，我们可以直接使用添加download属性的 a 标签。如果是图片，视频等等，我们可以将资源数据先请求到本地，然后再配合Web API使用和添加download属性的 a 标签来实现下载，这种方法适合所有可以成功下载到本地的资源类型，下面我们一起来看看怎么实现：

1. 设置 ` responseType = blob`，通过 Ajax 请求目标资源；
2. 资源返回后，创建一个隐藏的 a 标签， 给 a 标签添加` download` 属性；
3. 使用` URL.createObjectURL` 方法将 Blob 数据转化为 blob: URL；
4. 赋值 blob:URL 给 a 标签的 href 属性。

```
function downloadByXHR (url) {
  var xhr = new XMLHttpRequest();
  xhr.open('GET', url);
  xhr.responseType = 'blob';
  xhr.addEventListener('load', function (evt) {
    createAnchor(xhr.response);
  });
  xhr.send();
}
```



其实不管是`利用 Canvas API 下载远程的图片`还是 `利用 Blob API 下载资源`， 最后都要使用
到 HTML `<a>` 元素的 download 的属性。如果想要使用`利用 Blob API 下载资源` 这个下载方案，可以使用一个具有兼容性下载工具 [FileSaver.js](https://github.com/eligrey/FileSaver.js)。它暴露了一个saveAs 方法，这个方法接收一个资源url或者 blob作为第一个参数。



需要注意几点：

1. 所有将远程资源请求到本地再做处理的方案中，对于非同源资源，需要允许跨域，

2. 即使用相同的对象作为参数，在每次调用 `createObjectURL` 方法时都会创建一个新的 URL 对象，
   当不再需要这些 URL 对象时，每个对象必须通过调用 `URL.revokeObjectURL` 方法来释放。如果不通过手动释放，
   要等到浏览器在卸载 document 的时候才会自动释放它们。
3. 如果下载的资源过大或者网络不好导致下载时间过长，不在页面中手动添加资源的下载状态，用户是无法感知到文件正在下载。我们可以通过监听 `progress` 事件来计算当前资源的下载进度

```
xhr.addEventListener('progress', function (evt) {
  let progress = ~~((evt.loaded / evt.total) * 100);
  console.log(progress);
});
```

4. 由于Blob的使用在浏览器中是有大小限制的，所以我们能够下载资源大小也是有限制的。在不同的浏览器中这个“大小”是不同的，详细内容可以参考[这里](https://stackoverflow.com/questions/28307789/is-there-any-limitation-on-javascript-max-blob-size)。



现在我们已经结束了探索之旅，一起来总结一下所有的方案吧。

* Content-Disposition: attachment
* `<a>` 标签的 download 属性
*  Canvas API 下载远程的图片
*  Blob API 下载资源

感谢各位看官的阅读，如果发现文中的任何问题，欢迎指正。



参考链接：

https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Disposition

https://developer.mozilla.org/en-US/docs/Web/HTML/Element/a

https://developer.mozilla.org/en-US/docs/Web/API/Blob/Blob

https://developer.mozilla.org/en-US/docs/Web/API/URL/createObjectURL





