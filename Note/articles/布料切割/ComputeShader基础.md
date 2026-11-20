# ComputeShader基础

以布料模拟为例

## 布料模拟实现方式

程序化生成CPU回读GPU

WPO 全部在GPU中实现

通常使用wpo实现

## 读取文件

文件目录

```
MyComputeShader/
├── .vscode/
│   └── settings.json
├── Content/
│   └── M_ClothWPO.uasset
├── Resources/
│   └── Icon128.png
├── Shaders/
│   └── Private/
│       └── MyCloth.usf
├── Source/
│   └── MyComputeShader/
│       ├── Private/
│       │   ├── MyClothCS.h
│       │   ├── MyClothWPOActor.cpp
│       │   ├── MyComputeShader.cpp
│       │   └── MyCSDispatcher.cpp
│       ├── Public/
│       │   ├── MyClothWPOActor.h
│       │   └── MyComputeShader.h
│       └── MyComputeShader.Build.cs
├── .gitignore
├── MyComputeShader.uplugin
└── README.md
```

### 第一步 查看.uplugin文件



### 第二步查看.Build.cs文件



```c++
        // 公开头文件中的基础类型、UObject 反射和 Actor 接口。
        PublicDependencyModuleNames.AddRange(
            new string[]
            {
                "Core",
                "CoreUObject",
                "Engine"
            }
        );
```

Public模块

| 模块          | 你的代码为什么用它                               |
| ------------- | ------------------------------------------------ |
| `Core`        | 提供 `FString`、`TArray` 等基础类型              |
| `CoreUObject` | 提供 `UObject` 对象系统和反射支持                |
| `Engine`      | 提供 `AActor`、`UStaticMeshComponent` 等引擎类型 |



```c++
        // 仅实现文件使用的渲染、插件路径和静态网格构建接口。
        PrivateDependencyModuleNames.AddRange(
            new string[]
            {
                "RenderCore",             // 全局 Shader、RDG 和渲染线程命令。
                "RHI",                    // GPU 资源、命令列表和功能级别。
                "Projects",               // IPluginManager 定位 Shader 源码目录。
                "MeshDescription",        // 顶点、顶点实例和多边形拓扑。
                "StaticMeshDescription"   // 静态网格位置、法线、切线和 UV 属性。
            }
```





| 模块                    | 在你的插件里负责什么                                    |
| ----------------------- | ------------------------------------------------------- |
| `RenderCore`            | 注册全局 Shader，用 RDG 调度 Compute Pass               |
| `RHI`                   | 提供 GPU 纹理、缓冲区和命令列表接口                     |
| `Projects`              | 通过 `IPluginManager` 找到插件目录，用来定位 `Shaders/` |
| `MeshDescription`       | 描述布料网格的顶点和三角形连接                          |
| `StaticMeshDescription` | 提供构建静态网格所需的 UV、法线和切线属性               |

Private模块

| 类型                        | 模块                                       | 什么时候需要                                                 |
| --------------------------- | ------------------------------------------ | ------------------------------------------------------------ |
| **Actor 基础依赖**          | `Core`、`CoreUObject`、`Engine`            | 大部分基于 `AActor` 的效果插件都会用到。                     |
| **Compute Shader 常用依赖** | `RenderCore`、`RHI`                        | 你这种用 RDG 调度、读写 GPU Buffer 或纹理的实现，通常都需要这两个。 |
| **按路径定位方式选择**      | `Projects`                                 | 用 `IPluginManager` 查找插件的 `Shaders/` 目录时需要。你的插件用了，所以要留。 |
| **按网格生成方式选择**      | `MeshDescription`、`StaticMeshDescription` | 你的布料用 C++ 生成细分网格，所以需要。若新效果直接用已有平面 Mesh，且代码没有调用这些接口，就可以省掉。 |
| **按渲染接口选择**          | `Renderer`                                 | 直接使用 `Renderer` 模块提供的接口时才加。你当前这套 RDG 调度无需额外加它。 |

### 第三步 查看MyClothWPOActor.h

基础模板

```c++
#include "MyClothWPOActor.generated.h"

UCLASS()
class MYCOMPUTESHADER_API AMyClothWPOActor : public AActor
{
    GENERATED_BODY()

public:
    // ...
};
```

这几部分配合使用：

- **`UCLASS()`** 标记这个类，让 UHT 识别并处理它。
- **`GENERATED_BODY()`** 引入生成的类注册、类型信息等声明，让它接入 UE 的对象系统。
- **`UPROPERTY()`、`UFUNCTION()`** 标记需要 UE 识别的属性和函数，供编辑器、蓝图等功能使用。

这个 `.generated.h` 由构建工具生成，通常放在该头文件所有 `#include` 的最后。你写好这些宏并编译，UE 就会生成配套代码。



**成员函数声明**

主要放置游戏的**Actor 生命周期相关方法声明**，例如蓝图中的Tick，BeginPlay

使用的方法：

| 方法                       | 是否必须自己写                          | 什么时候需要                                                 |
| -------------------------- | --------------------------------------- | ------------------------------------------------------------ |
| `AMyClothWPOActor()`       | 按需写，省略时可由 C++ 生成默认构造函数 | 创建默认组件、设置默认属性。本代码中创建网格组件并加载材质   |
| `OnConstruction()`         | 按需重写                                | 希望在编辑器放置 Actor、修改参数后立即更新预览。本代码中生成布料网格 |
| `Tick(float DeltaSeconds)` | 按需重写，并启用 Tick                   | 需要每帧持续执行操作。本代码中累计时间并提交布料模拟         |
| `BeginPlay()`              | 按需重写                                | 开始游戏时初始化运行资源，例如创建 RT、初始化模拟状态        |
| `EndPlay()`                | 按需重写                                | Actor 离开游戏时清理资源，例如结束 PIE、切换关卡或销毁 Actor |

