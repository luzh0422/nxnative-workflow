# 当前进度

本文件记录本工作区的流程验证状态。通用操作维护在 Skill 与流程 reference；单游戏技术结论按 nxnative 的规则维护在对应游戏文档。

## 当前状态快照（2026-11-01）

以下为续接依据，后面的记录保留历史上下文，不能将早期的缺工具、未克隆或未构建状态作为当前结论。

- 用户已确认「整体流程已经通了」。茶杯头 1.3.7 游戏 IPA 已构建、校验并传入 iPad mini A17 Pro 的 LiveContainer，设备内导入记录存在。
- 首次启动的 `JITLess mode is required since iOS 26` 提示，已由用户确认在 LiveContainer 导入其所称的 JIT License 后解决。此设置现为 Skill 独立检查点。
- 游戏主产物：`games/Cuphead-iOS/Cuphead.ipa`；输入、中间产物、隔离工具、下载与日志均已迁入同一工作区。复建入口为该目录的 `rebuild.sh`，不因整理文档而重跑构建。
- 宿主产物：`LCRebuild/build/LiveContainer+SideStore-SaveSync.ipa` 及同名 `.validation.json`；Xcode 构建成功、18 项核心测试通过、163 个保留资源一致。
- Impactor 与官方 idevice_pair 已本地安装。开发者模式、Remote Pairing、SideStore 和 JIT-less 的后续变动仍按实际设备反馈核对；不重复生成有效配对。
- 用户的整体成功反馈不扩展为未记录的性能、长时间稳定性、全部输入/声音细项或双设备 iCloud 同步通过。版本与证据见 [已验证基线](validated-baseline.md)。
- 本次固化：Skill 按阶段路由，补齐 [全流程与验收](runbook.md)、[游戏编译](game-build.md)、[宿主构建](livecontainer-build.md)、[设备导入](game-install.md) 和 [视频口播与分镜](video-tutorial.md)。视频面向普通玩家，目标 10–12 分钟；交付脚本，尚未生成视频。
- 固化验证：在 Git 忽略的 `games/_tools/skill-validation/` 中隔离补齐 PyYAML，官方 `quick_validate.py` 现已通过；15 份 Markdown 的本地引用与 21 段 shell 语法检查通过，Git diff 空白检查通过。未重新构建、签名、安装或变更设备配置。
- 提交状态：用户已要求将 LiveContainer 的 Save Sync 功能单独提交，生成 `7930cdf11649803d65b7709179ad7c81c25f8b38`，18 项核心测试再次通过，子仓工作区干净。随后用户要求提交 Skill；基础仓应锁定该功能提交，并包含发现链接、Skill、流程参考和配套配置。两个仓库均尚未推送，远程可复现性待发布后验证。
- 提交范围：保留已删除准备脚本的状态；忽略本地 IPA 和源码补丁副本，不将游戏、密钥、配对、证书或缓存归入基础仓提交。历史中的本机绝对路径已转为仓库相对路径，签名后的设备应用身份不作为共享示例。

## 历史记录

## 初始基线

- 目标：基于 nxnative 打通 Switch 游戏到 iOS 的流程，并随验证逐步构建 Skill。
- 源码位置：本 Skill 目录旁的 `../nxnative/`。
- 初次检查 revision：`93578ab`（后续运行应重新记录实际 revision）。
- 已完成：检查仓库 `AGENTS.md`、移植指南、构建脚本选项；建立 Skill 入口、源码导航与进度记录。
- Skill 校验：`quick_validate.py` 因当前 Python 缺少 `PyYAML` 未能运行；使用 Ruby YAML 解析完成替代检查，名称、描述、界面提示词、模板占位清理及 reference 链接检查通过。
- 验证范围：仅文档与源码静态检查，尚未执行游戏调查、Mac 构建、IPA 构建或真机测试；尚未检查本机工具链是否齐备。
- 游戏名称、Title ID、游戏版本、输入目录、密钥路径、目标设备：尚未确定。
- 下一步：确定首个游戏及本地输入位置，检查构建依赖，选择已有 profile 的构建路径或新游戏的调查路径。

## 后续记录方式

每个实质步骤记录日期、源码 revision、游戏版本与 build ID（已知时）、目标、实际命令与必要环境变量、退出状态、产物或日志路径、观察结果、未验证项和下一步。不要写入密钥内容或复制大段日志。

