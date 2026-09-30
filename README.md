# UE Restore Tools

将 Unreal Engine 中的几何缓存与受支持的 VAT 动画还原到 Houdini，继续编辑、处理或导出。

本仓库提供两款独立的 Windows 工具：**ABC Restore** 用于 Geometry Cache，**VAT Restore** 用于顶点动画纹理。按资产类型下载即可，两者互不依赖。

## 下载与选择

| 你的资产 | 使用工具 | 下载 |
| --- | --- | --- |
| 已导入 UE 的 Geometry Cache `.uasset` | ABC Restore v2 | [下载 ZIP](./ABCRestore_Tool_v2.zip) |
| 原始 FBX 网格及配套 VAT 贴图 | VAT Restore v2 | [下载 ZIP](./VATRestore_Tool_v2.zip) |

下载后**完整解压**，双击对应文件夹内的 `Launch.vbs`。无需提前打开 UE 或 Houdini，也无需单独安装 Python；运行时仍需本机安装并授权 Houdini，ABC Restore 还需要 Unreal Engine。

> VAT 的贴图名称并不代表统一编码。当前工具支持下文列出的两种材质函数编码，并非任意 VAT 都能直接还原。

## 运行环境

| 条件 | ABC Restore | VAT Restore |
| --- | --- | --- |
| 系统 | Windows | Windows |
| Houdini | 已安装并授权，提供 `hython.exe`，许可证支持 Alembic 导出 | 已安装并授权，提供 `hython.exe` |
| Unreal Engine | 需要安装可读取该资产的版本 | 已取得所需网格、贴图及参数后，解码不需要启动 UE |
| C++ 编译环境 | 使用附带匹配模块时无需编译；其他引擎构建首次使用需要 | 不需要 |

已验证 Houdini **21.0.440**。ABC Restore 已完成 **UE 5.6.1** 自动选择与 **UE 5.8.3** 自动编译、提取测试。其他版本尚未逐一验证。

首次编译 UE 提取模块时，需要所选引擎支持的 **Visual Studio C++ 工具链、Windows SDK 与 UE 构建工具**，且工具目录需可写。

## ABC Restore v2

从 UE 编辑器中的 **Geometry Cache `.uasset`** 读取动画，生成可播放的 **Alembic、Houdini 工程和逐帧 `.bgeo.sc` 缓存**，不需要导入前的原始 ABC 文件。

### 使用步骤

1. 双击 `Launch.vbs`。
2. 选择 Geometry Cache `.uasset`。如有同名 `.uexp`、`.ubulk` 或 `.uptnl`，请保留在旁边。
3. **UE 路径留空**，让工具自动匹配本机引擎；也可手动选择 `Engine/Binaries/Win64/UnrealEditor-Cmd.exe`。
4. 选择 Houdini 的 `bin/hython.exe` 和输出父目录。
5. 点击“开始还原”，等待提取、转换及逐帧验证完成。
6. 打开生成的 Houdini 工程，或将 `Restored.abc` 导入 DCC。

帧率和帧数从缓存采样时间读取。每次任务创建独立结果目录，源资产复制到独立提取工程后处理，不保存回原工程。

### 引擎版本如何匹配

工具分别读取 `.uasset` 包头中的**最后保存版本**与所属 `.uproject` 的**工程关联版本**，再查找本机已安装的 UE。

例如：资产最后保存于 **5.3.2**，但所在工程关联 **5.6**，工具会优先匹配本机 **5.6** 系列引擎。最后保存版本并不代表资产最初的制作版本。

- 版本字段缺失或包头结构未知时，使用可获取的工程版本，或提示手动选择。
- 按引擎 BuildId 匹配提取模块；其他引擎构建首次使用时自动编译，之后复用 `ModuleCache`。
- 不自动下载 UE，不通过修改 BuildId 绕过兼容检查。
- 自动编译不等于支持所有未来版本；UE 接口变化可能需要更新提取源码。

### 输出内容

```text
结果目录/
├── Restored.abc          # Alembic 动画
├── Restored.hip          # Houdini 工程，扩展名可能随许可证变化
├── cache/               # 逐帧 .bgeo.sc 缓存
├── validation.json      # 帧信息与验证结果
└── _extraction/          # 引擎识别记录、提取工程、原始数据及日志
```

移动 Houdini 工程时，请一起移动 `cache` 文件夹。`Restored.abc` 可单独使用。

### 支持范围

- 编辑器中的未烹饪 Geometry Cache，支持 Streamable 轨道、动态拓扑，以及单轨道或同步多轨道的均匀网格采样。
- 保留可用的顶点位置、法线、UV、颜色与材质编号，转换为 Houdini 的米制、Y 向上坐标。
- 导出网格采样时刻的轨道变换和可见性；不额外采样网格帧之间独立的变换或可见性关键帧。
- 不支持从打包游戏资源直接提取，不适用于普通静态网格、骨骼网格或 T3D/COPY 文本。
- 不恢复 UE 材质、关卡 Actor 变换或导入压缩前已丢失的数据。非均匀采样、不同步轨道和其他未支持轨道会报错。

## VAT Restore v2

根据网格、动画纹理及对应解码规则，还原逐帧几何体或 Houdini 粒子。输出动画可脱离 UE 材质播放。

### 支持的模式

