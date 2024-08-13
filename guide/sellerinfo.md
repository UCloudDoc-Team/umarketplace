<a name="Odrj7"></a>
# 服务商相关
<a name="dMPpy"></a>
## 入驻流程
![image.png](/images/%E5%85%A5%E9%A9%BB%E6%B5%81%E7%A8%8B.png)
<a name="NfMY5"></a>
<a name="ftkKc"></a>
## 入驻步骤
<a name="jTqEL"></a>
### 1. 商务洽谈
可通过工单、在线咨询等方式联系UCloud ，洽谈合作事宜。
<a name="oKk0Z"></a>
### 2. 签署协议
确认合作模式、合作条款后，正式签订合同，确立合作关系。
<a name="tQAiF"></a>
### 3. 注册 UCloud 云平台账号
注册UCloud 云平台账号，详情请参见[UCloud 账号注册流程](https://docs.ucloud.cn/register/register_flow)。
<a name="tCoyK"></a>
### 4. 制作产品镜像
<a name="ZlMUQ"></a>
请根据制作镜像的环境不同，选择相应的指导文档。<br />

#### 4.1 基于云平台制作 Linux 镜像
##### step1 创建主机
在控制台 [console.ucloud.cn](https://console.ucloud.cn/uhost/uhost?hpc=false&gpu=false) 创建主机，主机选择不要挂载数据盘。

##### step2 进入主机部署业务
对于从 UCloud 控制台上的Linux基础镜像制作镜像，为了保证镜像的正常使用，在部署自己业务时需要满足以下要求：
- 安装监控 Agent ，详情请参见 [监控代理](https://docs.ucloud.cn/umon/agent) 。 
- 检查 auditd 进程正常运行。auditd是Linux 审计框架（Linux Audit Framework）的一部分，用于实现系统审计功能。auditd进程负责收集、存储和分析系统的各种事件和日志，以便管理员监视系统的活动并检查安全性问题。
```
ps -ef|grep auditd|grep -v  grep |grep -v '\['
```

- 不可删除cloud-init软件包及其相关服务。
- 对于Ubuntu22.04及以后的版本不可以开启firewalld服务。
- 不要在`/data`目录中存放文件，该目录为系统盘挂载点，如果有文件的话，会导致新建的机器的数据盘没法自动挂载。
- 如果系统中存在Python，不可以删除或升级及系统上Python版本，如果要安装其他版本的Python话，不要修改python或者python3的软连接。
- 业务不要强依赖`IP`/`主机名`/`mac地址`等动态信息，因为这些信息在创建新主机时会进行更新。
- 不建议修改以下文件的内容(如果文件不存在可以忽略)：
   - /etc/modprobe.d/blacklist-nouv.conf
   - /etc/modprobe.d/mlx5.conf
   - /boot/grub2/grub.cfg
   - /etc/default/grub
   - /usr/lib/systemd/network/99-default.link
   - /usr/lib/NetworkManager/conf.d/00-server.conf

##### step3 清理系统
- 执行`cloud-init clean`命令清理本实例的初始化数据。
- 执行`rm ``-f /usr/local/ucl``oud/.cache/metadata.json`，清理`cloud-init`缓存的数据。
- 检查`/etc/hosts`中的内容，清理主机名。
- 根据自己业务的需要，清理系统的日志以及操作记录等。
<a name="ybRrB"></a>

#### 4.2 基于云平台制作 Windows 镜像
##### step1 创建主机
在控制台 [console.ucloud.cn](https://console.ucloud.cn/uhost/uhost?hpc=false&gpu=false) 创建主机，主机选择不要挂载数据盘。<br />

##### step2 进入主机部署业务
对于从 UCloud 控制台上的Windows基础镜像制作镜像，为了保证镜像的正常使用，在部署自己业务时需要满足以下要求：
- 安装监控 Agent ，详情请参见 [监控代理](https://docs.ucloud.cn/umon/agent) 。 
- 不可删除cloudbase-init软件包及其相关服务(如果没有的话可以忽略)。
- 业务不要强依赖`IP`/`主机名`/`mac地址`等动态信息，因为这些信息在创建新主机时会进行更新。
- 不要修改系统`virtio`驱动的版本，除非已经明确该版本存在问题。
- 不建议修改以下文件夹中的内容(如果不存在可以忽略)：
   - C:\Program Files\Cloudbase Solutions

##### step3 清理系统
- 清理`Cloudbase-init`相关(如果不存在可以忽略),在`PowerShell`中执行
```
Remove-Item -Path "HKLM:\SOFTWARE\Cloudbase Solutions\Cloudbase-Init" -Recurse -Force
Set-Content -Path "C:\Program Files\Cloudbase Solutions\Cloudbase-Init\log\cloudbase-init.log" -Value ""
```

- 根据自己业务的需要，清理系统的日志、事件、更新缓存、操作记录等。
<a name="pVQ84"></a>

#### 4.3 本地制作 Linux 镜像要求
##### 磁盘要求
您在制作云市场镜像过程中对磁盘分区时，需满足如下要求：

- `SWAP`分区 制作镜像时不要使用`SWAP`分区（交换分区）。
- 磁盘大小 系统磁盘最小大小设置为20 GiB。

如果您是使用本地的`KVM`虚拟机来制作镜像，建议虚拟机使用以下配置：

- 磁盘总线 使用`virtio`驱动作为磁盘的总线。
- 磁盘格式 使用`Qcow2`格式作为磁盘格式。

##### 必备的软件和工具
- `virtio`驱动<br />
  确保系统已经包含virtio驱动，可以使用命令`lsmod | grep virtio`来判断是是否加载了该模块。
- `net_failover`<br />
  如果要制作快杰镜像需要内核要等于或者大于`4.19`，确保含有`net_failover`驱动，可以使用命令`lsmod | grep net_failover` 来判断是是否加载了该模块。
- `tzdata-legacy`<br />
  如果是`Ubuntu24.04`系统，需要安装`tzdata-legacy`来添加旧的时区信息到系统中。
- `cloud-init`<br />
  您在制作云市场镜像时需要安装`cloud-init`，以保证运行该镜像的实例能成功完成初始化配置。`cloud-init` 不可以使用社区版，需要使用`UCloud版本`，[cloudbase-init](/umarketplace/guide/sellerinfo.md#cloud-init下载链接)。


##### 系统配置
- 关闭`Firewalld`服务
> 该项在Ubuntu22.04和Ubuntu24.04系统中是必做的
```
systemctl stop firewalld
systemctl disable firewalld
```

- 配置网络

如果是使用NetworkManager来管理网络需要进行如下配置。
```
cat > /usr/lib/NetworkManager/conf.d/00-server.conf << EOF
This configuration file changes NetworkManager's behavior to
what's expected on "traditional UNIX server" type deployments.
See "man NetworkManager.conf" for more information about these
and other keys.
[main]
Do not do automatic (DHCP/SLAAC) configuration on ethernet devices
with no other matching connections.
no-auto-default=*
Ignore the carrier (cable plugged in) state when attempting to
activate static-IP connections.
ignore-carrier=*
EOF
```
如果系统中含有`/usr/lib/systemd/network/99-default.link`文件，还需要执行下面的命令
```
sed -i 's/^AlternativeNamesPolicy/# AlternativeNamesPolicy/' /usr/lib/systemd/network/99-default.link
```

- 配置启动文件
```
sed -i '/^GRUB_TIMEOUT_STYLE/d' /etc/default/grub
sed -i 's/^GRUB_TIMEOUT=.*/GRUB_TIMEOUT=5/' /etc/default/grub  
sed -i 's/^GRUB_CMDLINE_LINUX=.*/GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0 console=tty1 console=ttyS0,115200n8"/' /etc/default/grub
```
`centos`/`redhat`/`rocky`执行:
```
grub2-mkconfig -o /boot/grub2/grub.cfg
```
`debian`/`ubuntu`执行:
```
update-grub2
update-initramfs -u
```

- 配置启动时的内核模块
```
cat > /etc/modprobe.d/blacklist-nouv.conf << EOF
blacklist nouveau
options nouveau modeset=0
EOFecho 'softdep mlx5_core pre: virtio_net' > /etc/modprobe.d/mlx5.conf
```

- 检查 auditd 进程正常运行
```
ps -ef|grep auditd|grep -v  grep |grep -v '\['
```

##### 清理系统
>下面的文件有可能不存在，如果不存在可以忽略
```
rm -f /etc/default/grub.d/50-cloudimg-settings.cfg 
rm -f /etc/netplan/50-cloud-init.yaml 
rm -f /etc/sysctl.d/99-cloudimg-ipv6.conf
```
发布镜像前，您可以根据自己本身情况进行清理历史记录以及日志等操作。<br />

##### cloud-init下载链接

| OS | package |
| --- | --- |
| **Rocky 9** | [cloud-init-23.4-7.el9.5.0.1.ucloud.noarch.rpm](https://uhost.cn-bj.ufileos.com/cloud-init/Rocky9.x/cloud-init-23.4-7.el9.5.0.1.ucloud.noarch.rpm) |
| **Rocky 8** | [cloud-init-23.4-7.el8.3.0.1.ucloud.noarch.rpm](https://uhost.cn-bj.ufileos.com/cloud-init/Rocky8.x/cloud-init-23.4-7.el8.3.0.1.ucloud.noarch.rpm) |
| **Redhat 8** | [cloud-init-23.4-7.el8.3.ucloud.noarch.rpm](https://uhost.cn-bj.ufileos.com/cloud-init/Redhat8/cloud-init-23.4-7.el8.3.ucloud.noarch.rpm) |
| **Ubuntu 20.04** | [cloud-init_24.1.3-1ubuntu1~20.04.5ucloud_all.deb](https://uhost.cn-bj.ufileos.com/cloud-init/Ubuntu20.04/cloud-init_24.1.3-1ubuntu1~20.04.5ucloud_all.deb) |
| **Ubuntu 22.04** | [cloud-init_24.1.3-1ubuntu2~22.04.5ucloud_all.deb](https://uhost.cn-bj.ufileos.com/cloud-init/Ubuntu22.04/cloud-init_24.1.3-1ubuntu2~22.04.5ucloud_all.deb) |
| **Ubuntu 24.04** | [cloud-init_24.1.3-0ubuntu3.3ucloud_all.deb](https://uhost.cn-bj.ufileos.com/cloud-init/Ubuntu24.04/cloud-init_24.1.3-0ubuntu3.3ucloud_all.deb) |
| **CentOS Stream 9** | [cloud-init-23.4-17.el9.ucloud.noarch.rpm](https://uhost.cn-bj.ufileos.com/cloud-init/centos-stream9/cloud-init-23.4-17.el9.ucloud.noarch.rpm) |
| **CentOS 8.x** | [cloud-init-22.1-8.el8.ucloud.noarch.rpm](https://uhost.cn-bj.ufileos.com/cloud-init/CentOS8.x/cloud-init-22.1-8.el8.ucloud.noarch.rpm) |
| **CentOS 7.x** | [cloud-init-19.4-7.el7.7.ucloud.x86_64.rpm](https://uhost.cn-bj.ufileos.com/cloud-init/CentOS7/cloud-init-19.4-7.el7.7.ucloud.x86_64.rpm) |
| **CentOS 6.x** | [cloud-init-19.4+11.g158c661c-1.el6.noarch.rpm](https://uhost.cn-bj.ufileos.com/cloud-init/CentOS6/cloud-init-19.4%2B11.g158c661c-1.el6.noarch.rpm) |

<a name="qPirL"></a>

#### 4.4 本地制作 Windows 镜像要求
##### 磁盘要求
您在制作云市场镜像过程中对磁盘分区时，需满足如下要求。

- 启动模式<br />
  建议使用BIOS模式进行启动，对于UEFI启动模式的镜像需要联系技术支持团队进行特殊处理。
- 磁盘分区<br />
  对于启动分区，如果有必须位于磁盘最前端，磁盘尾部为windows系统分区(C盘)，如果在系统分区后面还有其他分区需要全部删除，并且将系统分区拓展到尾部。
- 磁盘大小<br />
  系统磁盘最小大小设置为40 GiB或者以上。

如果您是使用本地的`KVM`虚拟机来制作镜像，建议虚拟机使用以下配置：

- 磁盘总线 使用`virtio`驱动作为磁盘的总线。
- 磁盘格式 使用`Qcow2`格式作为磁盘格式。

##### 必备的软件和工具

- `virtio`驱动<br />
  `windows`必须安装`virtio`驱动，其中网络和磁盘的驱动必须安装，`virtio` 驱动可以从 [fedorapeople](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/) 上进行下载，建议`Windows Server 2012`及以下使用`0.1.164`版本，`Windows Server 2012`以上使用`0.1.229`版本。
- `cloudbase-init`<br />
  `Windows Server 2016`及以上可以使用`cloudbase-init` 来进行初始化，下载地址：[cloudbase-init](/umarketplace/guide/sellerinfo.md#cloud-init下载链接)。

##### 系统配置

- `BootScript`<br />
  对于没有安装`cloudbase-init` 的系统，需要使用预埋启动脚本的形式来初始化系统，在`运行`-->`gpedit.msc`-->`计算机配置`-->`Windows设置`-->`脚本`-->`启动`-->`添加` 中的脚本名中添加`C:\bootstrap.bat`。
- 远程桌面<br />
  开启系统的远程桌面功能。
- 防火墙<br />
  建议关闭所有防火墙。
- 安装安全补丁更新<br />
  建议您安装最新的安全补丁。
- 账户策略<br />
  查看`控制面板`->`系统和安全`->`管理工具`->`本地安全策略`->`账户策略`->`账户锁定策略`->`账户锁定阀值` 如果该值不为0的话，需要将该值设置为0。

##### 清理镜像信息
为了提高系统的安全性，在镜像发布前，建议您清理镜像制作过程中的日志、历史记录、残留文件等，尽可能减小镜像的大小。
- 清理浏览器记录。
- 清理Windows更新相关日志。
- 清理事件记录。
- 清理系统日志。
- 清理临时文件。
- 清理Windows更新产生的临时文件和日志文件。
<a name="RkVtB"></a>
### 5. 提供产品信息
| 类型 | 内容 |
| --- | --- |
| 产品 | 镜像ID<br />账号邮箱<br />提供产品服务的相关端口号<br /> |
| 产品介绍文档 | 产品名称<br />产品简介<br />产品激活方式<br />售后联系方式及服务时间，仅对第三方提供售后的场景<br />产品使用文档<br />产品介绍文档<br />产品图标：svg格式<br /> |

<a name="Av2Ku"></a>
### 6. 等待UCloud审核、配置镜像产品
UCloud 基于服务商提供的产品制作镜像，进行安全审核，并上架 UCloud 云市场。
<a name="KalUt"></a>
### 7. 校对上线产品
产品上线后，您可在云平台检查镜像产品名称、配置、文档等信息是否有误，如有问题可联系UCloud 调整。
<a name="K7QVq"></a>
### 8. 完成入驻
当您在 UCloud 云市场看到镜像产品确认没有问题，即入驻完成。
