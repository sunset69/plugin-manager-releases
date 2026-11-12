# assets 目录说明

`Auto Release` 工作流会把这里的 `*.apk` 与 `*.zip` 作为 Release 附件上传。

## 为什么 APK 放在仓库里

发布仓库只存放「可分发的成品」，App 源码与签名在私有开发仓库，GitHub Actions
无法直接构建（需要私有仓库源码、Android SDK 版本与签名密钥）。因此采用：

1. 本地完成 arm64 release 构建；
2. 把生成的 `PluginManager-release-arm64.apk` 覆盖到本目录；
3. 更新根目录 `version.json` 的 versionCode / versionName / size / sha256；
4. 推送 `v<versionName>` tag，由工作流创建或更新 Release 并上传附件。

如果以后希望完全由 CI 构建，需要：把源码放进可被 Actions 访问的仓库，配置
`KEYSTORE_BASE64`、`KEYSTORE_PASSWORD`、`KEY_ALIAS`、`KEY_PASSWORD` 等 secrets，
并在工作流里增加 Gradle 构建步骤。代价是签名密钥要交给 Actions 保管。

## 更新步骤

1. 用新 APK 覆盖本目录同名文件（或删除旧 APK 后放入新的）。
2. 确认 `version.json` 的 `size` 与 `sha256` 与新 APK 一致，工作流会校验。
3. `git push` 后推送 tag：`git tag v1.3.0 && git push origin v1.3.0`。
4. 也可以在 Actions 页面手动运行 `Auto Release`，输入要补发的 tag。

重复触发不会新建 Release：存在则更新标题/正文，附件以 `--clobber` 覆盖同名文件。
