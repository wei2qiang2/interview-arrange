安装virtulabox

安装vagrant

注意两者之间的版本是否匹配,不匹配需要修改配置文件，详见

[修改配置文件]: https://blog.csdn.net/github_38336924/article/details/103832157

下载centos的virtualbox

`<http://www.vagrantbox.es/>` 在这个地址找到需要的box镜像地址，用第三方软件下载或者百度云下载

添加box

`vagrant add boxname`(上个步骤下载的box) --name（指定名称） 

2.2.9添加box命令

`vagrant box add CentOS-7-x86_64-Vagrant-1902_01.VirtualBox.box --name centos_1`

2.2.9移除box命令

`vagrant box remove centos_1`

查看box列表

`vagrant box list`

初始化

`vagrant init boxname` 生成Vagrantfile配置文件

启动

`vagrant up` 在有Vagrantfile文件下的目录下执行命令

建立ssh连接

`vagrant ssh`

安装网络工具（ifconfig命令）

`sudo yum install net-tools`

修改网络配置（自带的是端口转发，相当于主机端口映射到虚拟机端口），修改Vagrantfile文件

在本机上查看virtualbox的ip

```powershell
以太网适配器 VirtualBox Host-Only Network:

   连接特定的 DNS 后缀 . . . . . . . :
   本地链接 IPv6 地址. . . . . . . . : fe80::1982:ad9d:4c2c:f572%12
   IPv4 地址 . . . . . . . . . . . . : 192.168.56.1
   子网掩码  . . . . . . . . . . . . : 255.255.255.0
```

修改配置文件

```shell
# config.vm.network "private_network", ip: "192.168.33.10"
config.vm.network "private_network", ip: "192.168.56.10
```

重新启动

`vagrant reload`

连接虚拟机

`vagrant ssh`

查看虚拟机ip

`ifconfig`

ping 主机