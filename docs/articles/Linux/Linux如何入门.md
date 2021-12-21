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



### 1.2、准备一台服务器

在学习Linux前，你需要准备一台Linux服务器，这里有两种方式：

- 使用vmware这种软件安装虚拟机
- 购买云服务器，比如阿里云、腾讯云

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



## 2、基础

需要掌握一些基本的Linux命令。

推荐菜鸟教程：

[https://www.runoob.com/linux/linux-install.html](https://www.runoob.com/linux/linux-install.html)

如果你是学生，时间充裕，我建议看一下《鸟哥的私房菜》，这本书还是很经典的：

![](https://cdn.jsdelivr.net/gh/DogerRain/image@main/img-202109/image-20211221154853360.png)



## 3、搭建服务

这部分是每个开发者必须要掌握的，常见如：

- Java环境
- Nginx
- Apache
- MySQL

搭建服务意味着你可以独立服务，作为一个Java开发工程师，开发、测试 环境一般都是自己搭建服务。

会搭建并不是最重要的，重要的是使用和配置，比如Nginx的配置、MySQL的配置等等。

## 4、进阶

- 会熟练使用 三剑客：awk、grep、sed
- 看得懂 top 、vmstart 结果，知道 磁盘、内存、CPU情况，知道Java程序出了问题如何定位、排查
- 熟练使用 vim，快捷键
- SSH客户端的快捷键
- 常见的第三方工具使用，比如阿里的 arthas、网卡监测 nload
- shell脚本编写
- Linux调优

## 5、其他

初学者在学习Linux的时候容易忘记Linux命令，只要多使用，总会熟能生巧。

在遇到比较复杂的命令时，可以记录到自己的笔记，方便查看。

醋酸菌本人就是通过 有道云笔记 记录一些零碎的知识点：

![](https://cdn.jsdelivr.net/gh/DogerRain/image@main/img-202109/image-20211221163738048.png)



