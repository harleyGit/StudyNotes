>#视频播放类的思维导图
![Video 思维导图](https://upload-images.jianshu.io/upload_images/2959789-34af7f3c3f4d6817.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<br/>
***
<br/>

>#AVPlayer
AVAsset：一个用于获取多媒体信息的抽象类，但不能直接使用
AVURLAsset：AVAsset的子类，可以根据一个URL路径创建一个包含媒体信息的AVURLAsset对象
AVPlayerItem：一个媒体资源管理对象，用于管理视频的基本信息和状态，一个AVPlayerItem对应一个视频资源
AVPlayer：负责视频播放、暂停、时间控制等操作
AVPlayerLayer：负责显示视频的图层，如果不设置此属性，视频就只有声音没有图像

