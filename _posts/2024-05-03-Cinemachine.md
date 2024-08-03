---
title: Cinemachine
author: BraveRunTo
date: 2024-05-03 +0800
categories: [Cinemachine]
tags: [unity]
---
## Cinemachine

参考文章：

https://zhuanlan.zhihu.com/p/516625841

https://docs.unity.cn/Packages/com.unity.cinemachine@2.9/manual/index.html

## 快速使用Cinemachine系统

Cinemachine是用于处理Unity相机相关逻辑的系统。Cinemachine为开发人员解决了在不同相机视角之间组合，混合，裁切，追踪目标的逻辑和数学运算。

### 虚拟相机

Cinemachine系统不会创建新的相机，而是指挥单个相机的属性在多个虚拟相机（CinemachineVirtualCamera）的视角之间进行切换。虚拟相机和相机之间是相互独立的，二者之间没有必须嵌套在一起的规则，例如对于一个存在虚拟相机的场景很有可能长得像下图所示：

![image-20240225113153615](https://github.com/BraveRunTo/picx-images-hosting/raw/master/image-20240225113153615.5fkhwkofib.webp)

Cinemachine中虚拟相机的主要任务为：

- 控制挂载CinemachineBrain的游戏对象在场景中的位置信息。当挂载CinemachineBrain的游戏对象和虚拟相机位置不一致时，Cinemachine会将挂载CinemachineBrain的游戏对象缓动到虚拟相机的位置
- 控制挂载CinemachineBrain的游戏对象聚焦的方位。
- 为挂载CinemachineBrain的游戏对象添加噪声，噪声可以为挂载CinemachineBrain的游戏对象添加横向和纵向的抖动，使视角更接近现实

所以只要我们将CinemachineBrain组件挂载在Unity相机的游戏对象上，我们就可以实现利用虚拟相机控制Unity相机视角，位置的功能。

在Cinemachine中，虚拟相机的开销并不算大，通常我们可以选择在一个场景中放置大量的虚拟相机。但建议同一时期只打开必要的虚拟相机。

官方建议是使用一个虚拟相机代表游戏中的一个具体的Unity相机视角。举个例子来说，如果现在有一个虚拟相机代表一个楼梯过道的监控的视角，那我们不应该使用同一个虚拟相机代表另一个房间的监控视角，而是应该为房间的视角单独再实例化一个虚拟相机进行控制。把虚拟相机想象为电影拍摄中不同角度的机位来进行处理。

通常情况下，同一个时间段只能有一个虚拟相机Live并对Unity相机具有控制权，但也有例外情况，比如当逻辑要切换当前的虚拟相机时，且发生混合时（表现是Unity相机从A虚拟相机的位置缓动到B虚拟相机的位置），这个时候就会同时存在两个虚拟相机都Live的情况。

### CinemachineBrain

CinemachineBrain在Cinemachine系统中承担着管理器的功能，CinemachineBrain驱动当前场景中的所有激活状态的虚拟相机。并将虚拟相机的属性实时映射在自身游戏对象的对应属性上。

你可以通过激活/失活虚拟相机的游戏对象来达到切换虚拟相机视角的功能，CinemachineBrain组件将选择最近被激活的优先级更高或者相同的虚拟相机作为下一个视角。在两个虚拟相机视角之间的切换过渡，可以在CinemachineBrain组件上进行调整。

### 移动和旋转虚拟相机

我们可以通过虚拟相机上的Body属性来指定当前虚拟相机如何在场景中移动，可以通过Aim属性指定当前虚拟相机如何在场景中旋转。

一个虚拟相机拥有两个目标，指定虚拟相机跟随游戏对象的Follow属性和指定虚拟相机注视游戏对象的Look At属性。

Cinemachine系统包含了一系列用于控制移动和注视的算法。每一个算法都用于解决一个特定的问题，并且通常都带有自定义参数。Cinemachine系统通过CinemachineComponent来实现这些算法。开发人员可以通过实现自己的CinemachineComponent来自定义虚拟相机的行为。

Body属性为开发人员提供了以下几种算法：

- **Transposer**: 将虚拟相机的坐标与跟随目标的坐标有固定的偏移量来进行跟随
- **Do Nothing**: 不移动虚拟相机
- **Framing Transposer**: 和Follow Target保持一个固定的屏幕偏移进行移动
- **Orbital Transposer**: 沿着一个围绕Follow Target的环形轨道进行移动
- **Tracked Dolly**: 沿着预定义的路线进行移动
- **Hard Lock to Target**: 和Follow属性的游戏对象使用相同的位置信息

Aim属性为开发人员提供了以下几种算法：

- **Composer**: 将单个跟随目标框定在相机视角内
- **Group Composer**: 将多个跟随目标框定在相机视角内
- **Do Nothing**: 不旋转虚拟相机
- **POV**: 基于用户输入旋转虚拟相机
- **Same As Follow Target**: 和Follow属性的游戏对象使用相同的旋转
- **Hard Look At**: 将Look At属性的游戏对象始终保持在相机视角的中心

### 虚拟相机的区域构图

在使用Framing Transposer, Composer, 和Group Composer时，自定义属性中有一些关于相机构图的概念：

- **Dead zone**: Cinemachine系统会保持目标在Dead Zone中
- **Soft zone**: 如果目标进入了Soft Zone，则Cinemachine会根据Damping的相关设置将目标拉回Dead Zone
- **Screen**: ScreenX，ScreenY属性用于设置Dead Zone的中心相对屏幕的位置，0.5为屏幕中心
- **Damping**: 模拟真实物理相机在操作时的滞后性。该值越小，则相机滞后性越小，即相机跟随目标跟的更紧一些。目标越难进入Soft Zone。否则相反。

注意，只有当Brain组件直接挂载在Camera对象上时，构图辅助才会出现。

### 通过噪声模拟相机抖动

现实世界的物理摄像机通常很笨重。它们由摄像师手持或安装在不稳定的物体上，如移动的车辆。使用Noise属性来模拟现实世界的电影效果。例如，你可以在跟随奔跑的角色时添加摄像机抖动，让玩家沉浸在行动中。

在每一帧更新时，Cinemachine添加与摄像机运动无关的噪声来跟踪目标。噪声不会影响相机在未来帧中的位置。这种分离确保了阻尼等属性的行为符合预期。

## 虚拟相机

在场景中为一个空游戏对象添加CinemachineVirtualCamera组件，这样我们就可以得到一个虚拟相机。通过虚拟相机的Aim，Body，Noise属性设置来告知虚拟相机如何改变其位移，旋转和其他属性。虚拟相机会将这些设置应用到CinemachineBrain的挂载对象上。

虚拟相机可能存在以下三种状态：

- **Live**: 通常情况下，同一个时间段只能有一个虚拟相机处于Live状态并对CinemachineBrain游戏对象具有控制权，但也有例外情况，比如当逻辑要切换当前的虚拟相机时，且发生混合时（表现是CinemachineBrain游戏对象从A虚拟相机的位置缓动到B虚拟相机的位置），这个时候就会同时存在两个虚拟相机都处于Live状态的情况。
- **Standby**: 激活态的虚拟相机，该状态下的虚拟相机不会控制带有CinemachineBrain游戏对象的各种信息，但是其依旧会正常更新位置，旋转等属性。
- **Disabled**: 此状态下的虚拟相机不会控制带有CinemachineBrain游戏对象的各种信息，也不会更新自身的属性。

### 虚拟相机属性

![image-20240225111253998](https://github.com/BraveRunTo/picx-images-hosting/raw/master/image-20240225111253998.2krtqs98qq.webp)

- Solo：该选项可以临时让一个虚拟相机处于Live状态，主要用于调试时适配虚拟相机的位置
- Game Window Guides：
- Save During Play：勾选后可以在运行时序列化属性调整，这样就不需要我们记住属性改动再回到编辑器模式进行调整了
- Priority：当前虚拟相机的优先级，优先级越大，则越容易被选为Live状态
- Follow：当前虚拟相机需要跟随的游戏对象，Body属性会使用此Target的位置信息，如果此项为空，则默认挂载CinemachineBrain的游戏对象会使用当前虚拟相机的位置。
- Look At：当前虚拟相机需要聚焦的游戏对象，Aim属性会使用此Target的旋转信息，如果此项为空，则默认挂载CinemachineBrain的游戏对象会使用当前虚拟相机的旋转信息。
- Standby Update：控制处于激活态，但是不是Live态的虚拟相机的属性更新频率
- Lens：其中包含一部分Unity相机的属性
- Inherit Position：当勾选此项，不论当前虚拟相机是否处于Live状态，都会强制该虚拟相机和挂载CinemachineBrain的游戏对象使用相同位置。
- Extensions：额外拓展组件

### Cinemachine场景工具

//todo

### Body属性

Body属性主要用于控制虚拟相机的位置。

#### DoNothing

选中此选项则Cinemachine不会移动该虚拟相机

#### 3rdPersonFollow

使用该模式，Cinemachine会以一种固定的位置和距离跟随Follow Target。

![image-20240117191515930](https://github.com/BraveRunTo/picx-images-hosting/raw/master/image-20240117191515930.8dws02wozm.webp)

此项会将挂载CinemachineBrain的游戏对象与FollowTarget保持一定的距离，并追踪目标的移动和旋转。Cinemachine通过4个点来确定这个距离：

- origin（A）：origin是Follow Target的位置，当Follow Target进行水平旋转时，虚拟相机会环绕origin进行水平旋转。
- shoulder（B）：该项相对于origin进行设置，会为相机视角增加一个Shoulder Offset属性向量偏移，以便可以完成自定义的相机位置。垂直旋转会围绕shoulder点完成。
- hand（C）：该项相对于shoulder点进行设置。hand的位置相对于shoulder点位加上了一个Vertical Arm Length的y轴偏移。当相机垂直旋转时，臂长影响跟随目标的屏幕位置。对于第一人称相机，这个值可以设置为0。
- camera（D）：该项相对于hand进行设置，该点的旋转永远平行于hand点，但会与hand点有一个camera distance的z轴距离差。也是最终的相机位置。

注意3rdPersonFollow只是Cinemachine提供的跟随方式，不涉及相机旋转相关的逻辑。建议不要尝试通过改变Vertical Arm Length和Shoulder Offset来实现相机围绕对象旋转。

属性参数：

- Damping：相机在各个方向移动时的阻尼
- Shoulder Offset：用于调整shoulder点位的偏移量
- Vertical Arm Length：用于调整hand点位的偏移量
- Camera Side：为shoulder点添加一个x轴的偏移，用于表示左肩视角或者右肩视角
- Camera Distance：相机距离hand点的距离
- Camera Collision Filter：指定那些层级是需要内置相机碰撞解决方案考虑的
- Ignore Tag：内置相机碰撞解决方案会忽视的Tag，一般建议Follow Target的tag要写在这里
- Camera Radius：指定在不调整其位置的情况下，摄像机可以接近可碰撞障碍物的距离

##### 控制相机

该模式下如果想要控制相机则需要自定义脚本进行控制。

##### 内置相机碰撞解决方案

虚拟相机在3rdPersonFollow选项下会有默认的碰撞解决方案来保证相机不会被阻挡视线，通过改变`Camera Collision Filter`属性来选择不同的预设碰撞解决方案。

#### FramingTransposer

该选项提供一个算法来保证相机与Follow Target之间保持一个固定的屏幕空间下的距离关系。一般来说该模式是为2d游戏或者正交相机设计的，但透视相机和3d环境也同样可以使用。

属性参数：

- Track Object Offset：追踪目标相对于当前Follow Target的偏移量
- Lookahead Time：根据跟随目标的运动情况调整虚拟摄像机与跟随目标的偏移量。Cinemachine估计了目标在未来几秒后的位置。这个特性对噪声动画很敏感，可以放大噪声，导致不理想的相机抖动。如果目标在运动时相机抖动过大，请调低此属性，或者使目标动画更平滑。
- Lookahead Smoothing：
- Lookahead Ignore Y：如果勾选则预测不会考虑Y轴的移动
- X/Y/Z Damping：相机运动的阻尼
- Target Movement Only：如果勾选，则相机的阻尼只对移动跟随有效，对相机旋转无效
- Screen X/Y：调整游戏物体在屏幕中的相对位置
- Camera Distance：相机距离跟随点的距离
- Dead Zone Width/Height/Depth：设置死域范围，对应死域的X，Y，Z轴
- Unlimited Soft Zone：如果勾选则SoftZone的范围是无限的
- Soft Zone Width/Height：设置Soft Zone的范围
- Bias X/Y：Moves the target position horizontally/vertically away from the center of the soft zone.

一般情况下，使用FramingTransposer + POV的组合就可以达到不错的相机表现。基本可以用于绝大多数的视角移动需求。

#### HardLockToTarget

虚拟相机使用和Follow Target一致的位置信息

#### OrbitalTransposer

该模式下Unity会将相机与Follow Target保持一个可变的距离关系，该模式提供了可选的用户输入，这可以帮助用户动态控制相机相对于Follow Target的位置关系。

该模式引入了`heading`的概念。`heading`指的是与当前Follow Target面向或者移动相同的方向。该模式在跟随Follow Target的同时会将相机的位置保持在这样一个位置：从Follow Target连向相机位置的向量的方位等于`heading`。

如果我们为该模式添加了输入控制器，则我们还可以控制相机在一个既定的环形轨道上进行运动。这也是该模式被称为Orbital Transposer的原因之一。

在制作车辆或者交通工具的跟随视角时可以选择该模式。

属性参数：

- Binding Mode：解释参考官方文档https://docs.unity.cn/Packages/com.unity.cinemachine@2.9/manual/CinemachineBindingModes.html
- Follow Offset：相机跟随Follow Target时的偏移量
- X/Y/Z Damping：相机在X，Y，Z轴移动时的阻力系数
- Pitch Damping：相机在X轴旋转时的阻力系数
- Roll Damping：相机在Z轴旋转时的阻力系数
- Yaw Damping：虚拟相机在Y轴旋转时的阻力系数
- Heading：
  - Definition：指定如何计算跟随目标的heading方向
    - Positon Delta：根据上次更新和当前帧的目标位置差来计算heading方向
    - Velocity：以目标身上的刚体组件的速度为标准计算heading方向
    - Target Forward：目标本地的forwar的方向
    - World Forward：世界空间的forward方向
    - Velocity Filter Strength：控制速度的平滑度，在Positon Delta和Velocity模式中可以进行设置
  - Bias：相对于heading方向的偏移量
- Recenter To Target Heading：当不存在玩家输入时是否将相机调整回heading方向
- X Axis：heading控制，此处设置玩家输入控制相关，其中的Input Axis Name需要与Input Manager中的虚拟轴名称一致，或者也可以通过自定义脚本控制Input Axis Value来实现输入控制的需求，具体如何操作请看下面章节：`替换输入系统`

#### TrackedDolly

这个算法通过引入Dolli Path资源来保证按照既定的轨迹来运行虚拟相机。Dolly Path是一系列路点的集合。为一个空游戏对象挂载CinemachinePath或者CinemachineSoomthPath以编辑轨道。

![image-20240303101840654](https://github.com/BraveRunTo/picx-images-hosting/raw/master/image-20240303101840654.6ik77gk9dt.webp)

- Resolution：轨迹点与点之间的采样数量，数量越高，移动约圆滑
- Appearance：如何在Scene视口内显示辅助Icon
- Looped：轨迹是否收尾相连
- Path Details：定义的路点集合

#### Transposer

这个算法将虚拟相机的坐标与跟随目标的坐标有固定的偏移量来进行跟随，也可以使用Damping属性。简单来说就是虚拟相机跟目标会有固定的位置差偏移。

参数基本与Orbital Transposer相同。这里不重复说了。

### Aim属性

Aim属性主要用于控制虚拟相机的方位。

#### DoNothing

该模式下Cnimachine不会选转该虚拟相机。

#### Composer

该算法用来控制镜头瞄准目标以面向目标坐标，最常用的算法，通常用来做跟随人物

#### GroupComposer

参数基本和Composer模式保持一致，但当Look At的目标是一个Cinemachine Target Group时，虚拟相机会自动调整相机的参数以确保所有目标可以被正确的框定在相机视角中。

Cinemachine Target Group组件可以理解为其永远都处于所有包含对象的轴心点位置以便GroupComposer可以正确将所有对象框定。

#### HardLookAt

该模式下，类似于为相机调用了transform.lookat方法，相机会始终注视Look At对象。

#### POV

在该模式下，Cinemachine允许用户自定义控制相机方位。可以使用内置集成的Input Manager。也可以自定义脚本控制。自定义脚本控制时和OrbitalTransposer一样，需要脚本控制Input Axis Value的值。

#### SameAsFollowTarget

该模式下，相机的方位会和当前Look At的对象保持一致。

### Noise属性

其本质就是通过噪音来实现虚拟相机的模拟抖动。Cinemachine 包含一个**基本的多通道 Perlin**组件，该组件将 Perlin 噪声添加到虚拟摄像机的运动中。**Perlin 噪声**是一种计算具有自然行为的随机运动的技术。

### 虚拟相机混合

除了在Cinemachine上直接设置默认的虚拟相机之间的过渡方式，还可以通过Custom Blend自定义每两个虚拟相机之间的过渡方式。使用Custom Blend需要新建或者使用现成的配置资源。该资源通过Brain组件上的Create Asset进行创建

![CinemachineRigSceneView](https://github.com/BraveRunTo/picx-images-hosting/raw/master/CinemachineRigSceneView.9rjb447r0g.webp)

### Manager Camera

Cinemachine中存在一种特殊的虚拟相机，官方称为Manager Camera。可以简单的理解为是多个简单虚拟相机组合成的复杂虚拟相机。在日常使用中，我们可以将Manager Camera与虚拟相机混合在一起进行使用。官方提供了以下几种Manager Camera：

- Free Look Camera
- Mixing Camera
- Blend List Camera
- Clear Shot Camera
- State-Driven Camera

#### Free Look Camera

通过挂载CinemachineFreeLook组件来获得该相机功能。该组件是提供第三人称摄像机体验的 Cinemachine 摄像机。相机围绕其对象运行，三个独立的相机装备定义了围绕目标的环。每个装备都有自己的半径、高度偏移、合成器和镜头设置。根据相机沿连接这三个装备的样条线的位置，对这些设置进行插值以给出最终的相机位置和状态。

其参数与虚拟相机一致，只是将3个虚拟相机封装为一个，由Top，Middle，Bottom三个部分操控。都可以选择自己的Soft Zone ，Dead Zone等各种参数数值。其主要特点是3个子空间的body都是选择OrbitalTransposer算法跟随 ，其Aim默认选择是Composer，可以选择其他观察模式。

#### Mixing Camera

通过使用CinemachineMixingCamera组件来获得该相机的功能。该组件会通过其自虚拟相机的加权平均值来计算Unity相机的相机属性。该组件最多支持8个子虚拟相机的属性混合计算。

#### Blend List Camera

通过使用CinemachineBlendListCamera组件来获得该相机功能。该组件可以理解为简化版的Timeline，只可以编辑虚拟相机。下图中的红框负责编辑虚拟相机的播放序列，可以设置停留时间和混合方式。默认如果不勾选Loop则Blend List Camera最后会一直保持最后一个虚拟相机的播放，所以最后一个虚拟相机不能选择Hold时间。Blend List Camera不能动态增加播放序列。

红框下面的列表Virtual Camera Children则负责生成Blend List Camera需要控制的子虚拟相机，注意子虚拟相机的名称需要有区分。

![CinemachineVCamProperties](https://github.com/BraveRunTo/picx-images-hosting/raw/master/CinemachineVCamProperties.6t710lzhj3.webp)

#### Clear Shot Camera

通过挂载CinemachineClearShot组件来获得该相机功能。当挂载该组件时，Unity还会为游戏对象挂载一个名为Cinemachine Collider的组件。带有 Cinemachine Collider 扩展的虚拟摄像机子项分析场景中的目标障碍物、最佳目标距离等。 Clear Shot 使用此信息来选择要激活的最佳子虚拟相机。 和Blend List Camera一样，我们需要手动在Virtual Camera Chidren中添加子虚拟相机才可以。

提示：要为所有虚拟摄像机子项使用单个 Cinemachine Collider，请将 Cinemachine Collider 扩展添加到 ClearShot GameObject 而不是其每个虚拟摄像机子项。 这个 Cinemachine Collider 扩展适用于所有孩子，就好像他们每个人都有那个 Collider 作为自己的扩展。 

注意：如果多个子摄像机具有相同的拍摄质量，则 Clear Shot 摄像机选择具有最高优先级的摄像机。

使用此相机可以实现类似于GTA的电影模式视角。

#### State-Driven Camera

通过挂载CinemachineStateDrivenCamera来获得该相机功能。该组件可以为每一个具体的动画状态机状态指定一个虚拟相机。核心参数和Blend List Camera类似，一个列表指定状态机状态和虚拟相机的对应关系，另一个列表用于指定StateDrivenCamera的子虚拟相机。由于动画状态机是图形结构，所以对于虚拟相机之间的切换功能是通过虚拟相机混合资源实现的。

![CinemachineSceneHierarchy](https://github.com/BraveRunTo/picx-images-hosting/raw/master/CinemachineSceneHierarchy.4g4ejeloce.webp)

### 虚拟相机拓展

扩展是增强虚拟摄像机行为的组件。 例如，Collider 扩展将相机移出阻碍相机对其目标视野的游戏对象。Cinemachine 包括各种扩展。 通过从 CinemachineExtension 类派生来创建您自己的自定义扩展。

拓展不是常规的Unity组件，需要通过虚拟相机的Add Extension来添加。

#### Cinemachine Collider

- Collide Against：这里设置的层级中的对象会被纳入遮挡考虑
- Ignore Tag：带有这些标签的对象不会被纳入遮挡考虑
- Transparent Layers：这里设置的层级中的对象永远都不会纳入遮挡考虑
- Minimum Distance From Target：忽略距离目标距离小于该值的障碍物
- Avoid Obstacles：勾选该项以保证Cinemachine可以正确的防止相机被遮挡
- Distance Limit：使用射线检测相机是否被遮挡的距离限制，如果遮挡物与相机的距离大于该限制，则认为没有被遮挡。如果为0则使用当前相机和观察对象的距离。一般使用0就可以
- Minimum Occlusion Time：除非遮挡物遮挡的时间超过了这里的值，否则不会进行对相机进行移动
- Camera Radius：可以理解为相机的边界
- Strategy：防止相机被遮挡的策略
  - Pull Camera Forward：向前推相机直到不被遮挡
  - Preserve Camera Height：防止被遮挡的同时保留相机的高度不变
  - Perserve Camera Distance：防止被遮挡的同时保留相机与目标的距离不变

- Maximum Effort：按照官方说法此项一般不需要更改
- Smoothing Time：如果相机被遮挡，则相机需要停留在原地的时间，适用于遮挡物较多的情况，设置该值可以有效的缓解由于遮挡物过多导致的相机平凡抖动问题
- Damping：相机回到默认位置时的阻尼
- Damping When Occluded：相机被遮挡时移动的阻尼

#### Cinemachine Confiner

该拓展可以限定相机的移动范围，我们可以为该拓展引用一个2D或者3D碰撞体，这样相机就只可以在当前碰撞体的范围内进行移动。

#### CinemachineFollowZoom

当目标前进或者远离相机时，该组件会通过调整相机的FOV来保证目标可以被框定在视野内

#### CinemachineStoryBoardr

该组件是一个开发用组件，主要用于进行场景编辑或者UI编辑时的预览图使用。

#### CinemachineImpulseListener

使用该组件可以让虚拟相机获得监听脉冲源的能力。想要使用该组件我们还需要在场景中添加最少一个脉冲源。

1. 首先需要为场景中的最少一个游戏对象添加CinemachineImpusleSource或者CinemachineCollisionImpusleSource组件
2. 为希望监听的虚拟相机添加CinemachineImpulseListener组件。

相机在监听到脉冲之后会表现为相机震动。

默认情况下该组件可以监听到所有的脉冲源，可以通过channel fitler字段来决定该组件过滤那些频道的脉冲信号。

#### CinemachinePostProcessing

使虚拟相机可以使用后处理V2的预设文件

#### CInemachineCameraOffset

Cinemachine 虚拟摄像机的附加模块，可为摄像机添加最终偏移量，可以选择为那个参数添加该位移加到哪一个参数（Body，Aim，Noise，Finalize）

#### CinemachineRecomposer

Cinemachine 虚拟摄像机的附加模块，可对摄像机组成进行最终调整。 它旨在用于Timeline上下文，您希望在其中手动调整程序或记录的相机瞄准的输出。

#### Cinemachine3rdPersonAim

Cinemachine 虚拟摄像机的附加模块，强制 LookAt 指向屏幕中心，消除噪音和其他校正。 这对于需要始终精确瞄准的第三人称风格瞄准相机非常有用，即使存在位置或旋转噪声也是如此。

### 替换输入系统

一些Cinemachine的组件比如Freelook，或者虚拟相机的POV模式都需要接受用户输入。默认Cinemachine使用InputManager接受用户输入。但Cinemachine同样支持开发人员使用自己的用户输入。通过让一个`MonoBehaviour`脚本实现`Cinemachine.AxisState.IInputAxisProvider`接口并将其挂载到虚拟相机上以实现输入替换的功能。

### 多相机使用虚拟相机

[Multiple Unity cameras | Cinemachine | 2.9.7](https://docs.unity.cn/Packages/com.unity.cinemachine@2.9/manual/CinemachineMultipleCameras.html)

## Cinemachine Brain

CinemachineBrain组件是一个你需要挂载在unity相机上或者相机对象的父级的组件。该组件驱动整个场景中所有的虚拟相机，虚拟相机的值最终会同步到CinemachineBrain组件的ControlledObject上。如果我们选择将CinemachineBrain和Unity相机对象分开，则我们需要手动改变ControlledObject对象，这样才能让相机的参数得到正确的修改。

## Timeline与Cinemachine

Cinemachine很容易和Timeline结合使用，比直接使用相机动画制作镜头更容易。使用Timeline可以激活、停用、混合虚拟相机。Timeline可以将Cinemachine、场景中的GameObject和其他资产组合在一起，以可视化方式创建、调整出丰富的过场动画，甚至是交互式的过场动画。 

提示：对于简单的镜头序列，可以不使用Timeline，直接使用Cinemachine中的Blend List Camera。 

使用Timeline时，Timeline会控制哪个相机激活，覆盖Cinemachine Brain基于虚拟相机优先级的决策。当Timeline播放完后，控制权返还给Cinemachine Brain，它会激活优先级最高的虚拟相机。 

你可以在Timeline中创建Cinemachine Track和使用Cinemachine Shot Clip控制虚拟相机。每个镜头片段都指向一个虚拟相机，Timeline播放时将其激活，播放后再将其禁用。使用一系列镜头片段来指定每个镜头的顺序和持续时间。 

想要两个虚拟相机之间直接切换，可以将clip相邻放置。想要两个虚拟相机之间混合过渡，可以重叠片段。 

## 应用实例

### 黑魂Boss锁定

在锁定模式中，通常会存在两个角色：玩家和Boss，首先我们需要将玩家和Boss放在一个TargetGroup中并让虚拟相机Look At该Target Group，模式选择Compser或者Group Composer，这样可以保证玩家和boss一定位于相机视野内。相机的跟随应当跟随玩家，且推荐使用Framing Transposer。

官方示例：`Assets/Samples/Cinemachine/2.9.7/Cinemachine Example Scenes/Scenes/BossCamera`

### GTA电影模式

想要实现类似于GTA总的电影模式，可以使用Cinemachine中的Clear Shot组件来实现。

官方示例：`Assets/Samples/Cinemachine/2.9.7/Cinemachine Example Scenes/Scenes/ClearShot`

### 炸弹落地效果

炸弹落地或者巨物落地时，相机应当产生抖动。可使用CinemachineImpulse相关的内容。

官方示例：`Assets/Samples/Cinemachine/2.9.7/Cinemachine Example Scenes/Scenes/Impulse`

### 第三人称射击

瞄准时使用AimingRig的解决方案，一般移动使用3rdPersonWithAimMode中的解决方案。且应该需要通过StateDrivenCamera来切换相机。

官方实例：`Assets/Samples/Cinemachine/2.9.7/Cinemachine Example Scenes/Scenes/3rdPersonWithAimMode`

官方实例：`Assets/Samples/Cinemachine/2.9.7/Cinemachine Example Scenes/Scenes/AimingRig`

### 第一次人称传送门

官方示例：`Assets/Samples/Cinemachine/2.9.7/Cinemachine Example Scenes/Scenes/Anywhere Door`

### 相机受阻FadeOut

官方示例：`Assets/Samples/Cinemachine/2.9.7/Cinemachine Example Scenes/Scenes/FadeOutNearbyObjects`

### 相机视角在特定范围内吸附

简单来说想要实现该功能，也需要使用TargetGroup，但是需要我们根据接近吸附物品的距离调整各个物品的权重。这样就可以有一个特定的视角吸附效果了。在2D游戏中比较常见。

官方示例：`Assets/Samples/Cinemachine/2.9.7/Cinemachine Example Scenes/Scenes/CameraMagnets`

### 交通工具驾驶视角

官方示例：`Assets/Samples/Cinemachine/2.9.7/Cinemachine Example Scenes/Scenes/FollowCam.unity`





