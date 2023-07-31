## 1. 简单介绍什么是WebRTC？

WebRTC，网页实时通信（Web Real-Time Communication）,他是一个可以不借助中间媒介在浏览器之间进行点对点连接的实时通讯的技术。
在通信的过程中会提供一些JavaScript接口，这个接口所创立的信道并不是像WebSocket一样，打通一个浏览器与WebSocket服务器之间的通信，而是通过一系列的信令，建立一个浏览器与浏览器之间（peer-to-peer）的信道，这个信道可以发送任何数据，而不需要经过服务器。并且WebRTC通过实现MediaStream，通过浏览器调用设备的摄像头、话筒，使得浏览器之间可以传递音频和视频。
WebRTC的点对点通信并不意味着不涉及服务器，这只是意味着正常的数据没有经过它们。至少，两台客户机仍然需要一台服务器来交换一些基本信息（我在网络上的哪些位置，我支持哪些编解码器），以便他们可以建立对等的连接。用于建立对等连接的信息被称为信令，而服务器被称为信令服务器。

## 2. Webrtc音视频会议之Mesh/MCU/SFU三种架构

#### Mesh架构
Mesh架构流量或带宽要求比较大，Mesh架构是利用Webrtc对等连接，在参与会议的各方之间开辟UDP通道，也就是两两进行P2P连接，把媒体流发给参与会议的各方，同时从参与会议的其它方获取媒体流，如上图四个参与方，总共8个连接，如果每个通道占用1M带宽，那每个端需要把自己的流发给其它三个端，也就是上行是3M带宽，同时从其它3个端获取流，也就是下行3M带宽，这样每个端上下行总共6M带宽；

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200622151300130.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZyZWRlcmlja19GdW5n,size_16,color_FFFFFF,t_70#pic_center)

#### MCU架构(MultiPoint Control Unit)
 MCU架构对服务器端压力比较大，MCU架构需要一个中心化的MCU服务器，编码、转码、解码、SFU与MCU有相似之处，但SFU不会对原视频流做编解码，它会把流直接转发给用户，而且用户可以对多路视频流进行任意拖拽，MCU服务器对4个媒体流解码后进行合并，4个流合并成一个媒体流，再发给4个参会人员；因此服务器的压力特别大；所以单台服务器能承受的参会人员特别少，需要加服务器和高级的GPU。
MCU架构占用带宽最小 ，从前面的描述和从上图中我们可以看到4个参会人员每个人上交一份媒体流如果还是按照1M来算，那上行每个端1M，同时从服务器端获取一份混合过的媒体流还是按照1M算，那每个端上下行总共就是2M；结合上面所述MCU架构
一个端同时能承受更多的人开启视频

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200622143322516.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZyZWRlcmlja19GdW5n,size_16,color_FFFFFF,t_70#pic_center)

#### SFU架构(Selective Forwarding Unit)
SFU与MCU有相似之处，但SFU不会对原视频流做编解码，它会把流直接转发给用户，而且用户可以对多路视频流进行任意拖拽，比如C4用户接收到C1、C2、C3三路视频流，C4可以把C1放在左上角或右下角，或把C2放在正中间，这样便大大增加了客户端的灵活性

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200622144153430.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZyZWRlcmlja19GdW5n,size_16,color_FFFFFF,t_70#pic_center)

但是这样做也有弊端，因为SFU不会对原视频做压缩处理，这便对下行带宽有很高的要求，若没达到要求，将会对音视频的质量产生影响

正是如此，WebRTC针对这种情况也提供了很多种解决方案，下面介绍其中最有名的两种方案

##### simulcast
 该方案是通过分层的方式，可以设置成两层、三层或是四层甚至更高层次的分辨率，比如最高层是640*360的分辨率，下一层是240*120的分辨率，再一层是80*60的分辨率。总之就是按比例的缩放，再上传的时候将三层同时上传，下发的时候SFU会判断整个带宽能否承载下行的数据，如果不能承载便选择低一个层次的分辨率看能否承载，若不能承载，再选择更低层次的，依次下去…

![img](https://img-blog.csdnimg.cn/img_convert/46b15817fd732a44b4b8f33811ea2da4.jpeg)

##### SVC
 SVC也是采用分层，但与simulcast不同，后者是独立的层，而前者是相互依赖的。SVC是依赖于最底层（核心层）的，这一层是最小的数据流，它的分辨率也非常低，把它解开的时候画面也是很模糊的，每来一层就会让它变得更清晰，再来一层就会让它恢复为原来的分辨率，通过SVC，当带宽不够的时候就传核心层，带宽够的时候传扩展层，带宽非常好的时候传边缘层

![img](https://img-blog.csdnimg.cn/img_convert/0bd7eb4c77a84d600f3316eb19b037b1.jpeg)