| 模式 | 所需输入 | 还原结果 |
| --- | --- | --- |
| Dynamic Remeshing 动态重网格 | 原始三角形 FBX、四通道 Lookup、HDR Position（EXR/HDR） | 动态拓扑逐帧网格，重新计算法线 |
| ParticleSprites 粒子面片 | 原始 FBX、Position、Rotation、材质实例的 Bound Min / Max | 可编辑粒子及朝向指定预览相机的面片 |

动态重网格模式对应已适配的 `MF_DynamicRemeshing` 编码，支持其 255 / 2048 查找表分支。该模式无需 Rotation、Color 或水面法线贴图。

粒子面片模式对应已适配的 `MF_VAT_ParticleSprites_Simplifed` 编码。在此编码中，Rotation 的 RGB 实际存储速度信息，不能仅凭文件名推断用途，也不能据此认为所有只有 pos/rot 的资产都受支持。

### 使用步骤

1. 双击 `Launch.vbs`，选择还原模式。
2. 选择从 UE 导出的原始 FBX 和该模式需要的贴图。
3. 粒子模式填写材质实例中的 `Bound Min` 与 `Bound Max`，各包含 X、Y、Z 三个值。
4. 选择 Houdini、输出目录，并设置正确的播放 FPS。
5. 点击“开始还原”，验证完成后打开 Houdini 工程播放。

请保留 FBX 的 UV0 与原始包围盒，不要在导出后缩放、重建网格或修改贴图数值。工具不会自动读取 UE 材质实例的播放速度覆盖值。

### 粒子模式输出

- `OUT_PARTICLES`：可编辑粒子，保留 `id`、`density`、`lower_mask`、`vat_velocity_raw`，并由轨迹差分计算 `v`。
- `OUT_BILLBOARD_PREVIEW`：面向 `VAT_PREVIEW_CAMERA` 的几何面片；不会自动跟随普通编辑器视口旋转。
- `CONTROLS`：调整面片大小、密度与低处大小混合、预览速度拉伸。若 UE 材质实例覆盖了默认值，需要相应调整。

### 支持范围

- 当前支持上述两种编码，不支持骨骼、刚体碎块 VAT、任意自定义编码、双位置贴图或 LDR Position。
- 动态重网格模式暂不支持仅由材质实例提供 Legacy 包围盒参数的变体。
- 粒子模式支持已适配的 Position Alpha 缩放分支，不支持任意替换为 Rotation Alpha 缩放。
- 还原整数纹理帧，未复刻 UE 亚帧插值、透明裁切、折射、水材质或额外 WPO 效果。
- 粒子速度拉伸为近似预览，不保证与 UE 的屏幕表现逐像素一致。
- 动态拓扑不能直接按点编号插值；贴图尺寸相同也不代表网格和贴图一定配套。

## 验证与结果管理

两款工具都会创建独立结果目录，并在生成后重新加载逐帧验证，将结果写入 `validation.json`。如果结果目录中仍存在 `INCOMPLETE.txt`，说明任务未完成全部验证。

ABC Restore 的测试样例为 **31 帧、24 FPS**：UE 5.6.1 与 5.8.3 的结果逐帧位置、三角形连接及可用点属性一致，ABC 读回位置误差为 **0**。这代表已测试样例的结果，不代表所有资产都能无损恢复。

VAT Restore 已回归验证 **36 帧动态重网格样例**，并检查粒子中心位置、缩放公式及相机变化对面片和轨迹的影响。

## 常见问题

### 双击 Launch.vbs 没有启动

确认已经完整解压。如果系统禁用了 VBS，可用 Windows PowerShell 执行对应命令，将路径替换为实际位置：

```powershell
# ABC Restore
powershell.exe -NoProfile -ExecutionPolicy Bypass -STA -File "D:\Tools\ABCRestore\ABCRestore.ps1"

# VAT Restore
powershell.exe -NoProfile -ExecutionPolicy Bypass -STA -File "D:\Tools\VATRestore\VATRestore.ps1"
```

### 找不到 Houdini 或出现授权错误

手动选择 `Houdini 安装目录\bin\hython.exe`，确认本机 Houdini 授权正常。ABC 输出还要求许可证具备 Alembic 导出能力。

### ABC Restore 无法匹配 UE / 编译失败

手动选择合适的 `UnrealEditor-Cmd.exe`。若首次编译失败，查看界面给出的 `ModuleCache/.../build.log`，检查 C++ 工具链、Windows SDK 与 UE 安装是否完整。

### ABC Restore 解码失败

确认输入为受支持的 Geometry Cache，配套二进制文件完整。查看结果目录中的 `_extraction/unreal.log`。

### VAT 还原后尺寸、速度或外观不对

检查模式、配套网格与纹理、Bound Min / Max、FPS 和材质实例覆盖参数。材质函数编码不匹配时，需要新增解码适配；仅调整文件名无法解决。

### 移动工程后缓存丢失

将生成的工程与旁边的 `cache` 文件夹一起移动，保留相对目录结构。

## 问题反馈

提交 Issue 时，请附上工具版本、UE / Houdini 版本、所选模式、错误日志，以及可复现问题的最小样例或截图。版本识别问题可同时提供 `_extraction/engine_detection.json`。

每个 ZIP 内附有对应工具的详细 README 与源码，可进一步查看解码规则、使用限制和诊断说明。
