# 关于GStreamer插件信息
## GStreamer官网
- [GStreamer官网](https://gstreamer.freedesktop.org/)

## 应用手册
- [应用手册](https://gstreamer.freedesktop.org/documentation/index.html)

## 一、GStreamer安装(Ubuntu)
注意区分`gstreamer0.10`和`gstreamer1.0`两个版本。

```bash
sudo add-apt-repository ppa:mc3man/trusty-media
sudo apt-get update
sudo apt-get install build-essential dpkg-dev flex bison autotools-dev automake liborc-dev autopoint libtool gtk-doc-tools

sudo apt-get install libgstreamer0.10-0 libgstreamer0.10-dev gstreamer0.10-tools gstreamer0.10-plugins-base libgstreamer-plugins-base0.10-dev gstreamer0.10-plugins-good gstreamer0.10-plugins-ugly gstreamer0.10-plugins-bad gstreamer0.10-ffmpeg

sudo apt-get install libgstreamer0.10-dev gstreamer-tools gstreamer0.10-tools gstreamer0.10-doc
sudo apt-get install gstreamer0.10-plugins-base gstreamer0.10-plugins-good gstreamer0.10-plugins-ugly gstreamer0.10-plugins-bad gstreamer0.10-plugins-bad-multiverse
```

可能需要的额外插件和软件：

```bash
sudo apt-get install gstreamer0.10-tools gstreamer0.10-x gstreamer0.10-plugins-base gstreamer0.10-plugins-good gstreamer0.10-plugins-ugly gstreamer0.10-plugins-bad gstreamer0.10-ffmpeg gstreamer0.10-alsa gstreamer0.10-schroedinger gstreamer0.10-pulseaudio
sudo apt-get install bison flex zlib1g

# mad解码插件
apt-get install libmad0-dev
apt-get install gstreamer0.10-plugins-ugly

# 安装音频库
sudo apt-get install gstreamer0.10-alsa

# 安装ffmpeg多媒体库
gst-ffmpeg --enable-liba52 enable GPLed liba52 support [default=no]
--enable-liba52bin open liba52.so.0 at runtime [default=no]
--enable-libamr-nb enable libamr-nb floating point audio codec
--enable-libamr-wb enable libamr-wb floating point audio codec
--enable-libfaac enable FAAC support via libfaac [default=no]
--enable-libfaad enable FAAD support via libfaad [default=no]
--enable-libfaadbin open libfaad.so.0 at runtime [default=no]
--enable-libgsm enable GSM support via libgsm [default=no]
--enable-libmp3lame enable MP3 encoding via libmp3lame
--enable-libvorbis enable Vorbis encoding via libvorbis, native implementation exists [default=no]
--enable-libx264 enable H.264 encoding via x264 [default=no]
--enable-libxvid enable Xvid encoding via xvidcore, native MPEG-4/Xvid encoder exists [default=no]
```

## 二、入门教程
- [Basic tutorials](https://gstreamer.freedesktop.org/documentation/tutorials/basic/hello-world.html)
- [Wiki手册](http://wiki.oz9aec.net/index.php?title=Gstreamer_cheat_sheet)
- [IBM社区gstreamer教程](https://www.ibm.com/developerworks/cn/linux/l-gstreamer/)

## 三、进阶应用
1. **播放视频文件**
   - 硬解(vaapi)播放MP4文件：
     ```bash
     gst-launch-1.0 filesrc location=FilePath/test.mp4 ! qtdemux ! vaapidecode ! vaapisink
     ```
   - 软解：
     ```bash
     gst-launch-1.0 filesrc location=FilePath/test.mp4 ! qtdemux ! avdec_h264 ! videoconvert ! videoscale ! video/x-raw,width=800,height=600 ! ximagesink
     ```

2. **播放RTSP视频流**
   - 硬解：
     ```bash
     gst-launch-1.0 rtspsrc location=rtsp://username:passwd@ipaddr:port latency=0 ! rtph264depay ! capsfilter caps="video/x-h264" ! h264parse ! vaapidecode ! vaapipostproc width=800 height=600 ! vaapisink sync=false
     ```
   - 软解：
     ```bash
     gst-launch-1.0 rtspsrc location=rtsp://username:passwd@ipaddr:port latency=0 ! rtph264depay ! capsfilter caps="video/x-h264" ! h264parse ! avdec_h264 ! videoconvert ! videoscale ! video/x-raw,width=800,height=600 ! ximagesink
     ```

3. **播放Udp视频流**
   - 硬解：
     ```bash
     gst-launch-1.0 udpsrc port=2101 ! h264parse ! vaapidecode ! vaapisink
     ```
   - 软解：
     ```bash
     gst-launch-1.0 udpsrc port=2101 ! h264parse ! avdec_h264 ! autovideosink
     ```

4. **GStreamer RTSP推流/拉流**
   - [gstreamer rtsp拉流播放](https://blog.csdn.net/yang_quan_yang/article/details/78846134)
   - [gstereamer rtsp推流](https://blog.csdn.net/zhuwei622/article/details/80348916)

5. **RTPbin Network/RTP**
   - 发送端：
     ```bash
     gst-launch-1.0 rtpbin name=rtpbin v4l2src ! videoconvert ! ffenc_h263 ! rtph263ppay ! rtpbin.send_rtp_sink_0 rtpbin.send_rtp_src_0 ! udpsink port=5000 rtpbin.send_rtcp_src_0 ! udpsink port=5001 sync=false async=false udpsrc port=5005 ! rtpbin.recv_rtcp_sink_0 audiotestsrc ! amrnbenc ! rtpamrpay ! rtpbin.send_rtp_sink_1 rtpbin.send_rtp_src_1 ! udpsink port=5002 rtpbin.send_rtcp_src_1 ! udpsink port=5003 sync=false async=false udpsrc port=5007 ! rtpbin.recv_rtcp_sink_1
     ```
   - 接收端：
     ```bash
     gst-launch-1.0 -v rtpbin name=rtpbin udpsrc caps="application/x-rtp,media=(string)video,clock-rate=(int)90000,encoding-name=(string)H263-1998" port=5000 ! rtpbin.recv_rtp_sink_0 rtpbin. ! rtph263pdepay ! ffdec_h263 ! xvimagesink udpsrc port=5001 ! rtpbin.recv_rtcp_sink_0 rtpbin.send_rtcp_src_0 ! udpsink port=5005 sync=false async=false udpsrc caps="application/x-rtp,media=(string)audio,clock-rate=(int)8000,encoding-name=(string)AMR,encoding-params=(string)1,octet-align=(string)1" port=5002 ! rtpbin.recv_rtp_sink_1 rtpbin. ! rtpamrdepay ! amrnbdec ! alsasink udpsrc port=5003 ! rtpbin.recv_rtcp_sink_1 rtpbin.send_rtcp_src_1 ! udpsink port=5007 sync=false async=false
     ```

6. **录音**
   ```bash
   gst-launch -e pulsesrc ! audioconvert ! lamemp3enc target=1 bitrate=64 cbr=true ! filesink location=audio.mp3
   gst-launch -e pulsesrc device="alsa_input.pci-0000_02_02.0.analog-stereo" ! audioconvert ! lamemp3enc target=1 bitrate=64 cbr=true ! filesink location=audio.mp3
   ```

7. **录视频**
   ```bash
   gst-launch-1.0 -e rtspsrc location=rtsp://admin:admin@192.168.1.2 ! rtph264depay ! "video/x-h264, stream-format=byte-stream" ! filesink location=test.264
   ```

8. **视频收发(监控，预览)**
   - 发送端：
     ```bash
     gst-launch v4l2src ! video/x-raw-yuv,width=128,height=96,format='(fourcc)'UYVY ! ffmpegcolorspace ! ffenc_h263 ! video/x-h263 ! rtph263ppay pt=96 ! udpsink host=127.0.0.1 port=5000 sync=false
     ```
   - 接收端：
     ```bash