更新当前状态以便续接；只有能解释重要决策的历史记录才需要保留。

## 基础仓与 submodule 配置

- 用户指定当前目录为 Skill 基础仓，包含 `https://github.com/seedds/nxnative` 与 `https://github.com/luzh0422/LiveContainer` 两个 submodule，路径分别为 `nxnative/`、`LiveContainer/`。
- 已准备根目录 `.gitmodules` 与 `README.md`，Skill 已记录两个子仓的定位和检查方式。
- 实际状态：基础仓尚未 Git 初始化；nxnative 是现有干净 clone，revision 为 `93578ab`，origin 为 `git@github.com:luzh0422/nxnative.git`；LiveContainer 尚未克隆。没有创建 gitlink、提交或推送。
- 阻塞证据：在根目录执行 `git init` 返回 `.git: Operation not permitted`，当前环境禁止该目录的 Git 元数据写入。
- 下一步：在本地终端按根目录 README 完成初始化与 submodule 注册，核对提交指针；随后继续首个游戏的流程验证。

## 初始化调用约定

- 已添加 `$nxnative-workflow 初始化环境` 入口及 `environment.md`：检查基础仓注册情况、保留本地开发现场、按锁定 commit 初始化两个子仓、检查工具链并记录结果。
- 本次仅更新 Skill 与文档，没有实际执行依赖下载、版本切换或工具安装。基础仓注册的已知阻塞仍待解决，初始化流程尚未端到端验证。

## LiveContainer 下载尝试

- 按用户要求执行 `git clone https://github.com/luzh0422/LiveContainer LiveContainer`，退出状态为 `128`，错误为 `Could not resolve host: github.com`。
- 当前执行环境无法解析 GitHub 主机，下载未完成，`LiveContainer/` 目录未创建；现有源码未改动。
- 下一步：在可访问 GitHub 的本地终端执行上述命令，下载后核对 remote 和 revision；基础仓的 submodule 注册仍需另行完成。
- 用户随后指定 SSH 地址 `git@github.com:luzh0422/LiveContainer.git`，已同步更新 `.gitmodules`、README 注册命令和 Skill 中的地址。
- 使用该 SSH 地址再次下载，退出状态为 `128`，错误为 `Could not resolve hostname github.com`；下载仍未完成。后续下载使用用户指定的 SSH 地址。

## 第二步调用约定

- 已添加 `$nxnative-workflow 打开游戏下载网址`，目标地址为用户指定的 `https://ns.nsyxck.com/#home`，通过浏览器打开。
- 已记录 macOS 默认浏览器打开命令、页面加载的验证标准及浏览器不可用时的处理方式；此步骤可以独立调用。
- 本次仅增加调用说明，未实际打开浏览器或验证站点加载。

## Skill 发现状态

- 排查确认 Skill 源文件位于根目录 `nxnative-workflow/`，尚未接入 `.agents/skills/`，因此不能仅凭文件存在就声称已出现在客户端选择器。
- 已核对官方本地发现规则，根目录 README 增加通过 `.agents/skills/nxnative-workflow` 相对符号链接接入的命令。
- 当前环境执行 `mkdir -p .agents/skills` 返回 `Operation not permitted`，尚未创建发现入口。需在本地终端执行 README 中命令；客户端是否显示仍需用户侧验证。

## 基础仓发布准备

- 用户指定基础仓远程地址为 `git@github.com:luzh0422/nxnative-workflow.git`。README 已使用实际克隆地址。
- 已新增 `scripts/prepare-repository.sh`：初始化基础仓、配置 origin、建立 Skill 相对符号链接、注册两个 submodule 并暂存发布文件；不提交、不推送。已有冲突的 origin 或子仓修改会保留并停止。
- 已在临时测试仓验证：准备与重复执行、符号链接和 gitlink 模式、基础仓 origin 配置、子仓 remote 保留、子仓修改保护、基础仓 origin 冲突保护。测试提交后的普通 clone 可解析 Skill 入口，不依赖 submodule 下载。临时测试目录已清理。
- 当前会话审批机制更新后，准备脚本已通过单次命令权限审核并在实际目录执行成功：基础仓已初始化，origin 已设为 `git@github.com:luzh0422/nxnative-workflow.git`，Skill 发现链接已创建，两个子仓已注册为 gitlink，发布文件已暂存。
- 未修改全局 Codex 配置，未创建提交或推送；客户端列表是否刷新仍待用户侧验证。其他使用者需在这些文件提交并发布后 clone，才能获得发现入口。
- 本地 LiveContainer 源码现已存在，当前 revision 为 `9108ae6`，检查时工作区干净；此前下载未完成的记录为历史状态。

