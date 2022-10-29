# YouSeeAR WebAR SDK

版本： 1.0.0 (首发公测版)

## 使用说明

本文假设您已学会threejs的基础使用，以下代码均以threejs库为例。

完整代码请查看 https://github.com/gentwolf-shen/youseear-threejs

### 1. 创建WebAR内容容器

所有由WebAR创建的资源均会添加到此容器下

```html
<div id="container"></div>
```

### 2. 引入 threejs 文件

```html
<script src="path/to/your/three.min.js"></script>
<!-- 本文以加载GLTF/GLB格式为例 -->
<script src="path/to/your/GLTFLoader.js"></script>    
```

### 3. 引入 YouSeeAR WebAR SDK 文件

***请不要修改“YouSeeAR.js”文件名***

```html
<script src="path/to/your/YouSeeAR.js"></script>
```

### 4. 创建 threejs 场景，相机等

```javascript
// threejs的使用请查看 https://threejs.org/
```

### 5. 创建 WebAR 根节点

```javascript
// 跟踪内容的根节点　
this.root = new THREE.Object3D();
this.root.matrixAutoUpdate = false;
// 默认不渲染，识别到目标后再渲染
this.root.visible = false;
// 添加到场景中
this.scene.add(this.root);

// anchor是跟随目标的节点，把需要跟随的模型添加到这个节点上就可以
this.anchor = new THREE.Group();
this.root.add(this.anchor);
```

### 6. 初始化 WebAR

```javascript
// container: WebAR内容节点，即第1步中的 container
this.youSeeAR = new YouSeeAR({ container: this.container });
```

### 7. 添加事件绑定

```javascript
// WebAR初始化完成事件
this.youSeeAR.on('Ready', (m) => {
    // 设置AR相机transform
    this.camera.projectionMatrix.elements = m;
    this.camera.updateProjectionMatrix();

    // 加载特征数据，特征数据的生成请访问 www.YouSeeAR.com
    this.youSeeAR.loadMarker('../../../features/feature.dat');
});

// 域名验证状态事件
this.youSeeAR.on('License', (status) => {
    console.info(`域名验证${status ? '' : '未'}通过`);
    if (!status) {
        // 验证未通过请暂停跟踪功能
        this.youSeeAR.pause();
    }
});

// 特征数据加载事件
this.youSeeAR.on('MarkerLoad', (markers) => {
    // markers为数组，当前版本仅支持一个跟踪目标
    // 设置anchor在容器中居中
    this.anchor.position.x = markers[0].x;
    this.anchor.position.y = markers[0].y;

    // 特征数据加载完成后，开始跟踪
    this.youSeeAR.start();
});

// 特征数据加载错误事件
this.youSeeAR.on('MarkerLoadError', (err) => {
    console.info(err);
});

// 识别到目标事件，当前版本仅支持一个跟踪目标
this.youSeeAR.on('TargetFound', (i) => {
    // 识别到目标后，设置为可视状态
    this.root.visible = true;
    console.info(`found ${i}`);
});

// 目标丢失事件
this.youSeeAR.on('TargetLost', (i) => {
    // 目标丢失后，设置为不可视状态
    // this.root.visible = false;
    console.info(`lost ${i}`);
});

// 跟踪事件，更新根节点的transform
this.youSeeAR.on('TargetTracking', (m) => {
    this.root.matrix.elements = m;
});
```

### 8. 打开相机
```javascript
// 打开相机，如果成功表示YouSeeAR的初始化完成
this.youSeeAR.camera().then(() => {
}).catch(err => {
    console.info(err);
});
```