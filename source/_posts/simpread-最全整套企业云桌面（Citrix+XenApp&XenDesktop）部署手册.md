---
title: 桌面云（Citrix+XenAppXenDesktop）
date: 2023-01-031 17:05:21
tags:
	
	- 桌面云
---


**一、案例概述**
----------

为了方便对公司办公计算机桌面系统的管理，公司需要搭建一套桌面虚拟化平台。公司运维工程师决定使用 Cirtix 桌面 虚拟化解决方案。  
Citrix XenServer 服务器虚拟化系统通过更快的应用交付、更高的 IT 资源可用性和利用率，使数据中心变得更加灵活、高效。在提供关键工作负载（操作系统、应用和配置）所需的先进功能的同时，也不会牺牲大规模部署必需的、易于操作的特点、

**二、案例前置知识点**
-------------

**1、桌面虚拟化介绍**
-------------

桌面虚拟化是指将计算机的桌面进行虚拟化，以达到桌面使用的安全性和灵活性，可以通过任何设备，在任何地点、任何时间访问在网络上的属于用户个人的桌面系统。如下图：  

![](https://pic2.zhimg.com/v2-63e75cdd23d2e58b25d078f0eba9ccad_r.jpg)

**2、XenServer**
---------------

XenServer 是由美国 Citrix 公司推出的一种服务器虚拟化平台，其功能强大、丰富，具有卓越的开放性架构、性能和存储集成。它是基于开源 Xen Hypervisor 的免费虚拟化平台，该平台引进了多服务器管理控制台 XenServer，具有强大管理能力。  
安装 XenServer 硬件要求如下：  
1、CPU：一个或多个 x86_64 位 CPU，最低 1.5GHz，建议 2GHz 以上或更快的多核 CPU。  
  
2、内存：最低 2GB，建议 4GB 或更多。  
  
3、硬盘：本地存储（PATA、SATA、SCSI），最低 46GB，建议 70GB 磁盘空间。  
  
4、网卡：100Mb/s 或更快的 NIC

**3、XenDesktop**
----------------

XenDesktop 安装向导是一种工具，自动完成虚拟桌面大型安装的创建和交付部分。此向导集成了 Citrix 组件，系统管理员可以快速创建多个桌面。

**4、XenCenter**
---------------

XenCenter 是在独立的计算机上运行的独立应用程序。通过 XenCenter 可以创建和管理虚拟服务器、虚拟机模板、快照、共享存储支持、资源池和 XenMotion 实时迁移。

**1）安装 XenCenter 操作系统要求如下：**

*   Windows 7 SP1、Windows 8.1、Windows 10（.NET Framework 版本号基于. NET4.6）。
*   Windows server 2008 SP2、Windows server 2008 R2 SP1（.NET Framework 版本号基于. NET4.6）。
*   Windows server 2012、Windows server 2012 R2（.NET Framework 版本号基于. NET4.6）。

**2）安装 XenCenter 硬件要求如下：**

*   CPU 主频最低为 750MHz，建议使用 1GHz 及以上。
*   内存最低为 1GB，建议使用 2GB 及以上最低为 100MB。
*   网卡为 100Mb/s 或更快的 NIC。
*   屏幕分辨率最低为 1024 X 768 像素。

**5、Desktop Delivery Controller**
---------------------------------

桌面传送控制器（Desktop Delivery Controller，DDC）是 XenDesktop 的一个组件，可以单独安装，也可以把所有组件安装在一起。该控制器安装在数据中心的服务器上，用于对用户进行身份验证，管理用户虚拟桌面环境的程序集，以及代理用户及其虚拟桌面之间的连接。它控制桌面的状态，根据需要管理配置启动和停止它们。其中的 Profile Management 还可以管理 Windows 环境中用户个性化设置 VDA

虚拟桌面访问（Virtual Desktop Access，VDA）是一种授权策略，是指每个访问虚拟桌面的设备都要获取的访问许可。它是通过许可访问虚拟桌面的设备，而不是许可虚拟桌面本身。

**三、案例环境**
----------

![](https://pic4.zhimg.com/v2-4a19824cba6b41a72eb0f74bba684cfb_r.jpg)

此案例用到的所有工具 可以访问网盘下载[链接：](https://link.zhihu.com/?target=https%3A//pan.baidu.com/s/1JEsDt-DI88DKz7aEn8PisQ%26shfl%3Dsharepset)[https://pan.baidu.com/s/1JEsDt-DI88DKz7aEn8PisQ&shfl=sharepset](https://link.zhihu.com/?target=https%3A//pan.baidu.com/s/1JEsDt-DI88DKz7aEn8PisQ%26shfl%3Dsharepset)  
提取码：v287  
复制这段内容后打开百度网盘手机 App，操作更方便哦

**四、问题分析**
----------

XenServer 对服务器的配置要求并不太高，处理器要求是一个或多个 64 位 x86 x86CPU，主频最低位 1.5GHz；内存要求最低为 2GB；硬盘本地连接的存储（PATA、SATA、SCSI），最低磁盘空间为 46GB；千兆网卡。由于服务器上要运行虚拟机，因此建议在实际生产环境中资源配置应该 根据应用规模适度调节。

**五、案例实施**
----------

**1、新建 XenServer：**
-------------------

选择典型安装，单击下一步  

![](https://pic4.zhimg.com/v2-45baa9304bda19476d554a11ad7daadf_r.jpg)

选择稍后安装操作系统，单击下一步  

![](https://pic1.zhimg.com/v2-082ffd13616f1057080ca83cfdc34410_r.jpg)

选择 Linux 操作系统，版本选择 Red Hat Enterprise Linux 5 版本，单击下一步  

![](https://pic4.zhimg.com/v2-cf19a6bb7d009daf57efe3e98526ea27_r.jpg)

编辑虚拟机名字，浏览安装位置，单击下一步  

![](https://pic4.zhimg.com/v2-af917f9085f8fd56693df370d7423517_r.jpg)

磁盘空间调为 500GB，单击下一步  

![](https://pic4.zhimg.com/v2-4fbd3bf0eaecc914118875fc47d354fb_r.jpg)

新建完成  

![](https://pic2.zhimg.com/v2-c946cfee449ac33e2621761f80232359_r.jpg)

编辑虚拟机处理器  

![](https://pic3.zhimg.com/v2-e70bc05e48a27e7b64fe9573d5e2ed52_r.jpg)

编辑虚拟机内存，网卡 VM2，挂载光盘，开启虚拟机  

![](https://pic2.zhimg.com/v2-05abc594973deb1684e6f92b42154069_r.jpg)

键盘选择 US，单击 OK  

![](https://pic3.zhimg.com/v2-a85cdeeca3d95ea05a1e7839261d2bca_r.jpg)

选择 OK 开始进行安装  

![](https://pic3.zhimg.com/v2-80956c101e07736b7e9622520f52194e_r.jpg)

阅读许可协议，单击 Accept EULA  

![](https://pic4.zhimg.com/v2-e334d81372f17164446ef038d2b993e7_r.jpg)

安装在 500G 硬盘，默认已经选中，单击 OK  

![](https://pic2.zhimg.com/v2-ce81c8a70ec948dc4e56c604f6574139_r.jpg)

选择从本地启动，单击 OK  

![](https://pic2.zhimg.com/v2-e0a78a52e8135a8e5faa385af3e11b2d_r.jpg)

不添加数据包，单击 NO  

![](https://pic2.zhimg.com/v2-e9bf8b8462e41b07b4acb0eadb5810f1_r.jpg)

选择跳过  

![](https://pic4.zhimg.com/v2-01af0f1a91efeac76876a7c68592bf8f_r.jpg)

配置密码  

![](https://pic3.zhimg.com/v2-94b834142fef7d7d1184ad5678e3db1a_r.jpg)

手动配置 IP 地址、网关子网掩码  

![](https://pic3.zhimg.com/v2-38d5f6f96eec1058375d13fd3bb0f846_r.jpg)

修改计算机名。配置 DNS  

![](https://pic2.zhimg.com/v2-dab98d8df8b79edeb71190ebeab95c35_r.jpg)

时区选择亚洲上海  

![](https://pic3.zhimg.com/v2-78f888ac0407afa96f8cb23c74868c92_r.jpg)

手动配置时间  

![](https://pic2.zhimg.com/v2-0ac5784be603a35e01ad6ca6c3bee221_r.jpg)

  
开始安装  

![](https://pic3.zhimg.com/v2-aebff25f45be0da2ae0696f63ffcf68e_r.jpg)

安装中  

![](https://pic1.zhimg.com/v2-9434c3f86c5a1d1df608da5f8f22d8c0_r.jpg)

配置时间  

![](https://pic2.zhimg.com/v2-98e0f4ab67f07465bf1ba2922668b311_r.jpg)

安装完成，重启即可  

![](https://pic3.zhimg.com/v2-d55e0a2cb6ead6d07c71a91b10ef8386_r.jpg)

**2、开启 DC01_AD ：**
------------------

![](https://pic4.zhimg.com/v2-915f6879ba8e0c38f56f5441d205d513_b.jpg)

配置 IP 地址  

![](https://pic4.zhimg.com/v2-2c8ec52379b7df54398e3556f64dc8b3_r.jpg)

添加域服务  

![](https://pic4.zhimg.com/v2-36ffad60c18dbf73ef93e74cdb972143_r.jpg)

一流下一步，安装即可  

![](https://pic1.zhimg.com/v2-838f110be01039480888e1551038e38c_r.jpg)

安装完成，设置为域控制器  

![](https://pic3.zhimg.com/v2-a6b85624507476c287b056c73303cc76_r.jpg)

选择添加新林，设置域名，单击下一步  

![](https://pic2.zhimg.com/v2-b06aea38ecbd6db4407fd38ee7297081_r.jpg)

设置域控制器密码  

![](https://pic4.zhimg.com/v2-a01dd7d0a757af680e96adb315cdd483_r.jpg)

默认下一步  

![](https://pic2.zhimg.com/v2-d19e65868d57bfc84ddda616dcff0571_r.jpg)![](https://pic4.zhimg.com/v2-ff8a061ada45a4dd6d178aaccae81387_r.jpg)![](https://pic2.zhimg.com/v2-fba7e089bd9ffc8b58b122a4f19c9259_r.jpg)![](https://pic4.zhimg.com/v2-a2eacca806995e7cec770d41ed1ce1df_r.jpg)

开始安装  

![](https://pic4.zhimg.com/v2-e2a06837180d4261548a80cc2d2db43f_r.jpg)

域管理员登录  

![](https://pic2.zhimg.com/v2-199e6fbcdf23d856bb541fad6feb4241_r.jpg)

关闭域防火墙  

![](https://pic1.zhimg.com/v2-d678833e05c0bc673991de030929cf20_r.jpg)

修改 DNS  

![](https://pic3.zhimg.com/v2-ee64179e5bf0b5502dfaf3272acecade_r.jpg)

域控制器添加数据库管理员账户 sqladmin  

![](https://pic2.zhimg.com/v2-f00e917d2a0cc37c29ef8cf983c737ad_r.jpg)

用户设置密码  

![](https://pic2.zhimg.com/v2-068ad049431c74e91e0ac8eff3ec5e05_r.jpg)

用户 隶属于 domain admins  

![](https://pic3.zhimg.com/v2-0e98212360e4917d87b763b6f2f1d2ee_r.jpg)

**3、开启 DC02_SQL：**
------------------

![](https://pic3.zhimg.com/v2-0660c1547b2e3faeb8d67858dbb88416_b.jpg)

配置 IP 地址、子网掩码和 DNS  

![](https://pic2.zhimg.com/v2-4893558855ef4bd51939b45b2d383be1_r.jpg)

DC02_SQL 加入域  

![](https://pic1.zhimg.com/v2-2bab48eb407504cff600af7a42fda5d8_r.jpg)

登录域  

![](https://pic3.zhimg.com/v2-1df59ea25e37eefa543d1f8916c7e192_r.jpg)

将 sqladmin 数据库管理员用户添加到本地 administrators 组  

![](https://pic2.zhimg.com/v2-5b11d0eda314cfd06a9028a87e9de921_r.jpg)

将本地管理员 administrator 禁用  

![](https://pic4.zhimg.com/v2-523e0aff1d1d0d6f7e87e845d6a12423_r.jpg)

注销使用数据库管理员 sqladmin 账户登录域  

![](https://pic4.zhimg.com/v2-b061c23481a37895c547c98d91973987_r.jpg)

关闭域防火墙  

![](https://pic2.zhimg.com/v2-02a698ade4897c9f437af1e777622359_r.jpg)

安装. NAT Framework 3.5  

![](https://pic4.zhimg.com/v2-cacb9c1a01041c13a2544592cacd657b_r.jpg)

一流下一步安装即可  

![](https://pic2.zhimg.com/v2-cf2e3d4d99a4e84e1045987c02f38b4d_r.jpg)

安装完成  

![](https://pic3.zhimg.com/v2-dd8e3e453827121b6446627627b6ac26_r.jpg)

切换 sql_server_2008_R2 光盘  

![](https://pic2.zhimg.com/v2-50558c3f65f77823410303d84ee094fd_r.jpg)

打开 DVD，选择全新安装  

![](https://pic4.zhimg.com/v2-9379886215af180a9938751198821f9b_r.jpg)

检查先决条件，通过后单击确定  

![](https://pic2.zhimg.com/v2-0a5aedbd88eaceda802927315dc45b61_r.jpg)

输入产品密钥  

![](https://pic1.zhimg.com/v2-362171ef41abd202dec1aa0ebc4cf7f8_r.jpg)

接受许可条款，单击下一步  

![](https://pic4.zhimg.com/v2-8b0ed31f7b99e38084fea4f7030a7ec7_r.jpg)

单击安装，开始安装  

![](https://pic2.zhimg.com/v2-c942ec579310e6ecbdf6baa83a5058a5_r.jpg)

一条警告，忽略下一步  

![](https://pic4.zhimg.com/v2-71a5005d63597a0200cb5d5930a3b9d7_r.jpg)

选择 SQL Server 功能安装，单击下一步  

![](https://pic3.zhimg.com/v2-f7399baf9e7852379475ed06b5808cbe_r.jpg)

勾选数据库引擎和管理工具，单击下一步  

![](https://pic4.zhimg.com/v2-ee4290e5f43f70b008883c8fde218b67_r.jpg)

默认下一步  

![](https://pic1.zhimg.com/v2-08c288707002b6c9f0572a3c485c9410_r.jpg)

选择默认实例，单击下一步  

![](https://pic2.zhimg.com/v2-a6b71fdcbfb327b8c8d831d26d2168c9_r.jpg)

单击下一步  

![](https://pic1.zhimg.com/v2-871f2b286de060e13b119f95b24fa4dc_r.jpg)

配置服务器账户密码  

![](https://pic2.zhimg.com/v2-21ba7c2c7db360e8c92b817043080b01_r.jpg)

选择混合模式，输入密码，添加当前账户  

![](https://pic4.zhimg.com/v2-33e4d074abe1cbcb2cfd3c6b5b305cff_r.jpg)

默认下一步  

![](https://pic4.zhimg.com/v2-b491a946f819dc2eb857034ff645420b_r.jpg)

先决条件通过单击下一步  

![](https://pic4.zhimg.com/v2-19ad29ad1e30170879f2575b522c5dbb_r.jpg)

单击安装  

![](https://pic2.zhimg.com/v2-2664ef9c0f7064ce87750c0a0c097efd_r.jpg)

安装完成  

![](https://pic4.zhimg.com/v2-2c4de4ab85d5f442a1a6e18783adce57_r.jpg)

**4、开启 DC03_XenCenter：**
------------------------

配置 IP 地址、子网掩码和 DNS  

![](https://pic3.zhimg.com/v2-11e51a69a08d08edc382e122f2d62af6_r.jpg)

DC03_XenCenter 加入域  

![](https://pic1.zhimg.com/v2-02c3cdfad608c1f4a08897ad6e42b0cc_r.jpg)

重启使用本地管理员登录域  

![](https://pic3.zhimg.com/v2-f71b4fdfe1f28a37f272d8bdc7206ac2_r.jpg)

关闭域防火墙  

![](https://pic2.zhimg.com/v2-1d3185db9deef3a2417309d7b28af729_r.jpg)

安装 windows 8.1-KB2919442-x64  

![](https://pic4.zhimg.com/v2-b599e2b6d6914b64e0c300838ae55aeb_r.jpg)

安装完成  

![](https://pic3.zhimg.com/v2-b1e0cfc2b4c2cd60b8ebf693a44c006e_r.jpg)

安装 windows 8.1-KB2919355-x64  

![](https://pic4.zhimg.com/v2-b78f4e52826ddc423985bee5ef686a07_r.jpg)

安装完成，重启计算机  

![](https://pic4.zhimg.com/v2-c4559bd19cb39fe8413577606bdbde97_r.jpg)

重启完成后安装 [http://Microsoft.NET](https://link.zhihu.com/?target=http%3A//Microsoft.NET) 2015  

![](https://pic4.zhimg.com/v2-d6dccc9d0ebf7c8c971a235071ed4a2f_r.jpg)

安装完成，重启计算机  

![](https://pic4.zhimg.com/v2-901940df2289900228e0e29d398c715f_r.jpg)![](https://pic3.zhimg.com/v2-ca66884aeb0bfa4c4365123b2a8e7982_b.jpg)![](https://pic3.zhimg.com/v2-afaddc345d0e21ce8b2767e55d02b1b2_r.jpg)

选择所有用户，单击下一步  

![](https://pic3.zhimg.com/v2-ecb8815fbd6ecc15e56e52d7ca035a62_r.jpg)

单击安装  

安装完成  

![](https://pic3.zhimg.com/v2-59be0a8779127efc98d63670194eb53e_r.jpg)

打开 Citrix XenCenter  

![](https://pic3.zhimg.com/v2-a6343c618a4cdcb99e7c9abba0c4326e_r.jpg)

连接服务器  

![](https://pic1.zhimg.com/v2-952c24cfa81cbf5a8849146f12535244_r.jpg)

连接成功，确定即可  

![](https://pic1.zhimg.com/v2-6e15b6d76312e81c2faead21fb355554_r.jpg)

切换 sql 光盘，安装 sql 客户端  

![](https://pic3.zhimg.com/v2-4f963249ea8fdcba5458f368236411aa_r.jpg)

选择全新安装  

![](https://pic4.zhimg.com/v2-6d67fd070f8e4d2aa141bd66a177f123_r.jpg)

检查先决条件，单击确定  

![](https://pic1.zhimg.com/v2-b351dfe285cb5ba9f9941134794f8f2c_r.jpg)

输入产品密钥  

![](https://pic3.zhimg.com/v2-a59e5a09523ad110e38b78f6564a26b2_r.jpg)

接受许可条款，单击下一步  

![](https://pic2.zhimg.com/v2-ac346c5bad18220afc15054980735fe9_r.jpg)

单击安装  

![](https://pic4.zhimg.com/v2-6f6393e821bf080aa0fa499b7f1e144f_r.jpg)

单击下一步  

![](https://pic4.zhimg.com/v2-32121b04956871ba7f82575bfcf0a983_r.jpg)

选择 SQL Server 功能安装，单击下一步  

![](https://pic2.zhimg.com/v2-086d16e09ef88ce8906e793e9ce7e365_r.jpg)

勾选管理工具，单击下一步  

![](https://pic4.zhimg.com/v2-d2d592275956ab6acfe711e0cf960a8f_r.jpg)

检查完成，单击下一步  

![](https://pic4.zhimg.com/v2-448d7fbb4da0556457911c0edeab8e37_r.jpg)

默认下一步  

![](https://pic3.zhimg.com/v2-0a174d2c9d76e81ed24b16d2772b9406_r.jpg)![](https://pic1.zhimg.com/v2-eeb4d9cad0777abd81ea8c93a5990ef4_r.jpg)![](https://pic2.zhimg.com/v2-c687fdccc009ce45674c124525198835_r.jpg)

单击安装  

![](https://pic2.zhimg.com/v2-6dc48406d7d3469297c74f8a561b2e71_r.jpg)

安装完成  

![](https://pic1.zhimg.com/v2-9c4b435f84cdf73cf829b25d8241495c_r.jpg)

连接数据库  

![](https://pic1.zhimg.com/v2-fac2cc2d64126e07271d0391607169f0_r.jpg)

输入 SQL 服务器名称，选择身份验证，输入登录名和密码，单击连接  

![](https://pic2.zhimg.com/v2-691bcd7a8d3b2c921123f6cfed338105_r.jpg)

成功连接数据库  

![](https://pic2.zhimg.com/v2-b69e722dd6f930aecccbf736fbcbac91_b.jpg)

切换 XenAPP_and_XenDesktop 光盘  

![](https://pic3.zhimg.com/v2-e25f16ab51309e5f93d3314bcdc836ae_r.jpg)

打开 DVD，启动 XenDesktop  

![](https://pic2.zhimg.com/v2-40c93061f6ebbea92e88286b0811dba5_r.jpg)

点击安装 Delicery Controller  

![](https://pic1.zhimg.com/v2-d1c8d9c596b7c3ae7dcf5a28f294c9d8_r.jpg)

接受许可条款，单击下一步  

![](https://pic4.zhimg.com/v2-de590ef2a1206ab810eebba0fe0ef1a3_r.jpg)

勾选安装全部核心组件，单击下一步  

![](https://pic3.zhimg.com/v2-2c3364d6993b38023a0329450640ab9e_r.jpg)

勾选安装 windows 远程协助，单击下一步  

![](https://pic2.zhimg.com/v2-efffd8f86133ee27ca7ae90cba2ef7c9_r.jpg)

防火墙规则选择手动配置，单击下一步  

![](https://pic1.zhimg.com/v2-691657a52c3b067d5df05db5dce510e4_r.jpg)

单击安装  

![](https://pic1.zhimg.com/v2-0e91e09aa580528fbcff78919ce0bcd4_r.jpg)

安装完成  

![](https://pic2.zhimg.com/v2-54ef73e6a7aefa71cbf14ff0407abbc9_r.jpg)

**基础环境已经全部搭建完成，接下来开始配置**

首先在域控制器创建一个组织单位  

![](https://pic3.zhimg.com/v2-e915587c7c15637b7a30ddd1127434aa_r.jpg)![](https://pic2.zhimg.com/v2-28701b38e003b5e7835e85de9d24cfe5_r.jpg)

Xen_Client 里创建 3 个测试账户  

![](https://pic1.zhimg.com/v2-32f7448475f0d098852bc4c65767a60c_r.jpg)

账户创建完成  

![](https://pic2.zhimg.com/v2-1319920b7d552431978c9bd6749eda49_r.jpg)

在 DC02_SQL 部署 DHCP  

![](https://pic3.zhimg.com/v2-3ec4c92ff3b7e0e148aa73d8811ca5b6_r.jpg)

一流下一步开始安装  

![](https://pic1.zhimg.com/v2-da10944dd11cf6e44abe8f111170e1a0_r.jpg)

安装完成  

![](https://pic1.zhimg.com/v2-a432a00dc356a0f887db934544297e58_r.jpg)

新建作用域  

![](https://pic2.zhimg.com/v2-d65e5b848455c271b09ef0d67af166f9_b.jpg)

单击下一步  

![](https://pic2.zhimg.com/v2-0d17e1486d0ddef90819749322907ea5_r.jpg)

编辑作用域名字  

![](https://pic1.zhimg.com/v2-7a57c63f58b7f4d9a454897637903044_r.jpg)

输入地址范围  

![](https://pic3.zhimg.com/v2-5cde1da55b318f74fd9efacc2244f32e_r.jpg)

默认下一步  

![](https://pic2.zhimg.com/v2-54bd194069de8d911c02662f2b86fafd_r.jpg)

选择否，单击下一步  

![](https://pic3.zhimg.com/v2-66c1f48763e1f0b8fafba7d74e1e58ba_r.jpg)

作用域新建完成  

![](https://pic3.zhimg.com/v2-62696089ba406ec0b0d19129e86fa972_r.jpg)

添加 DNS  

![](https://pic2.zhimg.com/v2-be93f85b4afe61545e934eb5a05ea7a5_r.jpg)

打开 DC03_XenCenter

复制 Windows 7 光盘到虚拟机，创建共享文件夹将光盘共享  

![](https://pic2.zhimg.com/v2-3c7b6a95185de4f9548fa3d05f9ff515_r.jpg)

打开 XenCenter，连接服务器  

![](https://pic4.zhimg.com/v2-9a96a2ff05fe320c87ad96c9e8aa1f2b_r.jpg)

连接共享  

![](https://pic3.zhimg.com/v2-a7074a4270d2e9c4107b274a5868a736_b.jpg)

选择 Windows 文件共享  

![](https://pic1.zhimg.com/v2-85a8da1c9874cefc519a0686035fbaf0_r.jpg)

编辑名字  

![](https://pic3.zhimg.com/v2-e6c368b2a15fc6a102d5b84ccaf0b7c2_r.jpg)

输入共享名称，  

![](https://pic3.zhimg.com/v2-f56f2080994edd41852b063b802f84c6_r.jpg)

选择向用户交付应用程序和桌面  

![](https://pic4.zhimg.com/v2-3c76fd5a199ba8dec0538216413559fb_r.jpg)

编辑站点名字，单击 下一步  

![](https://pic1.zhimg.com/v2-6b1e437b5ae841da774ae86ac8f89e94_r.jpg)

连接数据库  

![](https://pic2.zhimg.com/v2-938be2f369fb20a97e97f2af42a62c0d_r.jpg)![](https://pic2.zhimg.com/v2-ddf058305866ab88a1b134ed07086b21_r.jpg)![](https://pic1.zhimg.com/v2-608d21012d85761aa607088d5a9c8104_r.jpg)

默认下一步  

![](https://pic1.zhimg.com/v2-8c101012d6dbf58c531db83dcc6d7858_r.jpg)

连接 XenServer  

![](https://pic4.zhimg.com/v2-dc1b49928faf8728facdec52cbe890bb_r.jpg)

编辑网络名字，单击下一步  

![](https://pic1.zhimg.com/v2-e048132bda17eabde7ba986c0cd38858_r.jpg)

默认下一步  

![](https://pic3.zhimg.com/v2-26d0c82eb9e26fd8eff0c9e2431042d6_r.jpg)![](https://pic2.zhimg.com/v2-b1b58aeac4074d55e49f36ba40a5518d_r.jpg)

选择否，以后加入，单击完成  

![](https://pic1.zhimg.com/v2-063b98377cc063e93a7c4d280cb99a14_r.jpg)

配置完成  

![](https://pic4.zhimg.com/v2-86d8f87504eb454f0a28316732a76063_r.jpg)

开启 win7_01  

![](https://pic3.zhimg.com/v2-55cef1f3caae3fc13dd1cf5135ea496a_r.jpg)

运行 cmd 释放 IP 地址重新获取 DHCP 自动下发  

![](https://pic1.zhimg.com/v2-b1d7a747187f33bd56399084f443bbb8_r.jpg)

Ipconfig/all 查看详细信息，查看 DNS 是否正确  

![](https://pic2.zhimg.com/v2-bfb5288052a1274ce9855cb1bd9ca0f9_r.jpg)

计算机加入域，重启计算机  

![](https://pic4.zhimg.com/v2-8e9684e0989ebe40863040942179e6b3_r.jpg)

将域控制器创建的 bob 账户添加到本地 administrators 组  

![](https://pic2.zhimg.com/v2-c8958aa2b7444c60ed34aa712abd5ef5_r.jpg)

禁用本地 administrator，注销使用 bob 登录域  

![](https://pic2.zhimg.com/v2-4da29efb92d2f2dda7b7e6eab80882fd_r.jpg)

Bob 登录域  

![](https://pic1.zhimg.com/v2-c17c8e8c5309210c1885d914d128a3b4_b.jpg)

切换光盘  

![](https://pic2.zhimg.com/v2-4550ed5bb8ea696ffbc94249ce236381_r.jpg)

选择 XenDesktop 交付应用程序和桌面  

![](https://pic4.zhimg.com/v2-7c676b701ad953587ecfde6c729c8f37_r.jpg)

单击 Virtual Delivery Agent for Windows Desktop OS  

![](https://pic4.zhimg.com/v2-d86a13c3a07f149a223028eb577cad3f_r.jpg)![](https://pic4.zhimg.com/v2-b2bbbb7eab7a46a8bf701a1f90effa03_b.jpg)

选择启用 Remote PC Access，单击下一步  

![](https://pic2.zhimg.com/v2-8c957e6a007dc29241f3de5480186251_r.jpg)

选择否，单击下一步  

![](https://pic2.zhimg.com/v2-e8c4769b35a34e57016fddff5ef19ea9_r.jpg)

根据需求是否勾选 Citrix Receiver，我这里用不到平板电脑或者手机，所有就不勾选了  

![](https://pic2.zhimg.com/v2-d6973c90c025e5b9c91313d12b4f1e59_r.jpg)

添加 Controller 地址  

![](https://pic2.zhimg.com/v2-340e7298f64ed8d9c5f78a187481cec9_r.jpg)

默认下一步  

![](https://pic1.zhimg.com/v2-c1a649263c587a7d6438e53980af7638_r.jpg)

防火墙选择手动，单击下一步  

![](https://pic1.zhimg.com/v2-628265d06dccf58350159450e7a6dc4c_r.jpg)

单击安装  

![](https://pic2.zhimg.com/v2-bd67432b657a8a148837cbed31ad549d_r.jpg)

安装完成，重启计算机  

![](https://pic2.zhimg.com/v2-0a46e90e05d46dbaa37c7f7bebe923e5_r.jpg)

开启远程访问  

![](https://pic2.zhimg.com/v2-ee8927918228423e682fbd9daddd176d_r.jpg)

XenCenter 计算机打开 Citrix Studio 应用程序创建计算机目录  

![](https://pic4.zhimg.com/v2-73720fc3747bfa648ba9d53b54068e7b_r.jpg)

选择未进行电源管理的计算机，单击下一步  

![](https://pic3.zhimg.com/v2-348ba3672722684ef4c8afd86132709a_r.jpg)

桌面体验选择静态桌面，单击下一步  

![](https://pic4.zhimg.com/v2-cdad8c368b6ea529440f7b4bff70dc7b_r.jpg)

添加计算机  

![](https://pic3.zhimg.com/v2-ea4124b0c70c0d040a93e0d761ea51ae_r.jpg)

编辑计算机目录名字，单击完成  

![](https://pic4.zhimg.com/v2-a8b9f489a6ba7183e1653bf334a0a373_r.jpg)

显示已注册  

![](https://pic4.zhimg.com/v2-b5078a194c942e85dac97698ebd00957_r.jpg)

创建交付组  

![](https://pic4.zhimg.com/v2-9f4e008bf0bc502a3699bd9bf4f0ee7b_r.jpg)

选择交付组计算机数量  

![](https://pic2.zhimg.com/v2-d7f4b8c4b2de812017b7bbefbd8f2ed5_r.jpg)

选择交付桌面，单击下一步  

![](https://pic2.zhimg.com/v2-9b7b18b2f4f9cdf70122bdf4f5ac28a9_r.jpg)

添加用户  

![](https://pic2.zhimg.com/v2-8b0bdfcb61e02d7d67721375d1188ebd_r.jpg)

选择自动  

![](https://pic2.zhimg.com/v2-0a58825f0cdd5adb705d23f6c5e6881d_r.jpg)

编辑交付组的名字  

![](https://pic3.zhimg.com/v2-a90105ac5add8b4d27097eea04fc2872_r.jpg)

开启 win7_02 客户端  

![](https://pic2.zhimg.com/v2-d378f8e869348dee19fabbb356f05291_r.jpg)

运行 cmd，ipconfig /renew 自动获取 IP 地址  

![](https://pic4.zhimg.com/v2-5232b46e2c84b51f9bd1732f25e802af_r.jpg)

打开浏览器访问安装  

![](https://pic1.zhimg.com/v2-755d9f098f6ca5b5228f04c656bd09f0_r.jpg)

安装 Citrix Receiver  

![](https://pic3.zhimg.com/v2-e415c69cf95fc439dacc1238e2f3057a_b.jpg)

安装完成  

![](https://pic4.zhimg.com/v2-4492a5a51b8bdf560e29af8c58d80ceb_b.jpg)

Win 7_01 更新清单，更新完成关机开启即可  

![](https://pic3.zhimg.com/v2-856994271703c65058c053c61b76d9ba_r.jpg)

Bob 登录  

![](https://pic3.zhimg.com/v2-0cb60c5e1adab2fff6666ec4eb3887ae_r.jpg)

正在连接  

![](https://pic1.zhimg.com/v2-36fd48b5b310ae11886c56d709f096cc_r.jpg)

交付静态桌面：

打开 XenCenter  

![](https://pic1.zhimg.com/v2-5ebfc556b2cda3b53ed9f1068d377100_r.jpg)

连接 XenServer  

![](https://pic2.zhimg.com/v2-fc0e8913b883f712ee517d40df282125_r.jpg)

新建存储  

![](https://pic3.zhimg.com/v2-a7074a4270d2e9c4107b274a5868a736_b.jpg)

选择 Windows 文件共享，单击下一步  

![](https://pic3.zhimg.com/v2-0923b6bc8a9a296f125b762652e54fae_r.jpg)

编辑共享名字，单击下一步  

![](https://pic4.zhimg.com/v2-ee5112a805f7977873e20e3d4aa6d30f_r.jpg)

输入共享位置，勾选其他用户，输入用户名和密码  

![](https://pic1.zhimg.com/v2-8dcbb646a825619e00d4ba63109295ac_r.jpg)

共享存储添加完成  

![](https://pic1.zhimg.com/v2-4b718eb3698dfe79d31c3bcb4dc43d1c_r.jpg)

新建 VM  

![](https://pic3.zhimg.com/v2-af12ee786a5afbdfb8a1c60f6d95e4d2_b.jpg)

选择操作系统，单击下一步  

![](https://pic4.zhimg.com/v2-47374bd5d3e84e510d9360a86f1381bf_r.jpg)

名字设置为 Windows 7  

![](https://pic2.zhimg.com/v2-40fa6dd7b4b4eaab233ac8d70eef0cb9_r.jpg)

选择操作系统光盘  

![](https://pic1.zhimg.com/v2-dbde7ea73c3b03369375e4ab94ef9b10_r.jpg)

将 VM 放在此服务器上，单击下一步  

![](https://pic1.zhimg.com/v2-5fc9c4260973ba0526d8985c35ff1cc8_r.jpg)

内存 2GB，CPU 1 个，单击下一步  

![](https://pic1.zhimg.com/v2-c807b703ee0b9a1df36bdf894e0f22fc_r.jpg)

磁盘大小设置为 50GB，单击下一步  

![](https://pic1.zhimg.com/v2-7fa8d57fd8892f738cec239da33cc9cc_r.jpg)

网络连接保持默认，单击下一步  

![](https://pic1.zhimg.com/v2-ed243318bf3268e9d70976f2fe26ab6c_r.jpg)

开始创建  

![](https://pic2.zhimg.com/v2-38f2dbf7fcd77eafa1a05f0b6d65f791_r.jpg)

自动开启虚拟机  

![](https://pic4.zhimg.com/v2-8cd22cc78781017c5bcbc0622d3d88eb_r.jpg)

选择现在安装  

![](https://pic3.zhimg.com/v2-7fa185cba5a382218d058956a1e1d75e_r.jpg)

接受许可条款，单击下一步  

![](https://pic3.zhimg.com/v2-01e005138fc6c95f2dda22425070892e_r.jpg)

创建 30GB 磁盘，单击下一步  

![](https://pic1.zhimg.com/v2-4fd738da7c1b72d610e82a2f3a19fcf0_r.jpg)

安装完成，编辑计算机名字  

![](https://pic2.zhimg.com/v2-cb7051f81b6e6f5c1147ef0cde4619dd_r.jpg)

密码忽略，我这里就不设置了  

![](https://pic4.zhimg.com/v2-f76f864652aec42395ba772b39126a03_r.jpg)![](https://pic3.zhimg.com/v2-ae8fddc76c2904e26d29fbf622ce8866_r.jpg)![](https://pic2.zhimg.com/v2-60311d37863298d8364930fb8eb92159_r.jpg)![](https://pic3.zhimg.com/v2-2c702678f669b0976a54e4847a4771ce_r.jpg)

虚拟机自动获取 IP 地址  

![](https://pic3.zhimg.com/v2-a0cae7d664bcddbbf8cc941139bbf97e_r.jpg)

关闭防火墙  

![](https://pic4.zhimg.com/v2-55c8bdc945ac1afeefc0862cb06826cb_r.jpg)

切换光盘安装 tools  

![](https://pic1.zhimg.com/v2-deec68d0b66af579ff857f8e2dd97260_r.jpg)

单击下一步  

![](https://pic1.zhimg.com/v2-91ddf7535f131cedb9112577f125bf28_b.jpg)

接受许可条款  

![](https://pic4.zhimg.com/v2-657a9a4a5ce37431c8984de8a6957283_b.jpg)

默认安装位置  

![](https://pic3.zhimg.com/v2-dcb971214101d27109c0e1519580f702_b.jpg)

单击安装  

![](https://pic3.zhimg.com/v2-bbad7f4478d029a1e51ab67671f282de_b.jpg)

安装完成  

![](https://pic4.zhimg.com/v2-a1a51944df8f1fa7f2add31bed3fb21b_b.jpg)

重启  

![](https://pic3.zhimg.com/v2-744a3e4c88517ccca3970d07bb2312e6_b.jpg)

关闭虚拟机转换为模板  

![](https://pic4.zhimg.com/v2-7597203397ca2cc7ce7d6ccdb53d5e97_r.jpg)

新建 VM 向导  

![](https://pic4.zhimg.com/v2-14264fc67b94cde2ca47c11539ed0c8b_r.jpg)

选择 VM 模板  

![](https://pic3.zhimg.com/v2-8a05e807e8bf715d0e1147cb3d9bc9a2_r.jpg)

编辑名称  

![](https://pic1.zhimg.com/v2-b791d6ea0c57a6b2d426f7c20765ca28_r.jpg)

默认下一步，内存 1GB  

![](https://pic1.zhimg.com/v2-21dd9d972d3631ec28286d1c060ef4c0_r.jpg)

完成，立即创建  

![](https://pic4.zhimg.com/v2-ef82feb683065cbe9b3274bcaab25fd3_r.jpg)

打开控制台，加入域重新启动  

![](https://pic1.zhimg.com/v2-4115516eb4787a633e3a1aa7ee1bc940_r.jpg)

切换管理员登录域  

![](https://pic3.zhimg.com/v2-e1370727933e27ffb2041b0db90d1e06_r.jpg)

关闭域防火墙  

![](https://pic2.zhimg.com/v2-1c425ac71ec476f8a98f2e023d877095_r.jpg)

切换光盘  

![](https://pic3.zhimg.com/v2-b982f03ee3d57555083e330cd0d010b6_r.jpg)![](https://pic1.zhimg.com/v2-4a75f44792eb6f2abd2fc9dab3f1054c_r.jpg)

选择创建主映像，单击下一步  

![](https://pic3.zhimg.com/v2-d4a8f6c6920e2038821140b84db0ff4a_r.jpg)

选择不安装 VDA  

![](https://pic4.zhimg.com/v2-d7bd73189eaeb25c104c5516329ccdb7_r.jpg)

  
  
此选项根据个人需求勾选选项  

![](https://pic2.zhimg.com/v2-7df58a50e933d21bea4fa74fac5c1da1_r.jpg)

连接 Controller 地址  

![](https://pic2.zhimg.com/v2-c2d655133bf8a5c4c7ccc72210b1ed25_r.jpg)

功能全选，单击下一步  

![](https://pic4.zhimg.com/v2-6ac8968da3f2768556d9a6fb520af50b_r.jpg)

防火墙规则选择手动，单击下一步  

![](https://pic4.zhimg.com/v2-f607158927aee30b589c46b022e8638f_r.jpg)

开始安装  

![](https://pic3.zhimg.com/v2-d5b78009a6d0afae2ff673c9f01dee0e_r.jpg)

安装完成，重启计算机  

![](https://pic1.zhimg.com/v2-7c57b576530f8c8ac9ef2cdb884035c8_r.jpg)

更新清单  

![](https://pic3.zhimg.com/v2-08b1567ead327be6591aab190e35ace6_r.jpg)

关机之后创建交付  

![](https://pic2.zhimg.com/v2-bd622c049f068aca342ebe4e92a1576d_r.jpg)

默认下一步  

![](https://pic1.zhimg.com/v2-e9513660c1271942683ee625a0abb614_r.jpg)

选择 Windows 桌面操作系统  

![](https://pic3.zhimg.com/v2-28b7091ff18fefd45f8258a9e442a596_r.jpg)

选择进行电源管理的计算机  

![](https://pic4.zhimg.com/v2-f4f7e981adff38299bf15c7a794aa257_r.jpg)

选择静态桌面  

![](https://pic1.zhimg.com/v2-a4ac48accb64b0e6db939b03ec260a14_r.jpg)

默认下一步  

![](https://pic3.zhimg.com/v2-2eaf1e0a3a4719b12ec6d15f32ab42b6_r.jpg)

虚拟机数量选 2 台，单击下一步  

![](https://pic1.zhimg.com/v2-edefaf763fc19168050f1d0c0dc32af0_r.jpg)

编辑账户命名方式  

![](https://pic4.zhimg.com/v2-047456ee8a04b0cbaa5fc9b89318baf3_r.jpg)

编辑名称  

![](https://pic4.zhimg.com/v2-262b3dc29cfadd2313b3efd2ffa6e0eb_r.jpg)

域控制器查看  

![](https://pic3.zhimg.com/v2-5dfc4178eb58b0b136448979a5abeaa2_r.jpg)

创建交付组，交付数量为 2  

![](https://pic1.zhimg.com/v2-89e28c73a5f0e7ee4282c01b61e1dce0_r.jpg)

交付桌面  

![](https://pic4.zhimg.com/v2-d57e79eec5b983ee71be4d3b9ee8a4ff_r.jpg)

添加用户  

![](https://pic3.zhimg.com/v2-0a6dc6b2a00660252da246b1b9fe6a1e_r.jpg)

选择自动，单击下一步  

![](https://pic3.zhimg.com/v2-43583e25c2567f34eea019ee4f28079a_r.jpg)

编辑名称  

![](https://pic2.zhimg.com/v2-a42a098b07c1d8080da5aebafc80441d_r.jpg)

更改交付用户  

![](https://pic4.zhimg.com/v2-0831d25ff941553a0516f5c71b03926f_r.jpg)![](https://pic4.zhimg.com/v2-c1c8c3233f5d48baa8655849e7e890db_b.jpg)

Win7_02 更改交付用户为 alice  

![](https://pic1.zhimg.com/v2-d0725bb24da8b347fc61860f20c8ff00_r.jpg)

开启 win7-01  

![](https://pic3.zhimg.com/v2-248c1cadbd1ee958eae798ae2bf6bbda_r.jpg)

检查远程访问  

![](https://pic1.zhimg.com/v2-7fe10a641ace78e105821aad751b60c8_r.jpg)

开启远程访问，添加 tom 账户  

![](https://pic4.zhimg.com/v2-b0f5c89e71f43563cfd94700c482c4db_b.jpg)

将 tom 设置为管理权限  

![](https://pic3.zhimg.com/v2-01afa4bbc19e09b9bacabcdb33911bba_r.jpg)

禁用 benet 用户  

![](https://pic3.zhimg.com/v2-7c9807356c0f4eceef22830a2fde64c6_r.jpg)

用客户端浏览器访问  

![](https://pic3.zhimg.com/v2-e0b8afc33b10cd3d3f5cb8355636a626_r.jpg)

成功登录  

![](https://pic2.zhimg.com/v2-a999a212f42c5f4b06ecd093ed3a6781_r.jpg)![](https://pic3.zhimg.com/v2-0d7e86ee76687e34322dffdfc77de2b2_r.jpg)

注销切换 Alice 登录  

![](https://pic2.zhimg.com/v2-8035dc7d2d0d12b86722dea0ec0a2bf1_b.jpg)

开启远程访问，添加 Alice 用户  

![](https://pic3.zhimg.com/v2-570aabf825e8e33ace617c22b0572662_r.jpg)

将 Alice 加入 administrators 组  

![](https://pic3.zhimg.com/v2-cb2995d48e8abe4d687fbca854f53c4a_r.jpg)

禁用 benet 用户  

![](https://pic1.zhimg.com/v2-d6a4322b036c246def94dcf82378b408_r.jpg)