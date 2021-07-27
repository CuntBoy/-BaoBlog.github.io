---
title: OSG踩坑记-默认光源
date: 2021-04-04 00:10:50
categories: 
- works
- GF
- OSG
tags:
- cpp
- osg
mathjax: true
---

### OSG-Viewer默认光源

```C++
osg::Node->getOrCreateStateSet()->setMode(GL_RESCALE_NORMAL, osg::StateAttribute::ON);    // 法线随着模型大小变化而变化。
osg::Node->getOrCreateStateSet()->setMode(GL_LIGHTING, osg::StateAttribute::OFF | osg::StateAttribute::OVERRIDE); // 关闭节点光源 并遵从父节点的光照设置
```

---

##### `不指定法向` -- <font color=red>仍然可以看到 光照效果 但是无法看到明暗变化 </font> 

### Viewer 默认光源的使用

#### 1、设置光照的相关的信息 

- `osg::Viewer默认存在缺省光源,但是需要进行部分设置,不然可能与你当前的场景不匹配`

- 你需要设置的信息包括: ambient、diffuse、specular、光照方向（可以不设置，使用缺省的）、光照位置、光的衰减参数。

> 设置光照  -- <font color=Green>衰减公式</font>
> $$
> F = \frac{1}{(k_c + k_l*d + k_q*d^2)}
> $$
>

```C++ 
	// 1. 获取缺省的光照
	osg::Light *light = viewer->getLight(); // 从当前的查看器 获取光照设置 
	light->setAmbient(osg::Vec4(0.4, 0.4, 0.4, 1.0));   // 环境光   OPENGL的默认值是 osg::Vec3(0.2,0.2,0.2);
	light->setDiffuse(osg::Vec4(0.5, 0.5, 0.5, 1.0));   // 漫反射   直接来源于光源
	light->setSpecular(osg::Vec4(0.2, 0.2, 0.2, 1.0));  // specular   高光 此处给的很低  
	// 由于此处我们使用的是平行光
	osg::Vec4 lightpos(x, y, z, 0.0f);   // 记住: 平行光的位置的最后一个分量必须是 0 
	light->setPosition(lightpos);        // 设置光照位置 
	// 光照方向设置
	// 需要用到高度角和方位角 -- 计算一个你需要方向 
	

	// 设置光的衰减 -- 只设置 常量衰减的话 --   无距离衰减
	light->setConstantAttenuation(1.0f);
	light->setLinearAttenuation(0.0f);
	light->setQuadraticAttenuation(0.f);
```

<font color=red> 产生平行光，位置分量的第四分量必须是0 </font>

#### 2、对节点设置好材料属性

- 材料的设置是针对你要显示的节点设置的、你可以创建一个材料并设置相关的颜色，这样就可以拥有一个更好的显示效果。