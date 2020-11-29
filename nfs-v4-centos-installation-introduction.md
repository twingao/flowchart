# CentOS 7下NFS v4的安装和使用

### 搭建nfs服务器

安装nfs和rpc服务。

	yum install -y nfs-utils
	yum install -y rpcbind

启动并配置自启动，注意需要先启动rpcbind。

	systemctl start rpcbind
	systemctl enable rpcbind
	systemctl start nfs
	systemctl enable nfs

创建和配置共享文件目录。

	mkdir /data/volumes -p
	chmod 777 /data/volumes
	
	vim /etc/exports
	/data/volumes                     192.168.1.0/24(sync,rw,no_root_squash)

重新导出文件系统。

	exportfs –r

查看导出的文件系统。

	showmount -e nfs
	Export list for nfs:
	/data/volumes    192.168.1.0/24

### nfs客户端

安装nfs客户端。

	yum -y install nfs-utils

配置nfs服务器hosts。

	vi /etc/hosts
	192.168.1.80 nfs

查看导出的文件系统。

	showmount -e nfs
	Export list for nfs:
	/data/volumes    192.168.1.0/24

将导出的文件系统mount到本地目录。

	mkdir /data/vol -p
	mount -t nfs nfs:/data/volumes /data/vol

在本地创建一个文件。

	echo "hello nfs" >> /data/vol/hello.txt

到nfs服务器查看该文件是否存在。

	ll /data/volumes/
	-rw-r--r-- 1 root root 10 11月 30 17:47 hello.txt
	cat hello.txt
	hello nfs