## 客户端选择器排查

- 当前会话的系统可用 Skill 清单已包含 `nxnative-workflow`，说明仓库级发现入口已被识别；用户反馈 `$` 选择器和 `/skills` 中尚未显示，客户端缓存或刷新状态尚未确认。
- 已核对发现链接可读、准备脚本检查通过，用户级配置没有显式禁用该 Skill 的条目。
- 将 `agents/openai.yaml` 的界面显示名从 `Switch → iOS · nxnative` 统一为 `nxnative-workflow`，减少按名称搜索时的歧义；这不代表已确认搜索故障的根因。
- 下一步：完全重启 Codex，进入本项目后检查 `$` 选择器或 `/skills`。目前尚未观察到用户界面的刷新结果。

## 第二步实际执行

- 用户已通过 `$nxnative-workflow` 调用第二步，当前会话已收到该 Skill 的显式加载内容。
- 执行 `open 'https://ns.nsyxck.com/#home'`：沙箱内调用失败，经单次命令权限审核后再次执行返回 `0`，已向系统默认浏览器发出打开请求。
- 尚未观察浏览器中的页面加载结果；游戏与本地输入目录仍待确定。

## 第三步：密钥获取验证

- 用户要求从 `https://prodkeys.one/` 自动获取候选 `prod.keys`。本机请求首页、`/ryujinx-prod-keys/`、`/download/` 返回 200；下载页实际上是密码表单，不能将 200 等同于文件可下载。
- 在详情页找到公开口令，按正常 WordPress 表单提交后取得下载链接。首次命令权限审核超时，允许的一次重试通过并完成请求；未猜测口令或绕过表单。
- 已下载页面标注 v22.5.0 的候选 ZIP（7,413 字节），包含 `prod.keys`、`title.keys`；仅提取前者，未使用附带的 title.keys。
- 临时工作目录：`/private/tmp/nxnative-keys-fphgwcco/`。整理后的文件为 `prod.keys`，原件为 `prod.original.keys`；两者权限均为 0600，文件在仓库外，系统临时目录可能被清理。
- 原件 231 个条目，其中 33 个 `(mariko_)?master_kek_source_*` 条目为错误的 17 字节，符合 nxnative 提取说明记录的异常。保留原件并移除这些异常行后，副本剩余 198 个条目，未猜测或修改密钥值。
- 整理后 SHA-256：`8a38c823f4d197256331f2e7900852bc5df53b8709fdbb39739c494c2be39a4e`。
- 已新增 Skill 第三步与 `keys.md` 获取流程。实际游戏提取尚未验证，目标游戏及持久保存目录仍待确定。

## Cuphead 1.3.7：首次 iOS 构建完成（2026-11-01）