其余方法：

| 扩展方法                     | 是否必须自己写 | 什么时候需要                             |
| ---------------------------- | -------------- | ---------------------------------------- |
| `PostInitializeComponents()` | 按需重写       | 所有组件初始化完成后，执行依赖组件的设置 |
| `Destroyed()`                | 按需重写       | Actor 被明确销毁时，执行额外处理         |
| `BeginDestroy()`             | 按需重写       | UObject 进入销毁流程时，启动底层资源清理 |

**成员变量声明**

**`UPROPERTY()`、`UFUNCTION()`** 标记需要 UE 识别的属性和函数，供编辑器、蓝图等功能使用。

```c++
// 网格拓扑保持不变，顶点的最终变形由材质计算。
UPROPERTY(VisibleAnywhere, Category = "Cloth")
TObjectPtr<UStaticMeshComponent> ClothMesh;

// 网格列数，也是变形纹理的宽度。
UPROPERTY(EditAnywhere, Category = "Cloth", meta = (ClampMin = "4", ClampMax = "64"))
int32 Columns = 25;

```

第一种只声明组件引用，实际组件在构造函数中通过 `CreateDefaultSubobject` 创建

| 部分                               | 含义                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| `VisibleAnywhere`                  | 属性在详情面板中可见，组件引用本身不可直接替换。组件自身的可编辑属性仍可调整 |
| `Category = "Cloth"`               | 在详情面板中归入 `Cloth` 分类                                |
| `TObjectPtr<UStaticMeshComponent>` | 保存一个静态网格组件的对象引用，配合 `UPROPERTY` 接入 UE 的对象引用追踪和垃圾回收 |
| `ClothMesh`                        | 变量名，用来访问承载布料的组件                               |

第二种变量的属性声明，数值为默认值

| 部分                 | 含义                                              |
| -------------------- | ------------------------------------------------- |
| `EditAnywhere`       | 可以在类默认值和关卡中 Actor 实例的详情面板里修改 |
| `Category = "Cloth"` | 同样归入 `Cloth` 分类                             |
| `meta = (...)`       | 为编辑器提供额外的属性设置                        |
| `ClampMin = "4"`     | 编辑器输入值的下限为 4                            |
| `ClampMax = "64"`    | 编辑器输入值的上限为 64                           |
| `int32`              | 32 位有符号整数                                   |
| `Columns = 25`       | 声明列数变量，并将默认值设为 25                   |

### 第四步 查看MyClothWPOActor.cpp



查看参数定义

```c++
class FMyClothDeformationCS : public FGlobalShader
{
    DECLARE_GLOBAL_SHADER(FMyClothDeformationCS);
    SHADER_USE_PARAMETER_STRUCT(FMyClothDeformationCS, FGlobalShader);

    BEGIN_SHADER_PARAMETER_STRUCT(FParameters, )
        // 网格尺寸、静止间距用于还原未变形顶点位置。
        SHADER_PARAMETER(FIntPoint, GridSize)
        SHADER_PARAMETER(FVector2f, Spacing)
        // 只读取当前粒子状态，并向支持 UAV 的浮点纹理写入结果。
        SHADER_PARAMETER_RDG_BUFFER_SRV(StructuredBuffer<Particle>, InputParticles)
        SHADER_PARAMETER_RDG_TEXTURE_UAV(RWTexture2D<float4>, ClothDeformation)
    END_SHADER_PARAMETER_STRUCT()

    static bool ShouldCompilePermutation(const FGlobalShaderPermutationParameters& Parameters)
    {
        // 与模拟着色器一样，仅在支持 SM5 的平台编译。
        return IsFeatureLevelSupported(Parameters.Platform, ERHIFeatureLevel::SM5);
    }
};
```

只需要修改Begin和End中间的参数，其他地方根据C++的类名进行修改



需要将cpp类拖入场景中进行实例化，随后执行cpp中使用的方法

#### 构造函数 AMyClothWPOActor::AMyClothWPOActor()

#### 构造回调函数 void AMyClothWPOActor::OnConstruction(const FTransform& Transform)

#### 开始游戏回调 void AMyClothWPOActor::BeginPlay()

#### 每帧更新回调 void AMyClothWPOActor::Tick(float DeltaSeconds)

#### 结束游戏回调 void AMyClothWPOActor::EndPlay(const EEndPlayReason::Type EndPlayReason)

#### 其他自定义实现方法

- 定义网格构建函数 void AMyClothWPOActor::BuildGrid()
- 自定义 GPU 任务提交函数 void AMyClothWPOActor::DispatchCloth(int32 Steps, TArray<FVector4f> InitialParticles)







# 引用网页

虚幻官方C++入门文档

https://dev.epicgames.com/documentation/unreal-engine/unreal-engine-cpp-quick-start?lang=zh-CN

Compute Shader 完整教程

https://www.cnblogs.com/ysh11o/p/19674687

个人ComputeShader案例代码

https://github.com/yinqinqinqin/MyComputeShader
