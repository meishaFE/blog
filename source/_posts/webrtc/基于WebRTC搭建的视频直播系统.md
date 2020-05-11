---
title: 基于WebRTC搭建的视频直播系统
date: 2020-03-24 13:00:00
tags:
- webrtc
- 直播
---

### WebRTC是什么？

`WebRTC`是无需依赖第三方插件或软件即可实现实时通讯的一种开源技术；它的出现使得浏览器之间的点对点通信变得可能。
​`WebRTC`可以通过一系列的信令，建立一个浏览器与浏览器之间（`peer-to-peer`)的信道，这个信道可以发送任何数据，而不用经过服务器。而且重要的是，`WebRTC`通过实现	`MediaStream`,通过浏览器调用设备的摄像头、话筒，使得浏览器之间可以实现音频，视频的传输。

### WebRTC的几个接口API介绍

- `MediaStream`:通过此api能够通过设备的摄像头、话筒获得视频、音频的同步流
- `RTCPeerConnection`:通过`RTCPeerConnection`使得`WebRTC`能够构建点对点稳定、高效的流传输组件
- `RTCDataChannel`：`RTCDataChannel`使得浏览器之间建立一个高吞吐量，低延时的信道，用于传输任意的数据

<!-- more -->

##### 代码块实现：

获取音视频本地流：

```javascript
const openMediaDevices = async (constraints) => {
  return await navigator.mediaDevices.getUserMedia(constraints);
}

try {
  const stream = openMediaDevices({'video':true,'audio':true});
  console.log('Got MediaStream:', stream);
} catch(error) {
  console.error('Error accessing media devices.', error);
}
```



获取硬件设备列表：

```javascript
async function getConnectedDevices(type) {
  const devices = await navigator.mediaDevices.enumerateDevices();
  return devices.filter(device => device.kind === type)
}

const videoCameras = getConnectedDevices('videoinput');
console.log('Cameras found:', videoCameras);
```



监听设备的变更：

```javascript
// 更新下拉列表数据
function updateCameraList(cameras) {
  const listElement = document.querySelector('select#availableCameras');
  listElement.innerHTML = '';
  cameras.map(camera => {
    const cameraOption = document.createElement('option');
    cameraOption.label = camera.label;
    cameraOption.value = camera.deviceId;
  }).forEach(cameraOption => listElement.add(cameraOption));
}

// 获取指定设备类型的数组
async function getConnectedDevices(type) {
  const devices = await navigator.mediaDevices.enumerateDevices();
  return devices.filter(device => device.kind === type)
}

// 初始化原始相机数组
const videoCameras = getConnectedDevices('videoinput');
updateCameraList(videoCameras);

// 监听设备的变化并更新列表
navigator.mediaDevices.addEventListener('devicechange', event => {
  const newCameraList = getConnectedDevices('video');
  updateCameraList(newCameraList);
});
```



`Constraints`约束对象设置，用于更好的用户体验：

```javascript
async function getConnectedDevices(type) {
  const devices = await navigator.mediaDevices.enumerateDevices();
  return devices.filter(device => device.kind === type)
}

// 设置指定设备相机的最小宽度和最小高度，并设置消音属性
async function openCamera(cameraId, minWidth, minHeight) {
  const constraints = {
    'audio': {'echoCancellation': true},
    'video': {
      'deviceId': cameraId,
      'width': {'min': minWidth},
      'height': {'min': minHeight}
    }
  }

  return await navigator.mediaDevices.getUserMedia(constraints);
}

const cameras = getConnectedDevices('videoinput');
if (cameras && cameras.length > 0) {
    // 打开第一个可使用的相机设备，设置分辨率为 1280x720
  const stream = openCamera(cameras[0].deviceId, 1280, 720);
}
```



本地播放：

```javascript
// 页面嵌入video标签并设置捕获流
async function playVideoFromCamera() {
  try {
    const constraints = {'video': true, 'audio': true};
    const stream = await navigator.mediaDevices.getUserMedia(constraints);
    const videoElement = document.querySelector('video#localVideo');
    videoElement.srcObject = stream;
  } catch(error) {
    console.error('Error opening video camera.', error);
  }
}
```



为了给用户更好的体验，页面video标签一般设置`autoplay playsinline`属性，并且设置`controls='false'`;

```html
 <video id="localVideo" autoplay playsinline controls="false"/>
```



------

对等连接：`WebRTC`虽然采用了`RTCPeerConnection`来在浏览器之间进行数据流传递，而且它们之间是点对点的，不需要经过服务器；但是仍然需要服务器来为我们传递信令，以此来建立这个通道；信令的协议可以是任意的，任由我们自己选择，但是这个信令我们将要交换的信息有以下几种：

1. session信息：初始化通信和报错
2. 网络配置：IP地址或者端口等等
3. 媒体适配：发送方和接收方的浏览器能够接受什么样的编码器和分辨率

