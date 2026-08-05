## 梳理 

1. 基础概念
	1. 为什么需要Ansible
		1. 开过虚拟机k8s集群的同学都知道，每台手动安装有多麻烦。而Ansible不需要多余安装，只要SSH就行
	2. Agentless 框架
	3. Control Node
	4. Managed Node
		- Ansible基于SSH通信，由Control Node主动管理 Managed Node，不需要在被管理机器安装Agent
	5. Inventory
	6. SSH通信
	7. 什么叫幂等性？：同一个任务执行一次和执行多次，最终结果一致

2. 安装与环境
	1. Ansible 安装
	2. SSH 免密登陆
	3. host 文件（这个有点像分组）
	4. ansible.cfg[^1]

3. Ad-hoc  单条命令
	1. ping    
		1. 格式 `ansible 主机组 -m 模块  -a  参数` 
		2. `ansible all -m ping`    `ansible aaa -m yum -a "name=nginx  state=present"`
	2. command
	3. shell
	4. copy
	5. file
	6. yum
	7. service/systemd
	8. 常用命令[^3]


4. Inventory （主机清单）
	1. 静态 Inventory （我说白了，就是学k8s集群时给改的host别名）
	2. 分组
	3. 变量
	4. 动态 Inventory


5. Module模块（和上面Ad-hoc命令一样）
	1. 文件管理
	2. 软件安装
	3. 服务管理
	4. 用户管理
	5. 网络配置
	6. docker/k8s 模块


6. Playbook 把命令写成YAML[^2]
	1. YAML语法
	2. tasks
	3. handlers （只有被notify触发，并且任务发生changed 时执行）
	4. variables
	5. template （根据变量生成文件）
	6. loops
	7. conditions
	8. tags


7. Jinjia2 模版
	1. {{变量}}
	2. 配置文件模版
	3. 动态生成配置


8. Role 角色
	1. 标准目录结构
	2. nginx role[^4]
	3. mysql role
	4. k8s role


9. 高级用法
	1. Vault 密码管理
	2. Galaxy 
	3. Callback
	4. AWX/Tower
	5. 动态资产管理





## 实战部署，Ansible接管

>新机器
```bash

ip a | grep -v '127.0.0.1' | grep -oE '([0-9]{1,3}\.){3}[0-9]{1,3}'


hostname
cat /etc/os-release

hostnamectl set-hostname    server1

systemctl enable  --now sshd 
systemctl status  sshd
```

>Contorl 控制端
```bash
hostnamectl set-hostname  ansible-control

yum install  -y  epel-release  
yum install  -y  ansible 
ansible  --version

#解决机器ip是谁的问题
vim /etc/hosts

ip1  server1
ip2  server2
ip3  server3


ping -c  2  server1



#ssh-keygen -t ed25519 -c "your_email"

ssh-copy-id  root@server1

ssh-copy-id  root@server2

ssh-copy-id  root@server3


ssh root@server3
#登上去连接测试




mkdir  -p /etc/ansible

#解决Ansible层面的管理和分组
vim /etc/ansible/hosts


[web]
server1 
server2 

[db] 
server3



ansible all  --list-hosts



vim /etc/ansible/ansible.cfg

[defaults]
#inventory → 我的服务器名单在哪
inventory = /etc/ansible/hosts 
 #remote_user → 默认使用谁SSH登录 
remote_user = root  
#host_key_checking → 是否检查SSH主机指纹
host_key_checking = False  
#forks → 同时操作多少台
forks = 10  


```
>开始批量操作
```bash
ansible all -m ping 
 
ansible all -m command -a "hostname"
ansible all -m shell   -a "free -h"
ansible all -m shell   -a "df -h"


ansible web -m ping

ansible web -m yum -a "name=nginx  state=present" 
ansible web -m  systemd -a "name=nginx  state=started  enabled=yes"

```


[Playbook演示](https://ncloud.eagleslab.com/Ansible.html#playbook%E5%89%A7%E6%9C%AC)  










[^1]: ```
	[defaults]
	#指定主机清单
	inventory=/etc/ansible/hosts
	#默认登陆用户
	remote_user=root
	#
	host_key_checking=False
	#并发执行数量
	forks=10
	
	timeout=30
	log_path=/var/log/ansible.log
		
	```

[^2]: ```bash
	#检查语法
	ansible-playbook  nginx.yaml  --syntax-check
	#执行
	ansible-playbook  nginx.yaml
	#模拟
	ansible-playbook  nginx.yaml  --check
	#指定主机
	ansible-playbook  nginx.yaml  -l  server1
	#指定tags
	ansible-playbook  nginx.yaml  --tags  install
	```

[^3]: ```bash
	ansible all  --list-hosts
	ansible-doc  yum
	ansible-doc  -l
	
	
	```

[^4]: Role 目录，生产最常见
	
	roles/nginx
	
	- tasks
		- main.yaml
	
	- handlers
		- main.yaml
	
	
	- templates
	- files
	- vars
	- defaults
	- meta
	
