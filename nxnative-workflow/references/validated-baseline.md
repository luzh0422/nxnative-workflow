# 已验证基线

这是本工作区 2026-11-01 的案例记录，不是对全部游戏、设备或未来版本的兼容保证。版本数字用于复现；新的运行以实际锁定源码和检测结果为准。

| 项目 | 本次记录 | 证据层级 |
|---|---|---|
| nxnative | `93578abad6704ae95fbf60e80027469f145b02cf` | 本地 Git 检查 |
| LiveContainer 源码 | `7930cdf11649803d65b7709179ad7c81c25f8b38`（基于 `9108ae6` 加入 Save Sync） | 功能已本地提交，尚未推送 |
| 茶杯头 | Cuphead 1.3.7；profile `0100A5C00D162000` | 提取模块与 profile 匹配 |
| Mac 工具 | Xcode 27.2；Metal 32023.922；XcodeGen 2.46.0；.NET SDK 10.0.403；Python 3.14.4 | 实际执行与构建日志 |
| 游戏 IPA | 5,121,301,620 字节，8,401 ZIP 条目 CRC 通过，arm64，最低 iOS/iPadOS 26.0 | 本地包校验 |
| 宿主合并包 | LiveContainer 3.8.11 + Save Sync 1 | BUILD SUCCEEDED、打包报告 |
| 宿主测试 | 18 项核心测试通过；163 个保留资源逐字节一致 | 本地测试/包校验 |
| 外部工具 | Impactor 2.7.2；idevice_pair 1.1.0 | 本地安装与架构检查 |
| 设备 | iPad mini A17 Pro | 实际设备查询 |
| 游戏传输/导入 | Documents 中文件大小一致；游戏元数据与安装时间存在 | 设备端检查 |
| JIT-less 启动阻塞 | 用户确认导入 JIT License 后已解决 | 用户设备反馈 |
| 整体流程 | 用户确认“整体流程已经通了” | 用户确认，不扩大为所有细项测试 |
| 双设备 iCloud、冲突恢复、长时间稳定性 | 尚无本次完整验收记录 | 不宣称已通过 |

本地游戏/工具/缓存位于 `games/Cuphead-iOS/`，宿主产物位于 `LCRebuild/build/`。记录均保持相对目录，账号、Team、设备 UDID、配对文件和密钥内容不作为共享配置。

案例游戏 IPA SHA-256：`3290f724141b94a60f77c05e311a713de039f640980ea052ffc2ae95cfc32eb3`。

案例宿主 IPA SHA-256：`7e16e369668ed2c5ba8fdf57454a82b961a48fc002896cf91131ed4303ecca9d`。

详细操作历史在 [进度记录](progress.md)。发布前的 clone 可复现性检查见 [端到端流程](runbook.md)。
