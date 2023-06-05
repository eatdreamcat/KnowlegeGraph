# [Graphics Rendering
图形渲染](https://www.realtimerendering.com/)

## Programming
编程

### C++

- Memory Partition
内存分区

### HLSL

- Texture

	- Sampling

		- Range (0~1)

			- Texture Addressing

				- WrapMode

					- Clamp

					- Repeat

					- Mirror

					- ...

			- Texture Filtering

				- Point

				- Bilinear

				- Trilinear

	- Load

		- Interger Indexing

		- Data of interger coordinate

- Sampler

## Hardware
硬件

### Pipeline Stage
管线阶段

- Abstract Stage

	- Geomtry
几何阶段

	- Raster
光栅化

	- Pixel/Fragment
片元阶段

		- 2x2 pixel quad as a unit, datas are shared like ddx and ddy
以2x2像素为一个单位进行的，在这个范围内数据都是共享的，主要是为了低成本计算ddx，ddy

	- Testing/Blending
测试混合阶段

		- Happened in ROP
发生在RenderOutProcess阶段

		- Atomically Operations
测试混合都是原子操作，

- Shader Model

	- Traditional

	- Shader Model 6.5

- Compute

### Infrastruction
基础架构

- GPU microarchitecture
微架构

	- [Pipeline Architecture Image](https://developer.nvidia.com/content/life-triangle-nvidias-logical-pipeline)

	- Logic Pipeline
逻辑管线

	- Logic Threads Hierarchy

		- Threads

			- Pixel

				- Core

		- Group

			- SM

				- Shared memory / L1 Cache

		- Grids

			- GPCs

				- L2 Cache / GMemory

	- Computation Hierarchy
计算层级结构

		- GPC(Graphics Processing Cluster) x4(maxwell)
以maxwell架构为例，有4个GPC

			- Raster Engine

				- decide by bounding box of triangle
根据三角形覆盖屏幕的区域，一个三角形可能被多个GPCs处理

				- face culling / z-cull

					- z-cull

						- 16*16 pixel as a unit

						- no z writing

						- TBR

						- in raster engine, is before early-z

				- early-z

					- 2x2 pixel as a unit

					- z writing

					- before pixel and after raster

				- 2x2 pixel quad as a unit work in pixel shaders

					- pixel thread outside triangle will be masked out(gl_HelperInvocation) which cause underutilizing
由于是按照2x2为单位的，因此可能存在一些片元在三角形外，但是线程还是会被占用，通过mask控制线程是否空跑。一定程度上影响了线程使用率。

						- if pixel is outside the triangle, gl_HeperInvocation = true

			- TPC (2xSMs)

				- SMs(Streaming Multiprocessor) eg:Fermi

					- 1 x PolyMorph Engine

						- Vertex Fetch

							- from triangle indices

						- Tessellator

						- Viewport Transform

							- clipping triangles

						- Attribute Setup

							- set the interpolants in pixel shader friendly format

						- Stream Output

					- 1 x Instruction Cache
指令缓存

					- 2 x Wrap Scheduler
线程束调度器

					  2 wrap scheduler means that we can dispatch two diff steps in one clock --- two lock-step
					  
					  
						- 32 threads as wrap
N卡中以32个线程为一个线程束

						- wrap cant be individual and independent of each other
每一个线程束都不能再被划分，且相互独立

						- completed at once or take several turns
根据指令耗时不同，有些线程束可能一轮派发就执行完，有些则会因为stall被调度多次，其中上下文都保存在register file中

							- wrap switch while memory loads or sf execute, switching wrap can make latency hiding.
当线程束执行到比较耗时需要等待的指令时，比如内存加载，这时候会被切换到另一个Wrap继续执行，通过Wrap的切换，可以隐藏掉这些延迟

							- threads has own registers in the register-file(on-chip) making switching fast
因为register file是on-chip的，因此这种线程束的切换，可以在一个时钟周期完成

					- 2 x Dispatch Unit
线程任务派发单元

					- 1 x Register File( 32768 x 32-bits )

						- on-chip(makeing switching fast)

						- limit space (more registers a shader program needs, the less threads/wraps have space)
当shader中对于寄存器的使用越多，因为register file空间有限， 则划分的线程束数量就比较少

							- less wraps cause less latency hiding
较少的线程束就意味着比较少的切换选择，不可避免会有一些等待的情况，会导致GPU利用率下降，延迟变高

					- 32 x Cores
核心计算单元

						- [Unified Processor
通用处理单元，在早期，DX9时代， vertex跟pixel的处理单元在物理上是独立的，并且个数不同。在DX10之后，统一成streaming processor，在运行时动态分配vertex跟pixel的processor个数。](https://download.nvidia.com/developer/cuda/seminar/TDCI_Arch.pdf)

							- why unified？

								- bad utilization in non-unifiedarchitecture
在非统一的架构中，核心利用率比较低

								- optimal utilization

						- Vector ALU

						- Scalar

							- up to 2x effective

					- 16 x LD/ST
数据读写单元

					- 4 x SFU
特殊函数计算单元

						- sin

						- cos

						- log

						- exp

					- 1 x 64 KB Shared Memory / L1 Cache
共享缓存，Shared跟L1在物理上一个位置，逻辑上做了区分，L1由硬件控制，Shared Memory的使用可由程序控制

					- 1 x Uniform Cache

					- 4 x Tex

					- 1 x Texture Cache

					- Tensor Core 

					- RT Core
用于光追加速

		- Crossbar

			- work migration across GPCs 

		- L2 Cache
GPC间共享缓存

		- Memory Controller
用于控制DRAM的读写

	- Memory Hierarchy
内存层级结构

- Flynn
弗林分类

  弗林分类中，SIMT其实被归为SIMD的一种拓展。
  SMIT is an extension of SIMD in Flynn Classification.
  
	- SIMD

		- single instruction multiple data

		- vector/matrix calculate 

			- Core - Vector ALU/ Scalar is better.

	- SIMT

		- single instruction multiple thread

		- wrap / wavefront - threads within processed in lock-step 

			- SM -  Threads Wrap

		- shared control logic leaves more space for ALUs

- Latency and Throughput
延迟与吞吐量

	- Limited Count of transistors

		- CPU

			- minimize latency

			- flow control

			- large cache

		- GPU

			- high thoughput

			- tolerate latency

		- trade-off

### Pipeline Architecture
管线架构

- IMR

- TBR

- TBDR

### Graphics API
图形API

- DX12

	- Features

		- Command queues and lists

		- Descriptor Tables

		- Concise PSO

		- VRS(Varible Rate Shading)

			-  adjusting the shading rate for different parts of a scene

		- Volume Tiled Resources

		- Conservative Raster

		- Raster Order View

		- Tiled Resources

		- Typed UAV Access

		- Bindless Textures

			- ShaderModel 4.0

			- access textures directly, without bind to limited number of image unit.

			- reduce overhead of gl driver resources binding management

		- Asynchronous Compute

		- Depth Bounds Testing

		- Sampler Feedback

		- Mesh Shaders

			- Base on Group

			- Access to Shared Memory

			- More control over actual hw execution

		- Programmable MSAA

	- Infrastructure 

		- Device

			- FeatureLevel

			- Adapter

		- Command

			- CommandQueue

				- Direct

				- Compute

				- Copy

				- Video

					- Video Encode

					- Video Decode

					- Video Process

			- CommandList

				- consider as interface , can reset with different commandAlloctor

				- Threads?

			- CommandAlloctor


				- Memory Block

				- Frame Buffers

		- Synchronize

			- CPU / GPU

				- Fence

				- Barrier

			- Inside GPU

		- Resources

			- DescriptorHeap / View

				- describe how hardware should view a resources

				- Type

					- RTV(RenderTargetView)

					- DSV(DepthStencilView)

					- SRV(ShaderResourceView)

					- UAV(UnorderAcessView)

					- CBV(ConstantBufferView)

					- Sampler

					- NumberType?

				- Sizeof

				- Address

		- SwapChain

			- RenderQueue

			- Format

				- Typeless

				- Int

				- Float

				- ...

			- BufferUsage

				- CPU Access

					- None

					- Dynamic

					- Read Write

					- Scratch

					- Field

				- Shader Input

				- Shader

				- Readonly

				- Discard on present

				- Unorder Access

				- RenderTarget Output

				- BackBuffer

			- SwapEffect

				- Discard

				- Sequential

				- Flip Sequential

				- Flip Discard

			- Scaling

				- Stretch

				- None

				- Aspect Ratio Stretch

			- Alpha Mode

				- Unspecified

				- Premultiplied

				- Straight

				- Ignore

				- Force DWORD

		- Pipeline

			- Traditional Stage

				- Input Layout

					- VertexBufferView

					- IndexBufferView

				- Vertex Shader

				- Raster State

				- Pixel Shader

				- Output Merge

					- Render Target View(s)

					- Depth/Stencil View

			- Shader Model 6.5

		- Root Signature

			- Parmeter

				- Type

					- DescriptorTable

						- Bindless

					- 32Bit Constants

					- Descriptor

						- CBV

						- SRV

						- UAV

				- Shader Visiblity

				- Union

					- DescriptorTable

					- Constants

					- Descriptor

			- Static Sampler

			- Flags

		- PipelineState

			- InputLayout

			- RootSignature

			- ShaderByteCode

				- BinaryBlob

			- RasterState

			- BlendState

			- Primitive

			- NumRenderTarget

			- Subobject

			- ...

- Vulkan

- Metal

### Vendor
产商

- Adreno
高通

- AMD

- Nividia

## Theory
理论

### Mathematics
数学

- Algebra
代数

	- MVP

		- Model

			- Scale

			- Rotation

				- LeftHand-Clockwise   (Unity C#/DX) 
左手坐标系下，正方向是顺时针（旋转轴正方向看向原点，也就是沿着轴负方向看）。 例如Unity坐标系

				- RightHand-Anti-clockwise （OpenGL/ Unity Shader）

			- Offset

		- View

		- Projection

	- Why homogeneous coordinates?

		- for affine transform --- offset

- Calculus
微积分

	- Monto Carlo
蒙特卡洛

		- Importance Sampling
重要性采样

			- cosine-weight

		- Multiple importance Sampling
多重重要性采样

- Statistics
统计学

- Geomtry
几何学

### 
Optics Theory
光学理论

- Geomtry Optics
几何光学

- Wave Optics
波动光学

### Radiometry
辐射度量学

### Colorimetry
色度学

- Color Space
颜色空间

- HDR/LDR

### Rendering Equation
渲染方程

### Shading Model
着色模型

- Empirical Model
经验模型

- [Physic Based Rendering
基于物理的渲染](https://pbr-book.org/3ed-2018/contents)

	- Scope of PBR

		- Physic Based Camera

		- Physic Based Material

		- Physic Based Lighting

	- Rendering Equation and BxDF

		- Optical Interactive    

			- Reflection

			- Transmission

				- Refraction

				- Diffusion

			- Scattering / Diffusion

			- Absorption

		- BSDF (Bidirction Scattering Distribution Function)

			- BSSRDF (Bidirection Subsurface Scattering Reflection Distribution Function)

				- BRDF

				- SSS

			- BSSTDF (Bidirection Subsurface Scattering Transmission Distribution Function)

				- BTDF

				- SSS

	- Key Points

		- Microfacet Theory
微平面理论

		- Fresnel Reflectance
菲涅尔反射

		- Energy Conservation
能量守恒

		- Substance Optical Properties

			- Metal

			- Non-Metal

		- Linear-Space & Gamma-Space

		- Tone Mapping

		- Light Interaction with Non-Optical-Flat Surfaces

	- Core Theory

		- Cook-Torrance BRDF

			- Microfacet

			- Energy Conservation

			- Fresnel

		- Disney Principled PBS

			- BRDF

				- Based Principle

					- Art orientation, not always physics correct

					- Directly Parmeters

					- Parmeters Range 0 ~ 1

				- BRDF

					- Diffuse

						- Disney Diffuse

					- Specular

						- Microfacet Cook-Torrance BRDF

							- Distribute Function

								- Normal Distribution

									- Generalized Trowbridge-Reitz (GTR)

							- Geomtry Function

								- Scope

									- Geomtry Visibility

									- View Visibility

								- Smith-GGX

							- Fresnel Term

								- Schlick

			- BSDF

	- Based on assumption Geomtry Optical

		- Light is a Ray

### XR

## Rendering
渲染

### Occulsion
遮蔽

- AO
环境光遮蔽

	- SSAO

		- Use Depth/Normal construct a hemi-sphere

		- Sampling and calculate percentage

	- HBAO

- Shadow
阴影

	- Planar Shadow

		- Flatten the model to planar

	- Projector Shadow

	- Shadow Map

		- CSM

		- PCF

		- PCSS

	- Unique Character Shadows

	- Per Object ShadowMap / Cached Shadow Maps

	- Capsule Shadows

	- Contact Shadows

### Lighting
光照

- Path
路径

	- Direct
直接光照

		- one bounce

	- Indirect
间接光照

		- more than one bounce

- Type
类型

	- Glossy

	- Diffuse

	- Specular

	- Emission

	- Reflection

		- IBL

		- SSR

			- Reconstruct World Position

			- Calculate Reflection Direction

			- RayMarching (Intersection Judgement by Compare the depth)

				- Acceleration by Mipmap

			- Drawback

				- OnlyScreenSpaceContent

				- No backface reflection

				- No suitable for big area

		- Planar Reflection

			- Only suitable for relatively flat planar

			- need to rendering a relflection map

			- extra cost for blur 

		- PPR

- GI
全局光照

	- LightMapping
光照贴图

	- Spherical Harmonic
球谐

		- characteristic
特性

			- rotation invariance
旋转不变性

			- direction dependent only
只和方向有关（因为是极坐标计算）

			- no distance decay
不会随距离衰减

			- no affacted by occulsion
不受遮挡影响

			- low frenquency information
保存低频信息

		- Application
应用

			- Environment Ligting

			- Irradiance Volume

	- Irradiance Volume

	- PRT

		- 《Tom Clancy's The Division》
《全境封锁》

	- IBL

		- Box Projection

			- CubeMap

				- Optimization

					- Dual-Paraboloid Map

		- Probe Blending

	- ScreenSpaceGI

	- Raytracing

		- Classfication

			- Recursive Raytracing

			- Path Tracing

			- Ray Casting

				- need a real mesh

			- Ray Marching

				- SDF

				- implict mesh

			- Photon Mapping

		- Application

			- Ray-tracing Shadows

			- Ray-tracing Reflection

			- Ray-tracing AO

			- Ray-tracing GI

			- Refraction

- Baked

	- LightMap(Per-Pixel)

		- Type

			- Static

				- ShadowMask

				- Diffuse

					- Direct

					- Indirect

				- AO

				- AHD (Ambient + Hightlight + Direction encoding)

			- Precompute

		- Drawback

	- Probe(Per-Mesh)

		- LightProbe

		- OcclusionProbe

		- Drawback

			- LightBleeding

			- Detail losing

			- Huge Mesh 

				- LPPV / Volumetric Lightmaps

					- Per-Pixel

					- Store Probe Position in 3DTexture

				- Adaptive Probe Volumes

	- ReflectionProbe

### Nature
自然

- Plants
植被渲染

	- GameObject

		- Optimization

			- Cull

				- Frustum Culling

				- Layer Culling

				- Occlusion Culling

			- Batcher

				- GPU Instancing

			- LOD

	- Terrain

		- Tree (Foliage)

		- Grass (Detail)

			- DensityMap

		- Optimization

			- Cull

				- Frustum Culling

				- Layer Culling

				- Occlusion Culling

			- Batcher

				- GPU Instancing

			- LOD

			- HISM

	- Shading

		- Grass

			- Model

				- Billboard

				- ...

			- Weight

				- UV / VertexColor

				- Color Gradient

			- Coloring

			- Winds

- Water
水体渲染

	- Gerstner Wave

	- FlowMap

	- FFT
快速傅里叶变换

	- Fluid Simulation
流体模拟

	- Caustic
焦散

	- Reflection
反射

	- Refraction
折射

- Atmosphere
大气渲染

	- 米氏散射

	- 瑞利散射

- Volume
体积渲染

	- Cloud
体积云

	- Fog
体积雾

	- Light
体积光

- Weather
天气渲染

	- Rain
雨天

	- Thunder
雷暴

	- Snow
雪天

	- Storm
风暴

- Terrain
地形

	- HeightMap

		- Sample per vertex

	- LOD

		- QuadTree (Unity)

			- Cell

				- 17x17

			- Resolution

				- 33x33

				- 65x65

				- 129x129

				- 257x257

				- 513x513

				- ......

		- Binary Triangle Trees

		- Geomtry Clipmap

		- GPU Driven

		- Tear Sealed

			- CDLOD

			- Stitch

			- LockEdge

	- Draw Instancing

		- Same cell 17x17

	- Shading (Unity)

		- Layers

			- ControlMap

				- RGBA - 4 Layers

			- SplatMap

				- RGB - Diffuse

				- Alpha - Smoothness

			- NormalMap

			- Mask

				- R = Metallic, G = AO, B = Height, A = Smoothness

		- Shaders

			- TerrainLit

			- TerrainAdd

			- TerrainBase

			- TerrainBasemapGen

		- VertexPass

			- GPU Instancing Vertex Position

			- Rebuild Normal

			- Calculate UV

		- FragmentPass

			- Hole Clipping

			- UV Calculation

			- SplatMap Mixing

				- 4 Layers

				- TerrainAdd Pass (more than 4 layers)

				- Blend One One

			- Per-Pixel Normal (Optional)

	- Blending

### Humans
人体渲染

- Skin
皮肤

	- SSS
次表面散射

- Eyes
眼球渲染

- Hair
头发渲染

	- Kajiya-Kay

- Cloth
布料渲染

### NPR
风格化渲染

- Chinese Ink
水墨风

- Cartoon
卡通渲染

	- Outline
外描边

- Pencil Style
素描风格

### VFX
特效

- Shield
护盾

- Distortion
扭曲

- Fire
火焰

- Broken
破碎

- Particles
粒子

### Post Processing
后效

- ColorGrading
颜色分级

	- LUT

- Bloom
泛光

- Blur
模糊

	- Gaussian Blur
高斯模糊

	- Radial Blur
径向模糊

	- Motion Blur
动感模糊

- Tonemapping
色调映射

### [Anti-aliasing
抗锯齿](https://www.hp.com/us-en/shop/tech-takes/what-is-anti-aliasing)

- Spatial anti-aliasing
空间上的抗锯齿

	- SSAA

	- MSAA

		- store sub-sample per pixel

		- calculate only one color value per pixel

		- store cost fast than quality improves

	- [CSAA/EQAA](https://developer.download.nvidia.com/SDK/9.5/Samples/DEMOS/Direct3D9/src/CSAATutorial/docs/CSAATutorial.pdf)

		- store boolean converage at 16 sub-samples

- Temporal anti-aliasing

	- TXAA

- Post-process anti-aliasing
后效抗锯齿

	- FXAA/MLAA 

	- SMAA

- AI Enhance

	- DLAA

## Project
工程

### Rendering Path
渲染路径

- Forward
前向渲染

- Deferred
延迟渲染

### GPU Driven
软光栅

### Unity

- SRP Batcher

### UE

- Lumen

- Nanite

### PBR Workflow

- Metallic Workflow

- Specular Workflow

## Optimization
优化

### CPU

- Cache Optimization
缓存优化

	- 
Locality Principle
局部性原理

		- Temporal Locality
时间局部性

		- Spatial Locality
空间局部性

	- Cache Consistency
缓存一致性

- Memory Optimization
内存优化

- Rendering Optimization
渲染优化

	- Multiple Threading
多线程

	- Batching
合批

		- Static Batching

		- GPU Instancing

			- Automatic

				- Uniform MeshInfo

				- Dynamic Gather Data to ConstantBuffer

			- DrawMeshInstanced

				- ConstantBuffer - 64KB

				- No LOD

			- DrawMeshInstancedIndirect

				- ComputerBuffer

					- Much larger than CBuffer

				- ComputerShader

					- GPU Infrustum Culling

					- Hi-Z

				- Require ShaderModel 4.5 

			- DrawMeshInstancedProdual

		- SRP Batcher

		- Dynamic Batcher

	- Culling
剔除

		- Frustum Culling

		- Layer Culling

		- Occlusion Culling

	- LOD
细节分级

### GPU

- Bandwidth
带宽

- GMemory
显存

- Shader Complexity
Shader复杂度

	- if / else  (Bit mask stack)
GPU中的分支，是通过bit mask stack来实现的，因为GPU没法或者说比较难做分支预测，且因为线程束的设计，因此在遇到分支的时候，只能通过空跑一些线程来实现。

		- threads in a wrap executed in lock-step

	- loops with varying iterations
在同一个线程束内的循环，如果线程间循环的迭代器不一致，也会导致利用率下降，本质上也是造成一种分支。

		- cause  underutilizing cores

	- special functions
特殊函数的调用需要依赖SFU去计算，SFU的个数相对普通的Core会少很多，且本身运算过程也比较复杂耗时

	- texture sampling / memory fetches

- LOD
着色分级

- Over Draw
过度绘制

	- Render Queue

	- Early-Z

	- Hi-Z

	- Z-Cull

	- Z-Prepass

### Resources Optimization
资源优化

- Texture Compress
贴图压缩

- Mipmap

- Bindless

