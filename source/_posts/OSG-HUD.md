---
title: OSG-HUD
date: 2021-07-26 22:40:11
categories:
 - works
 - GF
tags:
- cpp
- osg
---

# OSG-HUD

- <p>"HUD"在渲染的场景中算是比较常见的存在: 游戏中的小地图(上帝视角)、实时状态、显示鼠标的实时位置、三维视角...</p>

<b style="color:red">如何创建HUD ？</b> 

<p>本质上说，HUD就是一个相机( 在`OSG`中可以作为节点)，只是不响应鼠标的操作。只需要设置好相关的参数，添加到场景的根节点就行</p>

<b style="color:red">实现要点，如下：</b>

- 关闭节点的光照，保证整个视口显示的场景的亮度是一样的。
- 关闭深度测试
- 调整渲染的顺序为最后渲染，让`HUD`显示在整个场景的最前方
- 设置参考帧为绝对参考帧
- 设置变换矩阵,不受父节点的影响
- 设置投影矩阵(平行投影或者透视),设置视口的大小。



## Create HUD

### 1、创建相机节点

```C++
osg::ref_ptr<osg::Camera> hud_camera = new osg::Camera;   // 创建相机的节点
osg::ref_ptr<osg::Geometry> geom = new osg::Geometry;      // 创建绘制的节点(也可以是文字)
```

### 2、设置投影矩阵

```C++
hud_camera->setProjectionMatrixAsOrtho2D(-3, 4, -3, 4, -1.0, 100);  // 2D 也可以 此处设置表示你裁剪的三维空间的实际的能表示的数值的大小
```

### 3、设置视口大小(ViewPort)

```C++
// void setViewport(osg::Viewport* viewport);
// void setViewport(int x,int y,int width,int height);
hud_camera->setviewport();  // 两种方式都是可以的 
```

### 4、设置渲染的顺序

```C++
hud_camera->setRenderOrder(osg::Camera::POST_RENDER);
```

### 5、设置参考帧

```C++
hud_camera->setReferenceFrame(osg::Transform::ABSOLUTE_RF);
```

### 6、设置不受父节点的影响

```C++
hud_camera->setViewMatrix(osg::Matrix::identity());
```

### 7、设置不获取焦点

```C++
hud_camera->setAllowEventFocus(false);
```

### 8、设置节点的更新回调

```C++
// void setUpdateCallback(Callback* nc);
// 需要重写一个节点的更新回调 修改节点的视图矩阵 
// 设置相机的三个参数  
// 视点 相机的位置 相机的向上方向
hud_camera->setUpdateCallback();
```

## 挂载节点

### 1、将前面创建的几何节点挂载到 相机上

```C++
osg::ref_ptr<osg::Geode> node = new osg::Geode;
node->addChild(geom.get());
hud_camera->addChild(node.get());
```

### 2、设置节点的属性

- 关闭光照 深度测试 & 打开混溶

```C++
auto states = node->getOrCreateStateSet();
states->setMode(GL_LIGHTING, osg::StateAttribute::ON);   //关闭灯光
states->setMode(GL_DEPTH_TEST, osg::StateAttribute::OFF);//关闭深度测试
states->setMode(GL_BLEND, osg::StateAttribute::ON);  
```



