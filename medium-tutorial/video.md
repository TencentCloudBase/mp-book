# 音视频增值服务

## 功能概述

本增值服务，整合了腾讯云的[实时音视频](https://cloud.tencent.com/product/trtc)能力，通过云开发的云函数和数据库的能力，简化了配置的拉取和房间的管理。

## 体验

<p align="center">
    <img src="https://main.qcloudimg.com/raw/5dffe79d2f0b28f7212c17c6a77be62e.png" width="500px">
    <p align="center">扫码体验</p>
</p>

## DEMO 源码

本章的案例代码在 [tcb-demo-video](https://github.com/TencentCloudBase/tcb-demo-video)。包含了小程序前端代码 （`client` 目录下）和云函数代码（`cloud` 目录下）。
## DEMO 接入流程

1. 登陆[小程序的管理后台](https://mp.weixin.qq.com/) , 选择【设置】> 【基本设置】 > 【服务类目】中添加允许视频直播类的管理。

<p align="center">
    <img src="https://main.qcloudimg.com/raw/0e318bae69e6912475ad7a81763bc5b8.png" width="800px">
    <p align="center">申请相关小程序类目</p>
</p>


2. 在小程序管理后台【开发】>【接口设置】中，将**实时播放音视频流**和**实时录制音视频流**打开。

<p align="center">
    <img src="https://main.qcloudimg.com/raw/6e5a2678a8dc7c9d2658917c3c1ef1a0.png" width="800px">
    <p align="center">允许音视频播放和录制</p>
</p>

3. 在小程序管理后台【开发】>【开发设置】中，配置以下三个域名：
    - https://official.opensso.tencent-cloud.com
    - https://yun.tim.qq.com
    - https://room.qcloud.com
    - https://webim.tim.qq.com

<p align="center">
    <img src="https://main.qcloudimg.com/raw/8d45af53bbe1d4f85219dedf65cc637d.png" width="800px">
    <p align="center">配置相关域名</p>
</p>

4. 通过[微信小程序第三方绑定流程](https://www.qcloud.com/login/mp?s_url=https%3A%2F%2Fconsole.cloud.tencent.com%2Fcam%2Fcapi)登录小程序对应的腾讯云帐号(需要小程序管理员权限)，然后到腾讯云的[实时音视频](https://cloud.tencent.com/product/trtc)，开通服务并购买体验包，进入控制台，并获取以下配置信息：

(1) SDKAppid 和 accoutType

<p align="center">
    <img src="https://main.qcloudimg.com/raw/300ea0eb8cbd15a614c303b6f1508ba4.png" width="800px">
    <p align="center">SDKAppid 和 accoutType</p>
</p>

(2) 点击下载公私钥，使用其中的 `private_key` 文件

<p align="center">
    <img src="https://main.qcloudimg.com/raw/e17faec4233f466196ffd9a4a26fc366.png" width="800px">
    <p align="center">private_key 文件</p>
</p>

5. 请使用[微信开发者工具](https://developers.weixin.qq.com/miniprogram/dev/devtools/download.html)打开源码，在根目录下的 `project.config.json` 文件，填写您的小程序 `appid`。

6. 请按以下步骤完成最后的配置r操作。
* 在云函数目录 `cloud/functions` 的函数 `webrtc-sig-api` 中，将 `private_key` 文件放到 `config` 目录下；
* 在每个云函数目录的 `config` 目录下，参照 `example.js` 文件，新建 `index.js` 文件，并配置好 `SDKAppid` 和 `accountType`；
* 上传部署所有的云函数;
* 最后，在云开发面板的数据库栏目中，创建 `webrtcRooms` 集合。

<p align="center">
    <img src="https://main.qcloudimg.com/raw/ad9a36f9dafde5721acf1b22499417cf.png" width="800px">
    <p align="center">创建 webrtcRooms 集合</p>
</p>

7. 在微信开发者工具直接预览即可。

## 源码介绍

### WebRTC能力

WebRTC 的能力的体验，主要是围绕源码中 `client/pages/webrtc-room` 里面的 `join-room` 和 `room` 两个目录，一个是手动输入房间号进入房间，另一个是视频房间，还涉及到的是 `cloud/functions/webrtc-sig-api` 目录，此云函数主要用于填写实时音视频的配置后，进行加密，将加密好的信息传到小程序端，才能正常使用 WebRTC 视频通话能力。

云函数 `webrtc-sig-api` 中，`WebRTCSigApi.js` 是官方提供的[签名逻辑文件](https://github.com/TencentVideoCloudMLVBDev/usersig_server_source/blob/master/nodejs/WebRTCSigApi.js)，而 `index.js`，不外乎是调用 `WebRTCSigApi.js` 中的方法，然后将 `privateMapKey`, `userSig` 提供到小程序，有了这两个信息，小程序端才能正常将直播流唤起。

### 房间管理

房间管理主要通过云开发的云函数和数据库实现。`webrtcRooms` 集合的数据格式如下：

<p align="center">
    <img src="https://main.qcloudimg.com/raw/1f06945c2b42e7aa3ee44dd6ba6a7951.png" width="800px">
    <p align="center">房间数据格式</p>
</p>

数据包括有房间创建者和观众的 `openid`，房间 id 和房间名、房间创建时间，还有房间的权限位 `privateMapKey`。

云函数分别有创建房间(webrtc-create-room)、进入房间(webrtc-enter-room)、退出房间(webrtc-quit-room)、获取房间信息(webrtc-get-room-info)、获取房间列表(webrtc-get-room-list)5个云函数。

* `webrtc-create-room` 函数主要用于创建房间，这里用到了数据库的读写，先要判断房间是否存在，如果不存在，则创建。

>! 该函数有一处逻辑值得解读下，此处是通过循环的方式，去检查房间 id ，以防生成了重复的 id。

```js
// 循环检查数据，避免 generateRoomID 生成重复的roomID
while (await isRoomExist(roomInfo.roomID)) {
    roomInfo.roomID = generateRoomID()
}
```

* `webrtc-enter-room` 函数主要用于进入房间，如果房间存在，则将用户的 `openid` 写入房间观众字段，如果房间不存在，则调用 `webrtc-create-room` 进行房间创建。

* `webrtc-quit-room` 函数主要用于退出房间，如果房间还有观众，则将退出者的 `openid` 清楚，如果没有观众了，则把房间数据清理掉。

* `webrtc-get-room-info` 函数主要用于获取房间数据。

* `webrtc-get-room-list` 函数主要用于获取房间列表数据。

