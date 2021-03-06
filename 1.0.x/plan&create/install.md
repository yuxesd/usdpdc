# USDP私有化部署流程

欢迎使用 USDP 构建大数据平台，接下来，我们将通过几个简单的步骤，完成 USDP 管理服务的部署流程，从而能够通过 Web 页面的方式，快速部署各类大数据服务与组件。

在开始安装之前，请至少准备如下节点资源，以供后续创建大数据集群。

私有化部署 USDP 需满足如下环境：
 * 节点数量：3 台及以上；
 * 网络：集群间节点内网通畅；
 * 内存：推荐 32G 以上；
 * CPU：推荐 8 核以上；
 * 磁盘：系统盘 60G 以上，数据盘根据业务需要调整；
 * 系统：CentOS 7.6(推荐)，或其他 CentOS 7 系列版本； 
 * 数据库：MySQL 5.7；

下面的部署安装流程以 USDP-Privatization-1.0.0.0 版本为例进行说明。



## 1. 下载安装包

通过给定地址，下载 USDP 离线安装包，得到 ``usdp-01-master-privatization-1.0.0.0.tar.gz`` 文件即可，整个文件大约 17 GB。



## 2. 创建安装目录

创建 ``/opt/usdp-srv/`` 目录，并将压缩包解压至该目录下。



## 3. 目录结构说明

解压后的子目录说明如下所示。

| 子目录       | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| agent        | USDP Agent 程序所在目录，为每个节点的作业端，无需手动启动、无需手动管理，无需手动管理； |
| bin          | 启动 udp-server 与 udp-agent 程序的脚本目录，无需手动管理；  |
| config       | 配置文件目录，主要用于配置 MySQL 的地址，其他无需修改；      |
| jmx_exporter | 监控的相关插件，无需手动管理；                               |
| recommend    | 部署服务时的默认勾选清单，无需修改；                         |
| repair       | 一键修复脚本；                                               |
| repository   | 服务组件资源目录，无需手动管理；                             |
| scripts      | 工具脚本目录，无需手动管理；                                 |
| server       | USDP Server 程序所在目录，无需手动管理；                     |
| sql          | 初始化 SQL 所在目录，无需手动管理；                          |
| templated    | 服务组件配置文件模板目录，无需手动管理；                     |
| verify       | USDP 私有化证书存储目录，无需手动管理；                      |
| versions     | USDP 版本目录，无需手动管理；                                |



## 4. 执行环境初始化

在运行 USDP 之前，需要对所有节点进行必要的配置，为了简化用户操作，USDP 提供傻瓜式的一键环境初始化脚本，包括自动安装 JDK，自动安装 MySQL，并初始化 USDP 数据，以及初始化系统软件环境等。

### 4.1 配置修复

在开始运行一键修复脚本之前，您需要提前配置 ``repair`` 目录下的相关文件如下：

* host_all_info.txt

  示例如下：

  ~~~shell
  127.0.0.1 your-node-root-password 22 udp01
  127.0.0.1 your-node-root-password 22 udp02
  127.0.0.1 your-node-root-password 22 udp03
  ~~~

  文件中每行为一个节点信息，从左至右依次为：内网IP，节点密码，SSH端口号，即将自动修改生效的主机名。

  `提示：在全量初始化部署过程中，host_single_info.txt 暂无须修改。`

* your.properties

  示例如下：

  ~~~shell
   # Repair all host in this cluster.
  
   host.all.info.Path=/opt/usdp-srv/usdp/repair/host_all_info.txt
   namp.server.ip=10.9.25.16
   namp.server.port=22
   namp.server.password=###qcbxzg521
   ntp.master.ip=10.9.25.16
   mysql.ip=10.9.25.16
   mysql.host.ssh.port=22
   mysql.host.ssh.password=###qcbxzg521
   mysql.password=1qaz!QAZ
  
   # Repair single host in this cluster.
  
   host.single.info.Path=/opt/usdp-srv/usdp/repair/host_single_info.txt
  
   # Common Settings.
  
   repair.log.dir=./logs
  ~~~

  > 上述代码解释如下：
  >
  > - 第 1 行：``host_all_info.txt`` 文件绝对路径；
  > - 第 2 行：填写未来即将部署 USDP-Server 的节点的内网 IP；
  > - 第 3 行：SSH 端口号，默认22；
  > - 第 4 行：填写未来即将部署 USDP-Server 的节点的密码；
  > - 第 5 行：选择某个节点作为 NTP 时间服务器；
  > - 第 6 行：选择某个节点作为 MySQL 服务器；
  > - 第 7 行：设置 MySQL 所在节点的 SSH 端口号，默认 22；
  > - 第 8 行：设置 MySQL 的 所在节点的密码；
  > - 第 9 行：``host_single_info.txt`` 文件绝对路径； 
  > - 第 10 行：修复过程中的日志输出目录；

