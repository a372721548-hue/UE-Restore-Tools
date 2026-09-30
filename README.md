# UE Restore Tools

一组面向 Unreal Engine 资产还原 / 动画重建的 Windows 独立工具，均以「双击 Launch.vbs 即用」的形式分发，无需提前打开 UE 或 Houdini，也无需安装 Python。

| 工具 | 下载 | 用途 |
|------|------|------|
| ABC Restore v2 | [ABCRestore_Tool_v2.zip](./ABCRestore_Tool_v2.zip) | UE Geometry Cache `.uasset` → Alembic (ABC) / Houdini 工程 / `.bgeo.sc` |
| VAT Restore v2 | [VATRestore_Tool_v2.zip](./VATRestore_Tool_v2.zip) | VAT（顶点动画纹理）→ 逐帧真实几何体 / Houdini 粒子 |

每个 zip 内附完整 `README.md`，包含使用步骤、支持范围、验证方式与故障处理。

## ABC Restore v2

- 输入 UE 编辑器的 **Geometry Cache `.uasset`**，输出可播放的 ABC、Houdini 工程与 `.bgeo.sc` 缓存。
- 自动从包头与 `.uproject` 识别资产保存版本并匹配本机引擎；缺失模块（`Extractor/Source`）首次使用时自动编译并缓存到 `ModuleCache`。
- 已实测 UE 5.6.1 自动选择、UE 5.8.3 自动编译提取，Houdini 21.0.440。
- 每次生成独立结果目录，含 `validation.json` 逐帧误差验证；失败不显示成功。

## VAT Restore v2

- **动态重网格模式**：输入原始 FBX + Lookup + Position（HDR/EXR），还原真实逐帧网格，法线重算，无需 Rotation/Color 贴图。
- **粒子面片模式（ParticleSprites）**：输入 Rotation/Position + Bound Min/Max，恢复每个粒子中心位置，保留 id / density / lower_mask / vat_velocity_raw，输出可编辑粒子与面向预览相机的面片。
- 解码保留 Lookup 原始 RGBA 数值，坐标转换为 Houdini Y 向上、米；生成后逐帧自动验证并写入 `validation.json`。

## 运行条件

- Windows
- 已安装并授权的 Houdini（需 Alembic / hython 能力）
- ABC Restore：使用附带模块可直接运行；其他引擎版本首次使用需 VS C++ 工具链 + Windows SDK + UE 构建工具

## 启动失败

若系统禁用了 VBS，用 PowerShell 运行：

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -STA -File "工具目录\xxxRestore.ps1"
```