- 输入来自 `games/已加速- 茶杯头 Cuphead/nsp非整合版/`：base NSP、v655360（1.3.7）update NSZ、DLC index 1 NSZ。使用 APFS clone 将原包复制到 `games/Cuphead-iOS/input/`；原始游戏包未修改，未暂存游戏产物。
- nxnative revision：`93578abad6704ae95fbf60e80027469f145b02cf`，profile `0100A5C00D162000`。目标主模块 build ID 为 `d02cd50295124a48968488416e7e692b`，与已有 profile 匹配。nxnative 源码工作区未产生修改。
- 工具链：arm64、Xcode 27.2（27B52）、XcodeGen 2.46.0、Metal 32023.922、隔离 .NET SDK 10.0.403、Python 3.14.4 venv。Python 3.14 内置 zstd，无需安装 backports.zstd。
- 补齐前实际检测：CLion 自带 .NET runtime 10.0.9，但没有 SDK；Metal 的 xcrun shim 存在，但运行明确报告缺少 Metal Toolchain。已通过官方组件下载安装 Metal，SDK 安装在 `games/Cuphead-iOS/tooling/dotnet/`，未修改全局 .NET。
- 下载遇低速和间歇网络错误后，保留完成分段并使用官方 CDN 并行补齐。Metal 官方资产 SHA-256、.NET SDK 官方发布元数据 SHA-512 均验证通过；未替换锁定的 shader translator。恢复下载工具及日志保留在隔离工作目录。
- 密钥使用整理后的候选文件副本 `games/Cuphead-iOS/prod.keys`；已成功解压 NSZ（NCA hash 验证通过）、提取并合并 base/update（548 文件、3.4 GB），DLC 提取成功。密钥适用于本次实际输入，不再仅是格式检查状态。
- 主流水线完成所有 8 个阶段并退出 0：代码重写，3,118 个 shader 提取和 iOS library 编译，134 个应用源文件编译，以及 IPA 打包。正式流水线 17:00:14–17:02:39，约 2 分 25 秒，不含依赖准备。
- IPA 主产物：`games/Cuphead-iOS/Cuphead.ipa`，5,121,301,620 字节；原交付副本保留在 `games/Cuphead-iOS/delivery/Cuphead.ipa`。SHA-256：`3290f724141b94a60f77c05e311a713de039f640980ea052ffc2ae95cfc32eb3`。
- 最终包验证：8,401 个 ZIP 条目 CRC 全部通过，架构 arm64；Info.plist 版本 1.3.7，bundle ID `org.nxnative.0100a5c00d162000`，最低 iOS/iPadOS 26.0，支持 iPhone/iPad；未含 provisioning profile，codesign 报告 `code object is not signed at all`。
- 使用了本机 Metal/spirv-cross 编译器，与 `dmsh_chain.lock` 记录的历史测试编译器版本不同，属于脚本支持的 own_compilers 路径。IPA 成功不代表真机渲染或运行已验证。
- 完整日志 `games/Cuphead-iOS/build.log`，阶段日志在 `input/0100A5C00D162000_build/logs/`；验证报告 `verification/report.json`。可用 `sh games/Cuphead-iOS/rebuild.sh` 复建（保留隔离目录，避免并行重复运行）。
- 未签名、未安装设备、未提交或推送。下一步由用户侧载并回传 `Documents/log.txt` 做 iOS 真机验证；IPA 已另存上述持久交付位置，本地资料已按后续用户要求迁入基础仓 Git 忽略的 `games/`，不随仓库发布。


## 构建资料迁入 games/（2026-11-01）

- 用户明确要求将下载资料、中间产物、工具链和 IPA 全部迁入 `games/`，替代本工作区此前使用临时目录/仓库外产物的约定。原始游戏包保持不动。
- 当前工作目录为 `games/Cuphead-iOS/`：原构建目录整体迁入；此前下载目录中的交付副本保存在 `delivery/Cuphead.ipa`，未删除副本。
- 密钥下载目录迁入 `keys-download/`，相关网站 HTML 保存到该目录的 `pages/`；原始密钥、整理后密钥、cookie jar 与 ZIP 均保留，不输出密钥内容。
- 根目录 `.gitignore` 已排除 `/games/`，检查确认游戏资料未被 Git 跟踪。Skill、密钥流程和 README 已同步本工作区的资料布局约定。
- 重建脚本已改为根据自身位置定位工作区与 nxnative；已修正 Python venv 启动器、激活脚本、下载辅助脚本及校验报告中的当前产物路径。历史构建日志保留原始内容。
- 迁移后已验证重建脚本语法、Python 模块导入、pip/NSZ 启动器及 .NET SDK 10.0.403 可用。原临时构建目录、原密钥临时目录和原下载交付目录均已移走。
- 迁移后的主 IPA 与交付副本 SHA-256 均与构建时一致：`3290f724141b94a60f77c05e311a713de039f640980ea052ffc2ae95cfc32eb3`。


## LiveContainer Save Sync：编译与合并 IPA（2026-11-01）

