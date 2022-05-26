

## 													使用vagrant快速搭建虚拟机

### 一、准备工作

```
1) 下载VirtualBox			https://www.virtualbox.org/
2) 下载vagrant 			https://www.vagrantup.com/
3) vagrant 官网 下载centos box镜像 也可前往 centos官网进行下载
```

> virtualBox 安装完后如下 会生成网卡 后续使用此网卡进行通信

![image-20220526125358714](https://raw.githubusercontent.com/lilu188011/img-repo/image-20220526125358714.png?raw=true)

> vagrant 官网镜像库

![image-20220526125706619](https://raw.githubusercontent.com/lilu188011/img-repo/image-20220526125706619.png?raw=true)

### 二、 开始安装

#### 	1. 准备好刚刚下载box文件  Vagrantfile

​			![](https://raw.githubusercontent.com/lilu188011/img-repo/image-20220526130024762.png?raw=true)

#### 	2.这里贴上Vagrantfile 文件(如果只装一台 对应boxes数组调整)

```
# -*- mode: ruby -*-
# vi: set ft=ruby :
Encoding.default_external = 'UTF-8'
boxes = [
	{
		:name => "master",
		:eth1 => "192.168.56.100",
		:mem => "2048",
		:cpu => "2"
	
	},
	{
		:name => "node1",
		:eth1 => "192.168.56.101",
		:mem => "2048",
		:cpu => "2"
	
	},
	{
		:name => "node2",
		:eth1 => "192.168.56.102",
		:mem => "2048",
		:cpu => "2"
	
	}
] 
Vagrant.configure("2") do |config|
  config.vm.box = "centos7"
	boxes.each do |opts|
		config.vm.define opts[:name] do |config|
			config.vm.hostname = opts[:name]
			config.vm.provider "vmware_fusion" do |v|
				v.vmx["memsize"] = opts[:mem]
				v.vmx["numvcpus"] = opts[:cpu]
			end
			config.vm.provider "virtualbox" do |v|
				v.customize ["modifyvm", :id, "--memory", opts[:mem]]
				v.customize ["modifyvm", :id, "--cpus", opts[:cpu]]
				v.customize ["modifyvm", :id, "--name", opts[:name]]
			end	

			config.vm.network :private_network, ip: opts[:eth1]
		end	
	end				
end

```

#### 3.vagrant 安装box

```
1) 添加本地镜像库
	vagrant box add centos7 ./virtubox.box
2) 查看镜像列表
	vagrant box list
3) 根据Vagrantfile 启动创建容器
	vagrant up		
```

### 三、 注意事项

#### 	一、Vagrantfile  boxes ip指定 需与 VirtualBox 生成的网卡ip在同一网段

####     二、 安装多台会导致生成虚拟机网卡地址一样  需进入virtualBox进行设置 如下

##### 	**1)  全局创建nat网络**

​			![image-20220526131109135](https://raw.githubusercontent.com/lilu188011/img-repo/image-20220526131109135.png?raw=true)

##### 2） 为虚拟机应用上刚刚创建的nat网络  刷新网卡地址

![image-20220526131441506](https://raw.githubusercontent.com/lilu188011/img-repo/image-20220526131441506.png?raw=true)

##### 3） 多机更改完后 建议以后从virtualbox 启动 不通过命令行 vagrant无法感知修改



### 四、 安装完成连接虚拟、开启远程连接

#### 		1）查看当前虚拟机

​					![image-20220526132029093](https://raw.githubusercontent.com/lilu188011/img-repo/image-20220526132029093.png?raw=true)

#### 		2)  连接虚拟机	

```
1.查看默认账号
	vagrant ssh-config
	关注:Hostname  Port  IdentityFile
	IP:127.0.0.1
	port:2222
	用户名:vagrant
	密码:vagrant
	文件:Identityfile指向的文件private-key
2.连接虚拟机
	vagrant ssh id/name 
3.切换root用户 默认密码 vagrant 开启远程连接
	su - root
	vi /etc/ssh/sshd_config
	修改PasswordAuthentication yes
	systemctl restart sshd
```



### 四、 附录 vagrant 常用命令

```
vagrant up                                               启动虚拟机

vagrant halt                                             关闭虚拟机

vagrant reload                                         重启虚拟机

vagrant reload --provision                       	重启虚拟机并更新相关配置

vagrant ssh                                             通过 SSH 登录

vagrantvagrant box list                            查看目前安装的box列表

vagrant box remove laravel/homestead  删除box镜像

vagrant destroy                                         删除虚拟机

vagrant status                                          查看当前 Homestead 虚拟机的状态

vagrant global-status  								查看当前所有虚拟机
```

