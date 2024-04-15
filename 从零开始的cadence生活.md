Cadence学习笔记之Ubuntu20.04双系统安装IC617，Calibre2019及常见问题一览

# 引言

这是一系列cadence的学习笔记，帮助初学者快速适应linux系统及避坑指南，同时后续会有一些数字仿真的内容持续更新

本篇博客建立在安装完ubuntu双系统的基础上，对于安装ubuntu双系统，网上的教程很多，这里不再做过多的赘述。
你可以访问如下链接来获取安装包：

```
链接：https://pan.baidu.com/s/1bEnmiZ57b_5nDpxlrfUILg?pwd=1234 
提取码：1234 
```

在安装完ubuntu系统后的操作如下：

# 步骤一：安装文件
（由于国内的源有些时候不太稳定，因此下列的包能获取安装多少获取多少）

## 1、安装依赖
使用快捷键 CTRL+ALT+T打开新的terminal，或者也可以直接在收藏栏中点击终端运行

sudo apt-get install命令常被用做从镜像源获取并安装库文件。

```
sudo apt-get install ksh
sudo apt-get install csh
```
如果不装或版本未更新的话运行virtuoso会报如下错误：

```
./virtuoso[114]: /cadence/IC617/bin/cds_plat: not found [No such file or directory]
./virtuoso[123]: /cadence/IC617/tools/bin/cds_plat: not found [No such file or directory]
virtuoso: ERROR: Cannot find a proper cds_plat in the hierarchy.
virtuoso: ERROR: Cannot identify the current platform.
virtuoso:          Check your installation.
```
若后面出现此错如可以重新装这两个包。
```
sudo apt-get install openjdk-8-jre openjdk-8-jdk
sudo apt-get install xterm
sudo apt-get install libncursesw5-dev
sudo apt-get install libxtst6:i386
sudo apt-get install libxi6:i386
```
安装license时，需要运行python文件获取license.dat文件： 
```
sudo apt-get install python
sudo apt install net-tools
```
由于现在大多数的电脑都是64位的，因此需要安装32位的library库来确保cadence的运行：
```
sudo apt-get install libstdc++6
sudo apt-get install lib32stdc++6
```
接着我们安装multiarch-support库，这个库可以直接通过deb文件直接安装：

我们可以访问如下的网址下载并安装：

> https://launchpad.net/ubuntu/bionic/+package/multiarch-support

