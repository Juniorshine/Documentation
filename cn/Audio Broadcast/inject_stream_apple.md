
---
title: 输入在线媒体流
description: 
platform: iOS,macOS
updatedAt: Sun Sep 29 2019 08:26:09 GMT+0800 (CST)
---
# 输入在线媒体流
## 功能描述
输入在线媒体流功能可以将音视频流作为一个发送端导入正在进行的直播房间。通过将正在播放的音视频导入到直播频道中，主播和观众可以一起收听/观看该媒体流并实时互动。

### 常用场景

- 赛事直播中，主播直接拉比赛的音视频流，实现主播和观众边看比赛边点评的功能。
- 同一直播间内，主播与观众一同欣赏电影、音乐、演出，并实时交流讨论。
- 无人机或网络摄像头直接采集视频，该视频作为在线媒体流导入直播频道中。

### 工作原理

![](https://web-cdn.agora.io/docs-files/1569397540228)

直播频道中的主播通过 Video Inject 服务器将在线媒体流拉到 Agora SD-RTN 中，导入到直播频道内。

- 频道内的连麦主播、普通观众都可以听/看到该媒体流。
- 如果主播开启了 CDN 旁路推流，该媒体流也会被推送到 CDN 上， CDN 观众就可以听/看到这路媒体流。

> - 支持的媒体流格式：RTMP、HLS 和 FLV。纯音频流也可作为在线媒体流导入直播频道。
> - 只有主播可以导入/删除在线媒体流，连麦主播、普通观众和 CDN 观众都不可以。

## 实现方法

在实现输入在线媒体流功能前，请确保你已在项目中完成基本的实时音视频功能。详见[开始互动直播](../../cn/Audio%20Broadcast/start_live_ios.md)。

参考如下步骤，在你的项目中实现输入在线媒体流功能：

1. 频道内主播调用 `addInjectStreamUrl` 方法向直播频道内导入指定在线媒体流。你也可以修改 config 的参数设置媒体流导入的分辨率、码率和帧率等参数，详见[`AgoraInjectStreamConfig`](https://docs.agora.io/cn/Audio%20Broadcast/API%20Reference/oc/Classes/AgoraLiveInjectStreamConfig.html)。

   > 频道内同一时间只允许输入一个在线媒体流。

   导入媒体流成功后，该媒体流会在直播频道内自动播放，频道内所有用户都会收到 `didJoinedOfUid` (`uid`: 666) 回调，导入媒体流的主播同时还会收到 `streamInjectedStatusOfUrl` 回调。

   > 如果导入媒体流失败，查阅 [API 参考](#api)排查问题。

2. 频道内主播调用 `removeInjectStreamUrl` 方法从直播频道内删除指定的已导入在线媒体流。

   删除媒体流成功后，频道内所有用户都会收到  `didOfflineOfUid` (`uid: 666`) 回调。

   > 主播退出频道后，无需再调用 `removeInjectStreamUrl` 接口。



### 示例代码

```swift
// Swift
// 导入在线媒体流
let config = AgoraLiveInjectStreamConfig()
  config.size = CGSize(width: 640, height: 360)
  config.videoGop = 30
  config.videoBitrate = 400
  config.videoFramerate = 15
  config.audioSampleRate = 48000
  config.audioBitrate = 48
  config.audioChannels = 1

  agoraKit.addInjectStreamUrl("media stream url", config: config)

// 删除在线媒体流
let urlPath = "Some online RTMP/HLS url path"
  agoraKit.removeInjectStreamUrl(urlPath)
```

```objective-c
// Objective-C
// 导入在线媒体流
AgoraLiveInjectStreamConfig *config = [[AgoraLiveInjectStreamConfig alloc] init];
  config.size = CGSizeMake(640, 360);
  config.videoGop = 30
  config.videoBitrate = 400
  config.videoFramerate = 15
  config.audioSampleRate = 48000
  config.audioBitrate = 48
  config.audioChannels = 1

  [agoraKit addInjectStreamUrl: @"media stream url" config: config];

// 删除在线媒体流
NSString *urlPath = @"Some online  RTMP/HLS url path";
  [agoraKit removeInjectStreamUrl: urlPath];
```

同时，我们在 Github 提供一个开源的 [Live-Streaming-Injection](https://github.com/AgoraIO/Advanced-Interactive-Broadcasting/tree/master/Live-Streaming-Injection) 示例项目。

<a name="api"></a>
### API 参考

- [`addInjectStreamUrl`](https://docs.agora.io/cn/Audio%20Broadcast/API%20Reference/oc/Classes/AgoraRtcEngineKit.html#//api/name/addInjectStreamUrl:config:)
- [`removeInjectStreamUrl`](https://docs.agora.io/cn/Audio%20Broadcast/API%20Reference/oc/Classes/AgoraRtcEngineKit.html#//api/name/removeInjectStreamUrl:)
- [`streamInjectedStatusOfUrl`](https://docs.agora.io/cn/Audio%20Broadcast/API%20Reference/oc/Protocols/AgoraRtcEngineDelegate.html#//api/name/rtcEngine:streamInjectedStatusOfUrl:uid:status:)
- [`didJoinedOfUid`](https://docs.agora.io/cn/Audio%20Broadcast/API%20Reference/oc/Protocols/AgoraRtcEngineDelegate.html#//api/name/rtcEngine:didJoinedOfUid:elapsed:)
- [`didOfflineOfUid`](https://docs.agora.io/cn/Audio%20Broadcast/API%20Reference/oc/Protocols/AgoraRtcEngineDelegate.html#//api/name/rtcEngine:didOfflineOfUid:reason:)

## 开发注意事项

主播在直播过程中启用输入在线媒体流，观众需要订阅主播才能观看外部视频。