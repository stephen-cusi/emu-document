# GStreamer 使用指南

## 安装与依赖

### 添加PPA源

为确保安装最新版本的GStreamer及其相关组件，可以使用如下命令添加第三方PPA源：

```bash
sudo add-apt-repository ppa:mc3man/trusty-media
```

### 更新软件包列表

更新本地软件包索引：

```bash
sudo apt-get update
```

### 安装构建工具与基础库

安装编译GStreamer所需的构建工具链与基础库：

```bash
sudo apt-get install build-essential dpkg-dev flex bison autotools-dev automake liborc-dev autopoint libtool gtk-doc-tools
```

### 安装GStreamer核心及插件

安装GStreamer核心库、开发工具以及一系列官方提供的插件集合，包括基础插件、好用插件、丑陋插件、坏插件以及FFmpeg兼容插件：

```bash
sudo apt-get install libgstreamer0.10-0 libgstreamer0.10-dev gstreamer0.10-tools gstreamer0.10-plugins-base libgstreamer-plugins-base0.10-dev gstreamer0.10-plugins-good gstreamer0.10-plugins-ugly gstreamer0.10-plugins-bad gstreamer0.10-ffmpeg
```

### 安装额外插件

若需要更完整的功能支持，可以安装以下额外插件：

```bash
sudo apt-get install gstreamer0.10-plugins-base gstreamer0.10-plugins-good gstreamer0.10-plugins-ugly gstreamer0.10-plugins-bad gstreamer0.10-plugins-bad-multiverse
```

### 可能需要的软件包

针对特定需求，可能还需安装以下软件包：

```bash
sudo apt-get install bison
sudo apt-get install flex
sudo apt-get install zlib1g
```

## 视频与音频处理

### 视频收发监控

#### 发送端（Server）

使用`v4l2src`从摄像头获取视频源，进行格式转换和编码后，通过RTP协议发送至指定地址：

```bash
gst-launch v4l2src! video/x-raw-yuv,width=128,height=96,format='(fourcc)'UYVY! ffmpegcolorspace ! ffenc_h263! video/x-h263! rtph263ppay pt=96! udpsink host=127.0.0.1 port=5000 sync=false
```

#### 接收端（Client）

接收端通过UDP接收RTP数据包，解复用、解码后显示到X窗口：

```bash
gst-launch udpsrc port=5000! application/x-rtp, clock-rate=90000,payload=96! rtph263pdepay! ffdec_h263! xvimagesink
```

#### 实时码流收发（Freescale平台）

**Server**：

```bash
gst-launch-v videotestsrc! video/x-raw-yuv,width=640,height=480! vpuenc codec=avc! rtph264pay pt=96! udpsink host=127.0.0.1 port=1234
```

**Client**：

```bash
gst-launch-vvv udpsrc port=1234 caps="application/x-rtp, media=(string)video, clock-rate=(int)90000, encoding-name=(string)H264"! rtph264depay! vpudec! mfw_isink
```

### 音频收发对讲

#### 模拟声音数据

**发送端（send.sh）**：

```bash
gst-launch-1.0 rtpbin name=rtpbin audiotestsrc! amrnbenc! rtpamrpay! rtpbin.send_rtp_sink_1 rtpbin.send_rtp_src_1! udpsink port=5002 rtpbin.send_rtcp_src_1! udpsink port=5003 sync=false async=false udpsrc port=5007! rtpbin.recv_rtcp_sink_1
```

**接收端（recv.sh）**：

```bash
gst-launch-1.0-v rtpbin name=rtpbin udpsrc caps=�application/x-rtp,media=(string)audio,clock-rate=(int)8000,encoding-name=(string)AMR,encoding-params=( port=5002! rtpbin.recv_rtp_sink_1 rtpbin.! rtpamrdepay! amrnbdec! alsasink udpsrc port=5003! rtpbin.recv_rtcp_sink_1 rtpbin.send_rtcp_src_1! udpsink port=5007 sync=false async=false
```

#### 真实声卡

**发送端（send.sh）**：

```bash
gst-launch-1.0 rtpbin name=rtpbin pulsesrc! amrnbenc! rtpamrpay! rtpbin.send_rtp_sink_1 rtpbin.send_rtp_src_1! udpsink port=5002 rtpbin.send_rtcp_src_1! udpsink port=5003 sync=false async=false udpsrc port=5007! rtpbin.recv_rtcp_sink_1
```

**接收端（recv.sh）**：

```bash
gst-launch-1.0-v rtpbin name=rtpbin udpsrc caps=�application/x-rtp,media=(string)audio,clock-rate=(int)8000,encoding-name=(string)AMR,encoding-params=( port=5002! rtpbin.recv_rtp_sink_1 rtpbin.! rtpamrdepay! amrnbdec! alsasink udpsrc port=5003! rtpbin.recv_rtcp_sink_1 rtpbin.send_rtcp_src_1! udpsink port=5007 sync=false async=false
```

#### 中国移动和对讲实时语音解码播放

```bash
gst-launch-1.0 udpsrc port=6000 caps=�application/x-rtp, media=(string)audio, clock-rate=(int)8000, encoding-name=(string)
```

#### 远程播放

**发送端（send.sh）**：

```bash
gst-launch-1.0-v filesrc location=Hopy_Always.mp3! decodebin! audioconvert! rtpL16pay! udpsink host=127.0.0.1 port=6000
```

**接收端（recv.sh）**：

```bash
```


### 参考资料与教程

#### 博客文章

1. [基于GStreamer的实时视频流分发](https://blog.csdn.net/sdjhs/article/details/51444934)
2. [GStreamer学习笔记：音视频合成HLS流并打包通过HTTP传输](https://blog.csdn.net/u010312436/article/details/53668083)
3. [GStreamer一个进程提供多路不同视频](https://blog.csdn.net/quantum7/article/details/82999132)
4. [GStreamer资料整理：摄像头采集、视频保存、远程监控、流媒体传输](https://blog.csdn.net/wzwxiaozheng/article/details/6099397)
5. [使用GStreamer进行摄像头采集和输出到文件及屏幕的相关测试](https://blog.csdn.net/shallon_luo/article/details/5400708)
6. [GStreamer中添加编解码器](https://blog.csdn.net/songwater/article/details/34855883)

#### 示例脚本与命令

- [GStreamer命令行示例](https://metalab.at/wiki/Gstreamer_One_Liners)
- [ARM平台基于嵌入式Linux的Gstreamer使用](https://www.eefocus.com/toradex/blog/16-05/379143_e4fcb.html)
- [常见GStreamer命令](https://blog.csdn.net/songwater/article/details/34800017)

#### Gstreamer组件

- [Gstreamer中好用的appsink和appsrc](https://blog.csdn.net/jack0106/article/details/5909935)
- [基于DM3730平台的Gstreamer音视频传输调试](https://blog.csdn.net/goalietech/article/details/24887955)
- [Gstreamer向appsrc发送帧画面的代码](https://blog.csdn.net/quantum7/article/details/82226608)
- [Gstreamer向appsrc发送编码数据的代码](https://blog.csdn.net