# Widora How-To

`by zymxjtu`

## 用Eclipse开发OpenWrt C/C++程序 交叉编译 远程调试

## 1 版权
这个 How-To 指引是基于 J.Kohler 的 OpenWrt C/C++ Devopement with Eclipse 的工作成果。 更新和增添了一些内容，以及Widora相关和独有的一些内容。

This work is licensed under the Creative Commons Attribution-ShareAlike 3.0 Unported License. To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/3.0/ or send a letter to Creative Commons, 444 Castro Street, Suite 900, Mountain View, California, 94041, USA.

## 2 版本历史
| 版本 | 日期 | 作者 | 更改记录 |
|--------|--------|--------|--------|
|    v0.1    |    6 Aug 2016    |   zymxjtu    |    第一版    |

## 3 介绍
使用Eclipse IDE, 我们可以有一个很方便和舒适的为OpenWrt目标设备做开发的开发环境， Eclipse提供了一套完整的开发套件。

使用交叉编译，我们可以不必受制于OpenWrt设备的资源限制；使用远程调试，可以很方便的设置断点，调试程序。

这篇文档解释了怎样使用Eclipse C/C++ IDE与OpenWrt的交叉编译工具链；怎样设置Eclipse做远程目标设备代码级别的调试和远程操控。

这里展示了怎样为OpenWrt目标设备编写代码，编译，还有调试程序，我们可以使用Eclipse为OpenWrt目标设备开发软件。

## 4 准备工作

### 4.1 准备工作

#### 4.1.1 目标设备准备工作

下面列出了目标设备需要的软件包:
1. 安装有DropBear 或者 OpenSSH，可以用SSH连接到设备
2. openssh-sftp-server
3. gdbserver
4. libstdcpp (可选，做C++开发需要)

openssh-sftp-server 和 gdbserver 都可以提前编译到OpenWrt的固件里，当然如果没有预装，我们可以简单的通过下面方法安装:
1. SSH 到 OpenWRT. (对于Widora，我们也可以通过板载串口终端，详细信息看使用指南`"Widora_用户指南_1_开始之前"`)
2. 运行 `opkg update` 然后 `opkg install libstdcpp`
3. 运行 `opkg update` 然后 `opkg install openssh-sftp-server`
4. 运行 `opkg install gdbserver` 安装 gdbserver。

  ![1.1.000](picture/000.001.png)

  ![1.1.000](picture/000.000.png)

对于Widora，如果Widora是连去一个路由器，然后开发电脑也是连接到同一个路由器，而不是直接连接到Widora，那么想要SSH访问Widora，我们需要更改Widora的防火墙设置。更改Widora防火墙的配置文件`/etc/config/firewall`，更改 config zone wan, option input 设置为 `ACCEPT`, 而不是默认的 `REJECT`, 保存改动之后，重启Widora:

默认:

  ![1.1.000](picture/000.002.png)

更改之后:

  ![1.1.000](picture/000.003.0.png)

然后远程SSH测试下可不可以SSH连到Widora （这里路由器为Widora分配的地址是192.168.8.180，你的环境分配的IP地址可能不同）：

  ![1.1.000](picture/000.003.1.png)


#### 4.1.2 OpenWRT 准备工作

准备 OpenWrt Buildroot 开发环境:

遵照“Widora-NEO开源硬件用户手册V04.pdf”。

以下连接供参考：
https://github.com/widora/openwrt_widora

http://wiki.openwrt.org/doc/howto/buildroot.exigence

对于Widora使用的OpenWrt Chaos Calmer版本，gdb存在一个问题，详细的信息可以参考下面链接：

https://dev.openwrt.org/ticket/22360

https://dev.openwrt.org/changeset/46298/trunk/toolchain/gdb

这个问题之后在我们调试程序的时候可能会导致如下问题:

  ![1.1.000](picture/000.005.png)

所以对于Widora，这里需要对源码的一个地方做一点改动。
 `{Widora源码目录}/toolchain/gdb/Makefile` (如下图所示加入一行 "--with-expat \\"):

  ![1.1.000](picture/000.004.png)

之后我们还需要安装依赖库"libexpat1-dev":
```
sudo apt-get install libexpat1-dev
```

在Terminal里面cd到Widora源码的根目录，然后执行 “make menuconfig”:

  ![1.1.000](picture/000.006.png)

