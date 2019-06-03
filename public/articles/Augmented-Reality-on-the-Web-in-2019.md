# 2019 年增强现实在 web 端的应用

作者： James Milner  
时间： May 21, 2019 7:08 am
地址：`https://www.sitepen.com/blog/augmented-reality-on-the-web-in-2019/`

![图图图](https://github.com/kaisa911/studyNotes/blob/master/public/image/Graphic.jpg?raw=true)

增强现实（AR）带来数字信息或媒体，并将其与我们对现实世界的体验交织在一起。近年来，增强现实在消费领域以两种主要形式变得明显：诸如 [Microsoft HoloLens](https://www.microsoft.com/en-us/hololens?SilentAuth=1&wa=wsignin1.0) 和 [Magic Leap](https://www.magicleap.com/) 等头戴式显示器以及移动设备上更广泛的可用体验。应用程序通常会抓住设备的摄像头，然后将数字工件强加到设备的视口中。流行的基于移动设备的增强现实应用程序的一些示例是`Pokemon Go`和`Snapchat Lens`。两者都为用户提供了应用程序，使数字世界与有形世界融为一体。

原生移动应用程序已经开始看到一些主流的 AR 成功，但 Web 上的支持已经不太受欢迎。传统上，Web 没有提供本机增强现实功能，并且工作一直专注于基于标记的 AR 与客户端库的方法。基于标记的跟踪允许通过识别场景中的特定图案（标记）来定向和放置对象。许多已建立的 JavaScript 库采用这种方法来实现 AR。

最近有一些基于 Web 的增强现实方法的牵引力，这些方法源于 2016 年本机虚拟现实（VR）功能的变化。向 WebVR 的转变始于[w3c 的报告](https://www.w3.org/2016/06/vr-workshop/report.html)，该报告随后将在 2017 年的 Firefox 和 Chrome 等浏览器中实现。

将工作融入 WebVR 和 WebAR 的举措与 [Immersive Web Community Group](https://www.w3.org/community/immersive-web/) 达成共识，现在正在开发一种名为 [WebXR Device API](https://www.w3.org/TR/webxr/) 的新 API。借助 Web 上增强现实的简史，让我们在 2019 年探索开发人员的当前状态。
