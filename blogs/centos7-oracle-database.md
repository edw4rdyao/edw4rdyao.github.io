# Centos7命令行静默安装Oracle19c

## 本人配置

**腾讯云轻量级服务器** `CPU`: `2`核 内存: `4GB` 系统：`CentOS 7.6` `80GB SSD`云硬盘

## 安装包下载与传输

进入[Oracle官网](https://www.oracle.com/database/technologies/oracle-database-software-downloads.html#19c)，选择`19.3 - Enterprise Edition (also includes Standard Edition 2)`，选择**`Linux x86-64 .ZIP`**版本。

使用Xftp方式传输文件至服务器，拷贝安装包`LINUX.X64_193xxx_db_home.zip`到`/opt`目录下。

XShell与Xftp已经为个人和家庭提供了免费版，进入[XShell官网](https://www.netsarang.com/zh/free-for-home-school/)，填写邮箱即可。

## 系统环境准备与配置

**1. 关闭防火墙**

```shell
$ systemctl  stop firewalld
$ systemctl  disable firewalld
```

**2. 关闭selinux**

```shell
$ vim /etc/selinux/config
# 修改 SELINUX=disabled
# 立即生效修改：
$ setenforce 0
# 查看是否生效
$ getenforce 
```

**3. 检查硬件**

- `RAM`：`>2G`(官方要求内存大于`2G`)
- `/tmp`目录：至少`1G`
- `SWAP`：**`RAM\*1.5`** `(RAM 1G\~2G);` **`=RAM`** `(RAM 2G\~16G);` **`16G`** `(RAM>16G)`

```shell
$ grep MemTotal /proc/meminfo
$ grep SwapTotal /proc/meminfo
$ df -h /tmp
$ free -h
$ uname -m
$ df -h /dev/shm
```

**4. Swap创建(如果swap区不够或者没有)**

```shell
# root用户下进行

# 创建8G大小的swap区
$ dd if=/dev/zero of=/myswapfile bs=1M count=8192
$ mkswap /myswapfile
$ swapon /myswapfile

# 在/etc/fstab文件末尾添加以下内容:
# /myswapfile    swap    swap    defaults    0    0
$ vim /etc/fstab

# 确认swap已经被使用，另外在/目录可以看到文件swapfile
$ swapon -s
```

**5. 修改系统版本，绕过安装时的系统检查**

```shell
$ vim /etc/centos-release
# 将原来的内容: CentOS Linux release 7.6.XXXX (Core) 改为：redhat-7
```

**6. hostname**

```shell
$ hostname
# 如果设置了hostname则添加主机名和IP保证hostname能ping通，没有设置hostname则跳过
$ vim /etc/hosts
# 添加 xxx.xxx.x.xxx(your ip) yourhostname
```

## 安装依赖

```shell
$ yum install -y bc binutils compat-libcap1 compat-libstdc++-33 elfutils-libelf elfutils-libelf-devel fontconfig-devel glibc glibc-devel ksh libaio libaio-devel libX11 libXau libXi libXtst libXrender libXrender-devel libgcc libstdc++ libstdc++-devel libxcb make smartmontools sysstat ipmiutil net-tools nfs-utils python python-configshell python-rtslib python-six targetcli 

$ rpm -qa bc binutils compat-libcap1 compat-libstdc++-33 elfutils-libelf elfutils-libelf-devel fontconfig-devel glibc glibc-devel ksh libaio libaio-devel libX11 libXau libXi libXtst libXrender libXrender-devel libgcc libstdc++ libstdc++-devel libxcb make smartmontools sysstat ipmiutil net-tools nfs-utils python python-configshell python-rtslib python-six targetcli |wc
```
总共33个包，其中这部分包非必须的依赖包，根据自己的实际情况也可以不安装

- `ipmiutil (for Intelligent Platform Management Interface)`
- `net-tools (for Oracle RAC and Oracle Clusterware)`
- `nfs-utils (for Oracle ACFS)`
- `python (for Oracle ACFS Remote)`
- `python-configshell (for Oracle ACFS Remote)`
- `python-rtslib (for Oracle ACFS Remote)`
- `python-six (for Oracle ACFS Remote)`
- `targetcli (for Oracle ACFS Remote)`

```shell
# 如果有缺少的包，单独处理
$ yum install -y bc binutils compat-libcap1 compat-libstdc++-33 elfutils-libelf elfutils-libelf-devel fontconfig-devel glibc glibc-devel ksh libaio libaio-devel libX11 libXau libXi libXtst libXrender libXrender-devel libgcc libstdc++ libstdc++-devel libxcb make smartmontools sysstat ipmiutil net-tools nfs-utils python python-configshell python-rtslib python-six targetcli 
No package ipmiutil available
```

不同的`OS`及`OS`版本、不同的`Oracle`版本，所需要的依赖包不同，详情可参考`Oracle`[官网帮助文档](https://docs.oracle.com/en/database/oracle/oracle-database/index.html)（`Oracle Database Documentation`):

选择对应的`Oracle`版本 -> 查看帮助信息：`Install and Upgrade -> Linux Installation Guides`

`RAC`参考：`Grid Infrastructure Installation and Upgrade Guide for Linux`

`DB`参考： `Database Installation Guide for Linux`

## 内核优化

```shell
$ vim /etc/sysctl.conf
```

将内容按照下列提示修改：

```shell
net.ipv4.icmp_echo_ignore_broadcasts = 1
net.ipv4.conf.all.rp_filter = 1
# 设置最大打开文件数
fs.file-max = 6815744
fs.aio-max-nr = 1048576
# 共享内存的页数，Linux共享内存页大小为4KB，4G内存按照官方设置为内存的1/2
# 我物理内存4G，设置为2G：2*1024*1024*1024/4K=(kernel.shmmax/4k)=524288
kernel.shmall = 524288
# 最大共享内存，官方文档建议是内存的1/2，我物理内存4G，设置为4G：4*1024*1024*1024 = 2147483648
kernel.shmmax = 2147483648
kernel.shmmni = 4096
kernel.sem = 250 32000 100 128
# tcp参数设置
# 可使用的IPv4端口范围(TCP/UDP协议允许使用的本地端口号)
net.ipv4.ip_local_port_range = 9000 65500
# 默认&最大的TCP数据接收窗口大小（字节）
net.core.rmem_default = 262144
net.core.rmem_max= 4194304
# 默认&最大的TCP数据发送窗口大小（字节）
net.core.wmem_default= 262144
net.core.wmem_max= 1048576
```

```shell
# 使内核参数立即生效，执行
$ sysctl -p  
# 或者 sysctl --system

# 修改oracle用户限制，提高软件运行性能，在/etc/security/limits.conf文件末尾添加配置项
$ vim /etc/security/limits.conf
# 加入：
# oracle           soft    nproc           2047
# oracle           hard    nproc           16384
# oracle           soft    nofile          1024
# oracle           hard    nofile          65536

```

## 开始安装数据库

>创建 oracle 目录，授权，cd到oracle安装包路径下，解压oracle到 $ORACLE_HOME 目录下，然后执行 runInstaller 安装

**1. 创建用户与目录**

```shell
# 创建Oracle相关用户和组
groupadd -g 54321 oinstall
groupadd -g 54322 dba
groupadd -g 54323 oper  #oper组非必须，也可以不创建
groupadd -g 54324 backupdba
groupadd -g 54325 dgdba
groupadd -g 54326 kmdba
groupadd -g 54327 asmdba
groupadd -g 54328 asmoper
groupadd -g 54330 racdba

# 创建oracle用户
useradd -u 54321 -g oinstall -G dba,asmdba,backupdba,dgdba,kmdba,racdba oracle

# 设置Oracle用户的密码
passwd oracle

# 创建目录
mkdir -p /opt/oracle
mkdir -p /opt/oraInventory
mkdir -p /opt/database
mkdir -p /opt/oracle/product/19.3.0
mkdir -p /opt/oracle/oradata
mkdir -p /opt/oracle/flash_recovery_area
mkdir -p /opt/oracle/product/19.3.0/db_1   #从18c开始，安装包必须解压到 $ORACLE_HOME 路径下进行安装!

chown -R oracle:oinstall /opt/oracle
chown -R oracle:oinstall /opt/oracle/oradata
chown -R oracle:oinstall /opt/oraInventory
chown -R oracle:oinstall /opt/database
chmod -R 775 /opt/oracle
```

**2. 授权**

```shell
# 配置Oracle用户的环境变量
$ su - oracle
# 添加环境变量配置内容
$ vim .bash_profile 
```

内容如下:

```shell
umask 022
#oracle数据库安装目录
export ORACLE_BASE=/opt/oracle
#oracle数据库路径
export ORACLE_HOME=$ORACLE_BASE/product/19.3.0/db_1
#oracle启动数据库实例名
export ORACLE_SID=orcl
#xterm窗口模式安装
export ORACLE_TERM=xterm
#配置时间格式
NLS_DATE_FORMAT="YYYY:MM:DDHH24:MI:SS"
#添加系统环境变量
export PATH=$ORACLE_HOME/bin:/usr/sbin:$PATH
#添加系统环境变量
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib
#防止安装过程出现乱码
#export LANG=en_US.gbk
export LANG=en_US.UTF-8
#设置Oracle客户端字符集，必须与Oracle安装时设置的字符集保持一致，
#export NLS_LANG=AMERICAN_AMERICA.ZHS16GBK
export NLS_LANG=AMERICAN_AMERICA.UTF8
```

```shell
# 生效环境变量
# 重新登录或者执行source 立即生效
$ source  /home/oracle/.bash_profile

# 检查环境变量是否生效：
$ echo $ORACLE_HOME
```

**3. 解压**

```shell
$ su - oracle
$ cd /opt/
$ unzip -q LINUX.X64_193000_db_home.zip -d $ORACLE_HOME
```

**4. 修改响应文件`db_install.rsp`**

统一复制响应文件到`/home/oracle/`下面，然后再授权和修改响应文件

`Oracle 19c`解压后`response`目录下，只包含` db_install.rsp` (用来安装`Oracle`软件），`dbca.rsp` 在 `$ORACLE_HOME/assistants/dbca/dbca.rsp`

```shell
$ cp -r $ORACLE_HOME/install/response  /home/oracle
$ vim /home/oracle/response/db_install.rsp    
# 修改设置下列参数
```

```shell
oracle.install.option=INSTALL_DB_SWONLY
# ORACLE_HOSTNAME=localhost  # 如果有设置过hostname则需要设置正确，否则可以不设置
UNIX_GROUP_NAME=oinstall
INVENTORY_LOCATION=/opt/oraInventory
ORACLE_BASE=/opt/oracle
ORACLE_HOME=/opt/oracle/product/19.3.0/db_1
oracle.install.db.InstallEdition=EE
oracle.install.db.OSDBA_GROUP=dba
oracle.install.db.OSOPER_GROUP=oinstall
oracle.install.db.OSBACKUPDBA_GROUP=backupdba
oracle.install.db.OSDGDBA_GROUP=dgdba
oracle.install.db.OSKMDBA_GROUP=kmdba
oracle.install.db.OSRACDBA_GROUP=racdba
oracle.install.db.rootconfig.executeRootScript=true
oracle.install.db.rootconfig.configMethod=ROOT
```

**5. 开始静默安装**

```shell
$ $ORACLE_HOME/runInstaller -silent -ignorePrereq -responseFile /home/oracle/response/db_install.rsp 
```

安装会提示安装的日志文件，可以通过日志文件查看静默安装的进展和信息:

```
You can find the log of this install session at:
/tmp/InstallActions2020-06-21_08-52-09PM/installActions2020-06-21_08-52-09PM.log
```

安装成功，提示信息如下：

```
Successfully Setup Software with warning(s).
Moved the install session logs to:
/opt/oraInventory/logs/InstallActions20xx-xx-xx_xx-xx-xxPM
```

```shell
$ cd /opt/oraInventory/logs/InstallActions20xx-xx-xx_xx-xx-xxPM
$ ls -l

# 安装完毕后，启动监听
$ su - oracle
$ lsnrctl start

# 查看默认监听端口1521的监听状态
$ netstat -an |grep 1521
```

## 创建数据库

```shell
$ cp $ORACLE_HOME/assistants/dbca/dbca.rsp /home/oracle/response/
$ vim /home/oracle/response/dbca.rsp
```

设置以下参数:

```shell
gdbName=orcl
sid=orcl
sysPassword=1q2w3e
systemPassword=1q2w3e
dbsnmpPassword=1q2w3e
# datafileDestination 为所有数据文件的目标位置，如果不设置默认为 $ORACLE_BASE/oradata
datafileDestination=/opt/oracle/oradata
# recoveryAreaDestination 为恢复区位置，如果不设置默认为 $ORACLE_BASE/flash_recovery_area
recoveryAreaDestination=/opt/oracle/flash_recovery_area
characterSet=AL32UTF8
# totalMemory 为2046MB，物理内存4G*50%，我设置为2G = 2048，注意：如果设置超过 kernel.shmmax 建库时会报错
totalMemory=2048
databaseType=OLTP
# Oracle自带的默认建库模板 General_Purpose.dbc，文件位于 $Oracle_HOME/assistants/dbca/templates/General_Purpose.dbc，这里设置等同于 dbca 加参数 -templateName General_Purpose.dbc 
templateName=General_Purpose.dbc
```

```shell
# 使用dbca.rsp静默建库（注意：#responseFile必须使用绝对路径）
$ dbca -silent -createDatabase -responseFile /home/oracle/response/dbca.rsp
# 注意：如果建库时提示 SYS 和 SYSTEM 密码太短，直接忽略该告警
```

建库成功👇

```
安装成功（建库大概分钟级别）：
...
60% complete
Completing Database Creation
66% complete
69% complete
70% complete
Executing Post Configuration Actions
100% complete
Database creation complete. For details check the logfiles at:
 /opt/oracle/cfgtoollogs/dbca/orcl.
Database Information:
Global Database Name:orcl
System Identifier(SID):orcl
Look at the log file "/opt/oracle/cfgtoollogs/dbca/orcl/orcl2.log" for further details.
```

```shell
# 登录查看实例状态：
$ sqlplus / as sysdba
# Oracle数据库操作...略
```

## 参考博客

[blog.csdn.net/sunny05296/article/details/56840775](https://blog.csdn.net/sunny05296/article/details/56840775)
