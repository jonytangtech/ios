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

<img width="987" alt="Pasted Graphic" src="https://user-images.githubusercontent.com/126937296/224756791-cb12e01c-2ef5-47e7-9a09-b5ba438d6f7d.png">

输入密码：

<img width="979" alt="Pasted Graphic 1" src="https://user-images.githubusercontent.com/126937296/224756848-bea11e7e-4c32-4ce6-b6c5-1f1fbc6ed9cb.png">

>选择第一个安装推荐的插件

等待安装：

<img width="983" alt="Pasted Graphic 2" src="https://user-images.githubusercontent.com/126937296/224756946-a42db3c6-04ad-45b8-99c6-f292c5a399f8.png">

有些会安装失败，点击重试:

<img width="982" alt="Pasted Graphic 3" src="https://user-images.githubusercontent.com/126937296/224757016-a4cea8ea-fb73-443a-96da-a4a1613826bb.png">

安装完成，创建管理员：

<img width="985" alt="Pasted Graphic 4" src="https://user-images.githubusercontent.com/126937296/224757067-f7d0574a-7cfa-463b-a3e4-e76af5759fb4.png">

创建完成：

<img width="615" alt="Pasted Graphic 5" src="https://user-images.githubusercontent.com/126937296/224757124-9dc7125c-d73c-4f91-acd6-790a8149d596.png">

点击按钮进入：

<img width="1240" alt="Pasted Graphic 6" src="https://user-images.githubusercontent.com/126937296/224757171-4481397a-c0ea-447d-98f0-da7dfec4f1b8.png">

报一些错：

<img width="1014" alt="Pasted Graphic 8" src="https://user-images.githubusercontent.com/126937296/224757241-970aa648-192e-4b91-94a5-d6867305cd6c.png">

根据提示，安装插件，重启jenkins:
```js
brew services start jenkins-lts
```
<img width="1236" alt="Pasted Graphic 9" src="https://user-images.githubusercontent.com/126937296/224757290-b12c2220-b440-4f41-81c3-445da669e3ca.png">

>已经没有红色的提示

可以看到最下面安装的插件：

<img width="625" alt="Pasted Graphic 10" src="https://user-images.githubusercontent.com/126937296/224757340-5126c2fe-1f9c-4638-b328-3e55936400f1.png">

配置环境变量，终端输入：

```js
echo $PATH
```
复制粘贴：

<img width="1081" alt="Pasted Graphic 11" src="https://user-images.githubusercontent.com/126937296/224757526-84c04590-5464-4d0e-9de1-2e4eb1a7cc96.png">

新建任务，选择第一个：

<img width="707" alt="Pasted Graphic 12" src="https://user-images.githubusercontent.com/126937296/224757560-35b80b44-ecbc-4386-a724-9f0cf2bc74ba.png">

选择git，输入仓库的https地址：

<img width="859" alt="Pasted Graphic 13" src="https://user-images.githubusercontent.com/126937296/224757611-2d237b35-f815-4988-aa4e-bd5da76e8fe9.png">

Credentials点击添加，用户名和密码输入github的账号和密码：

<img width="941" alt="1__#$!@%!#__Pasted Graphic" src="https://user-images.githubusercontent.com/126937296/224757638-1c208b01-8221-4bf0-8aa3-78b4a221faea.png">

勾选丢弃构建：

增加构建步骤 -> 执行shell：<img width="617" alt="1__#$!@%!#__Pasted Graphic 1" src="https://user-images.githubusercontent.com/126937296/224757690-a1a096ce-3f06-47b9-8922-a30002c01534.png">

输入github用户名和密码

<img width="761" alt="Pasted Graphic 14" src="https://user-images.githubusercontent.com/126937296/224757845-6001f41d-f790-487e-a9e9-cc4ad129c627.png">

增加构建步骤 -> 执行shell：

<img width="381" alt="1__#$!@%!#__Pasted Graphic 2" src="https://user-images.githubusercontent.com/126937296/224758007-bf25dfad-f356-4b5b-a9ef-900a370a28ad.png">

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
 构建打包上传
 
 ## 参考：
- [iOS Fastlane自动构建打包、发布、部署jenkins](https://www.jianshu.com/p/dac1ce3d7de8)
- [Jenkins-iOS自动打包流程](https://www.jianshu.com/p/531c959b8cf8)
- [Jenkins 实现iO项目自动打包](https://www.jianshu.com/p/68a19f28c51a)

