---
title: UnityShader-6 阴影
published: 2024-04-24
description: 《Unity Shader入门精要》简陋笔记。关于阴影——阴影映射(shadow map)。
tags: [CG, Unity, Shader]
category: CG
draft: false
---

本文是《Unity Shader入门精要》的简单笔记之一，只有非常基础简单的一部分，主要关注Unity shader的概念、语法、用法等。不涉及基本的图形学原理。  

:::important
本文是Unity内置管线的Shader，使用CG着色器语言以及以.cginc结尾的各种依赖。而Unity内置管线正在过时，Unity正逐渐全面转向SRP，它更多使用HLSL着色器语言，以及以.hlsl结尾的各种依赖与内置变量/函数（这些内置依赖与工具的用法发生了较大变化）。虽然语法变化很少，而且仍然可以在SRP下使用CG和\*.cginc写Unity Shader，但建议考虑了解更新的技术。  
关于新的SRP管线及其Shader，强烈推荐[catlikecoding](https://catlikecoding.com/)，尤其是它的SRP教程能从零搭建一个有必要功能的自定义渲染管线，大大加深对渲染管线、Shader以及各种图形学基本理论的理解。  
:::

阴影当然是Shadow Map。Unity shader 控制阴影的方法。  
  
## Unity阴影机制概述  
不同于我们实现shadow map算法的简单demo，unity的每个材质都有自己的shader，而投射阴影是物体、体现阴影的是材质。因此，unity把阴影图的实现分为阴影投射和接收阴影两个步骤。前者通过 *meshrenderer* 和*Light* 控制，前者开启或关闭shadow cast，后者选择阴影属性，比如soft or not。这里的软阴影只是对边缘做不同程度的模糊来实现的（原本是抗阴影边缘锯齿的算法，后来发现可以用来做软阴影效果）。这里的投射阴影，是在渲染**非透明**物体而需要投射阴影的时候都有的一个固定的“阶段”，它寻找一个特定的Pass：ShadowCaster（是LightMode的Tags）。我们不需要特意去实现它，因为Unity会不断找 Fallback 指定的Pass，而我们一般会在自己的Unityshader里设置 Fallback 为 VertexLit 或者 Diffuse，这里面也有Fallback，而这个Fallback里就有ShadowCaster的Pass。  
  
后者也通过Meshrenderer控制。则是根据第一步shadow cast计算出来的深度图，在正常渲染中加上阴影的衰减，并且根据设置通过一些卷积方法计算软阴影效果。  
  
想要Unity正确处理阴影，以及为我们的shaders准备适当的备件，需要在光源中选择适当的阴影类型(shadow type), 并且在meshrenderer中开启 shadow cast 以及 receive shadow。  
  
Unity的各种材质可以有各种shader，而第一步计算深度缓冲显然是要对“全局”计算的，而一个shader的 ShadowCaster Pass 只能计算自己这个物体在光源视角下的深度，因而需要有一个把所有这样单独的深度图混合起来的过程。而结果就是 Unity 的*屏幕空间阴影映射*。这是一种在延迟渲染中产生阴影的技术，它有一定的兼容性限制。但我们不需要考虑这些，Unity已经封装好了。我的理解就是，把各种纹理弄出来的各种阴影图合在了一起，并且保证纹理坐标和屏幕坐标等值映射，屏幕坐标就是它在深度纹理中的纹理坐标。  
  
## Shadow Cast  
在 Shadow Map 的过程中，shadow cast的实质就是shadow map的第一步：在光源的位置渲染深度图，保存在一张纹理（缓冲）中。  
  
在Unity的Shadow Cast是一个 `Tags {"LightMode"="ShadowCaster"}` 的Pass执行的。Shadow Cast是Unity渲染中的一个阶段，这个阶段，所有开启了阴影投射的纹理都会运行有上面那个 Tags 的Pass。一般不用实现它（当然需要指定Fallback），它会找到 Fallback，而Fallback我们一般会用Unity内置的shader，这些shader自己的Fallback里可能会一层层嵌套到一个有这个Pass的shader。实际上，这个Pass在：`builtin-shaders-xxx->DefaultResourcesExtra->Normal- VertexLit.shader`。  
  
它是长这样的：  
```  
Pass {  
    Name "ShadowCaster"  
    Tags { "LightMode" = "ShadowCaster" }  
  
    CGPROGRAM  
    #pragma vertex vert  
    #pragma fragment frag  
    #pragma multi_compile_shadowcaster  
    #include "UnityCG.cginc"  
  
    struct v2f {  
        V2F_SHADOW_CASTER;  
    };  
  
    v2f vert( appdata_base v )  
    {  
        v2f o;  
        TRANSFER_SHADOW_CASTER_NORMALOFFSET(o)  
        return o;  
    }  
  
    float4 frag( v2f i ) : SV_Target  
    {  
        SHADOW_CASTER_FRAGMENT(i)  
    }  
    ENDCG  
  
}  
```  
这个shader主要都是宏，并且是比较庞大的宏。它会检验当前目标平台的兼容性、用户的设置等等，选用不同的阴影投射技术，比如上面的屏幕空间阴影映射，在无法兼容它的时候就会使用其它方法。  
  
而和shadowmap技术一样，阴影投射其实是完全走完了一次渲染管线，而默认情况里我们会提出物体的背面，导致一些特别的情况无法投射阴影。比如Unity内置的平面，它只有一个面，但也分了正反面，有时候它会被意外剔除。类似这种情况，我们将 meshrenderer 的 cast shadow 设置为 `two-sides`。  
  
## 接收阴影  
接收阴影需要我们自己写一些东西，但主要是用Unity提供的宏和宏函数，主要是三个：`SHADOW_COORDS` 、`TRANSFER_SHADOW` 和`SHADOW_ATTENUATION`。shadow map的原理我们已经知道，这里不说了。  
  
首先需要引入一个新的cginc文件：`"AUtoLight.cginc"`.  
  
定义vertex to fragment 的输入结构体 v2f，并使用宏 `SHADOW_COORDS()` 来定义其中的一个域，让Unity能用自己的方式获取当前顶点在深度图中的纹理坐标。  
  
```  
struct v2f {  
    float4 pos : SV_POSITION;  
    float3 worldNormal : TEXCOORD0;  
    float3 worldPos : TEXCOORD1;  
    SHADOW_COORDS(2)  
};  
```  
注意 `SHADOW_COORDS` 的参数，它实际上是 unity 语义 `TEXCOORDx` 后面那个数字 x，它需要是一个空的 TEXCOORD，这里我们前面用了 TEXCOORD0, TEXCOORD1，所以给的参数是2.  
  
此外，还需要注意，由于宏是字符替换，而Unityshader中阴影相关的工作都是由宏去进行，所以需要保证一些形式和宏里面的统一来确保正常访问：a2v 结构体中的顶点坐标这个域的变量名一定是 `vertex`，并且它在输入顶点着色器的时候用的形参变量名一定是 `v`；v2f 结构体中的顶点（屏幕坐标）位置（SV_POSITION）的域的名称一定是 `pos`。  
  
在顶点着色器中，需要添加一个操作让unity自己处理输出结构体中的深度图纹理坐标：  
```  
struct v2f vert(a2v v){  
	v2f o;  
	TRANSFER_SHADOW(o);  
}  
```  
  
在面元着色器中，只需要添加一行来直接获取阴影导致的光照衰减：  
```  
fixed4 frag(v2f i) : SV_TARGET {  
	fixed shadow = SHADOW_ATTENUATION(i);  
	// 把这个shadow乘到 环境光ambient之外的光照成分上。  
}  
```  
这是个蛮复杂的宏，考虑了各种光源类型，等等......  
  
## 统一管理阴影和光照衰减  
阴影和光照衰减都是一个系数，要乘以到除了环境光ambient之外的光照成分上，这两者在用法上是一样的。因此，Unity提供了一种统一的方式来处理这两种东西：`UNITY_LIGHT_ ATTENUATION`。它会同时处理阴影和光照衰减，最后的得到一个综合了两种效果的系数。  
  
前面都和上文差不多，只有fragment着色器的有点不同：  
```  
fixed4 frag(v2f i) : SV_TARGET {  
	UNITY_LIGHT_ATTENUATION(atten, i, i.worldPos);  
	return fixed4(ambient + (diffuse + specular) * atten, 1.0);  
}  
```  
可见，因为要计算光照衰减，所以v2f结构体额外多了 worldPos 域来把世界坐标也存储下来。  
  
## 透明物体和阴影  
正常情况下 Unity 的透明纹理没有阴影。它不投射阴影也不接收阴影。默认的透明物体Pass一般会Fallback到 `Transparent/VertexLit`，这里没有 ShadowCaster Pass。  
  
不过，我们也可以让它接收阴影，做法是开启 receive shadow，然后在shader中像不透明物体那样计算阴影衰减和光照衰减。不过，由于透明物体关闭了深度写入，所以可能会有一些不正常的效果（？）。  
  
此外，我们也可以强行将它的Fallback改成 Diffuse 或者 Specular 这些非透明物体的shader，这样它就有了 ShadowCaster 的Pass，最后能像不透明物体那样投射和接收阴影。可见要么是完全阴影、要么是完全不要阴影。真实世界中的投射效果是做不到的，这是全局光照才能实现的。  
