# ComputeShader部署

## 背景

UE 5.8 创建支持C++的模板

下载Visual Studio 代码编辑器

## 第一步 创建插件

1. 打开UE项目，点击左上角编辑-插件-添加
2. 选择Blank模板
3. 填写插件名称（后续不能再更改） **MyComputeShader**
4. 点击创建

此时会弹出vs编辑器，关闭UE编辑器

## 第二步 创建所需要的文件

在UE项目中，创建文件的方式有多种，

- 一种是在VS中右键目录打开菜单，选择新建项和新建依赖项

- 另一种是在资源管理器中直接新建文件和文件夹

- 还有一种在UE编辑器里面新建，在UE编辑器中，新建C++类，并继承父类能够同样实现手动创建文件的作用



在新建之后，都需要在uproject启动文件**点击右键重新编译**才能在项目中看到新建的文件



在VS中的目录栏中找到Plugins - MyComputeShader目录，该路径是插件根目录

在根目录中创建Shader-Private-MyComputeShader.usf （ **这个是CS实现的主要文件**）

在./Source/MyComputeShader目录下面的**Public**目录创建 MyComputeShaderCS.h MyCSDispatcher.h MyCSDispatcher.h

在 ./Source/MyComputeShader目录下面的**Private**目录创建 MyComputeShaderCS.cpp MyCSDispatcher.cpp MyCSBlueprintLibrary.cpp

```
项目根目录/
└── Plugins/
    └── MyComputeShader/                        插件根目录
        ├── MyComputeShader.uplugin
        ├── Shaders/                          ← 手动创建
        │   └── Private/                      ← 手动创建
        │       └── MyComputeShader.usf       ← 手动创建
        └── Source/
            └── MyComputeShader/
                ├── MyComputeShader.Build.cs
                ├── Public/
                │   ├── MyComputeShader.h      (模块头文件)
                │   ├── MyComputeShaderCS.h    ← 手动创建（Shader 声明）
                │   ├── MyCSDispatcher.h       ← 手动创建（调度器声明）
                │   └── MyCSBlueprintLibrary.h ← 手动创建（蓝图接口）
                └── Private/
                    ├── MyComputeShader.cpp     (模块实现)
                    ├── MyComputeShaderCS.cpp   ← 手动创建（Shader 实现）
                    ├── MyCSDispatcher.cpp      ← 手动创建（调度器实现）
                    └── MyCSBlueprintLibrary.cpp← 手动创建（蓝图接口实现）
```

## 第三步 修改.uplugin文件

`.uplugin` 是 UE 的**插件描述文件**，采用 JSON 格式。UE 通过它识别插件、读取插件面板里的显示信息，并决定插件中的 C++ 模块在什么环境、什么阶段加载。

> [!NOTE]
>
> **关键修改是 LoadingPhase**：
>
> **为什么要 `PostConfigInit`？** 因为 `AddShaderSourceDirectoryMapping` 必须在引擎编译着色器之前调用。 `PostConfigInit` 是配置系统初始化之后、引擎完全启动之前的阶段。 如果使用默认的 `Default` 加载阶段，调用时机太晚，着色器目录映射不会生效。

```json
{
    "FileVersion": 3,                                    //.uplugin 描述文件的格式版本。发布插件新版本时通常保持为 3
    "Version": 1,                                        //	插件的整数版本号，发布新版时可以递增。
    "VersionName": "1.0",                    //	插件面板里显示的版本字符串，可以写成 "1.0.1"。它和 Version 分别填写。
    "FriendlyName": "MyComputeShader",                   //	插件面板里显示的名字。插件标识来自 .uplugin 文件名
    "Description": "A simple compute shader plugin",     //插件面板里的简介、
    "Category": "Rendering",                             //分类
    "CreatedBy": "YourName",                             //作者信息。
    "CanContainContent": false, //插件不提供 Content/ 下的 UE 资产，比如材质、蓝图和贴图。插件仍然可以在 Shaders/ 下放 .usf 和 .ush 源文件。
    "IsBetaVersion": false,                             //插件面板不显示 Beta 标记。
    "Installed": false,              //描述符没有把插件标记为已安装的分发插件。项目是否启用它，由项目的插件配置等机制决定。
    "Modules": [                                        //Modules 决定 C++ 模块如何加载
        {
            "Name": "MyComputeShader",
            //Name: "MyComputeShader" 指定模块名，通常对应 Source/MyComputeShader/MyComputeShader.Build.cs，并与 IMPLEMENT_MODULE 中的模块名一致。这里模块和插件恰好同名，一个插件也可以声明多个模块。
            "Type": "Runtime",
            //Type: "Runtime" 表示模块可用于编辑器和打包后的游戏。只供编辑器使用的工具模块通常用 Editor。
            "LoadingPhase": "PostConfigInit"
            //LoadingPhase: "PostConfigInit" 表示 UE 在配置系统初始化后、引擎完全初始化前加载此模块。
        }
    ]
}
```

"Type" 

- Runtime
- Editor

"LoadingPhase"

- Default
- PostConfigInit
- PreDefault

## 第四步 修改Build.cs

打开 `Source/MyComputeShader/MyComputeShader.Build.cs`：

```c++
using UnrealBuildTool;

public class MyComputeShader : ModuleRules
{
    public MyComputeShader(ReadOnlyTargetRules Target) : base(Target)
    {
        PCHUsage = PCHUsageMode.UseExplicitOrSharedPCHs;

        PublicDependencyModuleNames.AddRange(new string[]
        {
            "Core",
            "CoreUObject",
            "Engine",
            "RenderCore",    // FRDGBuilder, FGlobalShader, AddShaderSourceDirectoryMapping
            "RHI",           // FRHICommandList, GMaxRHIFeatureLevel, GPU 资源类型
            "Projects"       // IPluginManager（用于定位插件路径）
        });

        PrivateDependencyModuleNames.AddRange(new string[]
        {
            "Renderer"       // SetComputePipelineState, SetShaderParameters, GetGlobalShaderMap
        });
    }
}
```

