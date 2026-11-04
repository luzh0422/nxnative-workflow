# nxnative-workflow

使用 Codex Skill 串起 Switch 游戏到 iOS 的适配、构建和安装流程。本文按实际演示顺序，以 **茶杯头（Cuphead 1.3.7）→ iPad → LiveContainer** 为例，介绍每一步的调用方式和完成标准。

本案例已完成游戏构建、设备导入，并由用户确认启动流程跑通。其他游戏需要核对对应版本的适配情况，具体记录见 [已验证基线](nxnative-workflow/references/validated-baseline.md)。

## 开始前

准备一台用于构建的 Mac、Xcode，以及通过 USB 连接的目标 iPad/iPhone。本案例使用 Apple Silicon Mac 和 iPad mini A17 Pro，生成的茶杯头 IPA 最低系统要求为 iOS/iPadOS 26.0；其他构建以实际要求为准。

先在终端克隆仓库：

```sh
git clone git@github.com:luzh0422/nxnative-workflow.git
cd nxnative-workflow
```

上述地址及 LiveContainer 子仓使用 SSH，需要配置可用的 GitHub SSH 身份和仓库访问权限。

用 Codex 打开克隆后的项目目录。仓库通过 `.agents/skills/nxnative-workflow` 相对符号链接提供 Skill 发现入口。若选择器里没有显示，检查是否打开了项目根目录、链接是否可读，再重启客户端检查。

下面以 `$nxnative-workflow` 开头的内容，都是**在 Codex 对话中输入的指令**。Skill 接受自然语言，可以附上游戏目录、目标版本、IPA 路径或设备信息。

整个流程会准备两份 IPA：

| 文件 | 用途 | 安装位置 |
|---|---|---|
| `Cuphead.ipa` | nxnative 生成的游戏包 | 导入 iPad 上的 LiveContainer |
| `LiveContainer+SideStore-SaveSync.ipa` | 包含 SideStore 和 SaveSync 功能的宿主合并包 | 通过 Mac 上的 Impactor 安装到 iPad |

## 1. 初始化整个仓库

```text
$nxnative-workflow 初始化整个仓库。
```

检查基础仓、Skill 入口，以及 `nxnative`、`LiveContainer` 和它们的嵌套依赖。缺失的源码按基础仓锁定的提交初始化；已有源码和本地修改会保留。需要一起检查构建工具时，也可以调用：

```text
$nxnative-workflow 初始化环境。
```

初始化后应确认源码已检出、子模块版本正确，并明确工具链缺项。首次构建可能还需要补齐依赖，具体见 [环境初始化](nxnative-workflow/references/environment.md)。

如需在终端手动初始化子模块，可从基础仓根目录执行：

```sh
git submodule update --init --recursive
```

## 2. 打开游戏下载地址，准备本地输入

```text
$nxnative-workflow 打开游戏下载地址。
```

该调用用浏览器打开流程中指定的地址：<https://ns.nsyxck.com/#home>。网页打开后，将准备好的游戏文件放入本地 `games/` 目录，并告知 Codex 实际路径。已有本地文件时可以直接继续。

茶杯头案例使用游戏本体、1.3.7 更新和 DLC。提取游戏还需要本地 `prod.keys`，可调用：

```text
$nxnative-workflow 准备游戏与密钥。
```

Skill 会核对输入路径、游戏版本和密钥格式，并在实际提取时验证适用性。游戏和密钥由使用者本地准备，保存在 Git 忽略的目录中；聊天和录屏只展示文件名及检查结果。详见 [密钥准备](nxnative-workflow/references/keys.md)。

## 3. 启动 subagent，完成茶杯头的 iOS 适配

```text
$nxnative-workflow 起一个subagent，完成“茶杯头”游戏的适配。
```

subagent 先检查现有输入、游戏 profile 和构建记录，再完成需要的适配与构建：提取并合并游戏内容、处理原生代码、转换 shader、编译并打包 IPA。

本案例的主产物和复建资料位于：

```text
games/Cuphead-iOS/
├── Cuphead.ipa       游戏 IPA
├── input/            输入与分阶段构建产物
├── tooling/          隔离工具链
├── verification/     校验报告
├── build.log         构建日志
└── rebuild.sh        复建入口
```

完成时核对 IPA 的版本、arm64 架构、完整性和哈希，并保存构建日志。已有匹配的 IPA 时，检查通过即可复用。具体构建步骤见 [游戏编译](nxnative-workflow/references/game-build.md)。

## 4. 编译 LiveContainer，准备并完成宿主安装

```text
$nxnative-workflow 完成LiveContainer的编译和安装准备。
```

这一步编译包含 SaveSync 修改的 LiveContainer，再使用打包脚本与指定基础 IPA 组装合并包。首次执行需要在本地准备脚本要求的基础包 `LCRebuild/LiveContainer+SideStore.ipa`，脚本会核对其哈希。

输出为：

```text
LCRebuild/build/LiveContainer+SideStore-SaveSync.ipa
LCRebuild/build/LiveContainer+SideStore-SaveSync.validation.json
```

Skill 检查构建和包校验结果，准备好 Impactor 与待安装文件。已有合格产物时直接复用。单独准备安装可调用 `$nxnative-workflow 准备LiveContainer安装`。

在 Mac 的 Impactor 中完成安装：

1. USB 连接并解锁 iPad，按提示信任电脑。
2. 将宿主合并 IPA 拖入 Impactor，选择目标 iPad，完成账号登录。
3. 已有安装需要覆盖升级时，沿用原来的 Apple ID/Team 和实际宿主 bundle ID。
4. 选择 **Only Register Main Bundle**，点击 **Install**，确认设备上能打开 LiveContainer。

