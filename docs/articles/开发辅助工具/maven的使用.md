Maven是一个跨平台的项目管理工具。作为Apache组织的一个颇为成功的开源项目，其主要服务于基于Java平台的项目创建，依赖管理和项目信息管理。

## Maven的安装：

Maven下载地址：https://maven.apache.org/download.cgi

Maven的安装需要依赖jdk，所以本地需要先安装jdk。

安装Maven之后，打开项目的`setting.xml`,可以配置本地仓库地址（默认在C盘），本地仓库地址就是你需要用到的jar的储存位置：

![ ](https://images-1253198264.cos.ap-guangzhou.myqcloud.com/image-20200526234043294.png)

然后配置环境变量：

1. 新建系统变量 `MAVEN_HOME` ,值为你的maven安装目录，例如我的是 `G:\apache-maven-3.5.4-bin`
2. 在 `path`后面加上 `%MAVEN_HOME%\bin`

window系统快捷键 Ctrl+R 输入cmd  ，回车 ，输入` mvn -v`, 输出以下即安装成功：

![ ](https://images-1253198264.cos.ap-guangzhou.myqcloud.com/image-20200526231731305.png)

安装成功后就可以在IDEA中配置Maven了，之后使用就可以使用Maven进行项目开发和打包了。



## Maven有什么用？

###  1. 依赖管理

我们平时使用的各种jar，例如连接MySQL，需要下载`mysql-connector-java-5.1.5-bin.jar`,使用spring框架，需要下载下面这堆jar

![ ](https://images-1253198264.cos.ap-guangzhou.myqcloud.com/image-20200526230328308.png)

下载了jar之后，你还需要手动添加进项目，这样子你的项目就可以使用jdbc、spring的API了，这样做太麻烦了，如果你使用Maven，就可以省去以上的工作，你只需要在项目的pom.xml文件引入依赖即可：

```xml
    <!-- 数据库驱动 -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        <version>5.1.35</version>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>4.0.2.RELEASE</version>
        </dependency>
```



这样子Maven就会自动帮你去Maven的中央仓库下载jar到本地。

maven的中央仓库地址：https://mvnrepository.com/  基本收集了所有的jar包。

### 2. 一键构建

我们的服务器要运行我们的项目，我们需要把项目打成jar或者war放到服务器的目录，例如Tomcat就是webaaps目录下面，Tomcat会自动识别war或者jar包（自动解压），我们只需要访问项目的路径即可进入到我们的项目。

如果你不想打成jar或者war，可以把编译后的.class文件直接放在webapps项目也是一样的效果，但是这种方法不高效。

打开IDEA，我们可以看到这里有很多maven的命令：

![ ](https://images-1253198264.cos.ap-guangzhou.myqcloud.com/image-20200526233020062.png)

你也可以自定义maven命令，进入项目的目录，输入`mvn clean package ` ，我们可以看到项目的输出目录就生成了jar

![ ](https://blog-1253198264.cos.ap-guangzhou.myqcloud.com/image-20200526233320788.png)

   这个jar就可以直接使用`java -jar MeiziTu-0.0.1-SNAPSHOT.jar` 命令运行了。

### 3. 项目拆分

有时候我们一个大项目，存在很多子项目，就可以使用maven进行拆分，让不同的子项目负责不同的功能模块。

![ ](https://images-1253198264.cos.ap-guangzhou.myqcloud.com/image-20200526232738803.png)

只需要在父项目的`pom.xml`文件加入配置即可：

![ ](https://images-1253198264.cos.ap-guangzhou.myqcloud.com/image-20200526233705682.png)



看到这里你大概知道maven有什么用了，你看到一个项目有`pom.xml`文件，就知道这个项目是使用的maven来管理的啦~



下面讲一下maven深入的用法：

## Maven深入用法

### 1. maven的jar使用流程

 

公司一般都有自己的私有仓库，毕竟公司项目都需要打包发布上线的，我们写的项目一般都会把代码上传到git仓库，使用第三方工具把代码构建形成一个jar，再丢到私有仓库。

这涉及maven的另外一个功能，就是maven在构建项目的时候，我们可以配置本地仓库地址，构建完成后就可以把jar上传到仓库，供大家使用，虽然本地也可以上传，比如下图的程序员A，但是这样子很危险，一般需要先提交代码到git，代码审核通过了，最后才能打成jar丢到私有仓库。

流程如下：

![ ](https://images-1253198264.cos.ap-guangzhou.myqcloud.com/流程.png)

**私有仓库**：存放我们需要的jar、我们在`pom.jar`可以引用，私有仓库找不到会去中央仓库下载。

**Git仓库**：Git是用来存放和管理代码的地方。私有仓库是管理jar。

**Jenkins**：可以理解为一个中介（功能很强大），打包、发布，用来持续集成。



### 2. 几个maven命令的区别

```xml
1、mvn compile  项目编译
2、mvn test 执行单元测试
3、mvn clean 清除jar、class文件

4、mvn pakage 打成一个jar或者war,一般在target目录
5、mvn install 打成一个jar或者war，布署到本地maven仓库
6、mvn deploy  打成一个jar或者war，布署到本地maven仓库和远程maven私服仓库


7、mvn versions:set -DnewVersion=xxxx  设置Maven的版本  
8、mvn dependency:tree  查看maven的依赖树（排查依赖很有效）
```



比如命令：

```xml
clean package -Dmaven.test.skip=true -P pro
```

执行`clean` ，跳过单元测试打包到target目录，并且选择 `pro`作为节点，这是Maven选择profile的功能，这里指定到pro环境：

> 即 `<id>pro</id>` 这里

```xml
<profiles>
        <!--开发环境 -->
        <profile>
            <id>dev</id>
            <properties>
                <!--自定义常量参数-->
                <spring.profiles.active>dev</spring.profiles.active>
            </properties>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
        </profile>
        <!--测试环境 -->
        <profile>
            <id>test</id>
            <properties>
                <spring.profiles.active>test</spring.profiles.active>
            </properties>
        </profile>
        <!--生产环境 -->
        <profile>
            <id>pro</id>
            <properties>
                <spring.profiles.active>pro</spring.profiles.active>
            </properties>
            <repositories>
                <repository>
                    <!--仓库id，repositories可以配置多个仓库，保证id不重复-->
                    <id>nexus</id>
                    <!--仓库地址，即nexus仓库组的地址-->
                    <url>http://localhost:8081/nexus/content/groups/public/</url>
                    <!--是否下载releases构件-->
                    <releases>
                        <enabled>true</enabled>
                    </releases>
                    <!--是否下载snapshots构件-->
                    <snapshots>
                        <enabled>true</enabled>
                    </snapshots>
                </repository>
            </repositories>
            <pluginRepositories>
                <!-- 插件仓库，maven的运行依赖插件，也需要从私服下载插件 -->
                <pluginRepository>
                    <!-- 插件仓库的id不允许重复，如果重复后边配置会覆盖前边 -->
                    <id>public</id>
                    <name>Public Repositories</name>
                    <url>http://localhost:8081/nexus/content/groups/public/</url>
                </pluginRepository>
            </pluginRepositories>
        </profile>
    </profiles>
```



所以个人本地不需要执行`mvn deploy`命令，只需要`install` 就会把jar生成到我们的本地的仓库地址，其他项目就可以引用了。



maven的仓库因为是国外镜像，可以配置阿里的镜像，这样下载jar就会很快：

> 配置参考：https://maven.aliyun.com/mvn/guide

打开 maven 的配置文件（ windows 机器一般在 maven 安装目录的 **conf/settings.xml** ），在`<mirrors></mirrors>`标签中添加 阿里云的 mirror 子节点:

```
<mirror>
  <id>aliyunmaven</id>
  <mirrorOf>*</mirrorOf>
  <name>阿里云公共仓库</name>
  <url>https://maven.aliyun.com/repository/public</url>
</mirror>
```
