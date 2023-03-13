# Jenkins持续集成


安装最新的 LTS 版本：

```js
brew install jenkins-lts
```
启动jenkins服务:

```js
jonytang@Mac ~ % brew services start jenkins-lts
==> Downloading 
https://formulae.brew.sh/api/formula.jws.json
#########################################
############################### 100.0%
==> Successfully started `jenkins-lts` (label: 
homebrew.mxcl.jenkins-lts)
jonytang@Mac ~ %
```
浏览器打开 `http://localhost:8080`:

![Pasted Graphic.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/caa41d8e13f147ea88b9b42fb43547af~tplv-k3u1fbpfcp-watermark.image?)

输入密码：

![Pasted Graphic 1.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0eaa7671f2fd4c688ac0838e605468ee~tplv-k3u1fbpfcp-watermark.image?)

>选择第一个安装推荐的插件

等待安装：

![Pasted Graphic 2.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/242204fc8d14416c9083ca4faa39f620~tplv-k3u1fbpfcp-watermark.image?)

有些会安装失败，点击重试:

![Pasted Graphic 3.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8e68b3f98c8042f5bf67ca3dff0232ad~tplv-k3u1fbpfcp-watermark.image?)

安装完成，创建管理员：

![Pasted Graphic 4.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4a49cb97168949aaacd72c04926a71e4~tplv-k3u1fbpfcp-watermark.image?)

创建完成：

![Pasted Graphic 5.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dd5a62b41bf44244913e043456e1ef85~tplv-k3u1fbpfcp-watermark.image?)

点击按钮进入：

![Pasted Graphic 6.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fa9b4518164f4aaa8ef685f35d890aa1~tplv-k3u1fbpfcp-watermark.image?)

报一些错：

![Pasted Graphic 8.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6680251c0b014332870b664c2c91c2ac~tplv-k3u1fbpfcp-watermark.image?)

根据提示，安装插件，重启jenkins:
```js
brew services start jenkins-lts
```

![Pasted Graphic 9.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/219f82d470524f75bddb5d42563d072d~tplv-k3u1fbpfcp-watermark.image?)

>已经没有红色的提示

可以看到最下面安装的插件

![Pasted Graphic 10.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8368ee6ef0a044df81f2c320855d5971~tplv-k3u1fbpfcp-watermark.image?)

配置环境变量，终端输入：

```js
echo $PATH
```
复制粘贴：

![Pasted Graphic 11.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8d016fe0199947c9be050d484f25384b~tplv-k3u1fbpfcp-watermark.image?)

新建任务，选择第一个：

![Pasted Graphic 13.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d0a92d283bde4535bd8e0211d917ca75~tplv-k3u1fbpfcp-watermark.image?)

选择git，输入仓库的https地址：

