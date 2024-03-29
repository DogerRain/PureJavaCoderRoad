> https://www.zhihu.com/question/451255512

说来也是很搞笑😅....

我自己的的经历就是**从来没真正系统的学过Linux**。

记得第一次安装虚拟机的时候，电脑装的是Ubuntu系统，第一次发现与那里还有这种系统，还有命令行；

第二次再操作，发现这又是个Centos的系统，我还在寻思着怎么没有界面呢？（当然这也是后知后觉了~）

我没有完整地看过任何Linux相关的书籍，那我是怎么熬过来的呢？

答案就是 **边用边学**。

> 我的建议是 边用边学 ，而不是边学边用，后者容易忘记。

我仅列出作为一名Java程序员，在求职、工作的时候，需要掌握哪些Linux知识。



## 1、入门

为什么要学习Linux呢？

因为大部分服务都是部署在Linux服务器，因为Linux系统具有天然的优势。

我们在windows上可以右击新建文件夹，而在linux上就需要通过 mkdir 的命令。

###  1.1、操作系统概念

和Linux打交道是因为操作系统这门课程，可以说相辅相成，可以在学**操作系统**的时候把Linux的知识点也掌握了。

如果你已经学了操作系统，那么再学习Linux就会事半功倍。

你需要简单的知道一些概念：

- 进程、线程
- 端口、防火墙、网卡
- 公网、局域网

### 1.2、操作系统的分类

常见的Linux系统版本：

