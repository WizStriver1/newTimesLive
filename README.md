# obs+node-media-server+flv.js实现录播和直播

## 实现思路

1. 下载obs软件，进行视频的录制
2. 通过node-media-server开启一个服务，在obs中推流到该服务器
3. 通过flv.js配合html5的video标签实现node-media-server中视频源的播放

## 开始实现

### obs的使用

首先在来源窗口创建一个屏幕捕捉/窗口/摄像头，实现一个创建。

视频的本质其实是一张张截下来的图片，我们需要将这一张张图片放到一个地方，然后前端就可以从这个地方读取，从而展示出来，因此在这之前我们需要开启一个服务，作为前端获取视频的源地址。

### node-media-server开启服务

新建一个空白的文件夹，执行**npm init**, 根据提示输入相关信息后，下载node-media-server
```
npm install node-media-server --save
```

新建一个入口文件index.js
```
const NodeMediaServer = require('node-media-server');

const config = {
  rtmp: {
    port: 1935,
    chunk_size: 60000,
    gop_cache: true,
    ping: 60,
    ping_timeout: 30
  },
  http: {
    port: 8000,
    allow_origin: '*'
  }
};

var nms = new NodeMediaServer(config)
nms.run();
```
然后在命令行中执行
```
node index.js
```

如果看到下面的提示，表示我们已经成功开启node-media-server服务了
```
5/3/2020 17:17:30 13172 [INFO] Node Media Server v2.1.8
5/3/2020 17:17:31 13172 [INFO] Node Media Rtmp Server started on port: 1935
5/3/2020 17:17:31 13172 [INFO] Node Media Http Server started on port: 8000
5/3/2020 17:17:31 13172 [INFO] Node Media WebSocket Server started on port: 8000
```

flv.js

flv.js是来自Bilibli的开源项目。它解析FLV文件喂给原生HTML5 Video标签播放音视频数据，使浏览器在不借助Flash的情况下播放FLV成为可能。具体的介绍请自行google哈，继续刚才的项目

新建一个index.html文件
```
<!DOCTYPE html>
<html>
<head>
   <meta charset="UTF-8">
   <title>直播</title>
</head>
<body>
   <script src="https://cdn.bootcss.com/flv.js/1.4.0/flv.min.js"></script>
   <video id="videoElement" width="100%" controls></video>
   <script>
       if (flvjs.isSupported()) {
           var videoElement = document.getElementById('videoElement');
           var flvPlayer = flvjs.createPlayer({
               type: 'flv',
               url: 'http://localhost:8000/live/hello.flv'
           });
           flvPlayer.attachMediaElement(videoElement);
           flvPlayer.load();
           flvPlayer.play();
       }
   </script>
</body>
</html>
```
### 推流设置

点击obs中的设置，进入设置页面，点击推流，如果是在本地直播的话，流类型选择自定义流媒体服务器，url填写和流名称（或者叫串流密钥）填写index.html设置的内容，本项目的串流密钥是hello。
如果通过bilibili等直播平台进行播放，这里就填写你bilibili上的直播链接和名称。
```
Settings -> Stream

Stream Type : Custom Streaming Server

URL : rtmp://localhost/live

Stream key : STREAM_NAME
```
注意上面url的构成： rtmp://hostname/pathname，最后不要加/，名称为hello，不要写hello.flv，url只填hostname就行了，不要加port。


点击obs的开始推流按钮
> 这里遇到一个坑，点击开始推流，按钮直接变成一个空白按钮，正常情况应该显示“停止推流”，导致花了很多时间去怀疑url或者服务器有问题，其实就是obs的问题，多重启几次，重启前到任务管理器把程序全部退出。

这时双击在浏览器打开index.html就可以看到直播啦，记得点击视频下方的开始按钮~