![1__#$!@%!#__Pasted Graphic.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/94863c248e304aa6a976f4bea2862cda~tplv-k3u1fbpfcp-watermark.image?)

Credentials点击添加，用户名和密码输入github的账号和密码：

![Pasted Graphic 14.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/579007658c83484698a41cbaee609fd2~tplv-k3u1fbpfcp-watermark.image?)

勾选丢弃构建：

![1__#$!@%!#__Pasted Graphic 1.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/241ccffd220649d6bbea29656a1de6ea~tplv-k3u1fbpfcp-watermark.image?)

增加构建步骤 -> 执行shell：

![1__#$!@%!#__Pasted Graphic 2.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aedbfcb27cc144978057f85c4c8dc244~tplv-k3u1fbpfcp-watermark.image?)

添加打包脚本，保存。
```js
 #!/bin/sh
export LANG=en_US.UTF-8
USERNAME="hugengwei"
# 1.设置配置标识,编译环境(根据需要自行填写 release ｜debug )
configuration="release"

# 工程名(根据项目自行填写)
APP_NAME="TuWanApp"

# TARGET名称（根据项目自行填写）
TARGET_NAME="TuWanApp"

# ipa前缀（根据项目自行填写）
IPA_NAME="点点开黑"

# info.plist路径
#project_infoplist_path="./${TARGET_NAME}/Info.plist"
# 取版本号
#bundleShortVersion=$(/usr/libexec/PlistBuddy -c "print CFBundleShortVersionString" "${project_infoplist_path}")

#bundleVersion=$(/usr/libexec/PlistBuddy -c "print CFBundleVersion" "${project_infoplist_path}")

# 日期
DATE=$(date +%Y%m%d-%H-%M-%S)
# 工程文件路径
ARCHIVE_NAME="${APP_NAME}_${DATE}.xcarchive"
# 存放ipa的文件夹名称（根据自己的喜好自行修改）
IPANAME="${APP_NAME}_${DATE}_IPA"

# 工程根目录#工程源码目录(这里的${WORKSPACE}是jenkins的内置变量表示(jenkins job的路径):/Users/xxx/.jenkins/workspace/TestDemo/)
# ${WORKSPACE}/TestDemo/ 中的TestDemo根据你的项目自行修改
CODE_PATH="${WORKSPACE}"

# 要上传的ipa文件路径 ${username} 需要换成自己的用户名
ROOT_PATH="/Users/${USERNAME}/Desktop/Jenkins"
ARCHIVE_PATH="${ROOT_PATH}/Archive/${ARCHIVE_NAME}"
IPA_PATH="${ROOT_PATH}/Export/${IPANAME}"
echo "ARCHIVE_PATH: ${ARCHIVE_PATH}"
echo "IPA_PATH: ${IPA_PATH}"
echo "IPA_PATH:\n${IPA_PATH}">> export_history.txt

# 导包方式(这里需要根据需要手动配置:AdHoc/AppStore/Enterprise/Development)
EXPORT_METHOD="AdHoc"
# 导包方式配置文件路径(这里需要手动创建对应的XXXExportOptionsPlist.plist文件，并将文件复制到根目录下[我这里在源项目的根目录下又新建了ExportPlist文件夹专门放ExportPlist文件])
if test "$EXPORT_METHOD" = "AdHoc"; then
    EXPORT_METHOD_PLIST_PATH=${CODE_PATH}/ExportOptions/AdHocExportOptions.plist
elif test "$EXPORT_METHOD" = "AppStore"; then
    EXPORT_METHOD_PLIST_PATH=${CODE_PATH}/ExportOptions/AppStoreExportOptios.plist
elif test "$EXPORT_METHOD" = "Enterprise"; then
    EXPORT_METHOD_PLIST_PATH=${CODE_PATH}/ExportOptions/EnterpriseExportOptions.plist
else
    EXPORT_METHOD_PLIST_PATH=${CODE_PATH}/ExportOptions/DevelopmentExportOptions.plist
fi

# 指ipa定输出文件夹,如果有删除后再创建，如果没有就直接创建
if test -d ${IPA_PATH}; then
    rm -rf ${IPA_PATH}
    mkdir -pv ${IPA_PATH}
     echo ${IPA_PATH}
else
     mkdir -pv ${IPA_PATH}
fi

# 进入工程源码根目录
cd "${CODE_PATH}"

# 执行pod
pod install

#mkdir -p build

# 清除工程
echo "++++++++++++++++clean++++++++++++++++"
xcodebuild clean -workspace ${APP_NAME}.xcworkspace -scheme ${APP_NAME} -configuration ${configuration}

# 将app打包成xcarchive格式文件
echo "+++++++++++++++++archive+++++++++++++++++"
xcodebuild archive -workspace ${APP_NAME}.xcworkspace -scheme ${APP_NAME} -configuration ${configuration} -archivePath ${ARCHIVE_PATH}

# 将xcarchive格式文件打包成ipa
echo "+++++++++++++++++ipa+++++++++++++++++"
xcodebuild -exportArchive -archivePath ${ARCHIVE_PATH} -exportPath "${IPA_PATH}" -exportOptionsPlist ${EXPORT_METHOD_PLIST_PATH} -allowProvisioningUpdates

# 删除工程文件
# echo "+++++++++删除工程文件+++++++++"
# rm -rf $ARCHIVE_PATH

# 蒲公英上传结果日志文件路径
PGYERLOG_PATH="${IPA_PATH}/upload_pgyer_log"
# 创建蒲公英上传结果日志文件夹
mkdir -p ${PGYERLOG_PATH}
# 创建蒲公英上传结果日志文
touch "${PGYERLOG_PATH}/log.txt"

 ```
- [iOS Fastlane自动构建打包、发布、部署jenkins](https://www.jianshu.com/p/dac1ce3d7de8)
- [Jenkins-iOS自动打包流程](https://www.jianshu.com/p/531c959b8cf8)
- [Jenkins 实现iO项目自动打包](https://www.jianshu.com/p/68a19f28c51a)

