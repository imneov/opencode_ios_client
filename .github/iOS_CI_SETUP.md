# iOS GitHub Actions 自动构建配置指南

## 概述

已为项目配置了GitHub Actions自动构建流水线，可以在推送代码时自动编译iOS应用并生成安装包。

## 工作流触发条件

- 推送到 `master` 或 `main` 分支
- 创建Pull Request到 `master` 或 `main` 分支
- 手动触发（通过GitHub网页端Actions页面）
- 创建Git标签时会自动发布Release

## 构建产物

- **IPA文件**: 可在iPhone上安装的应用包
- **构建日志**: 完整的编译过程日志
- **自动Release**: 当推送标签时自动创建GitHub Release

## 代码签名配置（可选）

### 如果不配置签名证书
- 应用会以Debug模式构建
- 生成的IPA需要通过TestFlight或其他方式分发
- 适用于内部测试

### 如果要配置签名证书（推荐）
需要在GitHub仓库的 Settings > Secrets and variables > Actions 中添加以下密钥：

#### 必需的Secrets:
1. **BUILD_CERTIFICATE_BASE64**: iOS开发者证书的Base64编码
2. **P12_PASSWORD**: 证书的密码
3. **BUILD_PROVISION_PROFILE_BASE64**: Provisioning Profile的Base64编码
4. **KEYCHAIN_PASSWORD**: 用于临时密钥链的密码（可以是任意安全密码）
5. **TEAM_ID**: Apple开发者团队ID

#### 如何获取这些值：

1. **获取证书和Provisioning Profile**:
   ```bash
   # 导出证书为.p12文件
   # 在Keychain Access中找到你的开发者证书，右键 > 导出 > 保存为.p12文件

   # 从Apple开发者中心下载Provisioning Profile
   # 访问 https://developer.apple.com/account/resources/profiles/list
   ```

2. **转换为Base64编码**:
   ```bash
   # 证书转Base64
   base64 -i YourCertificate.p12 | pbcopy

   # Provisioning Profile转Base64
   base64 -i YourProfile.mobileprovision | pbcopy
   ```

3. **获取Team ID**:
   - 登录Apple开发者中心
   - 在Account页面查看Team ID

## 使用方法

### 1. 自动构建
- 推送代码到main/master分支即可触发构建
- 构建完成后在Actions页面下载IPA文件

### 2. 手动构建
- 访问GitHub仓库的Actions页面
- 选择"iOS Build and Archive"工作流
- 点击"Run workflow"按钮

### 3. 发布版本
```bash
# 创建并推送标签
git tag v1.0.0
git push origin v1.0.0

# GitHub会自动创建Release并附带IPA文件
```

## 安装IPA到iPhone

### 方法1: 通过TestFlight（推荐）
1. 上传IPA到App Store Connect
2. 邀请测试用户
3. 用户通过TestFlight安装

### 方法2: 直接安装（需要设备UDID在Provisioning Profile中）
1. 下载IPA文件
2. 使用Xcode或其他工具安装到设备

### 方法3: 通过第三方分发平台
- 蒲公英（pgyer.com）
- fir.im
- diawi.com

## 故障排除

### 常见问题：

1. **签名失败**
   - 检查证书和Provisioning Profile是否匹配
   - 确认Team ID正确
   - 验证设备UDID已添加到Provisioning Profile

2. **构建失败**
   - 检查Xcode版本兼容性
   - 确认项目设置正确
   - 查看构建日志定位具体错误

3. **IPA无法安装**
   - 确认设备UDID在Provisioning Profile中
   - 检查证书是否过期
   - 验证Bundle ID匹配

## 自定义配置

如需修改构建配置，编辑 `.github/workflows/ios-build.yml` 文件：

- 修改Xcode版本：更改 `XCODE_VERSION` 环境变量
- 调整构建配置：修改 `xcodebuild` 参数
- 更改触发条件：修改 `on` 部分的配置

## 安全注意事项

- 所有敏感信息都存储在GitHub Secrets中，不会暴露在代码中
- 临时密钥链在构建完成后会自动清理
- 证书和Provisioning Profile仅在构建过程中临时使用