然后:
1. 设置Widora的 "Target System", "Subtarget", "Target Profile".
2. 选择 [ * ] Build the OpenWrt SDK
3. 选择 [ * ] Package the OpenWrt-based Toolchain
4. 选择 [ \* ] Advanced configuration options (for developers) ---> Enable [ \* ] Toolchain Options ---> Enable [ \* ] Build gdb
5. 保存设置.

  ![1.1.000](picture/000.007.png)

  ![1.1.000](picture/000.008.png)

  ![1.1.000](picture/000.009.png)

  ![1.1.000](picture/000.010.png)

  ![1.1.000](picture/000.011.png)

  ![1.1.000](picture/000.012.png)

  ![1.1.000](picture/000.013.png)

执行 “make toolchain/install” . 这一步的目的是准备我们之后需要用到的交叉编译工具链还有程序调试所需要的gdb.

  ![1.1.000](picture/000.014.png)

  ![1.1.000](picture/000.015.png)


#### 4.1.3 Eclipse 准备工作

下载你喜欢的 Eclipse IDE for C/C++ Developers 版本.

对于这篇指引，我们用的是 Eclipse IDE for C/C++ Developers Eclipse Luna SR2 (4.4.2) Linux.

http://www.eclipse.org/downloads/packages/release/luna/sr2

选择你偏爱的版本和平台 (32位 / 64 位).

解压缩，把Eclipse放到你偏爱的目录，执行Eclipse，选择你偏爱的workspace位置。**注意：Eclipse运行需要Java Runtime**。

  ![1.1.000](picture/000.016.png)

  ![1.1.000](picture/000.017.png)

  ![1.1.000](picture/000.018.png)

  ![1.1.000](picture/000.019.png)

我们可能需要安装额外的eclipse软件包：`Help → Install new Software → All Available Sites`. 选择 **“Mobile and Device Development”** 下的 **“C/C++ GCC Cross Compiler Support”** 还有 **“Remote System Explorer End-User Runtime”**.

如果你使用的是 Eclipse IDE for C/C++ Developers, 很可能这两个软件包已经默认安装了。 如果你像查看这两个软件包，去掉点选 **“Hide items that are already installed”**.

  ![1.1.000](picture/000.020.png)

  ![1.1.000](picture/000.021.png)


## 5 工程配置

### 5.1 Eclipse 交叉编译工程的配置

#### 5.1.1 配置 eclipse

新建一个工程: `File → New C++ Project (resp. C project)`:

  ![1.1.000](picture/000.022.png)

  ![1.1.000](picture/000.023.png)

下面的设置是和你的目标设备还有你的OpenWrt开发环境相关的，你需要查看你开发环境独特的设置。

这里我们需要设置交叉工具链的路径还有prefix。

进入Widora源码的根目录，执行 `find ./staging_dir -path "./staging_dir/toolchain*" -name *openwrt-linux`

  ![1.1.000](picture/000.024.png)

当撰写这篇指引时用的开发环境返回的结果如下：

`./staging_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/mipsel-openwrt-linux`

因此 “Cross compiler prefix” 我们填写 `mipsel-openwrt-linux-`.

然后需要注意的是 “Cross compiler path” 需要填写的是**绝对路径**:

`[你Widora的源码目录]/staging_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/`

**不要忘了你的开发环境应该会和写这篇指引所用的环境不同，不能闭着眼睛复制黏贴!**

填入你的设置然后Finish按钮结束。

  ![1.1.000](picture/000.025.png)

#### 5.1.2 Eclipse 工程设置

在这里，我们会创建一个简单的Hello World程序做示范，展示如何设置eclipse对远程目标设备进行代码级别的调试和远程访问。

为源码创建一个src文件夹。

`File → New Source Folder (src)`

`File → New Source File (src/HelloWidora.cpp)`

```
#include <iostream>
using namespace std;
int main()
{
	int i = 0;
	for (i=0; i<10; i++)
	{
		cout << "Hello OpenWrt" << endl;
	}
	return 0;
}
```

  ![1.1.000](picture/000.026.png)

如果一切配置正确的话，现在你应该可以交叉编译代码了 `Project-Build all` 。

生成的程序肯定是不能在开发电脑上直接运行的，因为我们是用交叉工具链编译的可以跑在Widora上的MIPS架构的二进制可执行文件。

  ![1.1.000](picture/000.027.png)

##### 5.1.2.1 远程目标设备的配置（Widora）

