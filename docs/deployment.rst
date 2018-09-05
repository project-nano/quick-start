.. deployment .

-------
系统部署
-------

Nano平台提供了Installer安装程序用于自动化部署，最新版本Installer可以通过 `官网下载 <https://nanos.cloud/zh-cn/download.html>`_ 或者 `Github发布页面 <https://github.com/project-nano/releases/releases>`_ 获取。

Installer会自行选择最合适的配置进行建议， **对于初次安装Nano的新用户，安装过程中尽量不要调整任何参数** ，如需调整参数、修改配置或者直接升级二进制文件，请在熟悉产品后参考 `产品Wiki <https://github.com/project-nano/releases/wiki>`_ 进行操作。

服务器要求：

- 支持虚拟化的X86服务器或者打开嵌套虚拟化（Intel VT-x/AMD-v）的虚拟机
- 2核4G内存50GB磁盘和一个网络设备
- CentOS 7.5(1804) Minimal
- 操作系统安装完成并且网络就绪
- 如有raid/lvm请先完成配置，再进行Nano安装

建议使用刚装完CentOS7.5的纯净系统开始安装，避免残留设置或者依赖版本有问题影响顺利部署。

模块安装
========

安装Nano平台，只需要解压并执行Installer即可。只需要选择需要在当前服务器部署的模块，Installer会自动完成参数配置、依赖安装和模块部署。

以v0.3.1为例，在shell执行以下指令：

::

  $wget https://nanos.cloud/media/nano_installer_0.3.1.tar.gz
  $tar zxfv nano_installer_0.3.1.tar.gz
  $cd nano_installer
  $./installer

Installer启动后首先要求输入要安装的模块，可以输入"0,1,2"+回车，在一个服务器安装所有模块，也可以输入"2"+回车只安装Cell。

Installer默认把模块安装在/opt/nano目录下，默认的通讯域标识为<"nano":224.0.0.226:5599>，对于初次安装或者网络内仅有一套Nano平台的用户，不建议调整参数，以免错误参数影响平台工作。

如果选择安装Cell模块，Installer会要求用户输入"yes"确认构建默认的桥接网络br0； **如果已经有其他程序设置的br0，建议先手工删除，再安装Cell，否则可能会导致云主机网络连接不正常** 。如果是以往Nano安装生成的br0则无影响，可以跳过。

Installer在安装过程中，会选择模块启动监听服务的网卡和地址，如果仅有一张网卡，Installer会自动选择并进行配置；如果存在多个网卡，Installer会列出设备清单要求用户选择Nano集群需要工作的网段。

假如服务器有两个网卡，eth0地址为192.168.1.56/24，eth1地址为172.16.8.55/24，如果希望Nano集群工作在172.16.8.0/24网段，则选择eth1即可。

 *请注意：Installer会首先使用自带RPM包安装依赖库，如果出现版本冲突，则尝试使用yum从网络获取更新版本；如果系统已经预装了会导致冲突的版本，请确保网络可用，以便顺利安装。*

模块启动
========

所有Nano平台模块都使用命令行控制，调用方式："<模块名称> [start\|stop\|status\|halt]"，支持的指令含义如下：

- start: 启动模块，故障打印错误信息，成功则输出版本及必要信息
- stop: 优雅停止模块，自动释放相关资源并通知相关模块
- status: 检查模块是否在运行中
- halt: 强制终止模块运行

模块安装完成后，需要启动模块以提供服务，模块默认安装在/opt/nano目录下。使用命令手动启动所有模块（假定所有模块安装在同一台服务器）， **请注意，必须首先启动Core** 。

::

  $cd /opt/nano/core
  $./core start
  $cd ../cell
  $./cell start
  $ cd ../frontend
  $./frontend start

FrontEnd模块成功启动后，Console会输出一个形如"192.168.6.3:5870"的监听地址，使用Chrome或者Firefox访问这个地址就可以开始通过Web门户管理Nano平台了。


平台准备
========

添加资源节点
............

Nano平台初次启动时，会默认创建一个名为Default的计算资源池，但是该资源池没有可用资源。你需要先将一个Cell节点添加到该资源池，以便有足够资源分配云主机。

在Web门户上，选择"Compute Pool"菜单，点击default资源池的"cells"按钮，进入资源节点清单：

.. image:: images/2_1_compute_pool.png

当前没有任何资源节点，点击"Add Cell"按钮，进入添加页面

.. image:: images/2_2_add_cell.png

在下拉菜单中，选择目前平台中已经发现并且尚未加入资源池的Cell节点，完成添加

.. image:: images/2_3_choose_cell.png

添加完成回到资源节点清单，可以看到新Cell已经加入资源池，并且处于可用状态。

.. image:: images/2_4_cell_online.png

此时，就可以在"Compute Pool"或者"Instances"菜单创建新主机实例了。

上传镜像
........

空白云主机并不能满足我们的日常使用要求，我们还需要安装操作系统和应用软件，Nano提供了多种手段能够快速部署可用云主机。

磁盘镜像
,,,,,,,,

磁盘镜像保存了模板云主机系统磁盘的数据，用户可以选择从预置的磁盘镜像克隆，新建云主机能够获得与模板云主机完全一致的系统和预装软件，有效减少系统重复安装部署的时间。

磁盘镜像中还可以通过预装Cloud-Init模块，配合Nano的CI服务，自动完成管理员密码初始化、系统磁盘扩容和自动数据盘格式化及挂载等配置任务。

Nano官网 `下载 <https://nanos.cloud/zh-cn/download.html>`_ 页面已经提供了CentOS 7.5 Minimal预置镜像（其中一个预装了Cloud Init）。

下载镜像，选择Web门户的"Images"=>"UPLOAD"上传到平台，后续创建云主机时就可以选择从镜像克隆了。

.. image:: images/2_5_upload_image.png

光盘镜像
,,,,,,,,

光盘镜像保存了ISO格式的光盘数据，可以加载到云主机中安装操作系统或者其他系统软件，通常用于定制模板云主机，详见云主机管理和平台管理章节。
