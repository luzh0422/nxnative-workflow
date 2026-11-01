# 准备 LiveContainer 安装

入口：`$nxnative-workflow 准备LiveContainer安装`。

将已构建的合并 IPA 交给外部 Impactor 安装工具，复用用户现有的签名与安装流程。Impactor 不作为本仓库 submodule。

## 核验本地产物

默认产物为基础仓 `LCRebuild/build/LiveContainer+SideStore-SaveSync.ipa`，对应报告为同目录的 `LiveContainer+SideStore-SaveSync.validation.json`。用户指定其他文件时使用该文件及其对应报告，不能把旧报告套用到新 IPA。

- 重新计算 IPA 的 SHA-256，与报告的 `output_sha256` 比对，核对文件大小。
- 检查 ZIP CRC 与重复路径，读取主 `Info.plist`，确认 LiveContainer 身份、版本与 Save Sync 标记符合本次构建。
- 按报告中的 `preserved_resource_sha256` 逐一验证保留资源，检查架构记录及主宿主未签名状态，确认没有意外的 provisioning profile。若报告缺失或校验不一致，先解决产物问题。

## 定位并打开 Impactor

本工作区实际安装位置为 `/Applications/Impactor.app`，已验证版本 2.7.2。其他使用者先查找本机已有 App，或采用其指定路径；缺失时说明依赖，在用户要求安装后使用 Homebrew 官方 cask 或官方安装包。

macOS 已验证的交接命令：

```sh
open -a /Applications/Impactor.app
open -R 'LCRebuild/build/LiveContainer+SideStore-SaveSync.ipa'
```

从基础仓根目录执行，或使用解析后的绝对路径。前一条启动工具，后一条在 Finder 选中文件。文件直接导入 App 的接口尚未验证，不声称执行 `open -a Impactor <ipa>` 已完成导入。

打开命令在沙箱内失败时，根据当前会话允许的权限审核机制处理。退出状态 0 表示系统接受打开请求，不证明窗口已显示、IPA 已导入或设备安装成功。

## 用户确认安装

提示用户将选中的 IPA 拖入 Impactor，确认正确设备，沿用其现有 Apple ID/Team 与已安装宿主的 bundle ID，选择 **Only Register Main Bundle** 后点击 Install。不能把其他设备或旧文档中的 team/bundle ID 当作所有用户的默认值。保留包内扩展和 SideStore 资源，按现有流程覆盖升级，避免为升级删除已有容器。

账号登录、双重验证及最终安装由用户在 Impactor 中完成；此次准备调用不读取账号存储、导出证书或配对文件，也不自动签名或安装设备。需要后续自动化时按用户明确授权的操作范围另行验证。

官方仓库包含 `plumesign` CLI，但检查过的签名参数没有暴露 `Only Register Main Bundle` 对应配置，默认 app 类型也不是 LiveContainer 专用配置。尚未证明其与 GUI 的合并包处理等价，不能直接替换为普通 CLI 签名安装命令。

## 结果记录

记录本次选择的 IPA 路径、版本、哈希、Impactor 路径/版本以及打开请求结果。实际安装成功、SideStore 刷新、JIT-less 和 iCloud 存档同步分别记录用户反馈或设备验证证据；不能由“校验并打开”推断这些阶段成功。

用户确认安装完成后，接入 [安装后配置与 SideStore 续签](sidestore-setup.md)：先在设备 **设置 → 隐私与安全 → 开发者模式** 中开启并完成重启确认，再按 USB Remote Pairing、导入激活与 Refresh All 的流程续接。可独立调用 `$nxnative-workflow 配置SideStore续签`，不重新执行安装或覆盖已有有效配对设置。

参考：[LiveContainer 官方安装流程](https://livecontainer.github.io/docs/installation/lc_sidestore)、[Impactor 官方仓库](https://github.com/claration/impactor)、[覆盖安装说明](https://impactor.claration.dev/docs/troubleshooting/faq/)、[CLI 签名实现](https://github.com/claration/impactor/blob/main/apps/plumesign/src/commands/sign.rs)。
