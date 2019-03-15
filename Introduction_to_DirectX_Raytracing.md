# Ray Tracing Gems

## Abstract
现代的图形APIs如DirectX 12对开发者暴露了更加底层的硬件接口访问和控制，这就经常导致对于初学者看到这种复杂冗长的代码会感到害怕。在这一章，我们希望一步步解密安装和使用DirectX的光线追踪。

## 3.1	介绍
在2018年的游戏开发者大会上，微软宣布了他们的 DirectX Raytracing (DXR)API,这扩展了Directx12的本地光线追踪支持。随着2018年10月份windows10的更新,这个API运行在所有的支持DX12的GPU上，不是使用专用硬件加速就是通过基于计算的软件调用。此功能为DirectX渲染器提供了新选项，从完整的，电影质量的路径跟踪器到更简陋的光线-光栅混合，例如，用光线跟踪替换光栅渲染的阴影或反射。

与所有图形API一样，在深入研究代码之前，一些先决条件非常重要。 本章假设读者了解光线跟踪的基本原理，并参考本书的其他章节，或者其他文本[^ 4,^ 10]了解l 基本原理。此外，我们假定读者熟悉GPU编程；了解光线跟踪shader，具有基本DirectX、Vulkan或OpenGL的经验。如果你有以前使用DirectX 12的经验可能对理解底层的细节有帮助。
## 3.2	概述

GPU编程有三个独立于api的关键部分组成
* （1）GPU设备编程
* （2）CPU主机端设置进程（the CPU host-side setup process）
* （3）主机和设备之间的数据共享。
在我们讨论每一个部分之前，3.3节介绍了重要的软件和硬件的要求，以便开始构建和运行基于DXR的程序。

然后我们讨论每一个核心的部分，3.4，3.5，3.6章节开始介绍如何去编写DXR的shaders.对于DXR的高级着色语言（HLSL）看起来和用c++写的串行的光线追踪器很相似。使用库来抽象主机端的图形API(例如Falcor),甚至初学者都能很快搭建有趣的GPU加速的光线追踪器。在3.1节有一个例子的展示，它是用一个简单的路径追踪器扩展的Falcor来呈现的。  
3.7节提供了主机断DXR的安装过程的概述，并且描述了the mental model驱动新的API。3.8节包括必要的DXR初始化、建立光线加速的必要数据结构和对tracing shaders 的编译。3.9和3.10章分别介绍了新的光线追踪 pipeline state objectsh和 shader tables,他们定义了host和GPU间的数据通信。3.11展示了如何配置和启动光线。最后3.12讨论了对于想要深入研究的读者来说的额外的学习资源。  

DirectX抽象了光线加速结构，与软件渲染器不同，选择此结构是影响性能的关键选择。今天的共识是，边界体积层次结构（BVHs）有着比其他结构更好的特性，因此本章的前半部分将加速结构称为边界体积层次结构（bounding volume hierarchies），即使DirectX不强制要求使用BVH。 第3.8.1节详细说明了加速结构的初始化
## 3.3	开始
想要开始搭建DirectX 光线追踪的应用程序，你需要一些标准工具。DXR近能运行在 Windows 10 RS5(或者更新的)，或者称作1809版本或者2018 10月更新。检查你的windwos 版本通过运行 winver.exe,或者打开Settings → System → About.

在确认了你的操作系统之后，安装并且更新包括DXR的头文件和静态库的windows SDK。这需要 windows 10SDK 10.0.17763.0或更高，这可能叫做windows 10 1809 版本。你需要visual studio 或者其他类似的编译器。不管是专业版还是免费的社区版的Visual studio 2017都可以.
最后，光线追踪要求GPU支持Directx12(检查 dxdiag.exe)。拥有硬件加速的光线追踪对于复杂场景和高分辨率的情况下拥有着更好的表现。在旧的GPU上每个像素跟踪几条射线可能是可行的，特别是当使用简单的场景或较低的分辨率时。由于各种原因，光线追踪要比光栅化的渲染器需要更多的内存。内存较少的硬件可能会表现出糟糕的性能。

