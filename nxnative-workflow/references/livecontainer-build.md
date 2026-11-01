# 编译 LiveContainer 与合并 SideStore

入口：`$nxnative-workflow 编译LiveContainer`。

使用用户选定的源码和已有 Save Sync 修改，先读子仓说明。已有匹配且校验通过的合并 IPA 可直接转 [安装交接](livecontainer-install.md)，不要求每次重编。

## 输入与工作区

- 源码：基础仓 `LiveContainer/`，保留未提交的用户修改，不自动切换提交。
- 基础合并 IPA：本案例为 `LCRebuild/LiveContainer+SideStore.ipa`。脚本 `LiveContainer/scripts/package_save_sync.py` 锁定基础包身份和 SHA-256，以该脚本为准，不能通过删除校验来接受其他包。
- 输出、DerivedData 和日志：`LCRebuild/build/`，已 Git 忽略。输出 IPA 已存在时选新名字，不覆盖或先删除。
- 先确认 Save Sync 源码、工程引用、打包脚本存在。基础仓的旧子模块指针不能替代未发布的新功能源码；见 [发布前检查](runbook.md)。

## 构建依赖

核对 OpenSSL 和 litehook 的锁定提交及源码存在。需要初始化且无用户开发现场时，从 LiveContainer 子仓执行：

```sh
git submodule update --init --recursive --depth 1 -- OpenSSL litehook
```

浅克隆只减少历史下载，不更换锁定版本。中断后可能出现 HEAD 正确但工作区未检出的情况；核对 `litehook/src/litehook.h` 等实际文件。仅在确认是本次中断创建的空工作区、没有用户内容时恢复锁定文件，不能把一般删除状态当作可自动清理对象。

如项目提供 SaveSyncCore 测试，可先运行：

```sh
swift test --scratch-path ../LCRebuild/build/swift-tests
```

本案例 18 项测试通过，这是核心本地夹具测试，不代表 iCloud/真实设备已验证。

## 编译与打包

以下命令从基础仓的 `LiveContainer/` 执行；路径按本工作区布局适配用户提供的 LCRebuild 示例。

```sh
DEVELOPER_DIR=/Applications/Xcode.app/Contents/Developer \
xcodebuild \
  -project LiveContainer.xcodeproj \
  -scheme LiveContainer \
  -configuration Release \
  -sdk iphoneos \
  -destination 'generic/platform=iOS' \
  -derivedDataPath ../LCRebuild/build/DerivedData \
  CODE_SIGNING_ALLOWED=NO \
  CODE_SIGNING_REQUIRED=NO \
  CODE_SIGN_IDENTITY= \
  ARCHS=arm64 \
  build
```

确认退出 0 和 `BUILD SUCCEEDED`，产物为 `LCRebuild/build/DerivedData/Build/Products/Release-iphoneos/LiveContainer.app`。保留原始日志；实际 Xcode 不在该位置时使用已核对的路径。

```sh
python3 scripts/package_save_sync.py \
  --base ../LCRebuild/LiveContainer+SideStore.ipa \
  --app ../LCRebuild/build/DerivedData/Build/Products/Release-iphoneos/LiveContainer.app \
  --output ../LCRebuild/build/LiveContainer+SideStore-SaveSync.ipa
```

这是本案例的输出名；文件已存在时必须换名。脚本通过后得到 IPA 和同名 `.validation.json`。报告需与本次 IPA 一一对应。

## 验收与交接

核对输出哈希、ZIP 完整性、主宿主身份/Save Sync 标记、arm64、无 provisioning profiles，以及基础包保留资源逐字节一致。主宿主未签名；嵌入组件的原 vendor/ad-hoc 签名可能保留，不能声称所有二进制都完全无签名。

本案例保留 163 个 SideStore/settings/intents/widget 文件，具体数量以所选基础版本为准。不将自行编译的其他 SideStore 替换入合并包。

将通过校验的文件与报告交给 Impactor。本步骤不登录账号、不安装设备、不执行 iCloud 存档同步；相关阶段按 [端到端流程](runbook.md) 分别验证。
