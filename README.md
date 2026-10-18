# 捕鱼大师 · iOS IPA 构建仓库

本仓库把单文件 HTML 游戏（`index.html`）打包成 iOS App（IPA）。

## 在线试玩（免安装）

仓库根目录的 `index.html` 通过 GitHub Pages 发布：

- iPhone Safari 打开 Pages 链接 → 分享 → **添加到主屏幕**，即可全屏横屏运行（无需签名、永久有效）

## IPA 构建

每次 push 到 `main`（或手动触发 Actions）会自动在 macOS runner 上：

1. `xcodegen` 生成 Xcode 工程（`ios-shell/`）
2. `xcodebuild` 编译 Release 版（未签名）
3. 组装 `Payload/FishLobby.app` 并压缩为 `FishLobby-unsigned.ipa`
4. 上传为 Actions artifact

### 下载 IPA

Actions → 最近一次 `Build IPA` run → Artifacts → `FishLobby-unsigned-ipa`

### 关于签名（重要）

本流水线产出的是**未签名 IPA**，只能用于：

- 有 Apple 开发者证书者自行重签：`codesign` + `zsign` / `fastlane`
- 越狱设备安装

要安装到普通 iPhone，需要以下其一（本仓库均未配置，需自行补齐）：

| 方式 | 要求 | 有效期 |
| --- | --- | --- |
| Apple ID 免费签名 | Apple ID + 专用密码 + Xcode | 7 天 |
| 个人开发者账号 | $99/年，证书 + provisioning profile | 1 年 |
| 企业签名 | $299/年 Apple Developer Enterprise Program | 1 年 |

配置签名时，在 workflow 的 xcodebuild 步骤加入：

```yaml
- name: Import certificate
  env:
    P12_BASE64: ${{ secrets.P12_BASE64 }}
    P12_PASSWORD: ${{ secrets.P12_PASSWORD }}
    PROFILE_BASE64: ${{ secrets.PROFILE_BASE64 }}
  run: |
    # 导入 p12 到临时 keychain，安装 mobileprovision，然后：
    # xcodebuild ... CODE_SIGN_STYLE=Manual DEVELOPMENT_TEAM=XXXXXXXXXX
    # 最后用 xcodebuild -exportArchive 导出签名 IPA
```

### 本地构建（有 Mac 时）

```bash
cd ios-shell
brew install xcodegen
xcodegen generate
open FishLobby.xcodeproj   # Xcode 选择签名团队后 Archive 导出
```

## 目录结构

```
index.html                      # 游戏本体（单文件，Pages 根目录）
.github/workflows/build-ipa.yml # IPA 构建流水线
ios-shell/
  project.yml                   # xcodegen 工程定义（横屏锁屏/全屏）
  App/                          # Swift 源码（WKWebView 容器）
  game/index.html               # 游戏副本（打进 App bundle）
```
