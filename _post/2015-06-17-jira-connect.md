---
layout: ecsoya
title: 基于Play的JIRA插件开发小结
category: Java
---

公司除了Eclipse插件，也有做[JIRA](https://www.atlassian.com/software/jira)的插件[Gantt Chart Project for JIRA](http://www.jiraproject.com/)。自从JIRA Cloud（Atlassian Cloud）发布之后，我们就在研究如何去开发JIRA Cloud上面的插件。JIRA Cloud插件又叫做Jira Connect Add-on。

### 工具
开发Jira Connect插件的工具有很多，作为一个Java开发人员，我首先找到的就是一个基于Java的工具，[Play Java Module for Atlassian Connect（atlassian-connect-play-java）](https://bitbucket.org/atlassian/atlassian-connect-play-java)。

`atlassian-connect-play-java`是一个基于[Play Framework](https://www.playframework.com/)的工具，能够帮我们快速的建立开发Jira Connect插件的环境。它的优势如下：

* 能够自动安装插件到http://localhost:2990/jira
* 能够验证JWT请求。
* 使用Play框架中的很多优秀的API。
* 能够使用JIRA中的REST API。
* 能够很好的整合[AUI](https://docs.atlassian.com/aui/latest/)。 


### 开发环境搭建

1. 下载并安装[Atlassian SDK](https://developer.atlassian.com/docs/getting-started/set-up-the-atlassian-plugin-sdk-and-build-a-project)。
2. 下载并安装[Play Framework 2.2.x](https://downloads.typesafe.com/play/2.2.6/play-2.2.6.zip)，Play2.2以上的版本变化较多，不适用于本文。

> 安装完之后，要确保系统环境变量中是否设置正确，可以通过`atlas-version`和`play version`在命令行中验证。

### JIRA Application with Play

1. **New Application**：打开命令行，通过`play new [appName]`命令，创建一个Java工程，记得要选择***Create a Simple Java application***。
2. **Start Application**：进入刚才创建的Application目录，运行命令`play`。
3. **配置IDE**：运行命令`eclipse`或`eclipse with-source=true`，将Play Application配置成Eclipse Java Application。
4. **在IDE中编辑**：将工程导入到IDE中。
5. **添加Atlassian的maven资源库**：在`build.sbt`文件中加入

		resolvers += "Atlassian's Maven Public Repository" at "https://maven.atlassian.com/content/groups/public"

		resolvers += "Local Maven Repository" at "file://" + Path.userHome + "/.m2/repository"
6. **添加atlassian-connect-play-java工具**：在`build.sbt`文件中加入
	
		libraryDependencies ++= Seq(
   			javaJdbc,
   			cache,
   			"com.atlassian.connect"          % "ac-play-java_2.10"                          % "0.10.1" withSources(),
		)
7. **atlassian-connect.json**：参考[Atlassian Connect Documentation](https://developer.atlassian.com/static/connect/docs/latest/guides/getting-started.html)，在根目录添加一个`atlassian-connect.json`文件。
8. **其它设置**：参考[atlassian-connect-play-java](https://bitbucket.org/atlassian/atlassian-connect-play-java#markdown-header-enables-multi-tenancy)进行其它设置。
9. **运行**：在命令行中输入`run`来运行。如果Jira已经启动，运行之后会自动安装到http://localhost:2990/jira，如果没有自动安装，请参考下面步骤。
10. **安装**：
	1. 运行本地Jira Cloud实例。
		
			atlas-run-standalone --product jira --version 6.5-OD-08-001 --bundled-plugins com.atlassian.bundles:json-schema-validator-atlassian-bundle:1.0.4,com.atlassian.webhooks:atlassian-webhooks-plugin:2.0.0,com.atlassian.jwt:jwt-plugin:1.2.2,com.atlassian.upm:atlassian-universal-plugin-manager-plugin:2.19.0.3-D20150522T180613,com.atlassian.plugins:atlassian-connect-plugin:1.1.37 --jvmargs -Datlassian.upm.on.demand=true
	2. 安装，play运行之后，默认的在`9000`端口，在Jira的插件管理页中，选择上传：http://localhost:9000/atlassian-connect.json。

> 
> 1. 最新的Jira Cloud版本可以在[Release Notes](https://developer.atlassian.com/static/connect/docs/latest/resources/release-notes.html)中取到。
> 2. `build.sbt`是用来添加所有依赖的文件，详细设置请参考[SBT Settings](https://www.playframework.com/documentation/2.2.x/SBTSettings)
> 3. `conf/routes`文件中存储的是HTTP到Java的路径映射，详细请参考[Play Java Routing](https://www.playframework.com/documentation/2.2.x/JavaRouting)
> 


### 参考资料
1. [Atlassian Connect Documentation](https://developer.atlassian.com/static/connect/docs/latest/guides/introduction.html)。
2. [Play Java Module for Atlassian Connect](https://bitbucket.org/atlassian/atlassian-connect-play-java)
3. [Play Framework](https://www.playframework.com/documentation/2.2.x/Home)
4. [Whoslooking-connect](https://bitbucket.org/atlassian/whoslooking-connect)
