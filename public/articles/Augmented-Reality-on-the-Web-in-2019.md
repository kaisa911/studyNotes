# 2019 年增强现实在 web 端的应用

作者： James Milner  
时间： May 21, 2019 7:08 am  
地址：`https://www.sitepen.com/blog/augmented-reality-on-the-web-in-2019/`

![图图图](https://github.com/kaisa911/studyNotes/blob/master/public/image/Graphic.jpg?raw=true)

增强现实（AR）带来数字信息或媒体，并将其与我们对现实世界的体验交织在一起。近年来，增强现实在消费领域通过两种主要形式变成现实：像 [Microsoft HoloLens](https://www.microsoft.com/en-us/hololens?SilentAuth=1&wa=wsignin1.0) 和 [Magic Leap](https://www.magicleap.com/) 等头戴式显示器以及移动设备上更广泛的可用体验。应用程序通常使用设备的摄像头来获取信息，然后将数字产物显示到设备的显示屏中。流行的基于移动设备的增强现实应用程序的例如`Pokemon Go`和`Snapchat Lens`。两者都为用户提供了应用程序，使数字世界与有形世界融为一体。

原生移动应用程序已经开始看到一些主流的 AR 成功，但 Web 上的支持已经不太受欢迎。传统上，Web 没有提供本机增强现实功能，并且工作一直专注于基于标记的 AR 与客户端库的方法。基于标记的跟踪允许通过识别场景中的特定图案（标记）来定向和放置对象。许多已建立的 JavaScript 库采用这种方法来实现 AR。

最近有一些基于 Web 的增强现实方法的牵引力，这些方法源于 2016 年本机虚拟现实（VR）功能的变化。向 WebVR 的转变始于[w3c 的报告](https://www.w3.org/2016/06/vr-workshop/report.html)，该报告随后将在 2017 年的 Firefox 和 Chrome 等浏览器中实现。

将工作融入 WebVR 和 WebAR 的举措与 [Immersive Web Community Group](https://www.w3.org/community/immersive-web/) 达成共识，现在正在开发一种名为 [WebXR Device API](https://www.w3.org/TR/webxr/) 的新 API。借助 Web 上增强现实的简史，让我们在 2019 年探索开发人员的当前状态。

## 基于标记的 AR 与 JavaScript 库

其中一个最受欢迎的 AR 库是 ar.js，它在 GitHub 上积累了超过 12,000 颗星。在深入研究 ar.js 之前，重要的是要了解它对 ARToolKit 的依赖性，这是一个用 C / C ++编写的长期建立且流行的本地跨平台增强现实库。 ar.js 库包装了一个 emscripten 端口 artoolkitjs，它是 ar.js 的环境全局依赖项所必需的，必须先加载。 ARToolKit 没有被主动维护，但是 fork，artoolkitX 仍然保持活跃状态 ​​。因此，支持状态目前不直观或简单，但今天足以在浏览器中实现 AR。

ar.js 支持 three.js 和 A-Frame 作为增强现实渲染目标的目标。 Three.js 是一个完整的 JavaScript 3D 图形库，为在 WebGL 上构建 3D 场景提供了必要的 API。 A-Frame 是一个用于构建虚拟现实体验的框架，采用 HTML，自定义元素和 DOM 的声明式方法。下面是一个查找 Hiro 标记（默认标记类型）的示例：

```html
<!DOCTYPE html>
<html>
  <head>
    <style>
      body {
        margin: 0px;
        overflow: hidden;
      }
    </style>
  </head>

  <body>
    <script src="https://jeromeetienne.github.io/AR.js/aframe/examples/vendor/aframe/build/aframe.min.js"></script>
    <script src="https://jeromeetienne.github.io/AR.js/aframe/build/aframe-ar.js"></script>

    <script>
      ARjs.Context.baseURL = 'https://jeromeetienne.github.io/AR.js/three.js/';
    </script>

    <!-- Create a A-Frame Scene and enable tracking with ar.js -->
    <a-scene embedded arjs="trackingMethod: best;">
      <!-- Create an A-Frame anchor for our 3D models, detect for marker -->
      <a-anchor hit-testing-enabled="true">
        <!-- Provide a rotating Box and Torus Knot -->
        <a-box
          position="0 0.5 0"
          material="opacity: 0.5; side:double; color:red;"
        >
          <a-torus-knot radius="0.26" radius-tubular="0.05">
            <a-animation
              attribute="rotation"
              to="360 0 0"
              dur="5000"
              easing="linear"
              repeat="indefinite"
            ></a-animation>
          </a-torus-knot>
        </a-box>
      </a-anchor>

      <a-camera-static />
    </a-scene>
  </body>
</html>
```

ar.js 还提供了一个以类似方式处理 three.js 的接口。总的来说 ar.js 是一个很棒的图书馆，但它有几个引用的当前问题。例如，尽管 ar.js 在 npm 上，但它并未捆绑为模块，因此目前无法与 Webpack 或 Rollup 等模块捆绑器一起使用，因此必须将其放入脚本标记以用作全局。

THREE AR 是我创建的一个新库，它基于 ar.js 和 artoolkitjs 构建，以提供用 TypeScript 编写的现代库。使用 THREE AR，artoolkit 与 artoolkit 所需的相机参数二进制数据捆绑在一起。封装这两个项意味着三个 AR 除了 three.js 之外没有外部环境依赖性。三个 AR 也采用基于 Promise 的方法，而不是传递回调，允许更现代的编码样式（例如，您可以使用 async / await）。以下是使用简单模式标记创建基本屏幕的示例，在本例中为 Hiro 标记（默认标记）：

```js
import * as THREEAR from 'threear';
import * as THREE from 'three';

const renderer = new THREE.WebGLRenderer({
  antialias: true,
  alpha: true
});

renderer.setClearColor(new THREE.Color('lightgrey'), 0);
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.domElement.style.position = 'absolute';
renderer.domElement.style.top = '0px';
renderer.domElement.style.left = '0px';
document.body.appendChild(renderer.domElement);

// Initialise the three.js scene and camera
const scene = new THREE.Scene();
const camera = new THREE.Camera();
scene.add(camera);

const markerGroup = new THREE.Group();
scene.add(markerGroup);

var source = new THREEAR.Source({ renderer, camera });

THREEAR.initialize({ source: source }).then(controller => {
  // Add a torus knot
  const geometry = new THREE.TorusKnotGeometry(0.3, 0.1, 64, 16);
  const material = new THREE.MeshNormalMaterial();
  const torus = new THREE.Mesh(geometry, material);
  torus.position.y = 0.5;
  markerGroup.add(torus);

  var patternMarker = new THREEAR.PatternMarker({
    patternUrl: 'patt.hiro', // the URL of the hiro pattern
    markerObject: markerGroup,
    minConfidence: 0.4 // The confidence level before the marker should be shown
  });

  controller.trackMarker(patternMarker);

  // run the rendering loop
  let lastTimeMilliseconds = 0;
  requestAnimationFrame(function animate(nowMsec) {
    // keep looping
    requestAnimationFrame(animate);
    // measure time
    lastTimeMilliseconds = lastTimeMilliseconds || nowMsec - 1000 / 60;
    const deltaMillisconds = Math.min(200, nowMsec - lastTimeMilliseconds);
    lastTimeMilliseconds = nowMsec;

    // call each update function
    controller.update(source.domElement);

    torus.rotation.y += (deltaMillisconds / 1000) * Math.PI;
    torus.rotation.z += (deltaMillisconds / 1000) * Math.PI;
    renderer.render(scene, camera);
  });
});
```

最后，awe.js 是一个来自 awe-media 的库，它为 ar.js 提供了类似的功能。不幸的是，awe.js 在过去两年似乎没有更新。有趣的是，awe.js 也有一个与 ARToolKit 接口的例子，尽管使用了另一种形式，使用了 ARToolKit 的 ActionScript 端口的 JavaScript 端口（port-ception！）。 awe.js 支持的另一个功能是使用设备传感器定位对象的基于位置的标记。
