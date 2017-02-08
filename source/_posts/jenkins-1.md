---
layout: post
title: 使用jenkins + git + 蒲公英 对 iOS 项目进行持续集成
date: 2015-11-14 19:58:17
tags:
- jenkins
- 持续集成
categories: 持续集成
---
该文章主要介绍了如果配置 jenkins ，从而完成自动编译打包发布的功能
<!-- more -->
## 持续集成解决什么问题？
>在项目进行到测试阶段的时候，每天需要给多台测试设备安装最新的 build 的版本让测试人员验证已经修改的 BUG，测试早上拿来一堆设备让你给他安装新版本，或者是让你打包一个 ipa 文件发给他们，在进行大规模测试时这样的行为每天要进行很多次。正是这些毫无价值的重复劳动浪费你大量的时间。构建持续集成则可以实现一键式发布最新 build 。

## 持续集成的基本流程
>使用 jenkins 自动拉取 git 仓库最新版本代码，并编译打包成 ipa 文件 使用脚本上传到蒲公英应用管理平台。



## 构建完善的持续集成系统
### 一、安装 jenkins
* 安装 jdk : 
	* jenkins 依赖 jdk ，所以需要安装先安装 jdk ，可以从 jdk 官网 [http://www.oracle.com/technetwork/java/javase/downloads/index.html](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 下载 dmg 进行安装。
* 下载 jenkins : 

	* 安装 homebrew ：[homebrew官网](http://brew.sh/)
			
			$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
	* 用 homebrew 安装 jenkins

			$ brew install jenkins //等待安装完成
	
	你也可以从 [jenkins](http://jenkins-ci.org/) 官网上下载最新的 jenkins.war 包
	
* 启动 jenkins :
  
	* 进入 jinkns 目录 homebrew 的安装路径，如果你是从官网上下载 war ，则 cd 进你 war 所在的目录（中间的 1.636 的版本号，根据你具体的安装版本而定，可以先 cd 到上层目录，然后 ls 来查看）
	
		`$ cd /usr/local/Cellar/jenkins/1.636/libexec/`		 		
	* 运行 jenkins :
	 	
		`$ java -jar jenkins.war --httpPort=8080`
	
	* httpPort 用来指定访问的端口

	* 等待运行完毕用浏览器打开 [http://localhost:8080/](http://localhost:8080/) 就可以看到 jenkins 的界面了。

jenkins 界面如下
![主界面](http://7xo9cc.com1.z0.glb.clouddn.com/blog_jenkins_home)
----------------------
### 二、安装插件
*  **安装所需的插件**  
 * GitLab Plugin
 * Gitlab Hook Plugin
 * Xcode integration
 * Environment Injector Plugin
 * Credentials Plugin
 * Keychains and Provisioning Profiles Management
 
 
* **安装插件步骤**
	* 点击左侧的**系统管理（Manage Jenkins）**=> **管理插件（Manage Plugins）**

     ![进入插件](http://7xo9cc.com1.z0.glb.clouddn.com/blog_manager_plugus.png)
     
    * 选择**可选插件（Available）**=> 在右上角**搜索框**分别输入上面提到的插件

    * 选中所有需要安装的插件之后点击 **直接安装（Install without restart）**

       ![插件下载](http://7xo9cc.com1.z0.glb.clouddn.com/blog_download_plugus.png)


  * 等待插件全部安装完毕之后就可以配置你的第一个工程啦
   
   
-------------------------
### 三、配置你的第一个job
1. 创建一个新的 jenkins job ,选择**构建一个自由风格的软件项目（Freestyle project）**。点击 OK 之后我们就进入到了一个配置界面。通过这个配置界面我们可以对我们的构建过程做出很多的自定义配置。
 
	![创建job](http://7xo9cc.com1.z0.glb.clouddn.com/blog_create_home)


2. 你可以指定一个 jenkins 的自定义的工作空间,在配置路径中可以使用 jenkins 内置的环境变量方便使用，具体环境变量详情访问： [jenkins wiki](https://wiki.jenkins-ci.org/display/JENKINS/Building+a+software+project) [内置的环境变量]([jenkins wiki](https://wiki.jenkins-ci.org/display/JENKINS/Building+a+software+project))。可以使用 **$+环境变量的方式来使用** 如 `$JOB_NAME`

  ![路径设置](http://7xo9cc.com1.z0.glb.clouddn.com/blog_path_setting.png)
  
  
3. 首先你需要在**源码管理（Source Code Management）**中选择 Git，输入你项目 git 地址，点击 Add 添加 git 地址的账号和密码。（当然你也可以使用 ssh 方式连接，这里用 http 连接方式）。在 **Branches to build** 里面填写你需要构建的分支。这样 jenkins 每次构建之前的时候都会从这个 git 的地址上拉取最新的代码。

![git设置](http://7xo9cc.com1.z0.glb.clouddn.com/blog_git_setting.png)


4. 接下来就需要添加一系列的构建步骤包括用之前安装好的 Xcode 插件对工程进行编译打包出来一个 ipa 文件，使用[蒲公英](www.pgyer.com)提供的上传 ipa 的接口把 ipa 文件上传到你的蒲公英账户中，之后可以通过蒲公英生成的链接下载并安装程序。
5. 添加Xcode构建步骤：在下面 **构建（Build）** 的部分点击 **增加构建步骤（Add build step）**，选择 Xcode ,首先输入你工程中需要编译的 Target 

![Xcode设置](http://7xo9cc.com1.z0.glb.clouddn.com/blogBulid_Xcode_Setting.png)

到此，如果你的电脑上已经安装了你项目所使用的证书的话，或者你项目中 code sign 设置的是自动，你可以先保存设置，回到 JOB 的主界面，点击**立即构建**则可以立即构建，如果构建成功的话则会在你设置的 ipa 输出文件夹中看见一个打包好的 ipa 文件。

![构建](http://7xo9cc.com1.z0.glb.clouddn.com/blog_BUILD_NOW.png)

**如果发现构建失败了！！！**
点击红色的构建记录，然后点击 **Console Output** 查看错误原因。


![错误信息](http://7xo9cc.com1.z0.glb.clouddn.com/blog_BUILD_CONT.png)
-----------------------------
到此步骤，一般可能会出现一下两个错误。

1. 证书错误(修改项目证书设置，jenkins 可以指定打包证书，接下里会有教程说明)
![证书错误](http://7xo9cc.com1.z0.glb.clouddn.com/blog_BUILD_FILE2.png)

2. 路径设置错误（大部分情况是项目工程真是路径和默认路径不符）
![路径错误](http://7xo9cc.com1.z0.glb.clouddn.com/blog_BUILD_FILE.png)

	**在配置中的这个地方设置一下真是的项目路径**
	![路径设置](http://7xo9cc.com1.z0.glb.clouddn.com/blog_xcode_path)
	
>解决了以上问题你就可以在 jenkins 的工作路径找到一个 ipa 文件啦
当！当！当！当！！！
完成了第一个里程碑!!!

---------
### 增加构建步骤将你的ipa文件在打包完成后自动上传至[蒲公英](www.pgyer.com)平台

* 首先你需要到[蒲公英官网](www.pgyer.com)上注册一个账号
* 注册完成之后可以到[蒲公英文档](http://www.pgyer.com/doc/api#uploadApp)中查看上传 App 的 API 以及上传应用所需要的 apiKey 和 uKey。
* 接下来进入 jenkins 的 job 的配置页面，在 Xcode 构建下面 增加一个构建步骤 选择 `Execute shell`。
* 再输入框中输入以下命令（利用 curl 工具 调用蒲公英 api 上传 ipa 文件）
 
	```
	curl -F "file=@$WORKSPACE/$BUILD_NUMBER/$VERSION-$BUILD_NUMBER.ipa" \
	-F "uKey=替换成你的uKey" \
	-F "_api_key=替换成你的apiKey" \
	-F "isPublishToPublic=2" \
	http://www.pgyer.com/apiv1/app/upload
	```
	其中 `file=@`后面的内容是你 ipa 存在的路径，就是你之前设置的 ipa 	输出路径 + ipa 文件名。


**设置完成保存之后，点击立即构建开始新的一次构建，等待其构建成功之后你就可以在蒲公英管理后台上看到你成功上传的 ipa 文件啦，然后你的设备通过蒲公英设置的自定义网址则可以下载完成安装！！！**

完成此工作之后，以后测试同学在让你发布一个新版本，你就可以在 jenkins 上一键发布，然后告诉她 发布好了！！ 去下载！！！

-------------
### 当然这个过程总体来说还不足够简便和实用，中间还有做的事情有很多。

* 在 jenkins 上自定义打包证书（开发证书、发布证书）
* 可以利用蒲公英提供的 api 来写一个简单的 **build下载器**，让你们测试的同学的设备上都安装上，集中管理你们公司的所有产品，提供便利的下载方式，还可以集成 jpush 实现版本发布完成，自动推送。
* 可以利用 jenkins 上提供的 api 在 **build下载器** 中添加构建按钮，这样以来测试同学就能自己去构建发布下载啦！
* 还可以在编译打包之前动态的修改程序的图片，在图标上添加版本号，或者 debug beta 等字样，轻松区分测试版本和正式版本。

之后教程我会逐一介绍上面功能的实现过程。