## 3.4	 DirectX的光线追踪管线
传统的GPU光栅化管线包含许多开发者可以编写传统的shader去控制图像生成的可编程的阶段。DXR介绍了一种新的光线图元和灵活的光线数据存储（见3.5.1）加上5个新的着色阶段，如图3.2中的简化管道图所示。这些着色器允许发射光线，控制光线/几何交点，并遮挡已识别的命中。

![3.2：一个新的DirectX光线跟踪管道的简化视图，包括五个新的着色器阶段(蓝色)：the ray generation, intersection, any-hit, closesthit, and miss shaders。复杂性发生在遍历循环中(大的灰色轮廓，大部分图)，根据边界体积节点和潜在的命中点对光线进行测试，并命令确定最接近的命中点。否未显示为从最接近命中和未命中着色器到Traceray()的潜在递归调用。](./image/3.2.jpg)


* 1.ray generation shader在管线的一开始，允许开发者使用内置的`TraceRay()`着色函数，确认那个光线将要发射。和传统的计算着色器一样，它在常规的一维、二维或三维样本网格上执行。
* 2.Intersection shaders，定义了对于光线和任意图元相交的计算。它为三角形和光线的相交提供了更好的性能。


* 3.Any-hit shaders[^1]允许可控地丢弃其它有效的交叉点，例如，在纹理查找后忽略alpha遮罩的几何体。
*尽管名称角这个，any-hit shaders每个交叉点不会每一次都运行，主要是出于性能原因。 默认情况下，它们可以运行每条光线的变量，与实现相关的次数。 仔细阅读规范以了解和控制更复杂用例的行为。*

* 4.closest-hit shader在沿每条光线的最近的交叉点处执行。 通常，这会计算交叉点处的颜色，类似于栅格化管道中的像素着色器。
* 5 当光线错过场景中的所有几何图形时，miss shader就会执行。例如，这允许查找环境映射或动态天窗模型。

正如您在介绍性教材[^9]中所发现的那样，请思考右侧的伪代码以获得简单的CPU光线跟踪器。 代码循环输出图像，为每条光线设置方向，遍历加速结构，在重叠的加速结构节点中计算几何交叉，查询这些交叉点是否有效，以及对最终结果进行着色
```
for x, y ∈ image.dims() do
    [1] ray = computeRay(x, y);
    closestHit = null;
    while
    leaf = findBvhLeafNode(ray, scene)
    do
        [2] hit = intersectGeometry(ray,leaf);
        if isCloser(hit, closestHit) then
            if [3] isOpaque(hit) then
                closestHit = hit;
    if closestHit then
        [4] image[x,y] = shade(ray,closestHit);
    else
        [5] image[x,y] = miss(ray);
```
至少对于标准的使用用例，新的DXR着色器上面这个简单的光线追踪器的部分内容具有对应关系。射线生成着色器的发射尺寸对应于图像尺寸。相机计算生成每个像素的光线发生在光线生成着色器中。

当光线遍历边界体积层次结构时，叶子节点中图元的实际交点在逻辑上出现在DirectX交点着色器中（intersection shaders），并且可以在 any-hit shader着色器中丢弃检测到的交点。 最后，一旦光线通过加速结构完成其遍历，它将在最近命中的着色器中着色或在未命中着色器中给出默认颜色。
## 3.5	新的支持DirectX光线跟踪的HLSL
1.Ray traversal functions 产生光线并允许控制它们的执行。
2.Launch introspection functions查询启动维度并识别当前线程正在处理的光线（或像素）。 这些函数在任何光线跟踪着色器中都有效。
3.Ray introspection functions 查询光线参数和属性，并且只要你输入光线（除光线生成着色器之外的所有光线跟踪着色器）都可以使用。
4.Object introspection functions查询对象和实例属性，只要您有输入图元（intersection, any-hit, and closesthit shaders），我们就能够使用它。
5.Hit introspection functions查询当前交集的属性。 属性主要由用户定义，因此这些功能允许在操作和命中着色器之间进行通信。 这些函数仅在 any-hit and closest-hit shaders中可用。
### 3.5.1 在HLSL中发射一种新的射线
最重要的新函数TraceRay()会发起一条ray。从逻辑上讲，这类似于纹理获取：它暂停着色器以获得一个可变(可能很大的)数目的GPU时钟。当结果可用于进一步处理时恢复执行。 光 Ray generation, closest-hit, and miss shaders都可以调用TraceRay（）。 这些着色器可以为每个线程发射零个，一个或多个光线。 基本光线发出的代码如下所示：
```
RaytracingAccelerationStructure scene ; // Scene BVH from C ++
RayDesc ray = { rayOrigin , minHitDist , rayDirection , maxHitDist };

 UserDefinedPayloadStruct payload = { ... < initialize here > ... };

TraceRay ( scene , RAY_FLAG_NONE , instancesToQuery , // What geometry ?
     hitGroup , numHitGroups , missShader , // Which shaders ?
    ray , // What ray to trace ?
    payload ); // What data to use ?
```
 UserDefinedPayloadStruct 包含在光线的生命周期内持续存在的每个射线数据。 在遍历期间使用它来维护光线状态并从`TraceRay（）`返回结果。 DirectX定义`RayDesc`结构以存储光线原点，方向以及最小和最大命中距离（命令包装在两个`float4`中）。 将忽略指定间隔之外的光线的交点。 加速结构通过主机API定义（参见第3.8.1节）。

