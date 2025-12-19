---
layout: post
title: FFmpeg 常用命令行
slug: ffmpeg cmd
categories: [FFmpeg]
tags: [FFmpeg]
---
## ffmpeg

### Metrics

#### VMAF

```bash
ffmpeg -f rawvideo -s 1280x720 -pix_fmt yuv420p -i ref.yuv -f rawvideo -s 1280x720 -pix_fmt yuv420p -i dis.yuv -filter_complex "libvmaf='model=version=vmaf_v0.6.1\\:name=vmaf|version=vmaf_v0.6.1neg\\:name=vmaf_neg':n_threads=8" -f null -
```



## ffprobe

### 基本信息

```bash
ffprobe  -hide_banner input.mp4

ffprobe -v error -select_streams v:0 -show_entries stream input.mp4

ffprobe -v error -select_streams v:0 -show_entries stream=codec_name,profile,level,pix_fmt,width,height input.mp4
```

### 帧数

```bash
# 直接读nb_frames
ffprobe -v error -select_streams v:0 -show_entries stream=nb_frames -of csv=p=0 input.mp4

# 如果没有nb_frames，使用count_packets统计
ffprobe -v error -select_streams v:0 -count_packets -show_entries stream=nb_read_packets -of csv=p=0 input.mp4

```
