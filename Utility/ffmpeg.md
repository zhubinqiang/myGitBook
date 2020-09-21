# ffmpeg 的使用

[TOC]

## FFmpeg 的使用格式
FFmpeg 的命令行参数非常多，可以分成五个部分。[^1]
```sh
ffmpeg {1} {2} -i {3} {4} {5}
```

五个部分的参数依次如下:
1. 全局参数
2. 输入文件参数
3. 输入文件
4. 输出文件参数
5. 输出文件

```sh
$ ffmpeg \
[全局参数] \
[输入文件参数] \
-i [输入文件] \
[输出文件参数] \
[输出文件]
```

ffmpeg参数中文详细解释 [^2]

## 转码
```bash
ffmpeg -i in.h264 out.m2v
ffmpeg -i in.m2v -c:v libx264 -c:a libfaac out2.h264
```

## 对比文件
```bash
ffmpeg -i input1.h264 -i input2.h264
```

## 提取音频文件
```bash
ffmpeg -i in.mp4 -vn -c:a copy out.mp3
```

## 截图
```bash
ffmpeg -i in.mp4 -r 4 images/%03d.png

ffmpeg -y -i in.mp4 -ss 00:00:24 -t 00:00:01 output_%3d.jpg
```

## 生成gif图片
```sh
ffmpeg -i in.mp4 -ss 4 -t 6 -f gif -s 320x240 -r 1 out.gif
```
-f: 格式 format
-s: 分辨率 size
-r: 帧率


## 剪切视频
```bash
ffmpeg -i in.flv -ss 4 -t 16 out.mp4
```

> 第4秒开始， 往后截取16秒

```sh
ffmpeg -i in.mp4 -ss 00:00:04 -t 6 ss.mp4
```

## 合并视频
```bash
printf "file '%s'\n" *.mp4 | ffmpeg -f concat -i - -c copy out.mp4
```

> 各个小文件需要相同的分辨率和编解码

## 拆分视频
```bash
ffmpeg -i in.mp4 -c copy -map 0 -f segment -segment_time 60 parts/part-%d.mp4
```

> segment_time 以秒为单位进行切片

```sh
ffmpeg -v verbose -i in.mp4 \
-f hls -c:v copy -c:a copy \
-hls_list_size 2 \
-hls_time 2 \
-hls_segment_filename '/tmp/ramdisk/camera/segment%01d.ts' \
-hls_flags delete_segments /tmp/ramdisk/camera/live.m3u8
```

参数如下[^3]：
-hls_time n: 设置每片的长度，默认值为2。单位为秒
-hls_list_size n:设置播放列表保存的最多条目，设置为0会保存有所片信息，默认值为5


## 字幕合成

字幕文件 in.srt
```
1
00:00:00,000  -->  00:00:03,000
大家好

2
00:00:05,000  -->  00:00:10,000
我们可以使用ffmpeg， 在视频中嵌入字幕

3
00:00:12,000  -->  00:00:20,000
有事请搜索: <font color="red">http://www.bing.com</font>
```

```bash
ffmpeg -i in.mp4 -i in.srt -c copy output.mkv
```

> 要支持文本字幕的视频格式, mkv是其中一种

## ffmpeg 在android下推流
```bash
ffmpeg -f scdev -framerate 30 -i /dev/fb0 \
-f rawvideo -pix_fmt nv12 -vcodec h264_qsv \
-preset veryfast -buffer_size 500k -g 250 \
-threads 6 -bf 0 -b:v 2000k -async_depth 0 \
-look_ahead 0 -f mpegts udp://ip:port
```

## 参考
[^1]: 阮一峰的网络日志: http://www.ruanyifeng.com/blog/2020/01/ffmpeg.html
[^2]: 雷霄骅 ffmpeg参数中文详细解释 https://blog.csdn.net/leixiaohua1020/article/details/12751349
[^3]: Lenix Blog https://blog.p2hp.com/archives/4824