![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/8193c486dfdc490da65f975b7348b772.png#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/f352681b30e6494582c7819076486efb.png#pic_center)
可以使用 sudo dpkg -i + 包名的形式来安装下载下来的deb包

接着安装libxp6库，我们可以访问如下网址下载并安装

>  http://launchpadlibrarian.net/183708483/libxp6_1.0.2-2_amd64.deb

下载完后依旧使用sudo dpkg -i + 包名的形式来安装下载下来的deb包
如果报错：dpkg: 错误: 无法访问归档 ’ *.deb ': 没有那个文件或目录的话，ubuntu网页下载默认在/home/##/download文件夹下，那我们只需要使用cd命令到download文件夹或在download文件夹中打开终端再次输入dpkg命令即可。

当然我们也可以使用sudo wget + 对应包网址的形式来获取我们所需要的deb文件

接着我们需要查看环境中java的安装情况，在终端中输入：

```
java -version
```
通过gedit命令来编辑一般的文本类文档：

```
sudo gedit /etc/profile
```
将下列的环境变量复制到文件的最后：

```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export JRE_HOME=$JAVA_HOME/jre
export CLASSPATH=$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
```
CTRL+S保存后让这个文件找到新增路径并使之生效，在终端中输入：

```
source /etc/profile
```
接着我们需要添加redhat-release：

```
sudo gedit /etc/redhat-release
```
在打开的文件中添加：

```
Red Hat Enterprise Linux release 6.12
```
保存并更改文件权限为读写：

```
sudo chmod 777 /etc/redhat-release
```
我们可以通过如下命令创建我们即将要安装cadence和calibre的文件夹：

```
sudo mkdir -p /opt/cadence/
sudo mkdir -p /opt/mentor/Calibre2019
sudo mkdir -p /opt/mentor/license
```
opt文件夹创建或删除文件夹是需要sudo权限的，因此我们也可以通过以下命令实现文件的可视化移动，删除及创建：

```
sudo nautilus
```
这样就可以打开一个无权限限制的窗口了。
接着我们需要创建一个临时文件夹并更改文件的权限：

```
sudo mkdir /usr/tmp
sudo chmod -R 777 /usr/tmp/
```
建立软链接：

```
sudo ln -s /usr/bin/mawk /bin/awk
sudo ln -s /usr/bin/basename /bin/basename
sudo ln -s /lib/x86_64-linux-gnu/libncursesw.so.5.9 /lib/libtermcap.so.2
cd /lib/x86_64-linux-gnu/
sudo ln -s libcrypto.so.1.1 libcrypto.so.6
```
这里如果提示你无法创建因为已经有了那可以跳过。
## 2、安装IC617，MMSIM15.1
下载的IScape文件使用sudo nautilus指令移动到opt文件下的cadence文件夹下，或使用sudo cp 需要移动的文件名 + 移动路径进行移动。
启动IScape，这是一个引导类的软件，使用完如果双系统分配的空间不足可以删去，使用sudo rm +文件名或者使用sudo rm -rf +文件夹名可删除：
新建terminal窗口输入如下命令：

```
cd /opt/cadence/IScape  #进入解压后软件包所放的目录
sudo chmod -R 777 /opt/cadence/IScape  #更改cadence文件夹的写入权限
sudo zcat IScape04.23-s010lnx86.t.Z | sudo tar -xvf -  #解压IScape04.23
cd /opt/cadence/IScape/iscape/bin  #进入iscape下bin文件夹
sudo ./iscape.sh	#启动IScape安装界面
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/ace2c1d4288145f59c3175100d9be2da.png#pic_center)
这里点击Preferences - Istallscape设置运行空间和文件的安装路径
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/008b62a632ac43a5ad10937f7b1ad30f.png#pic_center)
进入Directories目录
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/bbf68bf45b3b4871b40d73d91a520949.png#pic_center)
第一个目录是运行空间，使用默认的路径
第二个是install文件夹安装的路径，将这里改为刚刚建的cadence文件夹：opt/cadence
都三个是downloads文件夹的安装路径，同样将这里改为：opt/cadence
点击ok确定后cancel
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/0da1ab1e09de421ba992161b2542e9c5.png#pic_center)
点击local directory/Media install 从本地安装包安装此文件。
首先是下载并解压完的IC06.17.700_Base文件
全部勾选，点击next全选安装，
接着就是安装时的一些确认了，有读秒的按ENTER
出现Synergy users must install these libraries窗口时全部输入y
出现Prepare libraries for AMS Designer窗口时输入2
出现Welcome to OA configuration Utility窗口时输入quit随后输入n
安装MMSIM的步骤同上
安装完后的界面如下所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/138d494c4eb140d6ad6d392832030fa4.png)
## 3、安装calibre
在安装calibre的过程中遇到很多问题，这里也详细展开说一说。
本文安装的为calibre2019，我们需要将下载的calibre2019包进行解压，解压后i面有且仅有一个aoj_cal_2019.3_15.11_mib.exe文件，我们将这个文件复制到/opt/mentor/Calibre2019文件夹下，同时在该文件夹下右击，打开终端，在该路径下输入命令：

```
sudo ./aoi_cal_2015.2_36.27_mib.exe
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/b05e22e8e91843789c7b3fbfe0bfb12d.png)
输入D ENTER
输入yes
这时如果没有出现安装进度条并报错仅需要再多输入三至五次即可，出现进度条并出现Successfully字样即表示安装成功。

# 步骤二：安装license文件
因为cadence和calibre属于两家不同的公司，因此其license文件也需要安装两次
首先我们解压下载下的patch文件夹，进入下载文件夹，右击，打开终端，输入以下命令：
## 1、安装IC617的license文件

```
python cdslicgen.py 	#生成license
sudo cp license.dat /opt/cadence/IC617/share/license/ 		#拷贝到license文件夹
sudo chmod 777 /opt/cadence/IC617/share/license/license.dat 	#设置为可读写
```
这时候如果不放心可以去对应的路径下看是否生成一个新的license.dat文件。

## 2、安装calibre的license文件

