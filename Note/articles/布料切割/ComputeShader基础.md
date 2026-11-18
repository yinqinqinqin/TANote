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

### 第四步 查看**`MyCloth.usf`**

