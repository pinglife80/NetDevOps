# 关于Python3的一些异常

::date:2020年11月14日

##  1.关于使用pprint时报的错误

- 错误异常：

  ![image-20201114222559852](E:\NetDevOps笔记\关于Python3的一些异常.assets\image-20201114222559852.png)

  pprint(output)调用，出现如下异常：

  ![image-20201114222528668](E:\NetDevOps笔记\关于Python3的一些异常.assets\image-20201114222528668.png)

- 异常原因

  pprint模块引入方法错误，需要从 from pprint import pprint 。用前者的方式导入pprint模块后调用需要加上前缀及pprint.pprint() ,后者直接pprint()即可 。

- 解决办法①调整引入模块方式，为 from pprint import pprint （推荐），程序中直接调用② 在程序中使用pprint.pprint()的方式调用。

  PS:a:这里层级目录切回到上一层级的快捷键时 Shift+Tab 

  ---

  ##  2.关于linux环境下pip安装的包pycharm无法识别的问题

- 问题显现

  有时候pycharm安装某些外部包模块会失败，需要通过pip3 去安装，但是发现通过pip3安装的包在pycharm中不识别。 

- 问题原因

  pip3安装的包通常情况下是安装在这个路径下的：/usr/local/lib/python3.7/dist-packages

  而通过pycharm安装的包是在项目的venv的项目虚拟环境下的，它不会去添加pip3的包安装路径，所以是查找不出来的。

- 解决办法

  ①在pycharm的项目解释器环境中添加pip3的包安装路径②在构建项目虚拟环境的时候勾选继承全局包路径，如下所示：

  ![image-20201114232551874](E:\NetDevOps笔记\关于Python3的一些异常.assets\image-20201114232551874.png)

2中方法的效果都是一样的，都是将所有的包路径包含进来，如下面所示，添加方法和继承全局包路径都是为了将下面4、5两条路径包含进去，第3条才是pycharm自己创建的包路径。

![image-20201114232714332](E:\NetDevOps笔记\关于Python3的一些异常.assets\image-20201114232714332.png)

PS：:a:这里pip3 如果环境中没有，可以通过 apt install -y  python3-pip 直接安装（debian环境命令，Redhat 使用yum）

pip3 list  命令可以查看pip3安装的所有包，pip3 show 包名 可以查看包安装的具体路径。

---