首先需要查询主机的硬件地址，打开终端输入：

```
ifconfig
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/67a932e47e964ecfa8825cfb44a43de5.png#pic_center)
图中框出的六位地址码即为硬件地址，我们需要在patch文件夹下运行终端并输入命令：

```
python mgclicgen.py e8:f4:08:c2:5b:dc	#将这一串替换成你的地址
sudo cp license.dat /opt/mentor/license/		#复制到mentor文件下的文件目录
sudo chmod 777 /opt/mentor/license/license.dat	#添加权限
```
# 步骤三：设置.bashrc文件
.bashrc文件是ubuntu提供的个性化可编辑执行文件，因此改动是没有太大影响的，在home目录中找到汉堡图标，点击，选择show hidden files，找到.bashrc文件进入编辑，同时我们点开我配置好的.bashrc文件。这里推荐一款工具，winmerge，可以通过建立两路对比来比较任意两个文本文件的不同，或者我们将图示代码以下的内容全部复制：
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/08cf4a789edc4478933228338d2348a1.png#pic_center)
这里有两个地方需要稍作修改：
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/81855434094e4741b55f408f6c914fb1.png#pic_center)
首先是这个地方，你需要将这些变量的声明改为你安装的路径，分别指向aoj_cal_2019.3.15.11文件夹及其下面的bin和lib文件夹，以及calibre的license.dat文件，这个后面在修改.cdsinit文件时也有作用。
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/ef58465cabcb455493139862bed60eb5.png#pic_center)
接着就是这里，我们需要将图中的homehost改为你自己的homehost名，这个很重要，不记得的可以打开终端，输入命令：

```
hostname
```
来查询自己的主机名，如果输入有误，那后续打开virtuoso时会报找不到端口的错误。
同时这里注意如果你安装了ros或者其他的ubuntu环境，建议将你的环境变量配置到.bashrc的最后，否则你可能无法使用其他的环境，如ros我需要将source opt/.../ros指令移动至文件最后。
接着我们需要将calibre集成到virtuoso界面内，我们首先要找到calibre的可执行文件其地址为：

```
/opt/cadence/IC617/tools.lnx86/dfll/cdsuser/.cdsinit
```
这里要注意区分.cdsinit文件和cdsinit文件，一个是C源代码文件，一个是文本文件。
将该文件复制到home目录下
打开该文件，在文件的最后添上如下的路径：

```
loadi(strcat(getShellEnvVar("CALIBRE_HOME") "/shared/pkgs/icv/tools/queryskl/calibre.skl"));
```
这里的CALIBRE_HOME就是.bashrc文件中所声明的路径变量的名称。
安装完成后，我们需要将IC617重新软链接到libstdc++.so.6
打开终端输入以下命令：

```
cd /opt/cadence/IC617/tools/lib/64bit
sudo rm libstdc++.so.6
sudo ln -s /lib/i386-linux-gnu/libstdc++.so.6
```

# 步骤四：解决sysname的linux内核版本问题
由于IC617能找到的资源几乎都是在相对较老的系统上运行的，因此为了适配新的Linux内核，我们还需要修改linux内核的配置文件。
首先我们需要知道自己的linux内核是几代版本,打开终端，输入命令：

```
uname -r
```
即可查询到自己的linux内核版本，如我这里返回到数据就是：

```
5.13.0-35-generic
```
接着我们去到：

```
/opt/cadence/IC617/share/oa/bin/sysname
```
这是一个sh脚本，我们点击进去编辑，若权限不够的话也可直接使用sudo gedit
命令。

```c
check_linux() {
    sysnames=$sysname

    version=`uname -r`
    machine=`uname -m`

    if [ -f "/etc/redhat-release" ]
    then
      longVersion=`cat /etc/redhat-release`
    elif [ -f "/etc/SuSE-release" ]
    then
      longVersion=`cat /etc/SuSE-release`
    elif [ -f "/etc/os-release" ]
    then
      longVersion=`grep PRETTY_NAME /etc/os-release | sed -e 's/.*"\(.*\)"/\1/'`
    else
      longVersion="UNKNOWN Linux"
    fi

    case $machine in
	ia64 )
          sysname="linux_rhas21_ia64$compiler"; sysnames="$sysname $sysnames";;
	*86 | *86_64 )	
	    case $version in
	        2.4.* )
                  # RHEL 2, RHEL 3
                  compiler="_gcc411"
		  sysname="linux_rhel30$compiler"; sysnames="$sysname $sysnames";;
	        2.6.[0-9]-* )
                  # RHEL 4, SLES 9
                  compiler="_gcc44x"
                  sysname="linux_rhel40$compiler"; sysnames="$sysname $sysnames";;
	        2.6.*)
                  # RHEL 5, RHEL 6, SLES 10, SLES 11, SLES 11 SP1
                  if [ "$OA_COMPILER" = "" ] ; then
                      compiler="_gcc48x";
                  fi
                  sysname="linux_rhel50$compiler"; sysnames="$sysname $sysnames";;
	        3.*)
                  # RHEL 7, SLES 11 SP2, SLES 12, Ubuntu 14
                  if [ "$OA_COMPILER" = "" ] ; then
                      compiler="_gcc48x";
                  fi
                  sysname="linux_rhel50$compiler"; sysnames="$sysname $sysnames";;
	        * )
                  check_global;;
	    esac;;
	*)
          check_global;;
    esac

}