如果你的程序编译成功，你需要在你的目标设备上执行你所写的程序，这个时候，eclipse提供的远程访问和远程调试功能就非常有用：

选择 `Window → Show View → Other… → Remote Systems`

  ![1.1.000](picture/000.028.png)

创建一个新连接:

  ![1.1.000](picture/000.029.png)

选择 Linux 然后下一步 “Next >”:

  ![1.1.000](picture/000.030.png)

现在填写目标设备的IP地址，然后下一步 “Next >”.

  ![1.1.000](picture/000.031.png)

选择 “ssh.files” 然后下一步 “Next >”.

  ![1.1.000](picture/000.032.png)

选择 “processes.shell.linux” 然后下一步 “Next >”.

  ![1.1.000](picture/000.033.png)

选择 “ssh.shells” 然后下一步 “Next >”. 然后 “Finish”.

  ![1.1.000](picture/000.034.png)

  ![1.1.000](picture/000.035.png)

你的 Eclipse IDE 看来是这个样子:

  ![1.1.000](picture/000.036.png)

##### 5.1.2.2 浏览你的目标设备

  ![1.1.000](picture/000.037.0.png)

输入你目标设备的用户名和密码:

  ![1.1.000](picture/000.037.png)

第一次, Eclipse 会让你设置安全存储的主密码, 设置你的主密码然后继续:

  ![1.1.000](picture/000.038.png)

  ![1.1.000](picture/000.039.png)

连接成功之后，你可以很方便的把文件拖曳到你的目标设备上(drag n' drop)，你甚至可以控制目标设备的进程(processes)，比如你可以kill一个目标设备上的进程。

  ![1.1.000](picture/000.040.png)

你也可以在eclipse里面打开一个SSH Terminal，拷贝执行文件：

  ![1.1.000](picture/000.041.png)

  ![1.1.000](picture/000.042.png)

##### 5.1.2.3 远程调试gdb Debugger的配置

要想做远程调试，首先我们需要创建一个调试配置:

左键点击“bug”图标旁边的箭头，选择“Debug Configurations”。

  ![1.1.000](picture/000.043.png)

然后创建一个新的 C/C++ Remote Application Debug Configuration:

  ![1.1.000](picture/000.044.png)

- 在 “Main” 选项页.
- 对于 “Connection”，选择之前创建的连接 (看“远程目标设备的配置（Widora）”).
- 不要忘了“Remote Absolute File Path for C/C++ Application”，这里设置的是你想要把程序放到Widora上的绝对路径。

  ![1.1.000](picture/000.045.png)

然后切换到 “Debugger” 选项卡设置开发主机的gdb文件.

在开发主机上，我们不能直接用默认的/usr/bin/gdb （Ubuntu默认），我们需要指向我们之前编译的gdb，还有工具命令的前缀也需要设置正确。

在Widora源码根目录执行: `find ./build_dir -executable -type f -name gdb |grep toolchain`。

当撰写这篇指引时用的开发环境返回的结果如下：
`./build_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/gdb-linaro-7.6-2013.05/gdb/gdb`

  ![1.1.000](picture/000.046.png)

在 “GDB debugger”, 我们需要填写的是**绝对路径**，对于写这篇指引时用到的开发环境是:

`/home/dev101/Desktop/Prj/openwrt_widora/build_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/gdb-linaro-7.6-2013.05/gdb/gdb`

**不要忘了你的开发环境应该会和写这篇指引所用的环境不同，不能闭着眼睛复制黏贴!**

其他的设置不需要改动，现在你可以按“Debug”按钮开始调试。

  ![1.1.000](picture/000.047.png)

##### 5.1.2.4 远程调试示例

  ![1.1.000](picture/000.048.png)

当你选择开始我们之前配置好的远程调试之后，eclipse的调试视图会打开：

  ![1.1.000](picture/000.049.png)

  ![1.1.000](picture/000.050.png)

上面的红色警告可以忽略，原因是当编译Widora现在所运行的这个固件的是后我们没有选择 `“Advanced configuration options (for developers)->Build Options->Debugging”`。


## 6 总结

恭喜，你现在已经有一个完整的OpenWrt的IDE开发环境了！你可以在Eclipse IDE里面开发你的OpenWrt C/C++ 程序，交叉编译它，设置断点，远程调试。

  ![1.1.000](picture/000.051.png)

  ![1.1.000](picture/000.052.png)