随后在 iPad 打开 **设置 → 隐私与安全 → 开发者模式**，按提示重启，并在重启后确认开启。

详见 [LiveContainer 构建](nxnative-workflow/references/livecontainer-build.md) 和 [Impactor 安装交接](nxnative-workflow/references/livecontainer-install.md)。

## 5. 生成 Pair File，配置 SideStore 和 JIT License

先准备官方配对工具及文件保存位置：

```text
$nxnative-workflow 准备生成pair file。
```

准备好后继续：

```text
$nxnative-workflow 生成pair file文件。
```

这里的 Pair File 指当前 iPad 的 **Remote Pairing `.plist` 文件**。使用官方 `idevice_pair`，首次使用需先完成工具安装；具体下载与安装说明见 [安装后配置](nxnative-workflow/references/sidestore-setup.md)。

在工具中操作：

1. 保持 iPad 通过 USB 连接、解锁并信任 Mac，选择这台设备。
2. 配对类型选择 **Remote pairing / RPPairing**，点击 **Create**。
3. 点击 **Validate**，确认验证成功。
4. 点击 **Save to file…**，将 `.plist` 保存到 `games/_devices/ipad/pairing/`，再传到同一台 iPad 的“文件”App。

已有当前设备的有效文件可以验证后复用。工具打开和目录准备完成后，仍要以 **Create、Validate、保存的实际结果**确认文件生成；遇到系统未授权界面自动化时，在工具中手动完成这些点击。

接着调用：

```text
$nxnative-workflow 配置SideStore续签。
```

在 LiveContainer 内置 SideStore 中依次设置：

- **Settings → Pairing File Management → Remote Pairing File → Import / Replace File**，导入刚保存的文件。
- 执行 **Activate** 和 **Set as Preferred**，确认 Active、Preferred 都显示 **`rppairing`**。
- 后端保持 **`idevice`**，连接 **LocalDevVPN**，完全退出并重新打开 LiveContainer，再执行 **Refresh All**，检查续签结果。

最后导入 **JIT License（JIT-less 签名证书）**：

```text
$nxnative-workflow 配置JIT-less。
```

在 LiveContainer 中进入 **Settings → JIT-less → Import Certificate from SideStore**，确认导入，再执行 **JIT-Less Mode Diagnose → Test JIT-Less Mode**。

茶杯头首次启动曾提示 `JITLess mode is required since iOS 26`，用户确认导入该证书后解决。开发者模式、Remote Pairing 和证书导入分别检查，续签成功后仍需完成 JIT-less 设置。

## 6. 将茶杯头 IPA 导入 LiveContainer 并安装

```text
$nxnative-workflow 把“茶杯头”的ipa文件导入LiveContainer，并进行安装。
```

Skill 定位已编译的 `Cuphead.ipa` 和目标 iPad，将游戏包传入实际安装的 LiveContainer，再发起容器内导入。本案例使用 Xcode 自带的 `devicectl` 完成传输和安装交接。

导入完成后检查：设备端 IPA 大小是否一致、游戏是否出现在容器中、版本和安装完成记录是否正确。已有匹配的安装时直接核验，保留现有游戏与存档。详见 [设备导入](nxnative-workflow/references/game-install.md)。

在 iPad 上打开 LiveContainer，启动茶杯头，检查标题画面、实际场景、输入、声音以及保存后重新进入的结果。需要继续排查时调用：

```text
$nxnative-workflow 验证运行。
```

## 重复调用与录屏

每一步都可以独立调用。先检查本地已有文件和设备状态，已经完成且核验通过的工作直接复用，缺失或失败的环节再继续处理。

录屏时可以约定：检查通过后简短回复 **“对应的工作已完成”**。需要手动确认、尚未生成文件或检查失败时，应说明实际状态，再完成剩余操作。

## 仓库与本地资料

| 路径 | 用途 |
|---|---|
| `.agents/skills/nxnative-workflow` | Skill 发现入口，指向 `../../nxnative-workflow` |
| `nxnative-workflow/` | Skill 定义、界面元数据和流程参考 |
| `nxnative/` | 游戏适配与构建源码，作为 submodule 锁定版本 |
| `LiveContainer/` | 宿主与 SaveSync 源码，作为 submodule 锁定版本 |
| `games/` | 本地游戏、密钥、配对文件、工具、构建缓存和游戏 IPA，Git 忽略 |
| `LCRebuild/build/` | 宿主编译产物、合并 IPA、报告和日志，Git 忽略 |

发布仓库时，将 Skill 源文件与发现链接一起提交；更新子仓后，先确保对应提交已推送且可访问，再发布基础仓中的子模块指针。游戏、密钥、配对文件、证书和构建产物保留在本地。详见 [发布前检查](nxnative-workflow/references/runbook.md#发布给其他人之前)。

## 进一步阅读

- [Skill 定义](nxnative-workflow/SKILL.md)：各项调用与执行规则。
- [完整流程与验收](nxnative-workflow/references/runbook.md)：阶段完成标准和常见问题。
- [已验证基线](nxnative-workflow/references/validated-baseline.md)：茶杯头案例的版本与证据。
- [可选存档同步](nxnative-workflow/references/save-sync.md)：双设备存档同步的配置和独立验收，实际 iCloud 往返仍需验证。
- [nxnative 架构梳理与鸿蒙适配边界](docs/nxnative-architecture.md)：离线构建、CPU 原生执行、系统兼容层、图形与 Shader 链路，以及鸿蒙适配需要保留、替换和验证的部分。
