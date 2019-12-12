# -
docker私人仓库

搭建 Docker 映像本地私有仓库
天津云计算专班学员刘敏
1.	引言
Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的映像中，然后发布到任何流行的 Linux或Windows 机器上，也可以实现虚拟化。容器是完全使用沙箱机制，相互之间不会有任何接口。在未来Docker的应用前进将无比广大，而Docker映像是Docker的重要基石。公有映像使用广泛，但由于网络问题有时会造成使用不便；私有映像个性化包含原始代码、配置等我们不希望外部人员获取的信息；这时Docker 映像本地私有仓库是我们的不二选择。基于此我开始搭建 Docker 映像本地私有仓库。
这里我采用两套方案分别搭建Docker映像本地私有仓库：
方案一，直接应用Registry映像和挂载本地文件夹实现无界面免注册私有仓库搭建。
方案二，借助Harbor搭建具有管理UI、访问控制、AD/LDAP集成、以及审计日志等企业用户需求的功能的私有仓库。
2.	硬件及网络配置
2.1 硬件配置
方案一
节点名称	CPU	内存	磁盘	操作系统镜像
Xiangmu1（服务机）	1VCPU	3GB	50GB	CentOS-7-x86_64-DVD-1804.iso
xiangmu2
（用户机）	1VCPU	3GB	50GB	CentOS-7-x86_64-DVD-1804.iso
方案二
节点名称	CPU	内存	磁盘	操作系统镜像
Xiangmu3（服务机）	1VCPU	3GB	50GB	CentOS-7-x86_64-DVD-1804.iso
Xiangmu4（用户机）	1VCPU	3GB	50GB	CentOS-7-x86_64-DVD-1804.iso
2.2网络配置
 
3.	基础环境搭建
3.1	安装VMware Workstation
3.2	安装虚机xiangmu1
选择相应的硬件配置，网络选择桥接。
3.3	启动虚机配置网络并验证
 
 
 
 
3.4	备份原有yum源，使用阿里yum源
[root@localhost ~]# cd /etc/yum.repos.d/
[root@localhost yum.repos.d]# ls
 
[root@localhost yum.repos.d]# mv /etc/yum.repos.d/CentOS-ase.repo /etc/yum.repos.d/CentOS-Base.repo.backup
 
[root@localhost yum.repos.d]# wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
   
[root@localhost yum.repos.d]# yum clean all && yum makecache
 
3.5	安装Docker
yum -y install docker
 
[root@localhost yum.repos.d]# systemctl start docker
[root@localhost yum.repos.d]# systemctl enable docker
 
3.6	进行Docker操作验证安装
docker docker.io/centos:latest拉取映像
docker image ls 列出下载的映像
docker run -it --name ceshi-centos docker.io/centos:latest /bin/bash 通过映像启动并进入命名为ceshi-centos 的container
exit  退出container
docker commit -a "ceshi" ceshi-centos ceshi-centos:1.0 将容器保存为映像
 
3.7 克隆已完成环境搭建的虚机xiangmu1 三次,分别命名xiangmu2xiangmu3xiangmu4，参照前文完成相应配置。
4.	方案一应用Registry直接搭建私有仓库
4.1	拉取Registry映像
docker pull registry:latest
 
4.2	运行Registry映像，创建名为server-registry的container，映射虚机端口5000，将虚机文件夹挂载到sever-registry上
docker run -d -p 5000:5000 --name server-registry -v /tmp/registry:/tmp/registry docker.io/registry:latest
 
4.3	将本地映像打标上传映像列出映像
docker tag ceshi-centos:1.0 localhost:5000/ceshi-centos:1.0
docker push localhost:5000/ceshi-centos:1.0
docker images
 
4.4	查看网络
 
4.5	配置用户机xiangmu2
启动虚机xiangmu2，进入/etc/docker/daemon.json修改文件内容为：
{ "insecure-registries":["192.168.1.111:5000"] }
4.6	从xiangmu2拉取上传的本地映像
 
4.7	经过验证，应用registry搭建私有仓库成功。关闭虚机xiangmu1xiangmu2
5.	借助Harbor 搭建私有仓库
5.1	安装docker-compose
5.1.1	从github上下载docker-compose二进制文件安装
Sudo Curl -L https://github.com/docker/compose/releases/download/1.16.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
 
5.1.2	添加可执行权限
sudo chmod +x /usr/local/bin/docker-compose
 
5.1.3	安装 docker-compose
yum install docker-compose
 
 
5.1.4	验证安装
docker-compose version
 
5.2	安装Harbor
5.2.1	下载Harbor 安装包
wget https://storage.googleapis.com/harbor-releases/harbor-offline-installer-v1.6.1.tgz
 
5.2.2	解压安装包
tar xvf harbor-offline-installer-v1.6.1.tgz
 
5.2.3	修改harbor.cfg 文件
# hostname设置访问地址，可以使用ip、域名，不可以设置为127.0.0.1或localhost，此处我设置为本地ip

hostname = 192.168.1.120
其它不动。
5.2.4	启动 Harbor
./install.sh
 
……………..
……………..
……………..
 
安装完成
5.2.5	查看依赖的映像及服务
 
 
5.3网页登录
打开浏览器地址栏输入：192.168.1.120
 
输入用户名：admin。默认密码：Harbor12345。登录
 
可以看到系统各个模块如下：
项目：新增/删除项目，查看镜像仓库，给项目添加成员等
日志：仓库各个镜像create、push、pull等操作日志
系统管理
用户管理：新增/删除用户、设置管理员等
复制管理：新增/删除从库目标、新建/删除/启停复制规则等
配置管理：认证模式、复制、邮箱设置、系统设置等
其他设置
用户设置：修改用户名、邮箱、名称信息
修改密码：修改用户密码
5.4新建一个名为ceshi 的项目
 
 
5.5配置用户机xiangmu4
启动虚机xiangmu2，进入/etc/docker/daemon.json修改文件内容为：
{ "insecure-registries":["192.168.1.120:5000"] }
5.6 从xiangmu4登录Harbor
docker login https://192.168.1.120
 
5.7 验证从xiangmu4上传映像
5.7.1 查看xiangmu4本机映像选择一个映像打标
      
5.7.2 上传打标映像
     docker push 192.168.1.120/ceshi/registry1
   
5.7.3 网页查看上传的映像
 
 
5.8 经过验证，借助Harbor搭建私有仓库成功。关闭虚机xiangmu3xiangmu4
