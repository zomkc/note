```vue
<template> 
  <button @click="videoadd">打开摄像头</button>
  <div class="video">
    <video id="self" autoplay ></video>
  </div>
</template>
<script setup>
function videoadd(){  //调用本地摄像头
  var self = document.getElementById('self')
  navigator.mediaDevices
  .getUserMedia({video: true,audio: true})
  .then((stream)=>{
    console.log(stream);
    self.srcObject = stream
  }).catch((err)=>{
    console.log("错误:"+err);
  })
}
</script>
```

		`navigator.mediaDevices.getUserMedia`接口提供访问连接媒体输入的设备，如照相机和麦克风，以及屏幕共享等。它可以使你取得任何硬件资源的媒体数据

​		浏览器要获取摄像头权限需要开启本地端口或者https服务

## 1. mediaDevices

mediaDevices上提供了4个方法和一个监听事件

方法：

- `enumerateDevices`:  获取系统可用的媒体输入和输出设备
- `getSupportedConstraints`： 返回一个输入设备可支持的**约束**属性
- `getDisplayMedia`： 开启屏幕共享
- `getUserMedia`： 开启媒体输入设备

事件：

- devicechange

### 1.1 enumerateDevices

enumerateDevices返回一个promise，如果正确执行可以得到一个MediaDeviceInfo的数组，每项分别有4个属性（都是只读）。

如果页面未获取浏览器设备权限，则返回的deviceId和label都是空字符串

```js
navigator.mediaDevices.enumerateDevices().then(res => console.table(res));
```

`deviceId`: 代表设备的id，随机生成，该网页与其他网页获取的id不同；

`label`: 设备的别名

`kind`: 枚举值 audioinput audiooutput videoinput，因为视频输出靠屏幕，因此没有videooutput这个选项

`groupId`: 如果设备是同一个物理设备，那么这些设备的groupId就是同一个



### 1.2 getUserMedia

​		浏览器获取视频、音频的入口

```js
navigator.mediaDevices.getUserMedia({ video: true, audio: true }).then((stream)=>{
    console.log(stream);
  })
```

​		初次调用后会询问权限策略：允许或者禁止。允许后会获取永久权限，如果想再改成询问，需要点击地址栏前的小叹号修改

​		返回的stream表示媒体流，内部可以包括多个媒体轨道track，比如视频轨和音频轨。

#### 1.2.1 stream

stream上有2个重要的属性 active 代表该流是否为活动状态 和 id 代表流的唯一值

简而言之，stream就是管理tarck的集合，在stream上有操作track的一些增删查方法：

- addTrack: 将track添加到stream中
- getTracks getVideoTracks getAudioTracks：顾名思义，会返回track list
- getTrackById: track是有id属性，可以根据id在stream中获取该track，如果不存在会返回null
- removeTrack: 传入值是MediaStreamTrack对象，而非trackId

同时stream也可以订阅track相关的事件：

- onaddtrack
- onremovetrack

除此之外stream上还有clone方法，返回MediaStream的克隆版本，并且会返回一个新的id；

#### 1.2.2 track

​		视频/音频轨道

```js
stream.getTracks()

属性(除了enabled外，全是只读)：

id： 代表唯一值
kind: "audio" | "video"
enabled: 表示该轨道是否可用，可以被手动设置，设置为false后，视频黑屏，音频静音
label: 和mediaDevices.enumerateDevices返回值设备的label相对应
muted: 是否静音
readyState: 枚举值， "live"表示输入设备正常连接，"ended"表示没有更多的数据

方法:
getConstraints(): 返回创建该轨道的约束（getUserMedia的参数），只以video为例，video可设置的选项很多

applyConstraints(): 给该轨道应用新的约束

getSettings(): 和getConstraints不同的是，getSettings会返回浏览器添加的默认约束和自己明确添加的约束，也就是轨道的所有约束

getCapabilities(): 方法返回一个 MediaTrackCapabilities 对象，此对象表示每个可调节属性的值或者范围，该特性依赖于平台和user agent。

clone(): 克隆一个track的备份，和stream一样，会产出一个新的id

stop(): stop后，readyState的状态就变成了ended

```

