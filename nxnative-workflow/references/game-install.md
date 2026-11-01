# 将游戏 IPA 导入 LiveContainer

入口：`$nxnative-workflow 安装游戏到LiveContainer`，可附 IPA 与目标设备。

此操作是将游戏放入已安装的 LiveContainer 并在容器内导入，不通过 Impactor 把游戏另装成独立系统 App。需要用户已要求传输/安装；仅编译游戏的任务不自动触发本步骤。

## 前置状态

确认游戏 IPA 已校验；目标设备与系统/硬件符合该构建；宿主安装、开发者模式、续签和 JIT-less 已按 [安装后配置](sidestore-setup.md) 核对。若缺证书，可保留已经完成的传输/导入，先处理 JIT-less，不要求重传大文件。

保持目标设备 USB 连接并解锁；设备/电脑出现信任提示时由用户确认。只定位本次设备和 LiveContainer，不扫描账号、证书、配对内容或无关 App 数据。

## 发现设备和宿主

优先使用本机已安装的 Xcode devicectl，先核对当前命令的 `--help`。示例从基础仓根目录执行，变量必须设为实际值：

```sh
xcrun devicectl list devices
DEVICE_ID='<目标设备标识>'
xcrun devicectl device info apps --device "$DEVICE_ID" \
  --search LiveContainer --include-container-paths
```

目标必须明确；多个候选设备/宿主无法确定时询问，不把案例设备的 UDID、Team、bundle ID 写成默认值。查询实际宿主 `bundleIdentifier`、`dataContainerPath` 和 `containerAccessible`；访问不被支持时改用 Finder 文件共享交接，不绕过设备权限。

列出宿主 Documents 顶层及 `Documents/Applications`，检查目标 IPA 与同 bundle 游戏是否已存在。已有文件不盲目覆盖，已有游戏的替换/保留选项交给用户选择，不删除存档。

## 文件传输

确认目标没有冲突后执行当前 CLI 支持的复制命令：

```sh
LC_BUNDLE='<设备上实际 LiveContainer bundle ID>'
IPA_PATH='<本地已校验 IPA 的绝对路径>'
xcrun devicectl device copy to --device "$DEVICE_ID" \
  --source "$IPA_PATH" --destination Documents/Cuphead.ipa \
  --domain-type appDataContainer --domain-identifier "$LC_BUNDLE"
```

`Cuphead.ipa` 是案例文件名，其他游戏选对应名字。5 GiB 左右文件需要确认设备容量和传输状态；保留回执到 Git 忽略的 `games/_devices/`。核对设备端文件大小与本地一致，不能只凭打开 Finder 声称传输成功。本次未做设备端完整文件回读 SHA-256，不补写不存在的校验结果。

## 容器内安装

有两种已理解的入口：

- 手动：用户在 LiveContainer 的导入入口选“文件”App 中的该 IPA。
- 已验证自动交接：当前源码支持 `livecontainer://install?url=<URL 编码的本地 file URL>`。可通过 `devicectl device process launch --payload-url` 传给实际宿主。

第二种入口必须按当前源码核对，不猜测固定容器 UUID。使用复制回执中的设备端 `destination` file URL，按 URL query 编码构造：

```python
from urllib.parse import urlencode
payload_url = 'livecontainer://install?' + urlencode({'url': destination_file_url})
```

```sh
xcrun devicectl device process launch --device "$DEVICE_ID" \
  --payload-url "$PAYLOAD_URL" "$LC_BUNDLE"
```

打开请求成功只是导入请求已发出。若出现替换选择、权限或其他对话框，由用户确认；不自动终止正在运行的游戏或替用户选择破坏性替换。原 IPA 位于 Documents 根目录，是否保留由用户决定，不为节省空间自动删除。

## 验证已导入

确认 `Documents/Applications/<实际游戏 bundle ID>.app/` 出现，按需要仅读取该游戏的 `Info.plist` 和 `LCAppInfo.plist`，核对名称、版本、补丁状态与 `installationDate`。本案例 `LCPatchRevision=7`；该数值不是所有未来版本的通用要求。

安装完成标记不代表签名/JIT-less 或游戏运行都正常。接着由用户启动并记录画面/错误。遇到 iOS 26 要求 JITLess 的提示，回到宿主导入 JIT License/证书，再诊断和重试；不能写配置标志伪装证书就绪。

本案例设备传输与安装记录在本地 `games/_devices/transfer/`，这些设备标识和路径不随仓库或视频公开。