第一个`TraceRay()`参数选择包含几何形状的BVH。简单的光线追踪器通常使用单一的BVH，但是独立地查询多个结构可以允许对于不同的几何类有着不同的行为(例如，透明/不透明，动态/静态)。第二个参数包含改变光线行为的标志，例如，指定对光线有效的附加优化。第三个参数是一个整数实例掩码，它允许每个基于实例的掩码跳过几何体；使用0xFF能够测试所有的几何图形。

第四个和第五个参数有助于选择要使用的命中组（hit group）。  intersection, closest-hit, and any-hit shader（其中一些可能为空）组成。 使用哪个集合取决于这些参数以及测试的几何类型和BVH实例。 对于基本光线跟踪器，每种光线类型通常有一个命中组：例如，主光线可能使用命中组0，阴影光线使用命中组1，全局照明光线使用命中组2.在这种情况下，第四个参数选择光线的类型，第五个指定不同类型的数量。

第六个参数指定要使用的miss shader。 这只是到加载的miss shader列表的索引。 第七个参数是要跟踪的光线，第八个参数应该是此光线的用户定义的持久性有效负载结构（user-defined persistent payload structure）.

### 3.5.2 在HLSL控制光线遍历
除了在光线启动时指定标志外，DirectX还提供了三个附加功能来控制intersection shaders和any-hit shaders中光线的行为。 在 自定义的intersection shaders中调用`ReportHit（）`以标识光线到达图元的位置。 这方面的例子如下：
```
if ( doesIntersect ( ray , curPrim ) ) {
    PrimHitAttrib hitAttribs = { ... < initialize here > ... };
    uint hitType = <user - defined - value >;
    ReportHit ( distToHit , hitType , hitAttribs );
 }
```
`ReportHit（）`的输入是到光线到交点的距离，用户可定义的指定命中的类型的整数，以及用户可定义的命中的属性结构。 命中类型可以通过`HitKind（）`转换为8位无符号整数来用于 hit shaders。 它可用于确定光线/图元间的属性，例如面部方向，但由于它是用户定义的，所以有着很高的自由度。当内置的三角形命中器发出报告时，`HitKind（）`返回`D3D12 HIT KIND TRIANGLE FRONT FACE`或`D3D12 HIT KIND`。 命中属性作为参数传递给any-hit and closest-hit shaders。 当使用三角形命中器时， hit shaders使用BuiltInTriangleIntersectionAttributes类型的参数。
另外，请注意，如果命中点是迄今为止遇到的最接近的命中，ReportHit()将返回true。调用any-hit shaders中的函数IgnoreHit()来停止处理当前的顶点。这会将返回到intersection shader(并且`ReportHit()`返回`false`)，并且行为类似于光栅中的丢弃调用(discard call)，但保留了对光线有效载荷的修改。