### 1.3 getDisplayMedia

​		调用后会在所有屏幕/应用/Chrome标签页中选择一个共享画面

​		因为是共享屏幕，即使约束不添加video选项，stream中也有默认添加一个video。其他约束、stream、track就和getUserMedia一样了

### 1.4 getSupportedConstraints

​		返回一个对象，该对象表明可以约束`MediaStreamTrack`的属性

​		`getUserMedia`和`getDisplayMedia`这两个方法接收一个对象作为参数，对象中可以分别对video和audio设置具体的参数（称之为constraints即约束）。

```js
navigator.mediaDevices.getUserMedia({
	video: {},
    audio: {},
})
```



​		通过`getSupportedConstraints`获取的属性就是设置约束的具体内容，这些属性可以分为通用设置、只对video设置、只对aduio设置

#### 1.4.1 通用设置 deviceId & groupId

​		通用设置，可以指定外设，可以通过`mediaDevices.enumerateDevices`来查询外设的deviceId或者groupId

#### 1.4.2 video width & height

​		这两个属性只对video来设置，设置方法一样，单拿一个来讲，一共有3种设置方式：

1.不设置：使用的是浏览器的默认值，如何获取默认值是多少？可以通过track.getSettings来获取

```js
navigator.mediaDevices.getUserMedia({ video: true });
```

2.精确值：宽度和高度分别设置2个值，有没有范围呢？有。如何获取范围值呢？可以通过track.getCapabilities来获取

```js
navigator.mediaDevices.getUserMedia({ video: {
	width: 640,
    height: 480,
} });
```

3.范围： 会在这个范围内尽量满足， ideal是最理想的值，如果ideal在可设置的范围内，那么就会应用ideal的值；如果min的值超出的浏览器的最大值，那么就会按浏览器的最大提供的能力来展示；

```js
navigator.mediaDevices.getUserMedia({ video: {
	width: { min: 1024, ideal: 1280, max: 1920 },
    height: { min: 776, ideal: 720, max: 1080 }
} });
```

#### 1.4.3 video frameRate

属性只对video生效，也分为不设置、精确值、范围值来设置，不再赘述

#### 1.4.4 video facingMode

一般用在移动设备上，选择开启前置或者后置摄像头

```js
navigator.mediaDevices.getUserMedia({ video: {
	facingMode: 'user'
} });

'user': 前置摄像头
'environment': 后置摄像头
'left':视频源面向用户但在他们的左边，例如一个对准用户但在他们的左肩上方的摄像机。
'right': 视频源面向用户但在用户的右边，例如，摄像机对准用户但在他们的右肩上

```

#### 1.4.5 video aspectRatio

设置采集图像的宽高比

```js
async function initMedia() {
    const stream = await navigator.mediaDevices.getUserMedia({
        video: {
            width: 640,
            aspectRatio: 2,
        }
    });

    // 宽度 640 高度 320
    localvideo.srcObject = stream;
}
```

#### 1.4.6 audio autoGainControl & noiseSuppression & echoCancellation

这3个属性通过`track.getSettings`来看默认值都是`true`。 autoGainControl是自动音量调节； noiseSuppression是噪声消除； echoCancellation是回声消除；

噪声消除和回声消除是浏览器内置算法消除的，一定不要设置成false

#### 1.4.7 audio channelCount & sampleRate & sampleSize

音频三要素，分别是声道数、采样率、位深。我在浏览器中得到的默认值是 1 48k 16bit 可以得到未压缩的码率：48k * 16 * 1 = 768kb/s

#### 1.4.8 audio latency

可以接受的延迟，值是数字，也可以设置范围

## 参考文档

```http
https://developer.mozilla.org/zh-CN/docs/Web/API/MediaDevices
```







## 建立对等连接

#### 
