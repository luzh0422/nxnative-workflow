# 安装后配置与 SideStore 续签

入口：`$nxnative-workflow 配置SideStore续签`。也用于 LiveContainer 安装完成后的续接。

本文使用用户提供的既有成功续签方式：Mac 官方 `idevice_pair` 经 USB 生成 Remote Pairing 文件，内置 SideStore 选择 `rppairing`、后端 `idevice`、连接 LocalDevVPN。该历史配置来源于用户说明；本次设备上的实际结果仍需逐步确认。不要因每次调用重新生成有效配对文件或重置已有设置。

## 1. 确认安装与目标设备

先确认目标 iPhone/iPad 已安装本次 LiveContainer 合并包，记录设备别名、系统版本和宿主版本。Impactor 打开请求、IPA 校验通过不等于已安装。每台设备使用自己的配对文件；不能将其他设备的记录、bundle ID 或 Team 当作当前设备配置。

检查用户已完成哪些步骤，仅续接缺失项。未收到设备反馈时提供对应操作步骤；不假装已在设备上完成设置，也不要求用户在聊天中提供设备密码、Apple ID 密码或配对文件内容。

## 2. 开启开发者模式

在 iPhone/iPad 执行：**设置 → 隐私与安全 → 开发者模式**，打开开关，按提示重启；重启后解锁设备，再次确认开启并在设备上输入密码。

若开关未显示，USB 连接 Mac、保持设备解锁并信任电脑，在 Xcode 的设备管理界面发起设备配对/配置后再检查。按所用 Xcode 版本使用 Devices and Simulators 或 Device Hub。确认该模式已开启后再继续本流程。

## 3. 用官方 idevice_pair 生成 Remote Pairing 文件

使用 [jkcoxson/idevice_pair 官方 Releases](https://github.com/jkcoxson/idevice_pair/releases)。先检查本机已有工具；缺失时提供正确 macOS 下载入口，用户要求安装后再下载/安装。工具支持多种传输和配对类型，本流程沿用 USB 与 Remote pairing，不自行替换成另一种类型。

工具安装是首次生成配对文件的前置条件，不能只列出下载链接就报告环境就绪。有安装授权时应完成下载、官方资产校验和安装，再进入配对步骤。本工作区已安装 `/Applications/idevice_pair.app` v1.1.0，支持 arm64 与 x86_64；官方 DMG 保留在 Git 忽略的 `games/_tools/idevice_pair/v1.1.0/`。其他机器仍须检查实际路径和版本。

1. USB 连接目标 iPhone/iPad，保持解锁，在提示时选择“信任此电脑”。
2. 打开 `idevice_pair`，选择 **USB 连接的目标设备**。
3. 配对类型选 **Remote pairing / RPPairing**，点击 **Create**。
4. 点击 **Validate**，确认验证成功；失败时先解决配对，不继续导入失败记录。
5. 点击 **Save to file…**，保存为 `.plist`。

官方工具当前的 Remote Pairing 要求 iOS/iPadOS 17.4 或更新版本，执行前核对目标系统。GUI 标签可能随版本变化，以明确的 Remote 配对类型和 Validate 结果为准；不要把 Lockdown 文件仅改名后当作 Remote 文件。

已有同设备的可用 Remote 文件时可先验证并复用。生成和验证应针对同一台当前设备；文件能被 plist 解析不代表它仍能完成配对。

在本工作区保存文件时使用 Git 忽略的 `games/_devices/<设备别名>/pairing/`，仅当前用户可读写。不提交、打印配对文件内容，不复制到受 Git 追踪的配置或进度记录。无需扫描或导出用户已有凭据目录。

## 4. 在内置 SideStore 导入并设为首选

由用户通过 AirDrop 或其他本地文件传输方式，将验证通过的文件传到**同一台设备**的“文件”App，然后打开 LiveContainer 内置 SideStore：

**Settings → Pairing File Management → Remote Pairing File → Import / Replace File**

选择刚保存的文件，执行 **Activate** 和 **Set as Preferred**，检查 **Active** 与 **Preferred** 均显示 **`rppairing`**。这些标签及 `idevice` 后端来自用户当前已验证流程；所用 SideStore 版本若标签不同，先核对实际界面和版本，再决定对应设置。

保留本机原有有效记录供排查；文件生成、文件导入、激活/首选设置是不同阶段，不能仅凭导入动作报告续签已就绪。

## 5. 验证续签

1. 后端保持 **`idevice`**。
2. 连接 **LocalDevVPN**。
3. 完全退出并重新打开 **LiveContainer**。
4. 进入内置 SideStore，执行 **Refresh All**。
5. 核对续签成功提示，以及 LiveContainer 的新有效期/到期时间。

如果失败，记录错误文本、SideStore 版本、选中的后端与 Active/Preferred 类型，先定位失败环节；不自动卸载容器、切换 Apple ID、重置证书或反复生成配对记录。此调用默认提供配置指导与核对，实际配对、导入和续签按用户在设备上的操作结果续接；需要操作设备的自动化时另按其明确授权范围执行。

续签成功后按 [LiveContainer 官方安装后流程](https://livecontainer.github.io/docs/installation/lc_sidestore) 检查 **Import Certificate from SideStore**，运行 **JIT-Less Mode Diagnose → Test JIT-Less Mode**。随后分别验证游戏启动和存档同步；续签成功不代表这些功能也已通过。

### 游戏启动提示 iOS 26 要求 JIT-less

本工作区的茶杯头已实际导入容器，但首次启动出现 `JITLess mode is required since iOS 26`。已在当前源码 `LiveContainer/LCBootstrap.m` 核对：iOS 26 上，LiveContainer 读不到证书密码配置时会在进入游戏前返回该提示。游戏目录及安装时间存在不能证明 JIT-less 已就绪。

引导用户返回 LiveContainer 的 **Settings → JIT-less → Import Certificate from SideStore**，确认导入，再进入 **JIT-Less Mode Diagnose → Test JIT-Less Mode**。证书导入后的按钮可显示 **Remove Certificate** 或 **Refresh Certificate from SideStore**；以诊断实际结果为准。若提示找不到证书，先回到内置 SideStore 确认已登录原账号并成功 Refresh All，再重试导入；需要手动导入时按官方流程在设备上导出/选择 `.p12` 并在设备上输入其密码，不要求在聊天中发送证书或密码。

不要直接写入证书密码标志来跳过启动检查，也不要为了处理该提示重置配对或卸载容器。实际导入和测试由用户在设备上确认，收到成功反馈后再继续验证游戏启动。

本工作区已有用户侧验证：在 LiveContainer 中导入用户所称的 **JIT License** 后，该启动提示已解决。因此安装后的续签/配对配置完成后，仍应确认 JIT-less 证书已导入并执行诊断测试，再验证游戏。此反馈只确认该阻塞已解除，不代替完整游戏画面、声音和存档同步测试。

## 记录与来源

记录开发者模式、Remote 文件生成与 Validate、导入、Active/Preferred 类型、后端、VPN、Refresh All、证书导入和 JIT-less 的实际状态。只保存必要结果及非敏感路径，不保存配对记录内容。缺少反馈的阶段标作待验证，不把用户提供的历史经验标作本次设备实测。

参考：[Apple 开发者模式说明](https://developer.apple.com/documentation/xcode/enabling-developer-mode-on-a-device)、[idevice_pair 官方用法](https://github.com/jkcoxson/idevice_pair)。SideStore 的具体菜单和 `rppairing` 首选组合以用户在本会话提供的流程为依据。