以使用Google的STUN服务器`stun:stun.l.google.com:19302`为例，代码如下：

初始化异步通信通道`SignalingChannel`

```javascript
// 设置一个异步通信通道，它将在对等连接建立期间使用
const signalingChannel = new SignalingChannel(remoteClientId);
signalingChannel.addEventListener('message', message => {
  // 接收来自远程客户端的数据
});

// 发送一个异步消息给远程客户端
signalingChannel.send('Hello!');
```



创建`RTCPeerConnection`对象作为发送方，使用`createOffer`方法创建`RTCSessionDescription`对象，设置为本地描述后通过信令通道发送到接收方，并且监听信号通道，用以监听接收方接收到的信息

```javascript
async function makeCall() {
  const configuration = {'iceServers': [{'urls':'stun:stun.l.google.com:19302'}]}
  const peerConnection = new RTCPeerConnection(configuration);
  signalingChannel.addEventListener('message', async message => {
    if (message.answer) {
      const remoteDesc = new RTCSessionDescription(message.answer);
      await peerConnection.setRemoteDescription(remoteDesc);
    }
  });
  const offer = await peerConnection.createOffer();
  await peerConnection.setLocalDescription(offer);
  signalingChannel.send({'offer': offer});
}
```



接收方：创建`RTCPeerConnection`实例来等待收到的报价，完成后，将使用`setRemoteDescription`来设置收到的信息；然后通过`createAnswer`来给收到的信息创建一个回应，并将此回应来设置为本地的描述`setLocalDescription`,最后通过自己的信令服务器发送给呼叫方

```javascript
const peerConnection = new RTCPeerConnection(configuration);
signalingChannel.addEventListener('message', async message => {
  if (message.offer) {
    peerConnection.setRemoteDescription(new RTCSessionDescription(message.offer));
    const answer = await peerConnection.createAnswer();
    await peerConnection.setLocalDescription(answer);
    signalingChannel.send({'answer': answer});
  }
});
```



连接建立监听：

```javascript
// 监听本地peerConnection的状态改变
peerConnection.addEventListener('connectionstatechange', event => {
  if (peerConnection.connectionState === 'connected') {
    // 连接成功
  }
});
```



------

上面介绍了`RTCPeerConnection`的对等连接过程，然后它还可以用来传递任意数据；可以通过`createDataChannel`来返回一个`RTCDataChannel`对象

```javascript
const peerConnection = new RTCPeerConnection(configuration);
const dataChannel = peerConnection.createDataChannel();
```



`RTCPeerConnection`实例还可以通过监听`datachannel`对象上的事件来接收数据通道`RTCPeerConnection`

```javascript
const peerConnection = new RTCPeerConnection(configuration);
peerConnection.addEventListener('datachannel', event => {
  const dataChannel = event.channel;
});
```



在将数据通道用于发送数据之前，客户端是需要等待直到打开为止，通过监听`open`事件可以做到这一点；同样的，`close`监听事件也类似

```javascript
const messageBox = document.querySelector('#messageBox');
const sendButton = document.querySelector('#sendButton');
const peerConnection = new RTCPeerConnection(configuration);
const dataChannel = peerConnection.createDataChannel();

// 打开时让文本域可用
dataChannel.addEventListener('open', event => {
  messageBox.disabled = false;
  messageBox.focus();
  sendButton.disabled = false;
})

// 关闭时灰置文本域
dataChannel.addEventListener('close', event => {
  messageBox.disabled = false;
  sendButton.disabled = false;
});
```



`RTCDataChannel`通过`send`函数来发送数据消息，发送数据的参数可以是字符串、`Blob`、`ArrayBuffer`、`ArrayBufferView`

```javascript
const messageBox = document.querySelector('#messageBox');
const sendButton = document.querySelector('#sendButton');

// Send a simply text message when we click the button
sendButton.addEventListener('click', event => {
  const message = messageBox.textContent;
  dataChannel.send(message);
})
```



远程对等方则通过`RTCDataChannel`监听`message`事件来接收发送的消息

```javascript
const incomingMessages = document.querySelector('#incomingMessages');
const peerConnection = new RTCPeerConnection(configuration);
const dataChannel = peerConnection.createDataChannel();

dataChannel.addEventListener('message', event => {
	const message = event.data;
	incomingMessages.textContent += message + '\n';
});
```



以上，就是近段时间对与`WebRTC`的一些简单的了解和概览，基于此，我们可以做的应用和场景覆盖面还是挺多的，之后打算研究基于node的信令服务器来搭建一个音视频聊天，多尝试，go！



### 参考链接

[cdn Api](https://developer.mozilla.org/zh-CN/docs/Web/API/WebRTC_API )

[Webrtc介绍](https://hpbn.co/webrtc )

[社区](https://webrtc.org.cn/)







