# 游戏编译

入口：`$nxnative-workflow 编译游戏`，可附游戏名、目录、密钥路径、目标版本与输出目录。

## 输入和复用

先读 nxnative 的 `AGENTS.md`、构建脚本参数及 `dm/games/<title ID>/README.md`。检查实际 profile/build ID、更新和 DLC。已有 IPA 时可先验证；只有需要重新构建才执行。

本地默认布局：

```text
games/<游戏工作区>/
  input/           原包副本及 <title ID>_build/
  prod.keys        本地密钥，不提交
  tooling/         必要的隔离工具
  <游戏名>.ipa
  build.log
  rebuild.sh
  verification/
```

保留原包；输入在其他文件夹时，可在 Mac 上使用 APFS clone 复制到工作区，避免为了构建移动或修改原文件。空间估算要包含原包/解压/更新合并/shader/IPA，不只看最终 IPA 的大小。

密钥先按 [密钥准备](keys.md) 核验，实际提取结果才说明适用性。新游戏无匹配 profile 时遵循调查流程，不承诺能直接运行。

## 工具实测

根据所检出代码检查 Xcode/iPhoneOS SDK、Metal、XcodeGen、.NET **SDK**、spirv-cross/相关图形工具、Python 与模块。可用性必须由实际执行确定：

```sh
xcode-select -p
xcrun -sdk iphoneos metal --version
xcodegen --version
dotnet --list-sdks
python3 -c 'import lz4, capstone, fontTools'
```

Python 环境和 PATH 要与执行构建的进程一致。NSZ/XCZ 输入还需要 nsz；zstd/backports 的条件以 Python 版本及当前脚本为准，本次 Python 3.14 使用内置 zstd。下载包可保存在 `games/_tools/` 或本游戏 tooling；官方安装到系统的工具维持其正常安装位置。

安装缺失依赖按会话授权推进；不要重复安装已有工具。当前会话权限失败先按工具返回处理，不把它当成组件缺失，也不关闭 TLS 校验。

## 构建命令

从基础仓根目录设定**实际路径**，以下为 shell 模板：

```sh
REPO_ROOT="$PWD"
GAME_WORK="$REPO_ROOT/games/<游戏工作区>"
cd "$REPO_ROOT/nxnative"
python3 scripts/build_ipa_from_nsp.py "$GAME_WORK/input" \
  -k "$GAME_WORK/prod.keys" -p '<实际包版本>' \
  -o "$GAME_WORK/<游戏名>.ipa" --progress lines
```

`<...>` 为待替换内容，不直接执行该模板。未指定 `-p` 时脚本可能选最新版本，必须先确认符合用户目标。隔离环境存在时，在本次进程设置 DOTNET_ROOT、DOTNET_CLI_HOME、NUGET_PACKAGES 和 PATH，不修改用户全局配置。

流水线按输入自动选择解压/提取合并、代码重写、shader 拆分/转换、主 App 编译和 IPA 打包。NSZ 解压阶段的 NCA 验证结果、提取模块 build ID、每阶段退出状态与日志都要保留。图形路径以实际 NVN/Vulkan/OpenGL profile 为准，不把茶杯头的阶段数量或 shader 数量变成通用要求。

已有缓存时先让脚本判断复用；不要为重试全删工作区。只启动一份针对同一工作目录的构建。

## 验收与复建

检查最终退出 0、文件大小、ZIP CRC/重复路径、Info.plist 的身份/版本/最低系统、主程序 arm64 和未签名状态，保存 SHA-256 与日志路径到本地 verification。需要提取检查时，不把 ZIP 中的路径原样不加校验地写入工作区之外。

记录源码提交、profile、包版本、模块 build ID、工具版本与命令。保存根据自身位置解析的重建脚本；移动工作区时还需修正 venv 绝对启动器并重新验证解释器/SDK，不能只移动目录就承诺复建环境可用。

本案例已有可执行复建脚本：

```sh
sh games/Cuphead-iOS/rebuild.sh
```

只有用户要求复建且本机目录仍齐备时才执行。案例详情见 [已验证基线](validated-baseline.md)。