| 版本名称     | 网 址                                    | 特 点                                                        | 软件包管理器                                        |
| ------------ | ---------------------------------------- | ------------------------------------------------------------ | --------------------------------------------------- |
| Debian Linux | [www.debian.org](http://www.debian.org/) | 开放的开发模式，且易于进行软件包升级                         | apt                                                 |
| Fedora Core  | [www.redhat.com](http://www.redhat.com/) | 优秀带桌面环境的系统，拥有数量庞人的用户，优秀的社区技术支持. 并且有许多创新 | up2date（rpm），yum （rpm）                         |
| CentOS       | [www.centos.org](http://www.centos.org/) | CentOS 是一种对 RHEL（Red Hat Enterprise Linux）源代码再编译的产物，由于 Linux 是开发源代码的操作系统，并不排斥样基于源代码的再分发，CentOS 就是将商业的 Linux 操作系统 RHEL 进行源代码再编译后分发，并在 RHEL 的基础上修正了不少已知的漏洞 | rpm                                                 |
| SUSE Linux   | [www.suse.com](http://www.suse.com/)     | 专业的操作系统，易用的 YaST 软件包管理系统                   | YaST（rpm），第三方 apt （rpm）软件库（repository） |
| Ubuntu       | [www.ubuntu.com](http://www.ubuntu.com/) | 优秀带桌面环境的系统，基于 Debian 构建，对新款硬件具有极强的兼容能力。 | apt                                                 |



目前市面上用的比较多的是CentOS，个人如果要学习用，建议用Ubuntu，因为是可视化的桌面，操作方便。

### 1.3、Shell

#### shell是什么

不同的系统命令有所差异，但是不会很大，取决于它们用的是什么Shell。

`Shell` 这个单词的原意是“外壳”，跟 `kernel`（内核）相对应，比喻内核外面的一层，**即用户跟内核交互的对话界面。**

> 简单的说，shell是一个程序，提供一个环境，这个环境只有一个黑框，用户从键盘输入命令，又称为命令行环境（ `command line interface` ，简写为 `CLI` ）
>
> `Shell` 收到命令后，发送给操作系统执行，并把结果返回。
>
> 同时`Shell`也提供了很多小工具，比如 vim、top、ll 等等

#### shell的分类

`Shell` 有很多种，不同的系统使用不同的`Shell` 

主要的 `Shell` 有下面这些：

- Bourne Shell（sh）
- Debian Almquist Shell （dash)；Debian、Ubuntu 默认使用
- Bourne Again shell（bash）；Centos、Fedora  默认使用
- C Shell（csh）
- TENEX C Shell（tcsh）
- Korn shell（ksh）
- Z Shell（zsh）
- Friendly Interactive Shell（fish）

可以查看一下自己的Linux上支持哪些 shell：

```bash
[root@VM-8-8-centos ~]# echo $SHELL
/bin/bash
[root@VM-8-8-centos ~]# cat /etc/shells
/bin/sh
/bin/bash
/usr/bin/sh
/usr/bin/bash
/bin/tcsh
/bin/csh
```

查看用的哪一种 shell：

```bash
root@VM-8-8-centos ~]# ls -l /bin/sh
lrwxrwxrwx 1 root root 4 Aug  7  2020 /bin/sh -> bash
```

不同的shell区别不会很大，只是个别语法区别，在于写shell脚本。

所以在编写 `.sh`  的shell 脚本的时候，必须在头部告知Linux使用哪一种 shell 脚本执行：

```shell
#!/bin/bash
echo "------Hello,Coder--------"
```



### 1.4、准备一台服务器

在学习Linux前，你需要准备一台Linux服务器，这里有两种方式：

- 使用vmware这种软件安装虚拟机
- 购买云服务器，比如阿里云、腾讯云

使用虚拟机的方式对电脑要求颇高，新手安装颇为复杂，可自行百度。



我推荐第二种方式，学生或者新用户一个月只需要10块钱，就能拥有一台自己的服务器了，后续还可以搭建自己的个人网站。

推荐一些云服务器：


| 云厂商 | 学生优惠                                                     | 新人优惠                                                     |
| ------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 腾讯云 | [学生特惠，1核2G5M宽带，仅需9元/1个月](https://cloud.tencent.com/act/cps/redirect?redirect=10004&cps_key=664b44b4e8e43b579d07036bf1c71060) | [星星海SA2云服务器，1核2G首年99元](https://cloud.tencent.com/act/cps/redirect?redirect=1063&cps_key=664b44b4e8e43b579d07036bf1c71060&from=console)（我目前用的） |
|        |                                                              | [新客户无门槛代金券，价值高达2860元代金券](https://cloud.tencent.com/act/cps/redirect?redirect=1040&cps_key=664b44b4e8e43b579d07036bf1c71060&from=console) |
|        |                                                              | [云产品限时秒杀，爆款1核2G云服务器，首年99元](https://cloud.tencent.com/act/cps/redirect?redirect=1062&cps_key=664b44b4e8e43b579d07036bf1c71060&from=console) |
| 阿里云 |                                                              | [精选云服务器1核2G 新人仅需87元/年](https://www.aliyun.com/minisite/goods?userCode=4lol8et7) |
| 百度云 | [1核2G 学生身份 9 元/1个月](https://cloud.baidu.com/campaign/campus-2018/index.html?unifrom=eventpage) |                                                              |
| 华为云 |                                                              | [精选云服务器2折起](https://activity.huaweicloud.com/cps/recommendstore.html?fromacct=0740541e-dec2-47db-99e9-b5bb524ccbf7&utm_source=aGlkX2txbGYyNDR0ZXlxc2ZwZg===&utm_medium=cps&utm_campaign=201905) |
| 青云   |                                                              | [https://www.qingcloud.com](https://www.qingcloud.com)       |



### 1.5、准备一个ssh工具

我们可以通过一些 ssh终端工具连接我们的Linux系统，这类工具具有较多功能，可以很方便的操作 Linux，比如 Xshell、SecureCRT、FinalShell等等。



当你一切就绪，连上你心心念念的服务器的时候，就会出现伴随着你终身难忘的语句：

```bash
[root@VM-8-8-centos ~]#
```

命令解析：

- `root`：表示用户名；
- `VM-8-8-centos`：表示主机名；
- `~`：表示目前所在目录为家目录，其中 `root` 用户的家目录是 `/root` 普通用户的家目录在 `/home` 下；
- `#`：指示你所具有的权限（ `root` 用户为 `#` ，普通用户为 `$` ）。

然后你就可以输入 Linux命令和它进行交互了。

## 2、基础

### 命令

作为一个程序员，必须要掌握一些基本的Linux命令。

![](https://cdn.jsdelivr.net/gh/DogerRain/image@main/img-202204/image-20210104215503268.png)

推荐菜鸟教程：[https://www.runoob.com/linux/linux-install.html](https://www.runoob.com/linux/linux-install.html)

### 书本

如果你是学生，时间充裕，我建议看一下《鸟哥的私房菜》：

![](https://cdn.jsdelivr.net/gh/DogerRain/image@main/img-202109/image-20211221154853360.png)

这本书还是很经典的，介绍十分详细，由于内容太多，但完全可以根据自己的方向挑选进行学习。

在线阅读：http://cn.linux.vbird.org

### 快捷键

在开始学习 `Linux` 命令之前，有这么一些快捷方式，是必须要提前掌握的，它将贯穿整个 `Linux` 使用生涯。

- 通过上下方向键 ↑ ↓ 来调取过往执行过的 `Linux` 命令；
- 命令或参数仅需输入前几位就可以用 `Tab` 键补全；
- `Ctrl + R` ：用于查找使用过的命令（`history` 命令用于列出之前使用过的所有命令，然后输入 `!` 命令加上编号( `!2` )就可以直接执行该历史命令）；
- `Ctrl + L`：清除屏幕并将当前行移到页面顶部；
- `Ctrl + C`：中止当前正在执行的命令；
- `Ctrl + U`：从光标位置剪切到行首；
- `Ctrl + K`：从光标位置剪切到行尾；
- `Ctrl + W`：剪切光标左侧的一个单词；
- `Ctrl + Y`：粘贴 `Ctrl + U | K | Y` 剪切的命令；
- `Ctrl + A`：光标跳到命令行的开头；
- `Ctrl + E`：光标跳到命令行的结尾；
- `Ctrl + D`：关闭 `Shell` 会话；

 

