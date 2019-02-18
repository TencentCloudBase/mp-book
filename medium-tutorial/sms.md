# 短信增值服务

## 功能概述

本增值服务，整合了腾讯云的[短信](https://cloud.tencent.com/document/product/382)能力，通过云开发的云函数和数据库的能力，简化了短信验证码、通知等场景。

## DEMO 源码

本章的案例代码在 [tcb-demo-sms](https://github.com/TencentCloudBase/tcb-demo-vidsmseo)。包含了小程序前端代码 （`client` 目录下）和云函数代码（`cloud` 目录下）。

## DEMO 接入流程
1. 请使用[微信开发者工具](https://developers.weixin.qq.com/miniprogram/dev/devtools/download.html)打开 `DEMO` 源码，在根目录下的 `project.config.json` 文件，填写您的小程序 `appid`。

2. 通过此[微信小程序第三方绑定流程](https://www.qcloud.com/login/mp?s_url=https%3A%2F%2Fconsole.cloud.tencent.com%2Fcam%2Fcapi)登录小程序对应的腾讯云帐号(需要小程序管理员权限)。

3. 在腾讯云的[短信控制台](https://console.cloud.tencent.com/ai)，开通短信服务。然后点击 “添加应用” 按钮，添加短信服务，便可开通国内短信、国际短信和语音短信三项服务，其中，语音服务需要有企业资质才可以使用。

4. 点击进入新添加的应用后，获取 `AppID` 和 `AppKey`。获取后请填写在对应的云函数的 `config/index.js` 配置文件中。

<p align="center">
    <img src="https://main.qcloudimg.com/raw/e4f158e139f1b4a202f3e743b1b28066.png" width="1000px">
    <p align="center">获取 AppID 和 AppKey</p>
</p>

5. 验证码功能

该功能使用到 `create-verification-code`，`verify-verification-code`,  `sms-single-send`, 和 `code-voice-send` 4个云函数。同时，需要创建名为 `verification` 的集合。

其中，`sms-single-send` 需要在【国内短信】->【短信内容配置】中，配置短信正文（短信模板），如果有需要同时可以配置短信签名。比如你配置的短信正文如下：

```js
你的验证码为：{1}，请于2分钟内填写。如非本人操作，请忽略本短信。
```

那么，请在 `create-verification-code` 的函数中的 `sendNormalSMS` 方法里，将该正文内容填入 `msg` 参数里。


而 `code-voice-send` 则需要企业资质才可以使用，但可以不需要设置模板。


6. 普通短信通知功能

该功能使用到 `sms-single-send-template`, `sms-multi-send`, `sms-multi-send-template`，3个函数都需要在【国内短信】->【短信内容配置】中，配置短信正文（短信模板），如果有需要同时可以配置短信签名。对于 `sms-single-send-template` 和  `sms-multi-send-template`，需要获取短信正文的 ID，并填写在 `config/index.js` 配置晨：

<p align="center">
    <img src="https://main.qcloudimg.com/raw/9e812b379921edae2a83a948e511857c.png" width="1000px">
    <p align="center">获取短信正文（模板）ID</p>
</p>

对于 `sms-multi-send`，跟 `sms-single-send` 一样，配置好短信正文后，需要将短信正文的内容放到 `config/index.js` 中的 `Msg` 参数里。

7. 上传语音文件功能

该功能使用到 `voice-file-upload` 云函数。将语音文件手动上传到云开发存储服务中后，将文件的 `fileID` 作为参数传给  `voice-file-upload` 上传到短信服务里，上传成功后短信服务会返回音频文件的 `fid`。

8. 语音短信通知功能

该功能使用到 `prompt-voice-send`, `tts-voice-send` 和 `file-voice-send`。 `prompt-voice-send` 和 `tts-voice-send` 都需要在【语音短信】->【语音内容配置】中创建语音正文模板，前者需要在 `config/index.js` 中传到正文的内容，而后者则需要填入正文的模板 ID。至于 `file-voice-send`，则需要上传语音文件且该文件经过审核后，利用该文件进行语音短信通知。

9. 正文模板审核
无论普通短信，语音短信的正文模板还是通过代码上传的语音文件，都需要进和审核。可添加 QQ `3012203387` 加快提审速度。

## 源码介绍

### 服务调用

该解决方案的主要功能，主要是直接调用 API，因此不再赘述，详细可以参考 [tcb-service-sdk]()，了解具体的使用文档。

### 验证码

该功能使用了短信的单发功能与语音验证码功能。整个业务逻辑如下：

1. 输入手机，点击 ”获取验证码“，会调用 `create-verification-code` 生成验证码数据，并返回验证码发送间隔和有效期，在前端开始倒计时禁止用户在发送间隔时间内不能重复生成验证参与。
2. `create-verification-code` 生成的验证码数据如下：
```js
{
    code, // 验证码
    validTime, // 验证码有效时间
    gapTime, // 验证码发送间隔
    createTime, // 验证码生成时间
    _openid, // 该验证码对应用户
    check, // 校验结果
}
```
3. 根据用户选择的类型，调用 `sms-single-send` 或 `code-voice-send`，进行普通短信或者语音短信的验证码发送。

4. 用户获取验证码后，输入验证码进行校验，此时会调用 `verify-verification-code` 进行校验，此时会通过用户的 `openid` 获取对应验证码数据，并校验验证码一致性与时效。