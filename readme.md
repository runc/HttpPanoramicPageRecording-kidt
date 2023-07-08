# HttpPanoramicPageRecording 帮助文档

<p align="center" class="flex justify-center">
    <a href="https://www.serverless-devs.com" class="ml-1">
    <img src="http://editor.devsapp.cn/icon?package=headless-ffmpeg&type=packageType">
  </a>
  <a href="http://www.devsapp.cn/details.html?name=headless-ffmpeg" class="ml-1">
    <img src="http://editor.devsapp.cn/icon?package=headless-ffmpeg&type=packageVersion">
  </a>
  <a href="http://www.devsapp.cn/details.html?name=headless-ffmpeg" class="ml-1">
    <img src="http://editor.devsapp.cn/icon?package=headless-ffmpeg&type=packageDownload">
  </a>
</p>

<description>

> ***快速部署一个Http触发的全景录制的应用到阿里云函数计算***

</description>

<table>

</table>

<codepre id="codepre">

</codepre>

<deploy>

## 部署 & 体验

<appcenter>

- :fire: 通过 [Serverless 应用中心](https://fcnext.console.aliyun.com/applications/create?template=HttpPanoramicPageRecording&type=direct) ，
[![Deploy with Severless Devs](https://img.alicdn.com/imgextra/i1/O1CN01w5RFbX1v45s8TIXPz_!!6000000006118-55-tps-95-28.svg)](https://fcnext.console.aliyun.com/applications/create?template=HttpPanoramicPageRecording&type=direct)  该应用。

</appcenter>

- 通过 [Serverless Devs Cli](https://www.serverless-devs.com/serverless-devs/install) 进行部署：
  - [安装 Serverless Devs Cli 开发者工具](https://www.serverless-devs.com/serverless-devs/install) ，并进行[授权信息配置](https://www.serverless-devs.com/fc/config) ；
  - 初始化项目：`s init HttpPanoramicPageRecording -d HttpPanoramicPageRecording`
  - 进入项目，并进行项目部署：`cd HttpPanoramicPageRecording && s deploy -y`

</deploy>

<appdetail id="flushContent">

# 调用函数

``` bash
# deploy
$ s deploy -y --use-local
# Invoke
$ s invoke -e '{"record_time":"60","video_url":"https://tv.cctv.com/live/cctv1/","output_file":"record/test.mp4", "width":"1920", "height":"1080", "scale": 0.75, "frame_rate":25,"bit_rate":"2000k"}'
```

调用成功后， 会在对应的 bucket 下， 产生 record/test.mp4 大约 60 秒 1920x1080 的全景录制视频。

其中参数的意义：

**1.record_time:** 录制时长

**2.video_url:** 录制视频的 url

**3.width:** 录制视频的宽度

**4.height:** 录制视频的高度

**5.scale:** 浏览器缩放比例

**6.output_file:** 最后录制视频保存的 OSS 目录

**7.frame_rate:** 录制视频的帧率(可不传递，默认帧率为30fps)

**8.bit_rate:** 录制视频的码率(可不传递，默认码率为1500k)

**9.output_stream:** 推流地址(可选参数,eg: rtmp://demo.aliyundoc.com/app/stream?xxxx)

其中 scale 是对浏览器进行 75% 的缩放，使视频能录制更多的网页内容

**注意:** 如果您录制的视频存在一些卡顿或者快进， 可能是因为您录制的视频分辨率大并且复杂， 消耗的 CPU 很大， 您可以通过调大函数的规格， 提高 CPU 的能力。

比如上面的示例参数得到下图:

![](https://img.alicdn.com/imgextra/i3/O1CN01fbUSSP1umgrF0cfFr_!!6000000006080-2-tps-3048-1706.png)

# 如何本地调试

直接本地运行， 命令执行完毕后， 会在当前目录生成一个 test.mp4 的视频

```bash
# 直接本地执行docker命令， 会在当前目录生成一个 test.mp4 的视频
$ docker run --rm --entrypoint="" -v $(pwd):/var/output aliyunfc/browser_recorder  /code/record.sh 60 https://tv.cctv.com/live/cctv1 1920x1080x24 1920,1080 1920x1080 1 25 2000k
```

调试

```bash
# 如果有镜像有代码更新, 重新build 镜像
$ docker build -t my-panoramic-page-recording -f ./code/Dockerfile ./code
# 测试全屏录制核心脚本 record.sh, 执行完毕后， 会在当前目录有一个 test.mp4 的视频
$ docker run --rm --entrypoint="" -v $(pwd):/var/output my-panoramic-page-recording  /code/record.sh 60 https://tv.cctv.com/live/cctv1 1920x1080x24 1920,1080 1920x1080 1 25 2000k
```

> 其中 record.sh 的参数意义:
>
> 1. 录制时长
> 2. 视频 url
> 3. $widthx$heightx24
> 4. $width,$height
> 5. $widthx$height
> 6. chrome 浏览器缩放比例
> 7. 帧率
> 8. 码率
> 9. 推流地址

# 原理

Chrome 渲染到虚拟 X-server，并通过 FFmpeg 抓取系统桌⾯，通过启动 xvfb 启动虚拟 X-server，Chrome 进⾏全屏显示渲染到到虚拟 X-server 上，并通过 FFmpeg 抓取系统屏幕以及采集系统声⾳并进⾏编码写⽂件。这种⽅式的适配性⾮常好， 不仅可以录制 Chrome，理论上也可以录制其他的应⽤。缺点是占⽤的内存和 CPU 较多。

**server.js**

custom container http server 逻辑

**record.sh**

核心录屏逻辑， 启动 xvfb， 在虚拟 X-server 中使用 `record.js` 中的 puppeteer 启动浏览器， 最后 FFmpeg 完成 X-server 屏幕的视频和音频抓取工作， 生成全屏录制后的视频

# 其他

如果您想将生成的视频直接预热的 CDN， 以阿里云 CDN 为例， 只需要在 server.js 上传完 OSS bucket 后的逻辑中增加如下代码：

[PushObjectCache](https://next.api.aliyun.com/api/Cdn/2018-05-10/PushObjectCache?lang=NODEJS&sdkStyle=old&params={})

> Tips 前提需要配置好 CDN

</appdetail>

<devgroup>

## 开发者社区

您如果有关于错误的反馈或者未来的期待，您可以在 [Serverless Devs repo Issues](https://github.com/serverless-devs/serverless-devs/issues) 中进行反馈和交流。如果您想要加入我们的讨论组或者了解 FC 组件的最新动态，您可以通过以下渠道进行：

<p align="center">

| <img src="https://serverless-article-picture.oss-cn-hangzhou.aliyuncs.com/1635407298906_20211028074819117230.png" width="130px" > | <img src="https://serverless-article-picture.oss-cn-hangzhou.aliyuncs.com/1635407044136_20211028074404326599.png" width="130px" > | <img src="https://serverless-article-picture.oss-cn-hangzhou.aliyuncs.com/1635407252200_20211028074732517533.png" width="130px" > |
|--- | --- | --- |
| <center>微信公众号：\`serverless\`</center> | <center>微信小助手：\`xiaojiangwh\`</center> | <center>钉钉交流群：\`33947367\`</center> |

</p>

</devgroup>