- 用户要求使用现有 `LiveContainer/` 中的存档同步修改，按 Release/iphoneos/arm64、禁用签名的 xcodebuild 配置编译，再调用原有 `scripts/package_save_sync.py` 保留基础 IPA 的 SideStore 资源。本次实际源码位置为基础仓 `LiveContainer/`，输出按用户指定布局保存到 `LCRebuild/build/`。
- 基础 IPA 为 `LCRebuild/LiveContainer+SideStore.ipa`，SHA-256 与打包脚本锁定值一致：`33831bb279d74f2da1be08eb3cc6d3646992a6120da32ffb763269696052b868`。根目录同名文件也已验证相同，均未覆盖。
- 补齐 OpenSSL `623c84da314e85363236507ca38a4bde65df21c3`、litehook `a7e9f599d7f0b3d77008ad62bf10cb10dab6eca9`。完整历史克隆遇网络中断后改用锁定提交浅克隆；中断期间 litehook 仅创建元数据、源码未检出，首次编译因此报缺头文件，恢复其锁定文件后重试成功。没有更换依赖版本。
- 存档同步核心 `swift test` 已通过 18 项测试，0 失败；测试使用项目本地夹具，不访问真实游戏存档或 iCloud。日志为 `LCRebuild/build/logs/swift-tests.log`。
- Xcode 编译日志 `LCRebuild/build/logs/xcodebuild-retry.log` 显示 `BUILD SUCCEEDED`；App 位于 `LCRebuild/build/DerivedData/Build/Products/Release-iphoneos/LiveContainer.app`，中间产物保留。
- 合并 IPA 为 `LCRebuild/build/LiveContainer+SideStore-SaveSync.ipa`，25,569,412 字节；SHA-256：`7e16e369668ed2c5ba8fdf57454a82b961a48fc002896cf91131ed4303ecca9d`。版本 LiveContainer 3.8.11，Save Sync 1，bundle ID `com.kdt.livecontainer`。
- 校验报告为 `LCRebuild/build/LiveContainer+SideStore-SaveSync.validation.json`；11 个主程序/框架/扩展二进制均为 arm64，163 个保留的 SideStore/settings/intents/widget 资源逐字节一致，ZIP CRC 与重复路径检查通过。主宿主未签名、无 provisioning profiles；原有嵌入组件的 vendor/ad-hoc 签名保持不变，未执行设备安装。
- 已比较构建前后的 tracked source diff，用户原有修改保持一致。构建前快照与构建、打包日志保存在 `LCRebuild/build/logs/`。根 `.gitignore` 已排除 `/LCRebuild/build/`；没有提交或推送。
- 实际设备、SideStore 续签与双设备 iCloud 同步仍待用户验证；核心测试和成功构建不能替代这些运行验证。


## Impactor 外部工具安装（2026-11-01）

- 用户确认以外部工具方式接入，并要求先安装 Impactor。检查时没有已有 App 或已安装 cask。
- 已通过 Homebrew 官方 cask 安装 Impactor 2.7.2，路径 `/Applications/Impactor.app`。安装包来源为官方 GitHub Release 的 macOS universal DMG，cask 锁定 SHA-256 为 `fa1a2ba0fb7e46dbe1b0ea5e66f12d04cc570693afd14fe3b43979e5a3cfe3be`。
- 首次命令权限审核超时，允许的一次重试通过并安装成功。已读取 Info.plist 及可执行架构，确认本地 App 可定位。
- 本次仅安装桌面工具，未启动 Apple ID 登录、签名、设备配对或安装 LiveContainer；后续交接流程仍待接入和验证。


## 准备 LiveContainer 安装：首次调用（2026-11-01）

- 用户显式调用 `$nxnative-workflow 准备LiveContainer安装`。已重新读取合并 IPA 及报告，SHA-256 为 `7e16e369668ed2c5ba8fdf57454a82b961a48fc002896cf91131ed4303ecca9d`，与构建记录一致；ZIP CRC/重复路径、LiveContainer 3.8.11 与 Save Sync 1 标记、无 provisioning profile，以及报告中 163 个资源的哈希均检查通过。
- 已确认 `/Applications/Impactor.app` 版本为 2.7.2。经具体命令权限审核后，系统 `open -a` 与 `open -R` 均返回 0，已请求打开 Impactor，并在 Finder 中选中 `LCRebuild/build/LiveContainer+SideStore-SaveSync.ipa`。
- 已向用户说明拖入 IPA、沿用原账号和 bundle ID、选择 Only Register Main Bundle 的后续操作。没有读取账号凭据或执行签名/设备安装，尚未观察到窗口/导入/安装结果。
- 已将本次调用入口与交接方法写入 Skill、`livecontainer-install.md` 与基础仓 README。


## 安装后开发者模式与 Remote Pairing 流程

