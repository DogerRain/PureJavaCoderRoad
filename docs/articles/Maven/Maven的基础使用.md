一个约定俗成的Maven项目的目录结构：

```xml
HelloWorld					工程名
|---src						源码
|---|---main				放主程序
|---|---|---java			存放java源文件
|---|---|---resources		存放框架或其他工具的配置文件
|---|---test				存放测试程序
|---|---|---java			存放java源文件
|---|---|---resources		存放框架或其他工具的配置文件
|---pom.xml					Maven的核心配置文件
```

## 构建配置文件的类型

构建配置文件大体上有三种类型

| 类型                  | 在哪定义                                                     |
| --------------------- | ------------------------------------------------------------ |
| 项目级（Per Project） | 定义在项目的POM文件pom.xml中                                 |
| 用户级 （Per User）   | 定义在Maven的设置xml文件中 (%USER_HOME%/.m2/settings.xml)    |
| 全局（Global）        | 定义在Maven全局的设置xml文件中 (%M2_HOME%/conf/settings.xml) |

## 仓库

Maven仓库有三种类型：

- 本地仓库（local）
- 中央仓库（central）
- 远程仓库（remote）

### 本地仓库

Maven本地仓库是你电脑上的某个目录地址。

Maven的`conf/setting.xml`文件可以配置：

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 
   http://maven.apache.org/xsd/settings-1.0.0.xsd">
      <localRepository>C:/MyLocalRepository</localRepository>
</settings>
```

Maven本地仓库保存着你项目所有的依赖（库的、插件的jar包等等）。当你运行Maven构建时，Maven会自动下载所有依赖的jar包到本地仓库中，这会帮助避免每次项目构建时项目的依赖参考都存储在远程的主机上。

使用`mvn install` 则可以把本地的包推到本地仓库，这样可以被本地其他项目引用。

### 中央仓库

Maven中央仓库是由Maven社区提供的仓库。它包含大量的常用库。 Maven 当未在本定仓库中找到任何的依赖时，Maven会使用下面的URL在中央仓库中搜索：[https://mvnrepository.com/](https://mvnrepository.com/ ) 

中央仓库几个关键概念：

- 中央仓库仓库是由Maven社区管理。　
- 中央仓库不需要配置。
- 搜索中央仓库需要接入互联网。

Maven社区提供了一个URL [**http://search.maven.org/#browse** ](http://search.maven.org/#browse)供用户浏览Maven中央仓库的内容。用这种方法，开发者可以搜索中央仓库中所有可用的库。

### 远程仓库

有时Maven在中央仓库中也找不到指定的依赖，这时Maven会停止构建进程并且输出错误信息到控制台。为了防止这样的情况，Maven提出了**远程仓库**的概念，即开发者自己定制的包含库或者其他项目jar包的仓库。

一般公司都有自己的远程仓库。

可以在pom.xml进行配置：

```xml
   <repositories>
      <repository>
         <id>companyname.lib1</id>
         <url>http://download.companyname.org/maven2/lib1</url>
      </repository>
      <repository>
         <id>companyname.lib2</id>
         <url>http://download.companyname.org/maven2/lib2</url>
      </repository>
   </repositories>
```

这个一般都是配置在全局文件 settings.xml 中的 	



当我们执行Maven构建命令时，Maven将开始按照下面的顺序搜索依赖库。

- 第1步 - 搜索本地仓库中的依赖，如果没有找到，进入第2步，否则若找到则进行后续的处理。
- 第2步 - 搜索中央仓库中的依赖，如果没有找到并且指定了远程仓库，则进入第4步，否则若找到，则下载依赖到本地仓库进行后续的查询。
- 第3步 - 如果远程仓库没有指定，Maven将简单地停止构建并且抛出异常（找不到依赖）。
- 第4步 - 搜素远程仓库中的依赖，如果找到则下载依赖到本地仓库进行后续的查询，否则Maven按预想地停止构建并且抛出异常（找不到依赖）。



