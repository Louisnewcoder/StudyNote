
Shader对材质(物体表面属性提供者）和纹理（物体图像数据提供者）进行采样和贴图

# 基本工作原理
Shader是一种用于图形渲染的程序，它定义了如何计算和呈现3D模型或2D图像的外观和效果。下面是Shader的工作原理的简要概述：

1. 输入几何数据：首先，Shader接收输入的几何数据，包括顶点位置、法线、纹理坐标等。这些数据通常由顶点缓冲区提供。

2. 顶点着色器（Vertex Shader）：顶点着色器对每个输入顶点执行操作。它可以对顶点进行变换、变形、计算光照等操作。顶点着色器的输出包括变换后的顶点位置、法线、纹理坐标等。

3. 图元装配（Primitive Assembly）：在图元装配阶段，根据顶点着色器的输出，将顶点组装成几何图元，如点、线、三角形等。
   
4. 几何着色器（Geometry Shader）（可选）：几何着色器对每个图元进行操作。它可以创建新的几何图元，或者对现有图元进行修改。几何着色器的输出通常是一个或多个图元。

5. 裁剪（Clipping）：在裁剪阶段，将超出视图空间的图元裁剪掉，只保留可见的部分。
 
6. 光栅化（Rasterization）：在光栅化阶段，将图元转换为像素。这包括确定每个图元覆盖的像素区域，并生成相应的片元。

7. 片元着色器（Fragment Shader）：片元着色器对每个片元（像素）进行操作。它可以计算片元的颜色，应用纹理采样、光照计算、阴影等效果。片元着色器的输出是最终的像素颜色。

8. 输出合并（Output Merging）：最后，将所有片元的颜色进行混合和合并，生成最终的图像。这包括深度测试、透明度混合等操作。

## 小节总结

几何物体的数据，比如顶点位置，法线，纹理坐标等是在进入Shader之前已经被准备好了的。

**Vertex Shader** （即顶点着色器）负责围绕着顶点进行操作，其输出设计顶点的位置，法线，以及纹理的坐标。

**Fragment Shader**（即片元着色器）负责围绕着像素进行操作，主要目的输出计算后的颜色。它将颜色，纹理采样，光照，阴影等效果根据程序员的指令综合处理后给出最终每个像素的颜色。

**Surface Shader** (即表面着色器）控制材质属性和参数，连接顶点着色器和片元着色器的接口。

# 常用着色器之间的关系

顶点着色器首先处理场景中物体的顶点，位置，光照，法线等。

表面着色器主要用于定义材质的属性和参数，并将这些信息传递给底层的顶点着色器和片元着色器。表面着色器通过 *指定的*  **输入结构体**来接受材质属性，并通过**输出结构体**将这些信息传递给顶点和片元着色器。

片元着色器可以*根据顶点着色器传递的信息*，计算每个像素的最终颜色。

顶点和片元着色器在每一帧都会在每个物体上执行。表面着色器对应用在物体上的材质进行计算。

# Shader 代码文件的基本结构

```c for Graphics
Shader "MenuNameInEditor/ShaderNameInEditor"{
	// Optional, 将显示在编辑器材质面板中， 同时作为参与各种Shader运算的参数
	Properties{
		// Property1
		// Property2
		// How to Define a Property 
		_PropertyNameForCode("NameForEditor",PropertyType) = Value 
		// no ';' needed 
		_MainTex("MyTexture",2D) = "white"{}
		// SubShader代码中使用的是 _MainTex ; 而在编辑器的材质面板中显示的是MyTexture; 这个属性的类型是 2D 类型， 默认值是 "white"
		// ......
	}
	// Subshader真正执行的代码模块，运行时，unity会从第一个Subshader开始确认是否可以执行，如果是则执行第一个，否则一次向下寻找。unity只会执行1个SubShader
	SubShader{
		Tags { "tagName1"="Value1" "tagName2"="Value2"} 
		// Subshader 下的tag是与渲染模式 渲染队列等宏观设置相关
		// 这些内容不能定义到 Pass的tag中
		// 在运行时，unity会按顺序执行所有的Pass通道内容
		Pass{
			Tags { "tagName1"="Value1" "tagName2"="Value2"} 
			// pass 下的tag是与 光照模式等细节操作相关
			//编写着色器函数的代码块,从CGPROGRAM标记开始，到ENDCG结束
			CGPROGRAM
			// code......
			ENDCG
		}
		pass{}
		......
	}
	SubShader{}
	......
	Fallback "diffuse" //如果没有Subshader可以运行则执行 Fallback指定的Shader类型， 这里是 diffuse 漫反射
}

```
## 关于Properties代码块相关的内容
### Property的类型
float 32位浮点数
half 16位中精度浮点数 -6万-6万 
fixed 11位低精度浮点数 -2 - 2 
e.g:
`_Float("MyFloat",float)=1 

//范围浮点数 e.g:
`_Range("亮度",range(0,100))=0

//浮点四元组 xyzw e.g:
`_Vector("MyVector",Vector)=(1,2,3)

//颜色 浮点四元组 rgba e.g:
`_Color("MyColor",Color)=(1,1,1,1) 

//纹理贴图 2的N次方大小的贴图 sampler2D e.g:
`_2D("MyTexture",2D)="white"{} 

//纹理贴图 非2的N次方大小的贴图 samplerRect e.g:
`_Rect("MyRect",rect)="white"{} 

//立方体纹理 e.g:
`_Cube("MyCubeMap",Cube)="white"{}


## SubShader代码块的相关内容

变量类型

1. 基本数据类型：
    
    - float：单精度浮点数。
    - half：半精度浮点数。
    - fixed：固定小数点数。
    - int：整数。
    - bool：布尔值。
2. 向量类型：
    
    - float2：二维浮点向量。
    - float3：三维浮点向量。
    - float4：四维浮点向量。
    - half2：二维半精度浮点向量。
    - half3：三维半精度浮点向量。
    - half4：四维半精度浮点向量。
    - fixed2：二维固定小数点向量。
    - fixed3：三维固定小数点向量。
    - fixed4：四维固定小数点向量。
    - int2：二维整数向量。
    - int3：三维整数向量。
    - int4：四维整数向量。
    - bool2：二维布尔向量。
    - bool3：三维布尔向量。
    - bool4：四维布尔向量。
3. 矩阵类型：
    
    - float2x2：二维浮点矩阵。
    - float3x3：三维浮点矩阵。
    - float4x4：四维浮点矩阵。
4. 纹理类型：
    
    - Sampler2D：二维纹理采样器。
    - Sampler3D：三维纹理采样器。
    - SamplerCUBE：立方体纹理采样器。


![[[Pasted image 20231110102233.png](https://github.com/Louisnewcoder/StudyNote/blob/master/Pasted%20image%2020231110102233.png)]]


===***Sampler***=== 类型是用于获取纹理数据的对象，声明与属性的同名纹理变量，就可以让变量获得属性的值
===***纹理坐标***=== texture-mapping Coordinates 即 UV，是一个float2的 二维向量，用于指定从纹理中获取像素颜色的位置。纹理坐标的范围通常是从(0, 0)到(1, 1)，表示纹理上的整个区域。

纹理采样器和纹理坐标传递给===***tex2D()***===方法，你可以获取纹理中指定位置的像素颜色。该方法返回一个包含纹理像素颜色的四维向量（RGBA值）。

```C
例如，你可以使用以下代码来获取纹理中(0.5, 0.5)位置的像素颜色：
sampler2D mySampler;
float2 uv = float2(0.5, 0.5);
float4 color = tex2D(mySampler, uv); 
// 也可以用float3 接  tex2D(mySampler, uv).rgb， 如果不想接alpha值的话可以这样做
```

===***TRANSFORM_TEX()***=== 方法是一个宏方法，里面有一系列的运算。主要作用是将纹理坐标从模型空间转到屏幕空间。第一个参数是模型空间的uv,第二个参数是一个纹理采样器。
它会根据纹理采样器的属性来对uv进行转换。 转换之后的uv 可以被用来在屏幕空间坐标系下对另一个纹理进行采样。通常，我们会在顶点函数中使用这个宏进行uv转换，然后在偏远函数中， 使用这个uv对原纹理进行采样，这样就可以将原纹理的像素正确的投射在屏幕空间下。


### 关于 struct appdata_full 中的信息
appdata_ 系列结构体内的相同属性的值是相同的，但是丰富程度不同。所以提到的内容并不会因为结构体不同而不同
```C for Graphics
struct appdata_full {
    float4 vertex : POSITION; // 模型的顶点，位置是模型坐标系下的位置
	// 最后一位w 代表队是裁剪范围，系统会自动使用这个值
    float4 tangent : TANGENT;
    float3 normal : NORMAL;   // 这个法线是面的法线
    float4 texcoord : TEXCOORD0; // 第一套uv坐标
    float4 texcoord1 : TEXCOORD1; // 第二套
    float4 texcoord2 : TEXCOORD2; // 3
    float4 texcoord3 : TEXCOORD3; // 4
    fixed4 color : COLOR; // 颜色
    UNITY_VERTEX_INPUT_INSTANCE_ID
};
```


```C for Graphics
struct v2f
{
    float2 uv : TEXCOORD0; // 屏幕坐标系的UV坐标
    float4 vertex : SV_POSITION; // 屏幕坐标系的位置
};
```

```C
sampler2D _MainTex; //属性中声明的纹理变量
float4 _MainTex_ST; //用于存储通过TRANSFORM_TEX()方法转换后的uv值，ST代表Scale和Transform
_MainTex_ST.xy //存储的是缩放值
_MainTex_ST.zw //存储的是偏移值

//特别变量 _Time 这个变量记录了游戏时间相关的信息
float4 _Time; //四个分量分别为（t/20,t,t*2,t*3)， 其中y分量即t,是当前场景加载到现在的游戏时间

```
### vert函数 - vertex shader函数
```C for Graphics
// 这里传入的参数就是 在渲染开始时cpu收集到的模型信息，可以自定义也可以使用上面提到的Unity给的结构体
v2f vert (appdata v) 
{
	v2f o; // 声明一个变量，作为返回值。类型刚好是v2f的结构体
	o.vertex = UnityObjectToClipPos(v.vertex); // 将模型的顶点值从模型自身的本地坐标转化为屏幕坐标系的值
	o.uv = TRANSFORM_TEX(v.uv, _MainTex); // 将模型的uv坐标值从本地uv空间转到屏幕坐标系的uv值，这里需要用一个后缀为_ST的， _MainTex_ST变量接收
	// TRANSFORM_TEX(v.uv,_MainTex) 等同于
	//o.uv = v.texcoord.xy * _MainTex_ST.xy  + _MainTex_ST.zw;
	return o; // 返回v2f```
}
```

### fragment函数 - fragment shader函数
```C
fixed4 frag() :SV_TARGET 
{
	return fixed4(1,1,1,1);
}

// SV_TARGET通知渲染器把函数输出结果存储到一个渲染目标中

```

# 语义
SV 系统数值  -- System Value
SV_POSITION -- 表示光栅化后的顶点坐标，裁剪空间的坐标
SV_TARGET -- 表示告诉渲染器把结果存储到渲染目标

POSITION -- 用模型空间的顶点坐标（填充它描述的变量的值）
NORMAL --  用模型空间的法线方向（填充它描述的变量的值）
TEXCOORD0 -- 用模型空间的法线方向（填充它描述的变量的值）
![[Pasted image 20231110105028.png]]

![[Pasted image 20231110105052.png]]




# 其他Unity内置变量

`float3 _WorldSpaceCameraPos 摄像机在世界空间中的位置
`float4 _ProjectionParams  //x= 1.0 (或-1.0. 如果正在使用一个翻转的投影矩阵进行渲染）， y= Near, z=Far, w=l.0+1.0/Far, 其中 Near Far 分别是近裁剪平面 和远裁剪平面和摄像机的距离
`float4 _ScreenParams // x=width , y=height, z= l.O + 1.0/width, w= 1.0+1.0/height, 其中width height 分别是该摄像机的渲染目标 (render target) 的像素度和高度


# UnityShader的Include 文件
![[Pasted image 20231110102826.png]]

# Unity内置函数
![[Pasted image 20231110103549.png]]

**UnpackNormal()**
该函数接受一个浮点型的压缩法线向量作为输入，并返回一个三维浮点型的法线向量。
因为法线向量通常用于表示表面的方向和光照计算，但在内存或纹理使用方面，它们可以占用较多的空间。为了减少内存占用或纹理传输的开销，法线向量可以通过压缩方式存储。而UnpackNormal函数就是用来还原原法线向量值，方便精准计算。
使用方法：
将法线纹理的采样结果作为参数传入函数参数列表
`o.Normal=UnpackNormal(tex2D(_BumpMap,IN.uv_BumpMap));`


# 渲染流水线基本信息

## 流水线阶段
渲染流水线一般分为3个阶段：应用阶段，几何阶段，光栅化阶段

**应用阶段**：由应用主导，开发者具有绝对的控制权，通常由CPU负责实现。***说的俗一点就是通过CPU通过应用程序把要渲染的内容的基本数据准备好。

a.准备好场景数据，eg：摄像机位置、视椎体、模型、光源等；
b.粗粒度剔除，剔除不可见的物体；
c.设置每个模型的渲染状态。（材质、纹理、shader）
d.***输出***渲染所需的几何信息，**渲染图元**。（点、线、三角面）

**几何阶段**：和每个渲染图元打交道，进行逐顶点、逐多边形操作。通常在GPU上进行。

a.把顶点坐标变换到屏幕空间中，再交给光栅器进行处理。
b.通过对输入的渲染图元进行多步处理后，输出屏幕空间的二维顶点坐标、每个顶点对应的深度值、着色等相关 信息，传递给下一个阶段。

**光栅化阶段**：使用几何阶段传递的数据来产生屏幕上的像素，渲染出最终的图像。通常在GPU上进行。

a.决定每个渲染图元中的哪些像素应该被绘制在屏幕上。
b.逐顶点数据进行插值，然后逐像素处理。

## 关于CPU与GPU之间的协作
应用阶段可分为下面3个阶段：

（1）把数据加载到显存中。 （硬盘->内存->显存）显卡对显存的访问速度更快，大多数显卡没有RAM的直接访问权利。

（2）设置渲染状态。 （定义了场景中的网格是怎样被渲染的.eg：使用哪个顶点着色器/片元着色器、光源属性、材质）

（3）调用DrawCall。(CPU向GPU发送一个渲染命令，一个DC会指向本次调用需要渲染的图元列表)

## GPU流水线
![[Pasted image 20231030165851.png]]
在这个流水线中我个人需要了解的是：
**顶点着色器（Vertex Shader）**：完全可编程。必须完成的基本工作是，把顶点坐标从***模型空间转换到裁剪空间***。最终得到归一化的设备坐标（Normalized Device Coordinates,NDC）
顶点着色器调用多少次取决于这个模型有多少个顶点。顶点着色器只操作单个顶点。
逐顶点光照 -- 效率最高，效果最差的光照运算。

**几何着色器（Geometry Shader）**：可选着色器， 执行图元的着色操作或生产更多的图元。

**裁剪（Clipping）**：不可编程，可配置。剔除不在摄像机视野内的顶点和三角图元的面片并且创建新的顶点。
![[Pasted image 20231030170228.png]]

**屏幕映射（Screen Mapping）**：不可配置，不可编程。 将每个图元的坐标转换到屏幕坐标中。
![[Pasted image 20231030170325.png]]

**片元着色器（Fragment Shader）**：根据上一阶段输出的数据插值得到输入信息。这个阶段则负责染色，输出一个或者多个颜色值。
所谓片元 ，***就是被模型占据的像素格子（没有被占据的并不称之为片元）

与顶点着色器一样，每次也只处理单个片元
![[Pasted image 20231030170355.png]]
ps： 我们使用uv 获得贴图采样的目的就是为了方便片元着色器快速处理上色渲染
也可以在片元着色器中进行光照处理，像在顶点着色器中处理一样， 在片元着色器中处理光照就是 逐像素光照。 

**逐片元操作**
核心执行内容就是决定每个片元的可见性，通过一系列的测试，比如***透明度，深度测试，模板测试。***
然后将留下来的片元与已存在颜色缓冲区的颜色进行合并。开启混合的就混合，没开启混合的则覆盖。

## 关于Drawcall
Drawcall就是Cpu调用图像应用程序接口来命令GPU进行渲染操作。

CPU和GPU之间的数据是通过**命令缓冲区**进行传输的。命令缓冲区包含了一个队列，由CPU向其中添加命令，GPU去读取命令，***添加和读取的命令都是相互独立的***。当CPU需要渲染一个对象时，就可以向命令缓冲区中添加命令，而当GPU完成上一个渲染任务后，就会从命令缓冲区中再取出一条命令并执行它。
*命令缓冲区中的命令还有很多种类，而DrawCall是其中的一种*，其他命令还有改变渲染状态等(例如改变使用的着色器，使用不同的纹理等)。
![[Pasted image 20231030163042.png]]
蓝色方框内的命令就是Draw Call，而红色方框内的命令用于改变渲染状态。我们使用红色方框来表示改变渲染状态的命令，是因为这些命令往往更加耗时。

***Drawcall太多会影性能***
每次调用Draw Call需要一系列的准备工作，如果Draw Call数量太多，那么CPU就会把大量的时间花费在提交Draw Call上，造成CPU的过载。也就是说，额外的操作越少越好。***复制1个10MB的文件的效率远高于复制10240个1kb的文件！！！***

**如何减少Draw Call？**
批处理(Batching)，就是把很多的小Draw Call合并成一个大Draw Call。Unity分为动态批处理和静态批处理：动态批处理是自动进行的，但是有一定的条件限制，比如顶点属性不得超过900，同一种材质和网格；静态批处理的物体不可移动，但是限制条件更少，可以通过合并网格来提高性能。

在开发过程中要尽量避免：
a. 使用大量很小的网格，如果要使用很小的网格时，考虑是否可以合并
b. 避免使用过多的材质，尽量在不同的网格之间共用同一种材质


# 光栅化 Rasterize/Rasterization
光栅化（Rasterization）是计算机图形学中的一个重要概念，用于描述**将几何图形转换为屏幕上的像素的过程**。在渲染管线中的光栅化阶段，将原始的几何图形（如三角形）转化为屏幕上的像素。

# Vertex Shader处理顶点的原理和目的
顶点着色器处理的顶点通常是从模型空间（Model Space）转换到屏幕空间（Screen Space）或裁剪空间（Clip Space）。这个转换过程是为了将三维模型的顶点坐标转换为最终在屏幕上显示的二维像素坐标。

在顶点着色器中，顶点经过一系列变换和计算，包括模型变换（Model Transform）、视图变换（View Transform）和投影变换（Projection Transform）。***这些变换将顶点从1（1）模型空间转换到 （2）世界空间（World Space），然后（3）转换到相机或观察者的视点空间（View Space），最后通过投影变换将其转换到（4）裁剪空间或屏幕空间。

从模型空间到世界空间的变换 就是 模型变换（Model Transform）
*ps: 相机空间就是以相机位置为原点所计算的坐标空间。 对应视图变换（View Transform）

裁剪空间是一个规范化的坐标空间，其中的顶点坐标的范围是[-1, 1]，对应于屏幕的可见区域。在裁剪空间中，还可以进行透视除法（Perspective Division），将坐标转换为标准化设备坐标（Normalized Device Coordinates，NDC），其范围是[-1, 1]。最后，标准化设备坐标会被映射到屏幕的实际像素坐标。
***个人理解所以一般情况下裁剪空间可以泛指屏幕空间，但并不是具体的屏幕像素坐标。不同设备屏幕像素，像素坐标也不同


转换到屏幕空间后，顶点的位置和其他属性（如颜色、法线、纹理坐标等）会被传递给后续的渲染阶段，如几何着色器、光栅化、片段着色器等，以进行进一步的处理和计算。

顶点着色器的任务是对每个顶点执行必要的变换和计算，以便将顶点从模型空间转换为屏幕空间或裁剪空间，并将其传递给后续的渲染阶段使用。


# 三大测试

共同点：三大测试都是决定是否显示当前像素
不同点：
深度测试用于处理前后遮挡关系
透明测试处理透明阈值
模板测试处理像素重合关系

关键点在于，无论什么测试，只要没通过测试，像素就会被舍弃，就不会被渲染。

# 关于深度测试
## 深度
像素点在3D世界中距离相机的远近。离相机越近，深度越小，反之越大。也就是z值
## 深度缓存
深度缓存存储着准备要绘制在屏幕上的像素点的深度值。这个值被用来执行深度测试。
当开启深度缓冲区绘制像素之前图形接口会先把像素的深度值和该像素在深度缓存存储的深度值进行比较，如果：
新像素深度值<深度缓存中存储的值，则新像素取代深度缓存区存储的值。即新像素在前，距离相机更近，遮挡其他物体；反之，但新像素深度值>深度缓存区存储的值，则新像素被遮挡，起颜色和深度值被丢弃。 

*ps :默认深度测试是开启的*

**默认情况下，后绘制的物体会遮挡先绘制的物体。**
在没有开启深度测试的情况下，gpu会绘制距离较近的物体，后绘制距离远的物体。
但是在这个情况下因为距离远的物体是后绘制的，反而会遮挡先绘制且离摄像机较近的那一个，这样的呈现的交过就是异常的。

ZWrite 是否写入深度缓存 
On/Off 默认值On

ZTest 是否进行深度测试
Greater/GEqual/Less/LEqual/Equal/NotEqual/Always（一定通过）/Never（一定不通过）/Off(和Always一样，一定通过)
默认值是LEqual （less or equal to）意思是当小于等于的时候通过测试

4种情况
**一：ZWrite开启 且 测试通过**
像素的深度能成功写入到深度缓存，因为测试通过，所以颜色会保留（写入颜色缓存）也就是可以成功显示
**二：ZWrite开启 但是测试不通过**
不写入深度缓存，也不会写入颜色缓存
**三：ZWrite关闭 且测试不通过**
不写入深度缓存，也不写入颜色缓存
**四：ZWrite关闭 但是测试通过**
像素深度不写入深度缓存，但是颜色写入颜色缓存

颜色能否写入决定能否正常显示； 深度能否写入影响是否更新深度值，即下一次比较的值是谁

===***要注意的是：***===
利用深度测试实现效果的时候，如果目的是为了显示自己，那么这个shader的pass 可以把深度值写入关掉，以免其他物体与之比较。 如果需要其他物体与之比较则打开。

并且可以考虑 使用多pass 设置不同的深度测试规则

# 关于模板测试 Stencil Test

用于处理重合的像素如何显示。以便更自由的进行自定义显示。
每一个像素都可以配置一个stencil值，在同一个像素上，一旦出现像素重合，就可以获取这个stencil值并根据条件进行处理。
这个stencil值是开发者定义的，随便自定义。
应用：在遮罩 迷雾 彩虹 ？？？共生存在？？

stencil{
	Ref value // Ref关键字 value自定义值 范围0-255， 重合时获取
	//ReadMask 读遮罩
	//WriteMask  写遮罩
	Comp comparisonFunction // 条件判断 Less Greater Equal....
	Pass // 满足条件后 如何处理，比如替换值，还是增减 Zero 归零 Replace 替换 Keep保留 IncrSat 加1但是不会超过255 DecrSat减一不会超过0
	Fail // 没有通过模板测试的处理
	ZFail // 通过测试的处理
}

# 关于光照

平行光 没有位置， 平行光就是向量
点光源有位置

一些常用重要属性和函数
`_LightColor0  // 该Pass所处理的逐像素光源颜色 ，不用纠结 0 1 2 3 ，因为当前pass值处理当前颜色
`_WorldSpaceLightPos0 // 该Pass所处理的逐像素光源的位置，但是平行光除外，因为平行光没有位置只有方向； 它是个float4， 如果是平行光的话，w分量是0，其他光w是1

`float3 WorldSpaceLightDir(float4) //输入模型空间坐标，得到光照方向
`float3 UnityWorldSpaceLightDir(float4) // 输入世界空间的位置，得到光照方向
`float3 ObjSpaceLightDir(float4) // 输入模型空间位置，得到 模型空间的光照方向



WorldNormal 世界法线是如何得到的？
1 得到法线贴图后，进行采样(（)采样贴图得到的是贴图的rgb)
2 将贴图的rgb3个值分别于顶点数据的 tangent，Bitangent，和法线相乘，并将结果相加

`worldNormal = red * tangent + green * Bitangent + blue * normal
如何获得副切线？
利用向量的叉乘，用切线叉乘法线。如果顺序反了，副切线的方向也反了
cross(tangent,normal)



Ambient 环境光
`UNITY_LIGHTMODEL_AMBIENT.xyz ; // 环境光的值

平行光
`_WorldSpaceLightPos0.xyz // 平行光的方向
`_LightColor0.rgb // 平行光的颜色


关于混合
如果不开启混合， 结果就是覆盖，最后一个覆盖前面的

正向渲染 Forward
什么情况下会导致性能下降？
光源和物体都多的时候

延迟渲染 deferred
延迟渲染可以解决正向渲染的弊端
延迟渲染进行一次光照计算
延迟渲染除了 颜色缓存和深度缓存之外，延迟渲染还会利用一个额外的缓冲区—— G缓冲区（Geometry）
G缓冲区存储的是表面信息

延迟渲染实现原理
两个pass
第一个pass 只计算片元是否可见，不计算光照。如果可见就将数据存入G缓冲区
存储信息：漫反射颜色 高光反射颜色  平滑度 自发光  深度等信息。 每个物体执行一次
第二个pass 利用G缓存的各个片元信息进行真正的光照计算，仅计算一次

这个延迟渲染的延迟，就是指光照计算的延迟

延迟渲染的有点
在一个着色器中进行所有的光照计算
大部分模型在执行自身着色器的时候，不需要计算光照，在场景接近完成的时候，才会使用光照信息 就好像在渲染一个二维图像。一般，延迟渲染的光照是发生在屏幕空间。

延迟渲染的缺点：
不支持真正的抗锯齿功能  // 算法问题
不能处理半透明物体 // 因为光照是发生在屏幕空间的，所以即使使用延迟渲染，半透明物体还是需要在正向渲染路径下完成
对显卡的要求，必须支持MRT shadermodel 3.0以上 ，支持深度渲染纹理和双面模板缓冲


G缓冲区，可以把G缓冲区存的变量理解为 几张渲染的纹理
RT0：ARGB32 RGB存储漫反射颜色，A通道不用
RT1：ARGB32 RGB存储高光反射颜色 A通道存储高光指数
RT2：ARGB3101010 RGB存储法线 A通道不用
RT3：ARGB32 存储自发光 
深度缓存 模板缓存 颜色缓存

阴影
能投射阴影的物体
能接收阴影的物体
被阴影忽略的物体

ShadowMap // 阴影渲染技术
首先把相机放置于光源重合的地方 （相机看不到的地方就是 该光源的阴影区域），这样问题就简化成了，判断一个片元是否为相机可见(判断是否有阴影, 通过深度测试)，如果不可见就不渲染阴影，不可见就渲染阴影。 所以深度测试结果决定是否有阴影。
如果场景中的光源开启了阴影，那么unity会为这个光源计算一张阴影映射纹理---- 就是ShadowMap

LightMode = ShadowCaster 可以生成阴影贴图
但是如果不写，引擎会从第一个pass开始找，一直找到fallback，一般fallback使用的引擎原生的shader都会有引擎写好的。所以unity总是会给我们生成一个ShadowMap



真实渲染 == 基于物理的渲染 PBR
非真实渲染 NPR

日式风格：
描边 ： 基于法线和视角 ， 过程式的几何描边，基于图像处理的边缘检测描边，几何轮廓边缘检测发等
着色：厚涂 和 celluloid


# 个人练习实验

## 实验1：如何通过vertex/fragment Shader 实现边缘自发光的效果

``` C
Shader "Unlit/NewUnlitShader"
{
    Properties
    {
        _MainTex ("Texture", 2D) = "white" {}
        _MainColor("MainColor",color)=(1,1,1,1)
        _Color("ShiningColor",Color)=(1,1,1,1)
        _ShiningRange("ShiningRange",range(0,1))=0
    }
    SubShader
    {
        Tags { "RenderType"="Opaque" }
        LOD 100
        Pass
        {
            CGPROGRAM
            #pragma vertex vert_test
            #pragma fragment frag
	        #include "UnityCG.cginc"

		// 没有使用自定义结构体，使用的是默认的appdata系列

            struct v2f
            //作为fragment函数的输入，其内部的内容更应该都是转换到齐次裁剪空间后的
            {
                float2 uv : TEXCOORD0; // 如果想使用纹理，需要用Transform_Tex转给这里的成员uv
                float4 vertex : SV_POSITION; // 必需品
                float3 normal :NORMAL; // 用于接收转换到齐次裁剪空间后的法线
                float3 viewDir : TEXCOORD1;
                // 一个自定义的float3 用于作为视角向量，这里使用第二套uv语义支持的变量接收一下
                // 通常使用没有用的某套uv语义存储一些值
            };
            sampler2D _MainTex;
            float4 _MainTex_ST;
            fixed4 _Color;
            fixed4 _MainColor;
            float _ShiningRange; // 控制发放的范围

            v2f vert_test(appdata_base v)
            {
                // 目标是实现通过视角向量和模型在世界空间的法线的点乘结果判断模型部位是否符合期待的发光标准
                // 所以要在进入片元函数之前把是将向量和模型的世界空间法线拿到
                v2f o;
                o.vertex = UnityObjectToClipPos(v.vertex); //把顶点从模型空间转换到齐次裁剪空间
                //将模型法线转换到世界空间下，并进行单位化
                o.normal =normalize(UnityObjectToWorldNormal(v.normal));
                //通过WorldSpaceViewDir函数得到模型顶点的视角向量，并进行单位化
                o.viewDir =normalize( WorldSpaceViewDir(v.vertex));
                //如果有纹理展示需求，就要在这里把纹理坐标从模型空间转到齐次裁剪空间
                o.uv = TRANSFORM_TEX(v.texcoord,_MainTex);
                return o;
            }
            // 拿到两个向量后，在偏远函数中进行点乘计算，目的是为了让上色更均匀
            // 之前在顶点函数中执行，但是上色不够均匀
            // 之所以在片元着色器中更均匀是因为，片元着色器为映射到齐次裁剪坐标系的顶点像素之间提供更平滑的颜色插值
            fixed4 frag (v2f i) : SV_Target
            {      
               //如果要展示纹理，先采样      
               fixed4 col = tex2D(_MainTex,i.uv);
               // 要的是越靠近边缘，越发光，中心不发光，所以取反，即不垂直的地方为1，垂直为0
               float rim = 1 - saturate(dot(i.viewDir,i.normal));
               if (rim>_ShiningRange) //符合的部分上色
                    col +=(_MainColor+_Color)* rim;
                return col;
            }
            ENDCG
        }
    }
}
```