- 用户补充 iPad 安装后需开启“设置 → 隐私与安全 → 开发者模式”，并提供曾成功续签的 USB idevice_pair Remote Pairing 生成、Validate、保存及内置 SideStore 导入流程。该成功历史来源于用户说明，未据此声称本次设备已经配置完成。
- 已加入 `$nxnative-workflow 配置SideStore续签`，将开发者模式/重启确认、Remote Pairing、导入并设置 Active/Preferred 为 rppairing、idevice 后端、LocalDevVPN、重启 LiveContainer、Refresh All 串接为安装后流程。
- 已核对 Apple 开发者模式资料和 jkcoxson/idevice_pair 官方 README；具体 SideStore 菜单与首选配置按用户提供的现有版本流程维护。已有有效文件可验证后复用，不要求每次重新配对。
- 配对文件使用本地 Git 忽略的 `games/_devices/<设备别名>/pairing/`，不输出或提交其内容。当前仅更新 Skill 和流程文档，未下载/安装 idevice_pair、未生成配对记录、未修改设备设置或执行续签。
- 当前设备的开发者模式、Validate、导入激活、Refresh All、证书导入及 JIT-less 结果仍待用户反馈。


## idevice_pair 前置工具安装（2026-11-01）

- 用户指出 Remote Pairing 流程需要先准备工具。检查本机未发现已有 idevice_pair，已从 jkcoxson/idevice_pair 官方 v1.1.0 Release 下载 macOS universal DMG。
- 官方资产 SHA-256 `9ff54171dccdb0a9b0e72ec89e43a479823c53ee54c2121fe9f053cc2cb940ce` 已核对一致；安装包及非敏感安装记录保存在 `games/_tools/idevice_pair/v1.1.0/`。
- 已安装 `/Applications/idevice_pair.app`，版本 1.1.0，bundle ID `com.jkcoxson.idevice_pair`，可执行文件包含 arm64 与 x86_64。安装磁盘映像已卸载。
- 首次下载的自动权限审核超时，允许的一次重试通过并完成下载；安装步骤另经权限审核完成。未关闭系统安全校验，未打开设备配对、生成文件或更改 SideStore 配置。
- 已在安装后流程中明确工具安装为首次 Remote Pairing 的前置条件；下一步是 USB 连接并解锁目标设备，选择 Remote pairing → Create → Validate，再保存文件。


## Cuphead：传入 iPad 并完成容器导入（2026-11-01）

- 用户明确授权将已编译 IPA 放入 iPad LiveContainer 文件夹并在设备导入。通过 Xcode devicectl 发现一台连接的 iPad mini（A17 Pro）；实际宿主为 LiveContainer 3.8.11，已确认实际签名后的宿主 bundle ID，数据容器可访问；该设备身份仅保留在本地回执中。
- 传输前确认 Documents 顶层无 Cuphead.ipa，Documents/Applications 为空；复制到设备 `Documents/Cuphead.ipa` 后，设备端报告 5,121,301,620 字节。原本地 IPA 保留。
- 按当前 LiveContainer 源码支持的 `livecontainer://install?url=<本地文件URL>` 通过 devicectl payload URL 调用容器内导入。随后从设备读取该游戏的 Info.plist 与 LCAppInfo.plist，确认 `Documents/Applications/org.nxnative.0100a5c00d162000.app`、Cuphead 1.3.7、LCPatchRevision 7 和 installationDate 已存在。没有覆盖既有游戏或读取存档/账号/配对文件。
- 本地设备传输回执、导入请求及游戏元数据保存在 Git 忽略的 `games/_devices/transfer/`。容器内安装完成，不等于运行已验证。
- 用户随后报告启动显示 `JITLess mode is required since iOS 26`。已在 LCBootstrap.m 核对该提示对应证书密码配置缺失；尚未认为游戏已成功运行。当前需用户在设备 Settings 导入 SideStore 证书，并执行 JIT-Less Mode Diagnose/Test，再继续游戏验证。
- 已将本次实际遇到的 JIT-less 提示及处理方式补入 SideStore 安装后流程。


## Cuphead：JIT-less 启动阻塞已由用户解决

- 用户反馈“解决了，是需要在 LiveContainer 中导入 JIT License”。已将此前 iOS 26 的 JIT-less 配置缺失提示标记为用户侧已解决。
- 安装后流程继续要求确认 JIT-less 证书导入与诊断测试，不能以配对文件配置或 Refresh All 成功替代。
- 此次反馈确认启动阻塞解除；完整游戏运行、声音和存档同步仍按各自实际测试结果记录。
