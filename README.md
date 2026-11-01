# nxnative-workflow

基于 nxnative 的 Switch → iOS 工作流 Skill，覆盖游戏构建、LiveContainer 宿主准备、SideStore 续签、JIT-less 和设备内导入。茶杯头 1.3.7 的整体链路已由用户确认跑通；具体版本和验证范围见 [已验证基线](nxnative-workflow/references/validated-baseline.md)。

完整操作见 [端到端流程与验收](nxnative-workflow/references/runbook.md)；讲解与录制见 [10–12 分钟口播、分镜与剪辑脚本](nxnative-workflow/references/video-tutorial.md)。当前视频交付为脚本，尚未生成视频文件。

## 克隆与使用

```sh
git clone git@github.com:luzh0422/nxnative-workflow.git
cd nxnative-workflow
```

在 Codex 中打开这个目录。仓库内的 `.agents/skills/nxnative-workflow` 相对符号链接是项目发现入口，无需每个人手动复制 Skill。发布时须将链接和 Skill 源文件一起提交。客户端未显示时，检查是否进入正确项目、链接能否解析，再重启客户端检查。

第一步：

```text
$nxnative-workflow 初始化环境
```

检查并初始化基础仓锁定版本的 `nxnative`、`LiveContainer` 及所需嵌套依赖，检查工具链并报告缺项。保留已有子仓的本地开发现场；源码初始化后仍需验证实际构建。

后续按当前进度调用，不必重跑已经完成的步骤：

| 调用 | 用途与参考 |
|---|---|
| `$nxnative-workflow 准备游戏与密钥` | 确认输入、更新/DLC、密钥格式及适用性；[密钥准备](nxnative-workflow/references/keys.md) |
| `$nxnative-workflow 编译茶杯头` | 输入提取、适配、打包与验证；[游戏构建](nxnative-workflow/references/game-build.md) |
| `$nxnative-workflow 编译LiveContainer` | 编译新宿主并组装保留 SideStore 的合并包；[宿主构建](nxnative-workflow/references/livecontainer-build.md) |
| `$nxnative-workflow 准备LiveContainer安装` | 校验合并 IPA/报告并打开 Impactor；[安装交接](nxnative-workflow/references/livecontainer-install.md) |
| `$nxnative-workflow 配置SideStore续签` | 开发者模式、Remote Pairing、后端/VPN、Refresh All；[安装后配置](nxnative-workflow/references/sidestore-setup.md) |
| `$nxnative-workflow 配置JIT-less` | 导入用户所称的 JIT License，执行 JIT-less 诊断；同上 |
| `$nxnative-workflow 安装游戏到LiveContainer` | 传输游戏 IPA、容器内导入和独立核验；[设备导入](nxnative-workflow/references/game-install.md) |
| `$nxnative-workflow 验证运行` | 启动、场景、输入、声音、保存及重启读档；[验收](nxnative-workflow/references/runbook.md) |
| `$nxnative-workflow 配置存档同步` | 两台设备授权同一云目录，独立验证往返、冲突和恢复；[同步流程](nxnative-workflow/references/save-sync.md) |
| `$nxnative-workflow 录制教程` | 读取完整口播、时间码、画面操作和剪辑说明 |

已有游戏文件时直接准备输入；需要时也可独立调用：

```text
$nxnative-workflow 打开游戏下载网址
```

用浏览器打开用户指定的 <https://ns.nsyxck.com/#home>。这是可选资源入口，打开网页不代表已经准备好本地输入。

流程有两个独立 IPA：**宿主合并包通过 Impactor 安装，游戏包导入 LiveContainer**。在 Impactor 中核对原账号、设备、bundle ID 和 **Only Register Main Bundle**。设备端续签就绪后，仍需在 LiveContainer 导入 JIT License/签名证书并做 JIT-less 测试；本次 iOS 26 启动提示已由用户确认通过导入该证书解决。

## 源码依赖

普通 clone 即可获得 Skill 文件。需要一次下载全部源码时可使用：

```sh
git clone --recurse-submodules git@github.com:luzh0422/nxnative-workflow.git
```

已有 clone 也可在根目录手动初始化基础仓记录的版本：

```sh
git submodule update --init --recursive
```

上述基础仓克隆命令及 LiveContainer 使用 SSH 地址，需要可用的 GitHub SSH 身份及仓库访问权限。网络或执行权限受限时，在本地终端初始化，再让 Skill 继续检查。升级上游版本应单独发起并验证后更新依赖指针。

## 仓库布局

| 路径 | 用途 |
|---|---|
| `.agents/skills/nxnative-workflow` | 提交到 Git 的相对符号链接，指向 `../../nxnative-workflow`，作为 Codex 发现入口 |
| `nxnative-workflow/` | Skill 的唯一源文件目录，含 `SKILL.md`、界面元数据及 references |
| `nxnative/` | submodule：<https://github.com/seedds/nxnative> |
| `LiveContainer/` | submodule：`git@github.com:luzh0422/LiveContainer.git` |
| `games/` | Git 忽略的本地游戏、密钥、工具链、构建中间产物与 IPA，不随仓库发布 |
| `LCRebuild/build/` | Git 忽略的宿主 DerivedData、合并 IPA、报告与日志 |

Skill 入口：[SKILL.md](nxnative-workflow/SKILL.md)。验证状态：[progress.md](nxnative-workflow/references/progress.md)。其他使用者 clone 后需自行准备 `games/` 中的本地资料。

## 本地构建目录

茶杯头本次构建保存在 `games/Cuphead-iOS/`：`Cuphead.ipa` 为主产物，`input/` 包含游戏输入及分阶段中间产物，`tooling/` 保存隔离工具链及下载，`keys-download/` 保存密钥下载与原件，`verification/` 保存校验报告，根目录保存日志。原始游戏包仍位于 `games/已加速- 茶杯头 Cuphead/`。

迁移后的重建脚本按自身位置解析路径，可从基础仓根目录执行：

```sh
sh games/Cuphead-iOS/rebuild.sh
```

## 维护者首次发布

基础仓包含发现链接、Skill 和两个 submodule 的注册信息。LiveContainer 的 Save Sync 功能已归档为本地提交 `7930cdf11649803d65b7709179ad7c81c25f8b38`，基础仓锁定该功能版本。**本地提交完成后，远程 clone 的复现仍取决于对应提交已推送且可访问。**

发布时先推送 LiveContainer 的功能提交，确认子仓远程能取得该提交，再发布包含对应 gitlink、Skill 和发现入口的基础仓提交。以后更新子仓时同样先审查功能、测试、文档和打包脚本。不要把本地密钥、配对、证书、基础 IPA 或构建缓存加入仓库。

检查发现链接为 `120000`、子仓为 `160000`，并以一次新的普通 clone 验证 Skill 发现，再验证锁定子仓初始化。详细检查见 [发布前检查](nxnative-workflow/references/runbook.md#发布给其他人之前)。本轮按用户要求创建本地提交，尚未推送。