在any-hit shaders调用`AcceptHitAndEndSearch()`，它接受当前命中信息，跳过任何未被搜索的BVH节点，并立即在closest-hit shader使用这个当前最接近的命中。 这对于优化阴影射线遍历很有用，因为这些光线只是确定是否有任何东西被击中而不会触发更复杂的着色和光照评估。
### 3.5.3其他HLSL内置函数
所有光线跟踪着色器都可以分别使用`DispatchRaysDimensions（）`或`DispatchRaysIndex（）`查询当前光线发射尺寸和线程索引。 请注意，两个函数都返回一个`uint3`，因为光线发射可以是一维，二维或三维的。

 对于内置，`WorldRayOrigin（）`，`WorldRayDirection（）`，`RayTMin（）`和`RayFlags（）`分别返回提供给`TraceRay（）`的原点，方向，最小遍历距离和光线标记。 在any-hit shaders和closest-hit shaders，`RayTCurrent（）`返回当前命中的距离。 在 intersection shader，`RayTCurrent（）`返回到最近命中的距离（在着色器执行期间可能会改变）。 在miss shader期间，`RayTCurrent（）`返回指定给`   TraceRay（）`的最大遍历距离。

 在intersection, any-hit, and closest-hit shaders中，可以使用许多object introspection 内联函数：
 * `InstanceID()`返回当前实例的用户定义标识符。
 * `InstanceIndex()`和`PrimediveIndex(`)返回系统为当前实例和原语定义的标识符。
 * `ObjectToWorld3x4()`和`ObjectToWorld4x3()`是从对象空间到世界空间。
 
 * `WorldToObject3x4()`和`WorldToObject4x3()`将矩阵从世界空间返回到对象空间。
 * `ObjectRaydirection()`和`ObjectRayOriue()`提供转换为实例坐标空间的光线数据

## 3.6	一个简单的HLSL光线跟踪示例

要提供更实际的实际工作示例，请考虑以下HLSL代码段。 它定义了一个由函数`ShadowRay（）`实例化的光线，如果光线被遮挡则返回0，否则返回1（即，一个“光线”。由于`ShadowRay（）`调用`TraceRay（）`，它只能在 ray generation,，closest-hit,or miss shaders。逻辑上，当我们明确地知道光线未被遮挡时，光线假定它被遮挡，除非miss shaders被执行，这使我们能够避免执行closest-hit（RAY FLAG SKIP） 并在遮挡发生（RAY FLAG ACCEPT FIRST HIT AND END SEARCH)）任何击中后停止。
```
RaytracingAccelerationStructure scene; // C ++ puts built BVH here

struct ShadowPayload { // Define a ray payload
	 float isVisible; // 0: occluded , 1: visible
	
};

[shader(" miss ")] // Define miss shader #0
 void ShadowMiss(inout ShadowPayload pay) {
	pay.isVisible = 1.0 f; // We miss ! Ray unoccluded
	
}

[shader(" anyhit ")] // Add to hit group #0
 void ShadowAnyHit(inout ShadowPayload pay,
	 BuiltInTriangleIntersectionAttributes attrib) {
	 if (isTransparent(attrib, PrimitiveIndex()))
		 IgnoreHit(); // Skip transparent hits
	
}

 float ShadowRay(float3 orig, float3 dir, float minT, float maxT) {
	 RayDesc ray = { orig , minT , dir , maxT }; // Define our new ray .
	 ShadowPayload pay = { 0.0 f }; // Assume ray is occluded
	 TraceRay(scene,
		 (RAY_FLAG_SKIP_CLOSEST_HIT_SHADER |
			 RAY_FLAG_ACCEPT_FIRST_HIT_AND_END_SEARCH),
		  xFF, 0, 1, 0, ray, pay); // Hit group 0; miss 0
	 return pay.isVisible; // Return ray payload
	
}
```

请注意，此代码使用自定义编写的isTransparent()函数来查询材料系统(基于基本ID和命中点)来执行alpha测试。有了这个地方，影子射线就可以发射了。由其他着色器投射；例如，简单的环境遮挡渲染器看起来如下：

```
Texture2D < float4 > gBufferPos, gBufferNorm; // Input G - buffer
RWTexture2D <float4 > output; // Output AO buffer

[shader(" raygeneration ")]
void SimpleAOExample() {
	uint2 pixelID = DispatchRaysIndex().xy; // What pixel are we on ?
	float3 pos = gBufferPos[pixelID].rgb; // AO rays from where ?
	float3 norm = gBufferNorm[pixelID].rgb; // G - buffer normal
	float aoColor = 0.0 f;
	for (uint i = 0; i < 64; i++) // Use 64 rays .
		aoColor += (1.0 f / 64.0 f) * ShadowRay(pos, GetRandDir(norm), 1e - 4);
	output[pixelID] = float4(aoColor, aoColor, aoColor, 1.0 f);

}
```

