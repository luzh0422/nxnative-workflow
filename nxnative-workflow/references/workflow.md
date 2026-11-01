# 流程与源码入口

下列路径相对于 nxnative 源码根目录。这是源码导航与通用命令模板；本次茶杯头实际执行记录见 [验证基线](validated-baseline.md)，构建步骤见 [游戏编译](game-build.md)。其他游戏运行前仍需确认当前脚本选项、依赖和输入路径。

## 按任务读取

| 当前任务 | 源码或文档入口 |
|---|---|
| 约束与目录布局 | `AGENTS.md` |
| 安装依赖与打包 | `README.md` 的 Pack an IPA；`docs/development.md` |
| 构建脚本和实际参数 | `scripts/build_ipa_from_nsp.py`，运行 `python3 scripts/build_ipa_from_nsp.py -h` |
| 游戏版本与 profile | `dm/games/README.md`、`dm/games/<id>/README.md`、`game.tsv` |
| 提取、调查与启动 | `docs/agents/01-extract.md`、`02-survey.md`、`03-bringup.md` |
| 图形转换 | `docs/agents/04-graphics.md`；按游戏实际 API 选择 NVN、Vulkan 或 OpenGL 路径 |
| 崩溃、卡死、错误画面 | `docs/agents/symptoms.md` → 对应 `docs/reference/` 条目；诊断方法见 `05-debugging.md` |

## 构建入口

从 nxnative 源码根目录执行，先将下列变量设为实际路径：

```sh
NSP_DIR='/path/to/game-folder'
KEYS_PATH='/path/to/prod.keys'
python3 scripts/build_ipa_from_nsp.py "$NSP_DIR" -k "$KEYS_PATH"
```

脚本默认选择目录中的最新版本；指定版本使用 `-p`，先核对该参数与游戏文件的对应关系。不要混入另一游戏的输入。

已有 profile 时先读取该游戏说明及已验证 build。没有匹配 profile 时，脚本会调查游戏，输出 `<NSP folder>/<title ID>_survey/survey.md`，再尝试生成临时 profile 和 IPA；临时产物可能无法启动。

仅对无匹配 profile 的游戏做静态调查，可使用：

```sh
python3 scripts/build_ipa_from_nsp.py "$NSP_DIR" -k "$KEYS_PATH" --survey-only
```

`--survey-only` 是无 profile 分支的选项，不应作为任意游戏的通用 dry-run。

需要 Mac 开发构建时使用：

```sh
python3 scripts/build_ipa_from_nsp.py "$NSP_DIR" -k "$KEYS_PATH" --platform mac
```

此模式生成 Mac 开发所需产物并输出运行命令，不生成 IPA。根据输出实际启动并观察，不能仅凭构建完成声称游戏运行。

## 产物与排障

- 构建中间产物及分阶段日志通常在 `<NSP folder>/<id>_build/`、`logs/`；以脚本实际输出为准。
- 临时 profile 在 `<NSP folder>/<id>_build/profile/<id>/`。手动运行工具时按仓库要求设置 `DM_GAMES_DIR`，并显式指定游戏：工具使用 `DM_GAME`，Make 与 `build_ipa.sh` 使用 `GAME`。
- 自动构建可能启动 Mac host 进行 heap 检查或运行时 shader 捕获；不能假设整个构建只有静态处理。
- 修改 shader chain 时遵循 `AGENTS.md` 中的版本头更新、对应游戏 shader 重建和 host 重建要求；不要用任意重新编译的 translator 替换锁定版本。
- 从第一处失败的阶段日志定位原因。复现记录包含源码 revision、游戏版本及模块 build ID、命令与必要环境变量、日志位置、观察结果。
- 真机验证记录设备、系统版本、测试场景及结果，结合用户回传的 `Documents/log.txt` 定位；未取得真机证据的阶段标记为未验证。
