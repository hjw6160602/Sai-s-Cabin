---
title: ARKit简介
date: 2017-07-15 15:31:40
---

![](https://saidicaprio.oss-cn-shanghai.aliyuncs.com/imgs/ARKitLogo.png) 

ARKit是WWDC发布iOS11系统所新增框架。ARKit是一个移动AR平台，提供高层次的API，可在iOS 11 + A9芯片及以上（iPhone 6s起）上使用。


Session 602 Introducing ARKit: Augmented Reality for iOS

https://developer.apple.com/videos/play/wwdc2017/602/

## 注意点：
* ARKit框架提供了两种AR技术，一种是基于3D场景(`SceneKit`)实现的增强现实，一种是基于2D场景(`SpriktKit`)实现的增强现实
* 一般主流都是基于3D实现AR技术，ARKit不仅支持3D引擎`SceneKit`，还支持2D引擎`SpriktKit`
* 要想显示AR效果，必须要依赖于苹果的游戏引擎框架，主要原因是游戏引擎才可以加载物体模型

ARKit需要的开发环境：
* 1.Xcode版本：Xcode9及以上
* 2.iOS系统:iOS11及以上
* 3.iOS设备：处理器A9及以上（6S机型及以上）
* 4.MacOS系统：10.12.4及以上（安装Xcode9对Mac系统版本有要求）

## 一、ARKit简介
### ARKit如何工作？

![](https://saidicaprio.oss-cn-shanghai.aliyuncs.com/imgs/ARKit框架结构.png) 

#### Step1. Tracking 追踪
* World tracking 世界跟踪
* 视觉惯性测距
* 无需外部设置 十分方便！

#### Step2. Scene Understanding 场景理解
* 平面检测(屋子的地板或桌子等大平面)
* Hit-testing 碰撞测试 
* 光线评估，根据环境光线确定虚拟物体的渲染

#### Step3. Rendering 渲染
* Easy integration
* 渲染生成 AR views
* 自定义渲染

#### 下图是一个<ARKit>与<SceneKit>框架关系图，通过下图可以看出

`ARKit`框架中中显示AR视图的`ARSCNView`继承于`SceneKit`框架中的`SCNView`，而`SCNView`又继承于`UIView`
`SCNView`的作用是显示一个3D场景，`ARScnView`的作用也是显示一个3D场景，只不过这个3D场景是由摄像头捕捉到的现实世界图像构成的

在一个完整的虚拟增强现实体验中，`ARKit`框架只负责将真实世界画面转变为一个3D场景，这一个转变的过程主要分为两个环节：
* 由`ARCamera`负责捕捉摄像头画面
* 由ARSession负责搭建3D场景

![](https://saidicaprio.oss-cn-shanghai.aliyuncs.com/imgs/ARKit框架流程图.png) 

> 综上所述，ARKit捕捉3D现实世界使用的是自身的功能，这个功能是在iOS11新增的。而ARKit在3D现实场景中添加虚拟物体使用的是父类SCNView的功能，这个功能早在iOS8时就已经添加（SceneKit是iOS8新增）

## 二、Tracking 追踪

![](https://saidicaprio.oss-cn-shanghai.aliyuncs.com/imgs/ARKitCapturing.png) 


### 2.1 影响追踪质量的因素
* 不能间断的传感器数据
* 特征显著的周边环境
* 静态场景

### 2.2 如何追踪？

##### `ARSession`的参与者主要有两个`ARWorldTrackingSessionConfiguration`与`ARFrame`

###### `ARWorldTrackingSessionConfiguration`（会话追踪配置）
跟踪设备的方向和位置,以及检测设备摄像头看到的现实世界的表面。它的内部实现了一系列非常庞大的算法计算以及调用了你的iPhone必要的传感器来检测手机的移动及旋转甚至是翻滚。

当`ARWorldTrackingSessionConfiguration`计算出相机在3D世界中的位置时，它本身并不持有这个位置数据，而是将其计算出的位置数据交给`ARSession`去管理，而相机的位置数据对应的类就是`ARFrame`



`ARCamera`只负责捕捉图像，不参与数据的处理。它属于3D场景中的一个环节，每一个3D Scene都会有一个Camera

它们三者之间的关系看起来如下图：

![](https://saidicaprio.oss-cn-shanghai.aliyuncs.com/imgs/ARKitSession关系图.png)

ARCamera在3D世界的位置看起来是这样的:

![](https://saidicaprio.oss-cn-shanghai.aliyuncs.com/imgs/ARKitCamera.png)


### 2.2 追踪状态？
#### 共有四种状态，在受限时，最好通知用户
![](https://saidicaprio.oss-cn-shanghai.aliyuncs.com/imgs/跟踪状态.png)
```swift
func session(_ session: ARSession, cameraDidChangeTrackingState camera: ARCamera) { 
	if case .limited(let reason) = camera.trackingState {
		// Notify user of limited tracking state
	} 
}
```

Session中断 或者 相机不可用，就追踪停止
```swift
func sessionWasInterrupted(_ session: ARSession) { 
	showOverlay()
}

func sessionInterruptionEnded(_ session: ARSession) { 
	hideOverlay()
	// Optionally restart experience
	...
}
```
## 三、Scene Understanding 场景理解

### 3.1 平面检测

![](https://saidicaprio.oss-cn-shanghai.aliyuncs.com/imgs/ARKitSceneUnderstanding.png) 

* 相对于重力的水平面
* 跨多个帧运行
* 表面对齐
* 平面合并

### 3.2 Hit-testing 碰撞测试
* 现实世界的光线交叉
* 使用场景信息
* 距离排序的结果
* 碰撞测试的类型
  * 有范围的存在平面
  * 存在平面
  * 预估平面
  * 特征点
  
![](https://saidicaprio.oss-cn-shanghai.aliyuncs.com/imgs/ARKitHit-Testing.png) 

根据类型的不同，碰撞测试的运作方法会不一样

### 3.3 光线评估
* 基于捕获图像的环境强度
* 默认为1000流明（光束的能量单位）
* 默认启用

## 四、 Rendering 渲染 
在一个完整的虚拟增强现实体验中，将虚拟物体现实在3D场景中是由`SceneKit`框架来完成中：每一个虚拟的物体都是一个节点SCNNode,每一个节点构成了一个场景SCNScene,无数个场景构成了3D世界。

### 4.2 自定义 Rendering 渲染
* 绘制摄像头北京
* 更新虚拟摄像头
* 更新光线
* 更新重力场下的transforms

## 五、总结
### ARKit工作流程
ARKit与SceneKit的配合：通过ARSCNView，绘制捕捉的图像，更新SCNCamera，更新光照，将SCNNode映射到ARAnchor上

![](https://saidicaprio.oss-cn-shanghai.aliyuncs.com/imgs/ARKit工作流程.png) 

* 1.ARSCNView加载场景SCNScene
* 2.SCNScene启动相机ARCamera开始捕捉场景
* 3.捕捉场景后ARSCNView开始将场景数据交给Session
* 4.Session通过管理ARSessionConfiguration实现场景的追踪并且返回一个ARFrame
* 5.给ARSCNView的scene添加一个子节点（3D物体模型）

`ARSessionConfiguration`捕捉相机3D位置的意义就在于: 能够在添加3D物体模型的时候计算出3D物体模型相对于相机的真实的矩阵位置(在3D坐标系统中，有一个世界坐标系和一个本地坐标系)