GetRandDir（）函数返回由曲面法线定义的单位半球内随机选择的方向，传递给ShadowRay（）的$1e^{-4}$ minT值是一个偏移量，有助于避免自相交（有关更多高级选项，请参阅第6章）
## 3.7	Overview of Host Initialization for DirectX Raytracing

## 3.8	Basic DXR Initialization and Setup

## 3.9	Ray Tracing Pipeline State Objects

## 3.10	Shader Tables

## 3.11	调度光线
完成3.8-3.10节中的步骤后，我们终于可以追踪光线了。 由于着色器表具有任意，灵活的布局，我们需要在光线跟踪开始之前使用D3D12 DISPATCH RAYS DESC描述我们的表。 此结构指向着色器表GPU内存，并指定要使用的光线生成着色器，错过着色器和命中组。 此信息使DXR运行时能够计算着色器表记录索引（在第3.7.1和3.10节中描述）。

接下来，指定光线调度大小。 与计算着色器类似，光线调度使用三维网格。 如果以二维方式调度光线（例如，对于图像），请确保深度尺寸设置为1; 默认初始化将其设置为零，这将产生无效。 配置着色器表指针和调度维度后，使用新的命令列表函数SetPipelineState1（）设置RTPSO，并使用DispatchRays（）生成光线。 以下是一个示例：


```
// Describe the ray dispatch .
 D3D12_DISPATCH_RAYS_DESC desc = {};

 // Set ray generation table information .
desc.RayGenerationShaderRecord.StartAddress =


shdrTable->GetGPUVirtualAddress();
desc.RayGenerationShaderRecord.SizeInBytes = shaderRecordSize;

// Set miss table information .
uint32_t missOffset = desc.RayGenerationShaderRecord.SizeInBytes;
desc.MissShaderTable.StartAddress =
shdrTable->GetGPUVirtualAddress() + missOffset;
desc.MissShaderTable.SizeInBytes = shaderRecordSize;
desc.MissShaderTable.StrideInBytes = shaderRecordSize;

// Set hit group table information .
uint32_t hitOffset = missOffset + desc.MissShaderTable.SizeInBytes;
desc.HitGroupTable.StartAddress =
shdrTable->GetGPUVirtualAddress() + hitGroupTableOffset;
desc.HitGroupTable.SizeInBytes = shaderRecordSize;
desc.HitGroupTable.StrideInBytes = shaderRecordSize;

// Set the ray dispatch dimensions .
desc.Width = width;
desc.Height = height;
desc.Depth = 1;

commandList->SetPipelineState1(rtpso); // Set the RTPSO .
 commandList->DispatchRays(&desc); // Dispatch rays !

```

## 3.12	深入挖掘和追加资源
在本章中，我们尝试概述DirectX光线跟踪扩展以及它们背后的相应心理模型。特别是，我们专注于使用DXR启动和运行所需的着色器和主机端代码的基础知识。无论您是编写自己的DirectX主机端代码还是有一些库（例如Falcor）为您提供，从这一点开始使用光线跟踪变得更加容易：基本设置完成后，添加更多光线跟踪效果通常就像更改几行着色器代码一样简单。

显然，我们有限篇幅的介绍性章节无法深入探讨。我们鼓励您探索提供基本DirectX基础结构代码，示例，最佳实践和性能提示的各种其他资源。

SIGGRAPH 2018课程“DirectX光线跟踪简介”[12]可在YouTube上获得，并提供深入的DXR着色器教程[11]，使用Falcor框架[2]抽象低级DirectX细节，让您专注于核心轻型运输细节。这些教程介绍了基础知识，例如打开窗口，创建简单的G缓冲区，使用环境遮挡渲染以及用于抗锯齿和景深的高级相机模型，直至全反射全局照明。图3.4显示了使用教程代码呈现的几个示例。

