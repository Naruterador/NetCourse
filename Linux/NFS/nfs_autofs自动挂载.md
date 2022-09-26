## NFS服务端配置
#### 1.新建共享目录及标记文件：
```shell
[root@localhost ~]# mkdir /opt/share
[root@localhost ~]# touch /opt/share/flag
```
#### 2.开放读写权限：
```shell
[root@localhost ~]# chmod -R 777 /opt/share
```
#### 3.修改配置文件：
```shell
[root@localhost ~]# vim /etc/exports
#写入：/opt/share 192.168.100.0/24(rw,sync)
```
#### 4.生效配置：
```shell
[root@localhost ~]# exportfs -r
```
#### 5.启动并开机自启NFS服务：
```shell
[root@localhost ~]# systemctl start rpcbind
[root@localhost ~]# systemctl start nfs
[root@localhost ~]# systemctl enable rpcbind
```
#### 7.查看挂载目：
```shell
[root@localhost ~]# showmount -e 192.168.100.10
```

## 在客户端配置autofs自动挂载
#### 1.先查看挂载目：
```shell
[root@localhost ~]# showmount -e 192.168.100.10
```
#### 2.安装autofs：
```shell
[root@localhost ~]# yum -y install autofs
````
#### 3.配置autofs挂载的主目录为/share：
```shell
[root@localhost ~]# vim /etc/auto.master
/share /etc/auto.test
```
#### 4.配置pub：
```shell
[root@localhost ~]# vim /etc/auto.test         
#创建autofs挂载配置文件
pub  -fstype=nfs,rw,sync,root_squash  192.168.10.10:/opt/nfsshare
#这里的pub是指子目录名称，最后nfs服务器的目录(192.168.10.10:/opt/nfsshare)会挂载到/share/pub下
```
#### 5.重启并开启自启：
```shell
[root@localhost ~]# systemctl restart autofs
[root@localhost ~]# systemctl enable autofs
```

#### 6.测试自动挂载
[root@localhost ~]# cd /share/pub
#只有当进入/share/pub目录后，nfs挂载才会被激活