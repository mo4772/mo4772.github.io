---
layout:     post
title:      Webrtc Fec分析(三) FEC的协商.md
subtitle:   Webrtc FEC
date:       2025-09-10
author:     mo4772
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Webrtc
    - FEC
---

## fec的协商
前面两篇文章介绍了fec的原理和处理流程，但都属于媒体层面。

要使用fec首先需要协商出fec，包括两点：是否使用fec；fec的payload type值。这些信息在sdp信息中携带。

### Red包
Red(Redundant)包是指冗余包，定义在RFC 2198中，是一种对冗余数据的封装格式。

虽然这个RFC文档的名字叫 RTP Payload for Redundant Audio Data，但是在webrtc中视频的fec包也被封装在了Red包中。



对于音频，有些音频格式的编码器本身就可以产生fec，比如opus，它是webrtc中的默认使用的音频格式，它可以产生fec数据，直接携带在音频的编码流中，称为带内冗余数据，它并不需要通过Red包格式封包。

对其的本身没有fec功能的音频编码格式，可以通过特定的方式生成冗余包，再通过Red包格式封包。



对于视频，ulpc fec(或者flex fec)被再次封装到Red包中。



在sdp协商时也会协商Red信息。

### sdp中默认就携带了fec信息
如下sdp，Red和Fec信息在最后面。