### 4.2 执行修复

完成上述步骤后，执行如下命令即可开始一键修复任务。
~~~shell
cd /opt/usdp-srv/repair
sh repair.sh initAll <your.properties 文件的绝对路径>
source /etc/profile
~~~

修复过程为完全离线的方式，等待一段时间后，即可将所有对应节点的环境准备完毕。



## 5. 启动 USDP

### 5.1 为USDP配置MySQL数据库

修改`/opt/usdp-srv/usdp/config/application-server.yml`文件，找到 `datasource` 配置片段，修改前如下：

```shell
datasource:
    type: com.zaxxer.hikari.HikariDataSource
    #    driver-class-name: org.gjt.mm.mysql.Driver
    driver-class-name: com.p6spy.engine.spy.P6SpyDriver
    url: jdbc:p6spy:mysql://udp01:3306/db_udp?useUnicode=true&characterEncoding=utf-8&useSSL=false
    username: root
    password: 1qaz!QAZ
```

修改url 中的udp01及 password 的值，修改后如下：

```shell
datasource:
    type: com.zaxxer.hikari.HikariDataSource
    #    driver-class-name: org.gjt.mm.mysql.Driver
    driver-class-name: com.p6spy.engine.spy.P6SpyDriver
    url: jdbc:p6spy:mysql://pusdp_master1:3306/db_udp?useUnicode=true&characterEncoding=utf-8&useSSL=false
    username: root
    password: ucloud.cn
```

### 5.2 启动 USDP

此时，进入 ``/opt/usdp-srv/usdp`` 目录后，执行如下命令，即可启动 USDP-Server：

~~~shell
bin/start-udp-server.sh 
~~~

大约等待 10 秒左右，即可通过浏览器访问 USDP：http://<your_host_ip>

`提示：<your_host_ip> 为启动 USDP-Server 的节点 IP 地址，如果为内网，则需要自行搭建 VPN，或通过内网互通的Windows机器的浏览器来访问该IP。`



## 6. 构建大数据集群

启动 USDP 服务之后，即可通过 WebUI 的方式，进行图形化界面操作，方便简单的完成您所需要的大数据集群的构建工作。

后续操作可以参看如何 [通过USDP创建第一个大数据集群](/usdpdc/1.0.x/plan&create/first_create)。



## 7. 新增节点前的修复步骤

当集群部署完毕后，如需要新增节点，同样需要准备好部署环境，对于新增节点环境修复，我们也提供一键式修复，需要做如下步骤：

### 7.1 检查your.properties 配置的信息

检查 your.properties 文件的参数项 host.single.info.Path，即新增节点信息存放的绝对路径。对于该配置文件其他参数项无需改动。

     host.single.info.Path=/opt/usdp-srv/usdp/repair/host_single_info.txt

### 7.2 配置host_single_info.txt文件

与4.1节中的全量修复类似， 文件中每行为一个节点信息，从左至右依次为：内网IP，节点密码，SSH端口号，即将自动修改生效的主机名。具体示例如下：

~~~shell
      127.0.0.1 your-node-root-password 22 udp01
      127.0.0.1 your-node-root-password 22 udp02
      127.0.0.1 your-node-root-password 22 udp03
~~~
修改完上述配置文件，即可进入 repair 目录执行如下修复命令


      bash repair.sh  initSingle  /opt/usdp-srv/usdp/repair/your.properties

> 注意：
>
> 1. 在host_single_info.txt文件中，仅需配置每次新增的节点信息即可，若存在已修复过的节点信息时，在下次运行“repair.sh  initSingle”指令前，请清除。
> 2. jdk 安装在 /opt/module 下面，不允许随意删除，否则 java 环境失效。