其他有用的教程包括那些侧重于低级主机代码的教程，包括激发本章后半部分的Marrs'API示例[3]，Microsoft的入门DXR示例[6]以及Falcor团队的低级示例[1]。此外，NVIDIA在其开发者博客上提供了各种资源，包括其他代码示例和演练[8]。
## 3.13	总结

我们已经介绍了DirectX光线跟踪的基本概述，我们希望能帮助你揭示使用DirectX组合基本硬件加速光线跟踪器所需的概念，此外还提供指向其他资源的网址以帮助您入门。
着色器模型类似于先前的光线跟踪API，并且通常干净地映射到传统CPU光线跟踪器的片段。主机端编程模型最初可能看起来复杂且不透明;请记住，设计需要支持任意的，大规模并行的硬件，这些硬件可能会产生着色器，而不会产生沿每条光线的连续执行历史记录的好处。新的DXR管道状态对象和着色器表有助于指定数据和着色器，因此这些GPU可以随着光线穿过场景而任意生成工作。

鉴于DirectX 12的复杂性和光线跟踪的灵活性，我们无法完全覆盖API。我们的目标是提供足够的信息以便入门。在定位更复杂的渲染时，您需要参考DXR规范或其他文档以获得进一步的指导。特别是，更复杂的着色器编译，默认管道子对象设置，系统限制，错误处理以及最佳性能提示都需要其他参考。

我们的入门建议：简单地开始。关键问题围绕正确设置光线跟踪管道状态对象和着色器表，这些使用更少，更简单的着色器更容易调试。例如，使用光栅化G缓冲器进行主要可见性的基本光线跟踪阴影或环境遮挡是良好的起点。

使用DirectX光线跟踪和现代GPU，射击光线比以往任何时候都快。但是，光线追踪并不是免费的。至少在不久的将来，您可以假设每个像素最多只有几条光线。这意味着混合光线栅格算法，抗锯齿，去噪和重建对于快速实现高质量渲染至关重要。本书中的其他工作提供了有关其中一些主题的想法，但许多问题仍未解决。
## References
[^1]	Benty,	N.	DirectX	Raytracing	Tutorials.	https://github.com/ NVIDIAGameWorks/DxrTutorials, 2018. Accessed October 25, 2018.

[^2]	Benty, N., Yao, K.-H., Foley, T., Kaplanyan, A. S., Lavelle, C., Wyman, C., and Vijay, A. The Falcor Rendering Framework. https:
//github.com/NVIDIAGameWorks/Falcor, July 2017.

[^3]	Marrs, A. Introduction to DirectX Raytracing. https://github.com/ acmarrs/IntroToDXR, 2018. Accessed October 25, 2018.

[^4]	Marschner, S., and Shirley, P. Fundamentals of Computer Graphics, fourth ed. CRC Press, 2015.

[^5]	Microsoft.	Programming Guide and Reference for HLSL.	https:
//docs.microsoft.com/en-us/windows/desktop/direct3dhlsl/ dx-graphics-hlsl. Accessed October 25, 2018.

[^6]	Microsoft.	D3D12	Raytracing	Samples.	https://github.com/
Microsoft/DirectX-Graphics-Samples/tree/master/Samples/Desktop/ D3D12Raytracing, 2018. Accessed October 25, 2018.

[^7]	Microsoft. DirectX Shader Compiler. https://github.com/Microsoft/ DirectXShaderCompiler, 2018. Accessed October 30, 2018.

[^8]	NVIDIA. DirectX Raytracing Developer Blogs. https://devblogs.nvidia. com/tag/dxr/, 2018. Accessed October 25, 2018.

[^9]	Shirley, P. Ray Tracing in One Weekend. Amazon Digital Services LLC, 2016. https://github.com/petershirley/raytracinginoneweekend.

[^10]	Suffern, K. Ray Tracing from the Ground Up. A K Peters, 2007.

[^11]	Wyman, C. A Gentle Introduction To DirectX Raytracing. http://cwyman. org/code/dxrTutors/dxr_tutors.md.html, 2018.

[^12]	Wyman, C., Hargreaves, S., Shirley, P., and Barre-Brisebois,´ C. Introduction to DirectX Raytracing. SIGGRAPH Courses,
2018.	http://intro-to-dxr.cwyman.org, https://www.youtube.com/ watch?v=Q1cuuepVNoY.