> a=fingerprint:sha-256 05:65:5C:96:5D:76:16:EF:10:08:C4:DD:00:30:5A:74:84:8C:32:D1:81:89:91:70:54:07:0D:D8:BB:4F:91:15
>
> a=setup:actpass
>
> a=mid:1
>
> a=extmap:14 urn:ietf:params:rtp-hdrext:toffset
>
> a=extmap:2 [http://www.webrtc.org/experiments/rtp-hdrext/abs-send-time](http://www.webrtc.org/experiments/rtp-hdrext/abs-send-time)
>
> a=extmap:13 urn:3gpp:video-orientation
>
> a=extmap:3 [http://www.ietf.org/id/draft-holmer-rmcat-transport-wide-cc-extensions-01](http://www.ietf.org/id/draft-holmer-rmcat-transport-wide-cc-extensions-01)
>
> a=extmap:5 [http://www.webrtc.org/experiments/rtp-hdrext/playout-delay](http://www.webrtc.org/experiments/rtp-hdrext/playout-delay)
>
> a=extmap:6 [http://www.webrtc.org/experiments/rtp-hdrext/video-content-type](http://www.webrtc.org/experiments/rtp-hdrext/video-content-type)
>
> a=extmap:7 [http://www.webrtc.org/experiments/rtp-hdrext/video-timing](http://www.webrtc.org/experiments/rtp-hdrext/video-timing)
>
> a=extmap:8 [http://www.webrtc.org/experiments/rtp-hdrext/color-space](http://www.webrtc.org/experiments/rtp-hdrext/color-space)
>
> a=extmap:4 urn:ietf:params:rtp-hdrext:sdes:mid
>
> a=extmap:10 urn:ietf:params:rtp-hdrext:sdes:rtp-stream-id
>
> a=extmap:11 urn:ietf:params:rtp-hdrext:sdes:repaired-rtp-stream-id
>
> a=sendrecv
>
> a=msid:31713af9-b809-4ac6-8b80-16dc45018c2f 77fe6f7c-5934-48d3-905e-04153ad21dd7
>
> a=rtcp-mux
>
> a=rtcp-rsize
>
> a=rtpmap:96 VP8/90000
>
> a=rtcp-fb:96 goog-remb
>
> a=rtcp-fb:96 transport-cc
>
> a=rtcp-fb:96 ccm fir
>
> a=rtcp-fb:96 nack
>
> a=rtcp-fb:96 nack pli
>
> a=rtpmap:97 rtx/90000
>
> a=fmtp:97 apt=96
>
> a=rtpmap:102 H264/90000
>
> a=rtcp-fb:102 goog-remb
>
> a=rtcp-fb:102 transport-cc
>
> a=rtcp-fb:102 ccm fir
>
> a=rtcp-fb:102 nack
>
> a=rtcp-fb:102 nack pli
>
> a=fmtp:102 level-asymmetry-allowed=1;packetization-mode=1;profile-level-id=42001f
>
> a=rtpmap:103 rtx/90000
>
> a=fmtp:103 apt=102
>
> a=rtpmap:104 H264/90000
>
> a=rtcp-fb:104 goog-remb
>
> a=rtcp-fb:104 transport-cc
>
> a=rtcp-fb:104 ccm fir
>
> a=rtcp-fb:104 nack
>
> a=rtcp-fb:104 nack pli
>
> a=fmtp:104 level-asymmetry-allowed=1;packetization-mode=0;profile-level-id=42001f
>
> a=rtpmap:105 rtx/90000
>
> a=fmtp:105 apt=104
>
> a=rtpmap:106 H264/90000
>
> a=rtcp-fb:106 goog-remb
>
> a=rtcp-fb:106 transport-cc
>
> a=rtcp-fb:106 ccm fir
>
> a=rtcp-fb:106 nack
>
> a=rtcp-fb:106 nack pli
>
> a=fmtp:106 level-asymmetry-allowed=1;packetization-mode=1;profile-level-id=42e01f
>
> a=rtpmap:107 rtx/90000
>
> a=fmtp:107 apt=106
>
> a=rtpmap:108 H264/90000
>
> a=rtcp-fb:108 goog-remb
>
> a=rtcp-fb:108 transport-cc
>
> a=rtcp-fb:108 ccm fir
>
> a=rtcp-fb:108 nack
>
> a=rtcp-fb:108 nack pli
>
> a=fmtp:108 level-asymmetry-allowed=1;packetization-mode=0;profile-level-id=42e01f
>
> a=rtpmap:109 rtx/90000
>
> a=fmtp:109 apt=108
>
> a=rtpmap:127 H264/90000
>
> a=rtcp-fb:127 goog-remb
>
> a=rtcp-fb:127 transport-cc
>
> a=rtcp-fb:127 ccm fir
>
> a=rtcp-fb:127 nack
>
> a=rtcp-fb:127 nack pli
>
> a=fmtp:127 level-asymmetry-allowed=1;packetization-mode=1;profile-level-id=4d001f
>
> a=rtpmap:125 rtx/90000
>
> a=fmtp:125 apt=127
>
> a=rtpmap:39 H264/90000
>
> a=rtcp-fb:39 goog-remb
>
> a=rtcp-fb:39 transport-cc
>
> a=rtcp-fb:39 ccm fir
>
> a=rtcp-fb:39 nack
>
> a=rtcp-fb:39 nack pli
>
> a=fmtp:39 level-asymmetry-allowed=1;packetization-mode=0;profile-level-id=4d001f
>
> a=rtpmap:40 rtx/90000
>
> a=fmtp:40 apt=39
>
> a=rtpmap:45 AV1/90000
>
> a=rtcp-fb:45 goog-remb
>
> a=rtcp-fb:45 transport-cc
>
> a=rtcp-fb:45 ccm fir
>
> a=rtcp-fb:45 nack
>
> a=rtcp-fb:45 nack pli
>
> a=rtpmap:46 rtx/90000
>
> a=fmtp:46 apt=45
>
> a=rtpmap:98 VP9/90000
>
> a=rtcp-fb:98 goog-remb
>
> a=rtcp-fb:98 transport-cc
>
> a=rtcp-fb:98 ccm fir
>
> a=rtcp-fb:98 nack
>
> a=rtcp-fb:98 nack pli
>
> a=fmtp:98 profile-id=0
>
> a=rtpmap:99 rtx/90000
>
> a=fmtp:99 apt=98
>
> a=rtpmap:100 VP9/90000
>
> a=rtcp-fb:100 goog-remb
>
> a=rtcp-fb:100 transport-cc
>
> a=rtcp-fb:100 ccm fir
>
> a=rtcp-fb:100 nack
>
> a=rtcp-fb:100 nack pli
>
> a=fmtp:100 profile-id=2
>
> a=rtpmap:101 rtx/90000
>
> a=fmtp:101 apt=100
>
> a=rtpmap:112 H264/90000
>
> a=rtcp-fb:112 goog-remb
>
> a=rtcp-fb:112 transport-cc
>
> a=rtcp-fb:112 ccm fir
>
> a=rtcp-fb:112 nack
>
> a=rtcp-fb:112 nack pli
>
> a=fmtp:112 level-asymmetry-allowed=1;packetization-mode=1;profile-level-id=64001f
>
> a=rtpmap:113 rtx/90000
>
> a=fmtp:113 apt=112
>
> **<font style="color:#DF2A3F;">a=rtpmap:116 red/90000</font>**
>
> **<font style="color:#DF2A3F;">a=rtpmap:117 rtx/90000</font>**
>
> **<font style="color:#DF2A3F;">a=fmtp:117 apt=116</font>**
>
> **<font style="color:#DF2A3F;">a=rtpmap:118 ulpfec/90000</font>**
>
> **<font style="color:#DF2A3F;">a=ssrc-group:FID 3791735673 2875250513</font>**
>
> **<font style="color:#DF2A3F;">a=ssrc:3791735673 cname:zH59X4Kl4ZqIleMd</font>**
>
> **<font style="color:#DF2A3F;">a=ssrc:3791735673 msid:31713af9-b809-4ac6-8b80-16dc45018c2f 77fe6f7c-5934-48d3-905e-04153ad21dd7</font>**
>
> a=ssrc:2875250513 cname:zH59X4Kl4ZqIleMd
>
> a=ssrc:2875250513 msid:31713af9-b809-4ac6-8b80-16dc45018c2f 77fe6f7c-5934-48d3-905e-04153ad21dd7
>



在sdp中看到，fec是一个独立的"流"(Red包)，payload type为116(Red)，Red流还有重传流(RTX，payload type为117)。ulpfec的payload type为118。



视频的fec被封装在Red包中，Red包中也可能包含了音视频的fec数据，可以通过ulpfec的payload type或视频包的ssrc(ulpfec包中的ssrc与视频流rtp包中的ssrc相同)来区分。

### 协商堆栈
#### sdp中携带fec
方法`AssignPayloadTypesAndDefaultCodecs`是生成本端sdp中媒体信息的方法，在它的实现中固定设置了Red和ulpfec，如下：

```cpp
std::vector<VideoCodec> AssignPayloadTypesAndDefaultCodecs(
    std::vector<webrtc::SdpVideoFormat> input_formats,
    const webrtc::WebRtcKeyValueConfig& trials) {
    if (input_formats.empty())
    return std::vector<VideoCodec>();
    static const int kFirstDynamicPayloadType = 96;
    static const int kLastDynamicPayloadType = 127;
    int payload_type = kFirstDynamicPayloadType;
    
    input_formats.push_back(webrtc::SdpVideoFormat(kRedCodecName));
    input_formats.push_back(webrtc::SdpVideoFormat(kUlpfecCodecName));
    
    if (IsEnabled(trials, "WebRTC-FlexFEC-03-Advertised")) {
        webrtc::SdpVideoFormat flexfec_format(kFlexfecCodecName);
        // This value is currently arbitrarily set to 10 seconds. (The unit
        // is microseconds.) This parameter MUST be present in the SDP, but
        // we never use the actual value anywhere in our code however.
        // TODO(brandtr): Consider honouring this value in the sender and receiver.
        flexfec_format.parameters = {{kFlexfecFmtpRepairWindow, "10000000"}};
        input_formats.push_back(flexfec_format);
    }
    
    .....
}
```

 `input_formats.push_back(webrtc::SdpVideoFormat(kRedCodecName));`

 `input_formats.push_back(webrtc::SdpVideoFormat(kUlpfecCodecName));`

 分别是Red和Ulpfec，所以sdp中默认就携带了。



其中也可以看到开启flexfec的条件。

#### 协商流程
先看一看sdp的协商堆栈

<!-- 这是一张图片，ocr 内容为：15181101811801181150181180150180151018118018018018018010180101811801510180180101801811801811180180101 1118111811181118111111811 1118181118181818111811181818181818181818181818 PERCOURESTON DREITERTRCRUCANICANCLENCLENTLENGINGGETRCEDSIRCEDS -->
![](https://github.com/mo4772/mo4772.github.io/raw/master/pic/fec/fec_3_code_1.png)



在调用`SetRemoteDescription(...)`就是设置远端sdp信息，开始进行sdp协商了。



紧接对它的调用会分发到work线程中，在`SetRemoteContent_w(...)`中处理，如下堆栈。

<!-- 这是一张图片，ocr 内容为：PSSECRRSCTON DCNTESESEBREWEBT- BSSTHREST IRPLKLARESTHREST BSK-BRIBDS ST.STSTHRESDEC3157 -RUNLL 53 PARTABERES ACTEGNG HASD ANPERSE.PRASE.PROST 683 SOLF FFROPUN FRANESANOUF PONT MON MON TOJENMEAD PERGBRGDLA ANLERTERTHEADINND            PESERREDTON DENTESHKELHENL HEBAID 'PDITI ES 252 -->
![](https://github.com/mo4772/mo4772.github.io/raw/master/pic/fec/fec_3_code_2.png)



最终会在`WebRtcVideoChannel::GetChangedSendParameters(...)`确定协商成功的媒体格式。

在其中会调用`SelectSendVideoCodecs(...)`函数，它返回的codec就是协商成功的媒体格式。

```cpp
std::vector<VideoCodecSettings> negotiated_codecs =
      SelectSendVideoCodecs(MapCodecs(params.codecs));
```



`negotiated_codecs`包含多个codec，选取第一个作为最终确定的媒体格式，如下代码：

```cpp
if (negotiated_codecs_ != negotiated_codecs) {
    if (negotiated_codecs.empty()) {
      changed_params->send_codec = absl::nullopt;
    } else if (send_codec_ != negotiated_codecs.front()) {
      changed_params->send_codec = negotiated_codecs.front();
    }
    changed_params->negotiated_codecs = std::move(negotiated_codecs);
}
```



#### fec相关的结构体的定义
`SelectSendVideoCodecs(...)`的声明，如下：

```javascript
std::vector<WebRtcVideoChannel::VideoCodecSettings>
WebRtcVideoChannel::SelectSendVideoCodecs(
    const std::vector<VideoCodecSettings>& remote_mapped_codecs)
```

`VideoCodecSettings`就是对应sdp中媒体格式描述的结构体，如下：

```cpp
struct VideoCodecSettings {
    VideoCodecSettings();

    // Checks if all members of |*this| are equal to the corresponding members
    // of |other|.
    bool operator==(const VideoCodecSettings& other) const;
    bool operator!=(const VideoCodecSettings& other) const;

    // Checks if all members of |a|, except |flexfec_payload_type|, are equal
    // to the corresponding members of |b|.
    static bool EqualsDisregardingFlexfec(const VideoCodecSettings& a,
                                          const VideoCodecSettings& b);

    VideoCodec codec;
    webrtc::UlpfecConfig ulpfec;
    int flexfec_payload_type;  // -1 if absent.
    int rtx_payload_type;      // -1 if absent.
  };
```

**关联了ulpfec，rtcx，codec。**

****

看结构体`UlpfecConfig`，如下：

```cpp
struct UlpfecConfig {
  UlpfecConfig()
      : ulpfec_payload_type(-1),
        red_payload_type(-1),
        red_rtx_payload_type(-1) {}
  std::string ToString() const;
  bool operator==(const UlpfecConfig& other) const;

  // Payload type used for ULPFEC packets.
  int ulpfec_payload_type;

  // Payload type used for RED packets.
  int red_payload_type;

  // RTX payload type for RED payload.
  int red_rtx_payload_type;
};
```

就是定义payload type和封装它的Red包的payload type。



## H264默认不会开启ulpFec
在跟踪代码时，发现这样一句输出日志：

> [002:598][7716] (rtp_video_sender.cc:106): Transmitting payload type without picture ID using NACK+ULPFEC is a waste of bandwidth since ULPFEC packets also have to be retransmitted. Disabling ULPFEC.
>

这次协商出的codec是H264，但是这个句打印提示将 ulpfec给关掉了(这里的意思是H264下nack与ulpfec不能共用，[nack与ulpfec](https://blog.csdn.net/dong_beijing/article/details/83934638))。对VP8和VP9没这句提示。



nack和ulpfec同时设置开启时，对H264会关闭ulpfec。在`ShouldDisableRedAndUlpfec`方法中的判断，如下：

```cpp
if (nack_enabled && IsUlpfecEnabled() &&
      !PayloadTypeSupportsSkippingFecPackets(rtp_config.payload_name)) {
     RTC_LOG(LS_WARNING)
        << "Transmitting payload type without picture ID using "
           "NACK+ULPFEC is a waste of bandwidth since ULPFEC packets "
           "also have to be retransmitte. Disabling ULPFEC.";
    should_disable_red_and_ulpfec = true;
}
```

可以关闭nack，单独开启ulpfec。



`PayloadTypeSupportsSkippingFecPackets`

```cpp
bool PayloadTypeSupportsSkippingFecPackets(const std::string& payload_name,
                                           const WebRtcKeyValueConfig& trials) {
  const VideoCodecType codecType = PayloadStringToCodecType(payload_name);
  if (codecType == kVideoCodecVP8 || codecType == kVideoCodecVP9) {
    return true;
  }
  if (codecType == kVideoCodecGeneric &&
      absl::StartsWith(trials.Lookup("WebRTC-GenericPictureId"), "Enabled")) {
    return true;
  }
  return false;
}
```









