# Unity-Shader
程序员的三大浪漫是编译原理、操作系统、图形学
计算机图形学第一定律：如果它看起来是对的，那么他就是对的。

## 渲染流水线
### 渲染流程的三个阶段：应用阶段、几何阶段、光栅化阶段

应用阶段：开发者主导的阶段，输出渲染所需的结合信息即渲染图元。
几何阶段：与渲染图元打交道，重要任务是输出屏幕空间的二位顶点坐标、每个顶点对应的深度值、着色等相关信息。
光栅化阶段：决定每个渲染图元中的那些像素应该绘制在屏幕上。

几何和光栅化阶段在GPU上处理。

### CPU和GPU之间的通信
- (1)把数据加载到显存：从硬盘加载到RAM再到GPU
- (2)设置渲染状态：渲染的状态
- (3)调用Draw Call：CPU告诉GPU渲染那个

### GPU流水线

![text](http://q59qahcgi.bkt.clouddn.com/IMG_0061%2820200222-124110%29.PNG)
绿色：完全可编程、黄色：可配置但不可编程、蓝色：由GPU设定、
实线：Shader由开发者编程、虚线：Shader可选编程

顶点着色器：完全可编程，实现顶点的空间变换、顶点着色等。
曲面细分着色器：可选，用于细分图元。
几何着色器：可选，逐图元的着色操作。
裁剪
屏幕映射：不可配置和编程，把图元坐标转换到屏幕坐标中。

三角形设置：固定函数，计算边界像素的坐标信息
三角形遍历：固定函数，判断像素是否被三角网格覆盖，弱覆盖则生成片元
片元着色器：完全可编程，逐片元着色、纹理坐标
逐片元着色：不可编程，但可配置，修改颜色、深度缓冲、进行混合 

### Shader的特色
- GPU流水线上一些可高度编程的阶段，而由着着色器编译出来的最终代码是会在GPU上运行的
- 有一些特定类型的着色器，如顶点着色器、片元着色器等
- 依靠着色器我们可以控制流水线中的渲染细节，列入用顶点着色器来进行顶点变换以及传递数据，用片元着色器来进行逐项渲染


## Unity Shader 基础

```
Shader "ShaderName"{
    Properties{
        //属性
    }
    SubShader{
        //显卡A使用的子着色器

        //可选
        "Tags"

        //可选
        "RenderSetup"

        Pass{
            //定义一个完整流程
        }
    }
    SubShader{
        //显卡B使用的子着色器
    }
    Fallback "VertexLit"
    //留后路
}
```

### <center>常见的渲染状态设置选项</center>

状态名称|设置指令|解释
|----- | -----|------
Cull|Cull Back/Front/ Off|设置剔除模式：提出背面/正面/关闭
ZTest|ZTest Less Greater/ LEqual/GEqual/Equal/NotEqual/Always|设置深度检测时使用的函数
ZWrite|Zwrite On/ Off|开启/关闭深度
Blend|Blend SrcFactor DstFactor|开启并设置混合模式

### <center>SubShader的标签类型</center>

标签类型|说明|例子
|--|--|--
Queue|控制渲染顺序，让透明物体在不透明物体之后渲染|Tags {"Queue"="Transparent"}
RenderType|对着色器分类，用于着色器替代|Tags {"RenderType"="Opaque"}
DisableBatching|一些SubShader在Unity的批处理时会出现错误，例如使用了模型空间下的坐标进行定点动画。这是可以通过该标签来直接指明使得否对该SubShader使用批处理|Tags {"DisableBatching"="True"}
ForceNoShadowCasting|控制使用SubShader的物体是否会投射阴影|Tags {"ForceNoShadowCasting"="True"}
IgnoreProjector|如果该标签值为True，那么使用该SubShader的物体将不会受Projector的影响，通常用半透明物体|Tags {"IgnoreProjector"="True"}
CanUseSpriteAtlas|当该SubShader使用于精灵(sprites)时，该标签设置为False|Tags {"CanUseSpriteAtlas"="False"}
PreviewType|知名材质面板将如何预览该材质。默认情况下，材质将显示一个球，我们可以通过把该标签的值设置为"Plane""SkyBox"来改变预览类型|Tags {"PreviewType"="Plane"}

### <center>Pass的标签类型</center>

标签类型|说明|例子
|--|--|--
LightMode|定义该Pass在Unity的渲染流水线中的角色|Tags {"LightMode"="ForwardBase"}
RequireOptions|用于指定当满足某些条件时才渲染该Pass，他的只是一个有空格分隔的字符串。目前，Unity支持的选项有SoftVegetation。|Tags {"RequireOptions"="SoftVegetation"}

# 基础光照
## 光源
在光学里，我们使用辐射度来量化光。
根据入社的光线数量和方向，无门可以计算出光线的数量和方向，我们称之为出射度。
辐射度和出射度的比值满足线性关系，其值为材质的漫反射和高光反射的属性。
## 吸收和散射
光线由光源发射出来后，就会与一些物体相交。通常，相交的结果有两个：散射和吸收。

散射只改变光线的方向，但不改变光线的密度和颜色。
光线散射后有两种方向：一种会散射带物体内部，成为折射或透射；一种会散射到物体外部，这种现象称为反射。
为了计算两种不同的散射方向，高光反射表示物体表面是如何反射光线的，漫反射表示有多少光线会被折射、吸收和散射出表面。

吸收只改变光线的密度和颜色，但不改变光线的方向。

## 标准光照模型
### 自发光 C<sub>emissive</sub>
用于描述当给定一个方向时，一个表面本身会向该方向发射多少辐射量。
自发光的计算也很简单，就是直接只用来该材质的自发光颜色：
__C<sub>emissive</sub>=M<sub>emissive</sub>__

### 高光反射 C<sub>specular</sub>
用于描述当光线从光源射到模型表面时，该表面会在完全镜面反射方向散射多少辐射量。
__Phong模型__
![test](http://q59qahcgi.bkt.clouddn.com/60C1A7D7BF90EA9AAE8857A085A0C1B7.png)
__$\vec{r}$ = 2($\vec{n}$ $\cdot$ $\vec{l}$)$\vec{n}$ - $\vec{l}$__
__C<sub>specular</sub> = (C<sub>light</sub> $\cdot$ M<sub>specular</sub>)max(0,$\vec{v}$ $\cdot$ $\vec{r}$)<sup>M<sub>gloss</sub></sup>__
其中，M<sub>gloss</sub>是材质的光泽度，也被称为反光度。它用于控制高光区域的亮点有多宽，其值越大亮点越小。M<sub>specular</sub>是材质的高光反射颜色，它用于控制该材料对于高反射光的强度和颜色。C<sub>light</sub>则是光源的颜色和强度。

__Blinn模型__
![test](http://q59qahcgi.bkt.clouddn.com/D03E4FB4603EFD64B3F382A902742408.png)
Blinn为了防止计算反向的$\vec{r}$引入了一个新向量$\vec{h}$，他是通过对$\vec{v}$和$\vec{l}$的取平均后再归一化得到的：__$\vec{h}$ = $\frac{\vec{v} + \vec{l}}{|\vec{v} + \vec{l}|}$__
__C<sub>specular</sub> = (C<sub>light</sub> $\cdot$ M<sub>specular</sub>)max(0,$\vec{v}$ $\cdot$ $\vec{h}$)<sup>M<sub>gloss</sub></sup>__
在实验中，入股摄像机和光源的距离足够远，Blinn模型会把$\vec{v}$和$\vec{l}$认为是定值，快于Phong模型，相反若是两者经常变化则Phong模型更快一些。

### 漫反射 C<sub>diffuse</sub>
用于描述当光线才光源照射到模型表面是，该表面会向每个方向散射多上辐射量。
漫反射光照符合兰伯特定律：反射光线的强度与表面法线和光线方向之间夹角的余弦值成正比。
__C<sub>diffuse</sub>=(C<sub>light</sub> $\cdot$ M<sub>diffuse</sub>)MAX(0, $\vec{n}$ $\cdot$ $\vec{l}$ )__
$\vec{n}$是表面法线，$\vec{l}$是指向光源的单位向量，M<sub>diffuse</sub>是材质的漫反射颜色，C<sub>light</sub>是光源的颜色。为了防止才背后来的光照照亮，我们使用最大值函数将其截取到0，以防止法线和光源方向的点乘为负值。

### 环境光 C<sub>ambient</sub>
用于描述其他所有的间接光照。
场景所有物体使用同一个环境光所以有：
__C<sub>ambient</sub>=G<sub>ambient</sub>__