```
打开后是这样一段代码，很显然没有我们需要的linux内核，我们仅需添加我们自己的Linux内核即可：

```c
check_linux() {
    sysnames=$sysname

    version=`uname -r`
    machine=`uname -m`

    if [ -f "/etc/redhat-release" ]
    then
      longVersion=`cat /etc/redhat-release`
    elif [ -f "/etc/SuSE-release" ]
    then
      longVersion=`cat /etc/SuSE-release`
    elif [ -f "/etc/os-release" ]
    then
      longVersion=`grep PRETTY_NAME /etc/os-release | sed -e 's/.*"\(.*\)"/\1/'`
    else
      longVersion="UNKNOWN Linux"
    fi

    case $machine in
	ia64 )
          sysname="linux_rhas21_ia64$compiler"; sysnames="$sysname $sysnames";;
	*86 | *86_64 )	
	    case $version in
	        2.4.* )
                  # RHEL 2, RHEL 3
                  compiler="_gcc411"
		  sysname="linux_rhel30$compiler"; sysnames="$sysname $sysnames";;
	        2.6.[0-9]-* )
                  # RHEL 4, SLES 9
                  compiler="_gcc44x"
                  sysname="linux_rhel40$compiler"; sysnames="$sysname $sysnames";;
	        2.6.*)
                  # RHEL 5, RHEL 6, SLES 10, SLES 11, SLES 11 SP1
                  if [ "$OA_COMPILER" = "" ] ; then
                      compiler="_gcc48x";
                  fi
                  sysname="linux_rhel50$compiler"; sysnames="$sysname $sysnames";;
	        3.*)
                  # RHEL 7, SLES 11 SP2, SLES 12, Ubuntu 14
                  if [ "$OA_COMPILER" = "" ] ; then
                      compiler="_gcc48x";
                  fi
                  sysname="linux_rhel50$compiler"; sysnames="$sysname $sysnames";;
            5.*)
                  if [ "$OA_COMPILER" = "" ] ; then
                      compiler="_gcc48x";
                  fi
                  sysname="linux_rhel50$compiler"; sysnames="$sysname $sysnames";;
	        * )
                  check_global;;
	    esac;;
	*)
          check_global;;
    esac

}

```
点击保存，新建终端，在其中输入如下命令：

```
$ ln -s  /cadence/IC617/share/oa/lib/linux_rhel50_gcc48x_64 unkown_64
```
到这里linux内核版本就大致设好了。
# 步骤五：测试
接着就是测试安装是否成功了，首先我们先确保calibre的安装，打开终端，输入以下命令：

```
calibre -gui
```
如果出现了calibre的小窗口即为安装成功，如果你的ubuntu之前安装过一款同样名为calibre的e-book阅读器那这串指令是没有用的，建议将e-book卸载重新进行测试。
接着就是virtuoso，打开终端输入：

```
virtuoso
```
打开小窗出现连接virtuoso server successful即表示安装成功，随意打开一个版图，在最顶端的window和help之间有calibre选项，至此calibre已成功集成到cadnece的菜单栏，至此安装成功。
