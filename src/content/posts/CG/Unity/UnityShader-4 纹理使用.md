---
title: UnityShader-4 纹理使用
published: 2024-04-24
description: 《Unity Shader入门精要》简陋笔记。关于声明、绑定、采样纹理。
tags: [CG, Unity, Shader]
category: CG
draft: false
---

本文是《Unity Shader入门精要》的简单笔记之一，只有非常基础简单的一部分，主要关注Unity shader的概念、语法、用法等。不涉及基本的图形学原理。  

:::important
本文是Unity内置管线的Shader，使用CG着色器语言以及以.cginc结尾的各种依赖。而Unity内置管线正在过时，Unity正逐渐全面转向SRP，它更多使用HLSL着色器语言，以及以.hlsl结尾的各种依赖与内置变量/函数（这些内置依赖与工具的用法发生了较大变化）。虽然语法变化很少，而且仍然可以在SRP下使用CG和\*.cginc写Unity Shader，但建议考虑了解更新的技术。  
关于新的SRP管线及其Shader，强烈推荐[catlikecoding](https://catlikecoding.com/)，尤其是它的SRP教程能从零搭建一个有必要功能的自定义渲染管线，大大加深对渲染管线、Shader以及各种图形学基本理论的理解。  
:::

纹理不是单纯的图片，而是一组以图片形式存下来的数据，可以存任何数据定义任何使用方式，是一个方便存储空间。  
  
## 在unityshader中使用纹理  
熟悉shader就好说。  
不同的纹理有不同的使用方法，比如法线贴图，置换贴图这些特殊贴图。为了一般性，这里以一个存储颜色的纹理为例。  
  
首先**声明这个纹理**，在properties和CGPROGRAM中。  
```  
Shader"..."{  
	Properties{  
		_MainTex("MainTexture", 2D) = "white" {}  
	}  
	SubShader{  
		Pass{  
			Tags {"LightMode"="ForwardBase"}  
			CGPROGRAM  
			#progma vertex vert  
			#progma fragment frag  
			sampler2D _MainTex;  
			float4 _MainTex_ST;  
			...  
			ENDCG  
		}  
	}  
}  
```  
\_MainTex 是这张纹理的名称，\_MainTex_ST 是Unity自动填入的，表示这个纹理的拉伸和平移（在纹理面板中设置的），S表示 Scale，T表示Transform. 模型存储的纹理坐标，要经过这个缩放、平移之后才到纹理上采样：`uv = texcoord.xy * _MainTex_ST.xy + _MainTex_ST.zw`. 可见_MainTex_ST xy 是scale，zw 是平移。  
  
然后设置输入输出**结构体，里面包含模型纹理坐标**，这里只写第一组.  
```  
// 在 CGPROGRAM中  
struct a2v{  
	float4 vertex : POSITION;  
	float4 normal : NORMAL;  
	float4 texcoord: TEXCOORD0;  
	...  
}  
// 在v2f中，TEXCOORD用于任意存数据.  
struct v2f{  
	float4 pos : SV_POSITON;  //最后的屏幕坐标  
	float2 uv: TEXCOORD0;  
	float3 worldPos: TEXCOORD1;  
	float3 wordNormal: TEXCOORD2;  
}  
```  
  
在 vert 中变换 texcoord 为 uv，传给fragment shader.  
```  
v2f vert(a2v i) {  
	v2f o;  
	o.uv = i.texcoord.xy * _MainTex_ST.xy + _MainTex_ST.zw;  
	//or o.uv = TRANSFORM_TEX(i.texcoord, _MainTex);  
	...  
	return o;  
}  
```  
其中 TRANSFORM_TEX为Unity内置，是一个宏定义，为：  
```  
// Transforms 2D UV by scale/bias property  
#define TRANSFORM_TEX(tex,name) (tex.xy * name##_ST.xy + name##_ST.zw)  
// 会议C宏定义中的##  
```  
  
最后，在frag中对纹理采样，按照纹理的身份进行光照计算。  
```  
fixed4 frag(v2f i) : SV_TARGET{  
	//这里将颜色纹理作为漫反射纹理.  
	float3 albedo = tex2D(_MainTex, i.uv).rgb;  
	float3 diffuse = _LightColor0.rgb * albedo * max(0, dot(worldNormal, worldLightDir));  
}  
```  
用作颜色的纹理，可以是漫反射纹理 diffuse，一般命名为 **albedo** 纹理，采样后直接乘以光色乘以角度得漫反射。还有 Specular 反射光色，采样后乘以光色乘以漫反射角度。  
  
## 纹理属性  
把一张图片复制到unity中去，会变成纹理，里面有一些属性。  
可以选择纹理类型，比如normal map 法线纹理等。可以设置纹理的缩放和平移（上面的ST），还有wrapmode，比如repeat, clamp，直接截断（大于1的变1，小于0的变0），还有其它的，见openGL笔记中的纹理部分。  
可以选择采样插值方法，比如最近邻，双线性，三线性。  
有个alpha from gray，勾选则按照每个像素的灰度值填充a通道。  
设置纹理类型为 advanced，还可以选择生成mipmap。  
还有尺寸，一般纹理大小以2的幂为佳（GPU的存储特点），不整可能会向下截取。还有压缩方式，等等。  
  
## 法线纹理/凹凸映射  
凹凸纹理其实是高度纹理，最后也会用于计算发现，见CG笔记中的纹理部分。法线纹理更多用，直接在像素中存储（切线空间下）的法线值。法线为单位向量，每个分量值在-1\~1，一般使用 `(normal+1)/2`的方式映射到0-1，使用的时候采样得到的数据 \*2+1.  
  
但是也有不同的压缩方法。一个在纹理属性中设置为 normal 类型的纹理，可以用 Unity内置的 `UnpackNormal(packednormal)` 来让Unity自动完成。  
  
法线纹理的使用有两种方式，一般是在顶点着色器中将各种向量变换到切线空间，然后在片段着色器中采样法向计算光照。注意*并不是在顶点着色器中采样法线贴图*，在顶点着色器中，法线空间的变换矩阵TBN使用原本的没有贴图的法线计算。  
  
## 渐变纹理  
渐变纹理是一种一维纹理，在动画中常用，能够显著地突出光照明暗的效果，属于非真实感映射。这里只用计算光照强度，但不用计算颜色（没有反射颜色、漫反射颜色等等，只用计算角度带来的光线明暗强度，也就是只计算光照模型中的角度项），然后用这个角度在渐变纹理中采样，将颜色直接作为显示的颜色。渐变纹理采样的颜色可能*只替换掉漫反射颜色*，而高光颜色可能还是要另算的。  
  
渐变纹理是一维的，而贴图是二维的，所以渐变纹理的颜色可能直接存在纹理对角线上。  
  
## mask遮罩纹理  
如题，就是个mask。如果在计算什么东西的时候希望屏蔽掉某些部分，只保留一些部分，（比如只计算部分区域的高光反射），就可以用mask。  
