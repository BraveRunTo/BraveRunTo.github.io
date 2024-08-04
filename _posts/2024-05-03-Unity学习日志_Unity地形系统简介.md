---
title: Unity学习日志_Unity地形系统简介
author: BraveRunTo
date: 2024-05-03 +0800
categories: [地形系统]
tags: [unity]
---
# Unity学习日志_Unity地形系统简介

## 地形创建：



## Terrain中的组件：

### Transform

### Terrain

1. ![1](https://github.com/BraveRunTo/picx-images-hosting/raw/master/unity_dixing/1.41xytmo2xl.webp)
2. 四大功能：
   1. paint Terrain，可以选择下面几种具体的模式：
      1. ![2](https://github.com/BraveRunTo/picx-images-hosting/raw/master/unity_dixing/2.361he6eehl.webp)
      2. Create Neighbor Terrains：创建临近地形。临近地形拥有和源地形相同的基础纹理。
      3. Raise or Lower Terrain：隆起或者下凹地形，其中下凹地形需要使用shift+左键。
      4. Paint Texture：绘制地形纹理，单个纹理会作用于整个地形，多个纹理则可使用地形刷选择绘制。
      5. Set Height：设置地形的高度，有高度了才可以绘制下凹地形。
      6. Smooth Height：平滑地形，去除隆起地形时产生的尖角。
      7. Stamp Terrain：创建尖角。
   2. paint Trees
      1. ![3](https://github.com/BraveRunTo/picx-images-hosting/raw/master/unity_dixing/3.esf63safu.webp)
      2. 首先需要有树的模型，点击Edit Trees可以添加。
      3. Mass Place Trees：在该地形中一次性随机种指定数量的树。
      4. Settings：
         1. Brush Size：地形刷大小。
         2. Tree Density：地形刷的种树密度。
         3. Tree Height：设置树高，和树高是否随机。
         4. Lock Width to Height：根据树高锁定树宽。
         5. Color Variation：设置色差。
   3. paint Details
      1. ![4](https://github.com/BraveRunTo/picx-images-hosting/raw/master/unity_dixing/4.64drhomnyz.webp)
      2. 本功能与paint trees同理。主要用于绘制地形表面的植被，如草和花。
   4. Terrain Settings
3. Brushes（地形刷）：
   1. 最先引入眼帘的是笔刷样式，右下角为添加新的笔刷样式。
      1. ![5](https://github.com/BraveRunTo/picx-images-hosting/raw/master/unity_dixing/5.54xo4ijwt7.webp)
   2. Brush Size：笔刷大小。
   3. Opacity：不透明度。决定一次涂抹的比例。

### Terrain Collider

1. ![6](https://github.com/BraveRunTo/picx-images-hosting/raw/master/unity_dixing/6.8ojlubmml7.webp)

### Terrain Settings

#### 1. Basic Terrain(基础地形)：

1. Draw：是否开启地形绘制；如果关闭则不在渲染地形。
2. Pixel Error：可以理解为游戏中的渲染距离，数值越大则同一距离下的图形越简陋，不准确。
3. Base Map Dist:数值距离内的贴图纹理为全分辨率，距离之外的则会合成低分辨率的纹理。
   1. 全分辨率：
      1. ![7](https://github.com/BraveRunTo/picx-images-hosting/raw/master/unity_dixing/7.1vyk7uwf6n.webp)
   2. 低分辨率：
      1. ![8](https://github.com/BraveRunTo/picx-images-hosting/raw/master/unity_dixing/8.4qr8dnblya.webp)
4. Cast Shadow：是否投射地形阴影。
5. Material：设置渲染地形的材质，分为标准，漫反射，镜面，自定义四种。
6. Reflection Probes：如果启动并且反射探头出现在场景中，反射纹理会从这个游戏对象和构建着色器设置的变量中获得。
7. Thickness：设置地形厚度，有助于防止高速移动的物体穿透地面。

#### 2. Tree&Detail Object(树与细节对象):

	1. Draw：是否绘制树和细节对象。
	2. Bake Light Probes For Trees：如果启用，Unity将为每棵树创建内部光照探针（不影响场景中的其他渲染器），并应用于树的渲染照明。
	3. Detail Distance：细节对象的可视渲染距离。
	4. Detail Density：细节对象的密度。
	5. Tree Distance：树的可视渲染距离。
	6. Billboard Start：树木被替换成广告牌时的距离。
	  	1. 变成了广告牌：
	       	1. ![9](https://github.com/BraveRunTo/picx-images-hosting/raw/master/unity_dixing/9.7ljwjfqspu.webp)
	 	2. 没有变成广告牌：
	     	1. ![10](https://github.com/BraveRunTo/picx-images-hosting/raw/master/unity_dixing/10.45hkrch5nn.webp)
	7. Max Mesh Trees：可见三维网格树的最大数量，超过这个数量树将由广告牌替代。

#### 3. Wind Settings for Grass(风对于草地的设置)：

	1. Speed：风吹草的速度。
	2. Size：风吹过草地时的涟漪大小。
	3. Bending：草被吹过之后的弯曲程度。
	4. Grass Tint：草的基色。

#### 4. Resolution(分辨率)：

	1. Terrain Width：地形高度。
	2. Terrain Length：地形长度。
	3. Terrain Height：地形高度。

## 水和雾的效果设置：

1. 水：
   1. 通常会将水的材质放在plane物体上，然会将plane安放在合适的地方。
2. 雾：
   1. 在Lighting视图中的Other Setting中勾选fog选项，并可以在此面板中进行具体的设置。