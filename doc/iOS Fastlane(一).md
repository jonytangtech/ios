![](https://github.com/fastlane/fastlane/raw/master/fastlane/assets/fastlane_text.png?raw=true)
# Fastlane
>fastlane是一个旨在简化 Android 和 iOS 部署的开源平台，可以自动化开发和发布工作流程的各个方面。
<!-- 
<p><a class="badge" href="https://twitter.com/FastlaneTools"><img alt="Twitter: @FastlaneTools" src="https://img.shields.io/badge/contact-@FastlaneTools-blue.svg?style=flat" />
 -->
## 一、安装Xcode命令行工具
为`fastlane`安装Xcode命令行工具：
```js
xcode-select --install
```
<img width="498" alt="Pasted Graphic" src="https://user-images.githubusercontent.com/126937296/224544049-2550b7bd-350c-4fc8-8862-76d557297c82.png">

如果安装过会提示：

```js
xcode-select: error: command line tools are already installed, 
use "Software Update" in System Settings to install updates
```
## 二、安装fastlane
> 提前配置好`HomeBrew`管理工具

```js
brew install fastlane
```
查看`fastlane`版本：

```js
fastlane -v
fastlane installation at path:
/usr/local/Cellar/fastlane/2.206.2/libexec/gems/fastlane-2.206.2/bin/fastlane
-----------------------------
[✔] 🚀
fastlane 2.206.2
```
将终端导航到项目目录并运行：

```js
fastlane init
```
命令执行：

```js
[✔] 🚀
[13:13:34]: Sending anonymous analytics information
[13:13:34]: Learn more at https://docs.fastlane.tools/#metrics
[13:13:34]: No personal or sensitive data is sent.
[13:13:34]: You can disable this by adding `opt_out_usage` at the top of your Fastfile
[✔] Looking for iOS and Android projects in current directory...
[13:13:34]: Created new folder './fastlane'.
[13:13:34]: Detected an iOS/macOS project in the current directory: 'FastlaneDemo.xcodeproj'
[13:13:34]: -----------------------------
[13:13:34]: --- Welcome to fastlane 🚀 ---
[13:13:34]: -----------------------------
[13:13:34]: fastlane can help you with all kinds of automation for your mobile app
[13:13:34]: We recommend automating one task first, and then gradually automating more over time
[13:13:34]: What would you like to use fastlane for?
1. 📸  Automate screenshots
2. 👩‍✈️  Automate beta distribution to TestFlight
3. 🚀  Automate App Store distribution
4. 🛠  Manual setup - manually setup your project to automate your tasks
```
>[13:13:34]你想用快车道做什么?
>1. 📸自动截屏
>2. 👩✈️自动测试分发TestFlight</br>
>3. 🚀自动化应用商店分销
>4. 🛠手动设置-手动设置您的项目自动化您的任务


这边选择`4`：

```js
[14:29:49]: --- Setting up fastlane so you can manually configure it ---
[14:29:49]: ------------------------------------------------------------
[14:29:49]: Installing dependencies for you...
[14:29:49]: $ bundle update
[14:31:36]: --------------------------------------------------------
[14:31:36]: --- ✅  Successfully generated fastlane configuration ---
[14:31:36]: --------------------------------------------------------
[14:31:36]: Generated Fastfile at path `./fastlane/Fastfile`
[14:31:36]: Generated Appfile at path `./fastlane/Appfile`
[14:31:36]: Gemfile and Gemfile.lock at path `Gemfile`

```
项目里多了这三个文件：

<img width="319" alt="Pasted Graphic 1" src="https://user-images.githubusercontent.com/126937296/224544077-d78d4d89-03ce-47e2-a866-68ea2b333c7f.png">

## 三、安装蒲公英插件

```js
fastlane add_plugin pgyer
```
注册蒲公英，拿到`API Key`和`User Key`

<img width="618" alt="1__#$!@%!#__Pasted Graphic" src="https://user-images.githubusercontent.com/126937296/224544029-ebed45e8-23f8-44be-a42c-3b60bba7db56.png">

简单配置一下：

```js
default_platform(:ios)

platform :ios do
desc "Description of what the lane does"
# 打包时候用的名称   例如 fastlane app
lane :develop do |options|
target = "FastlaneDemo"
configuration = "Debug"
gym(scheme: target, configuration: configuration, export_method:"development")
pgyer(api_key: "xxxxxxxxxx”)
end
end
```
终端运行：

```js
fastlane develop
```
结果：
```js
+------+------------------+-------------+
|           fastlane summary            |
+------+------------------+-------------+
| Step | Action           | Time (in s) |
+------+------------------+-------------+
| 1    | default_platform | 0           |
| 2    | gym              | 78          |
+------+------------------+-------------+

[17:39:32]: fastlane.tools finished successfully 🎉
```
可以看到上传蒲公英成功：

![2__#$!@%!#__Pasted Graphic.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/19143787fec547bca08abe1e78e7615a~tplv-k3u1fbpfcp-watermark.image?)

点击应用信息：

![Pasted Graphic 4.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2c90fb9ea34b4e9a8c3684595feeb4da~tplv-k3u1fbpfcp-watermark.image?)

在项目本地也会生成一个`ipa`包

![Pasted Graphic 3.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9ca8e1f1e8ae4d188317e8ceb74fd5ec~tplv-k3u1fbpfcp-watermark.image?)
## 四、分发到`Appstore`


```js
# 发布到appstore
  lane :to_appstore do

    # 先获取当前项目中的bundle version + 1
    @build_version = get_info_plist_value(path: "#{$info_plist_path}", key: "CFBundleVersion").to_i + 1

    # 针对于 iOS 项目开发证书和 Provision file 的下载工具
    sigh(
      force: true,
      output_path: "./fastlane/crets"
    )

    # 设置 bundle version
    set_info_plist_value(
      path: "./so/Supporting Files/so-Info.plist", 
      key: "CFBundleVersion", 
      value: "#{@build_version}"
    )

    # 针对于 iOS 编译打包生成 ipa 文件
    gym(
      workspace: "#{$project_name}.xcworkspace",
      scheme: "#{$project_name}",
      clean: true,
      configuration: "Release",
      export_method: "app-store",
      output_directory: "ipa_build/release",
      output_name: "#{$project_abbreviation}"
    )

    # 用于上传应用的二进制代码，应用截屏和元数据到 App Store
    deliver(
      force: true,# 上传之前是否成html报告
      submit_for_review: false,# 上传后自动提交审核
      automatic_release: true,# 通过审后自动发布
      skip_binary_upload: false,# 跳过上传二进制文件
      skip_screenshots: true,# 是否跳过上传截图
      skip_metadata: false,# 是否跳过元数据
    )
  end
```
运行：

```js

[16:03:13]: $ bundle exec fastlane FastlaneDemo
[16:03:13]:
[16:03:13]: Get started using a Gemfile for fastlane https://docs.fastlane.tools/getting-started/ios/setup/#use-a-gemfile
+-----------------------+---------+--------+
|               Used plugins               |
+-----------------------+---------+--------+
| Plugin                | Version | Action |
+-----------------------+---------+--------+
| fastlane-plugin-pgyer | 0.2.4   | pgyer  |
+-----------------------+---------+--------+

[16:03:14]: ----------------------------------------
[16:03:14]: --- Step: Verifying fastlane version ---
[16:03:14]: ----------------------------------------
[16:03:14]: Your fastlane version 2.212.1 matches the minimum requirement of 2.68.2  ✅
[16:03:14]: ------------------------------
[16:03:14]: --- Step: default_platform ---
[16:03:14]: ------------------------------
[16:03:14]: Driving the lane 'ios FastlaneDemo' 🚀
[16:03:14]: ----------------------------------
[16:03:14]: --- Step: get_info_plist_value ---
[16:03:14]: ----------------------------------
[16:03:14]: ------------------
[16:03:14]: --- Step: sigh ---
[16:03:14]: ------------------

+-------------------------------------+-------+
|          Summary for sigh 2.212.1           |
+-------------------------------------+-------+
| force                               | true  |
| adhoc                               | false |
| developer_id                        | false |
| development                         | false |
| skip_install                        | false |
| include_mac_in_profiles             | false |
| ignore_profiles_with_different_name | false |
| skip_fetch_profiles                 | false |
| include_all_certificates            | false |
| skip_certificate_verification       | false |
| platform                            | ios   |
| readonly                            | false |
| fail_on_name_taken                  | false |
+-------------------------------------+-------+

[16:03:14]: To not be asked about this value, you can specify 
it using 'username'
[16:03:14]: Your Apple ID Username:
```
输入`Apple ID`去测试 ......

## 参考：
- [fastlane](https://docs.fastlane.tools/)
- [蒲公英官网文章](https://open.pgyer.com/9W0Hy3)
- [使用Fastlane自动部署APP](https://www.jianshu.com/p/2383fe17b9ec/)




