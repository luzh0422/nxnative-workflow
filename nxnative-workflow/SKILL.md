---
name: nxnative-workflow
description: "基于 nxnative 完成 Switch 游戏到 iOS 的构建、LiveContainer 合并包准备、Impactor 安装交接、SideStore 续签与 JIT-less 配置、设备内游戏导入及运行验证。支持分步执行、断点续接和按实际证据完善流程。"
---

# nxnative-workflow

以本仓库锁定的 nxnative 与 LiveContainer 为基础，复现已跑通的游戏构建和设备安装链路。技术路线为 ARM64 原生执行适配、必要指令重写、系统服务适配及图形到 Metal 的转换；具体游戏能力以 profile 和实际运行结果为准。

## 入口与分流

用户要求“全流程”“从头开始”或“梳理流程”时，先读 [端到端流程](references/runbook.md)，确认目标和当前已完成阶段；不因从头讲解就重新下载、构建或安装。用户要求继续时，先读 [进度记录](references/progress.md) 顶部的当前状态，再按需查看历史。

| 调用或等价意图 | 执行参考 | 交付结果 |
|---|---|---|
| `初始化环境` | [环境初始化](references/environment.md) | 锁定源码与工具链检查结果 |
| `打开游戏下载网址` / `打开游戏下载地址` | 下方资源入口 | 浏览器打开请求 |
| `准备游戏与密钥` | [密钥准备](references/keys.md) | 输入路径、版本和密钥检查结果 |
| `编译游戏` / `编译茶杯头` | [游戏编译](references/game-build.md) | 未签名游戏 IPA、日志与校验记录 |
| `编译LiveContainer` | [宿主编译](references/livecontainer-build.md) | 未签名合并 IPA 与 validation.json |
| `准备LiveContainer安装` | [Impactor 交接](references/livecontainer-install.md) | 校验 IPA，打开 Impactor/Finder |
| `配置SideStore续签` | [安装后配置](references/sidestore-setup.md) | 开发者模式、Remote Pairing、续签状态 |
| `配置JIT-less` / `导入JIT License` | [安装后配置](references/sidestore-setup.md) | 证书导入与诊断结果 |
| `安装游戏到LiveContainer` | [设备导入](references/game-install.md) | 文件传输、容器内安装及独立验证 |
| `验证运行` | [端到端验收](references/runbook.md) | 实际设备证据与未验证项 |
| `配置存档同步` | [存档同步](references/save-sync.md) | 文件夹授权与独立双设备验收 |
| `录制教程` / `如何讲解` | [10–12 分钟口播与分镜](references/video-tutorial.md) | 可执行的真实录屏计划 |

完整调用示例：`$nxnative-workflow 编译游戏 游戏目录=… 密钥路径=…`。参数是自然语言输入，执行时解析为真实路径，不把示例中的省略号当作路径。

## 资源入口

仅在用户要求打开资源网站时，用浏览器打开用户指定的 `https://ns.nsyxck.com/#home`。macOS 可执行：

```sh
open 'https://ns.nsyxck.com/#home'
```

保留路由 `#home`。打开成功不等于页面已加载，更不等于游戏已下载；以本地输入文件为后续依据。已有文件时直接准备输入，不要求重复访问网站。网站是可选入口，不是环境初始化的必需依赖。

## 工作区与续接

- 先解析 `.agents/skills/nxnative-workflow` 符号链接，取得真实 Skill 目录；其上一级为基础仓。源码为基础仓的 `nxnative/`、`LiveContainer/`。修改或构建子仓前读取适用的 `AGENTS.md`、当前文档与实际代码。
- 使用基础仓记录的提交，不擅自更新远程分支。初始化前保留已有修改、未推送提交和有效分支现场；不以 reset/clean/stash 自动清理用户工作。宿主存档同步功能必须已经包含在对应子仓提交中，不能只凭本机未提交文件就声称别人可复现。
- 默认本地布局为 Git 忽略的 `games/<游戏工作区>/`、`games/_tools/`、`games/_devices/`。现有宿主产物按用户指定保存在 Git 忽略的 `LCRebuild/build/`。保留原始输入、完整日志和可复建缓存，不将长期资料仅放系统临时目录。
- 私钥、prod.keys、证书密码、配对内容和账号令牌不输出、不提交。公开教程只展示脱敏后的路径、版本和结果。
- 缺少游戏/设备信息时先从现有文件和状态定位；只有影响下一步且无法推断的信息才询问。缺少依赖时核对实际版本与执行结果，不能将 PATH 或沙箱问题当成整机未安装。
- 授权范围沿用会话。构建不自动包含设备操作；用户明确要求传输或安装时按已有授权执行，不重复确认。Impactor 登录和安装、开发者模式、Remote Pairing 导入、JIT-less 证书确认等界面步骤按设备实际反馈续接，不能假装已点击或成功。

## 验证与排障

始终区分：输入可读、提取成功、IPA 生成、包校验通过、宿主安装、续签、JIT-less 就绪、游戏导入、游戏启动和存档同步。用户说整体流程已通可记录为用户确认；没有观察到的画面、帧率、长时间稳定性和双设备同步仍不作额外保证。

- 失败先查当前阶段日志，再查 [源码与症状入口](references/workflow.md)，只重试相关阶段。保留已完成的输入、下载和缓存。
- 新游戏先调查 profile/SDK/图形 API，不复制其他游戏固定地址；特例放 profile 并按 build ID 限定。未匹配的 profile 或版本不能承诺生成后可运行。
- JIT-less 证书导入是首次运行前的独立检查点；配对文件、开发者模式和 Refresh All 成功均不能替代它。遇到本次已定位的 iOS 26 提示，按 [安装后配置](references/sidestore-setup.md) 处理，不跳过检查。
- SideStore 续签与 iCloud 游戏存档同步是两个流程；同步功能是否通过以真实双设备结果判断。

## 固化与维护

[已验证基线](references/validated-baseline.md) 保存本次案例的版本与证据；通用步骤维护在各阶段参考中；[进度记录](references/progress.md) 只将最新状态作为续接依据。单游戏/底层发现分别按 nxnative 文档规则写入游戏说明和子系统参考，Skill 保留链接，避免复制手册。

修改后运行可用的 `skill-creator/scripts/quick_validate.py`，再检查本地引用和关键流程边界。文档变更不触发真实签名、配对、安装或全量重建。提交与推送只在用户要求时执行。
