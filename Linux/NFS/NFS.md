## NFS概述
  - NFS：Network file system，网络文件系统
    - 由sun公司1984年推出，用来在网络中的多台计算机间实现资源共享(包括象文件或cd-rom)
    - 设计的目的是：实现在不同系统间交互使用，所以它的通信协议采用与主机和操作系统无关的技术
    - NFS Server可以看作是File Server，它可以让你的PC通过网络将远端得NFS SERVER共享出来的档案MOUNT到自己的系统中，在CLIENT看来使用NFS的远端文件就象是在使用本地文件一样
    - NFS协议从诞生到现在有多个版本：NFS V2（rfc1094）,NFS V3（rfc1813）（最新的版本是V4（rfc3010）

- **NFS服务器的安装**
 - 可以由多种安装方法：
   - 在安装linux系统时选择安装nfs服务对应的组件；(多数linux发行版本默认安装)
   - 使用yum或者dnf安装:
     ```shell
     dnf install nfs-server
     ```
   - 安装nfs的rpm套件包(手动安装),使用如下命令:
    `rpm -ivh rpm包`
   - 需要5个RPM包:
     - setup-*：　 共享NFS目录在/etc/exports中定义 (linux默认都安装)
     - initscripts-*： 包括引导过程中装载网络目录的基本脚本 (linux默认都安装)
     - nfs-utils-*：　 包括基本的NFS命令与监控程序 
      - portmap-*：　 支持安全NFS RPC服务的连接 
      - quota-*：　　　 网络上共享的目录配额，包括rpc.rquotad （这个包不是必须的） 
 - 也可以去下载nfs的源代码包，进行编译安装； 

>  RPC(Remote Procedure call) 说明
 >> NFS本身是没有提供信息传输的协议和功能的，但NFS却能让我们通过网络进行资料的分享，这是因为NFS使用了一些其它的传输协议。而这些传输协议用到这个RPC功能的。可以说NFS本身就是使用RPC的一个程序。或者说NFS也是一个RPC SERVER.所以只要用到NFS的地方都要启动RPC服务，不论是NFS SERVER或者NFS CLIENT。这样SERVER和CLIENT才能通过RPC来实现PROGRAM PORT的对应。可以这么理解RPC和NFS的关系：NFS是一个文件系统，而RPC是负责负责信息的传输。

- **NFS的进程**
  - nfs在系统中的后台守护进程：
    - nfs
  - nfs服务需要启动的其他进程：
    - rpc.nfsd：接收从远程系统发来的NFS请求，并将这些请求转化为本地文件系统请求；
    - rpc.mountd：执行被请求的文件系统的挂接和卸载操作；
    - rpc.portmapper：将远程请求映射到正确的NFS守护程序；
    - rpc.statd：在远程主机重启时，提供加锁服务；
    - rpc.quotaed：提供硬盘容量的管理能力，磁盘限额
  `rpcinfo -p #可以查看所要的守护进程时候正常运行`
<br>
<br>
- **NFS服务器启动的端口**
 
```
 # lsof -i|grep rpc
 portmap   1931 daemon   3u IPv4   4289   UDP *:sunrpc
 portmap    1931 daemon     4u IPv4    4290    TCP *:sunrpc (LISTEN)
 rpc.statd 3206 statd    3u IPv4   7081    UDP *:1029
 rpc.statd 3206 statd    6u IPv4   7072    UDP *:838
 rpc.statd 3206 statd    7u IPv4   7085    TCP *:1031 (LISTEN)
 rpc.mount 3483   root    6u IPv4   7934       UDP *:691
 rpc.mount 3483   root    7u IPv4   7937       TCP *:694 (LISTEN)
 ```

## NFS配置:
- NFS服务的主配置文件：
```shell
/etc/exports：
格式：[共享的目录] [主机名或IP(参数,参数)]

当将同一目录共享给多个客户机，但对每个客户机提供的权限不同时，可以这样： 
[共享的目录] [主机名1或IP1(参数1,参数2)] [主机名2或IP2(参数3,参数4)]

第一列：欲共享出去的目录，也就是想共享到网络中的文件系统；

第二列：可访问主机
192.168.152.13 指定IP地址的主机 
nfsclient.test.com 指定域名的主机 
192.168.1.0/24 指定网段中的所有主机 
*.test.com        指定域下的所有主机 
*                       所有主机 

第三列：共享参数
下面是一些NFS共享的常用参数： 
 ro                    #只读访问 
 rw                    #读写访问 
 sync                  #所有数据在请求时写入共享 
 async              #NFS在写入数据前可以相应请求 
 secure             #NFS通过1024以下的安全TCP/IP端口发送 
 insecure          #NFS通过1024以上的端口发送 
 wdelay            #如果多个用户要写入NFS目录，则归组写入（默认） 
 no_wdelay      #如果多个用户要写入NFS目录，则立即写入，当使用async时，无需此设置。 
 Hide              #在NFS共享目录中不共享其子目录 
 no_hide           #共享NFS目录的子目录 
 subtree_check   #如果共享/usr/bin之类的子目录时，强制NFS检查父目录的权限（默认） 
 no_subtree_check    #和上面相对，不检查父目录权限 
 all_squash               #共享文件的UID和GID映射匿名用户anonymous，适合公用目录。 
 no_all_squash         #保留共享文件的UID和GID（默认） 
 root_squash             #root用户的所有请求映射成如anonymous用户一样的权限（默认） 
 no_root_squas           #root用户具有根目录的完全管理访问权限 
 anonuid=xxx             #指定NFS服务器/etc/passwd文件中匿名用户的UID
 anongid=xxx             #指定NFS服务器/etc/passwd文件中匿名用户的GID
```

- 配置示例:

```shell
/tmp　　　　　*(rw,no_root_squash) 
/home/public　192.168.0.*(rw)　　 *(ro) 
/home/test　　192.168.0.100(rw) 
/home/linux　 *.the9.com(rw,all_squash,anonuid=40,anongid=40) 
```

## 相关命令
- export命令:
- 如果我们在启动了NFS之后又修改了/etc/exports，是不是还要重新启动nfs呢？这个时候我们就可以用exportfs命令来使改动立刻生效，该命令格式如下：

```shell
exportfs [-aruv] 
-a #全部mount或者unmount /etc/exports中的内容 
-r #重新mount /etc/exports中分享出来的目录 
-u #umount目录 
-v #在export的时候，将详细的信息输出到屏幕上。 
``` 
- 具体例子:

```shell
# exportfs -au 卸载所有共享目录
# exportfs -rv 重新共享所有目录并输出详细信息
```

- 启动NFS 

```shell
service portmap start 
service nfs start 
```

- 检查NFS的运行级别：
  
```shell
chkconfig --list portmap 
chkconfig --list nfs 
```

- 根据需要设置在相应的运行级别自动启动NFS： 

```shell
chkconfig --level 235 portmap on 
chkconfig --level 235 nfs on 
```

- 查看NFS的运行状态，对于调整NFS的运行有很大帮助 
`nfsstat`

- 查看rpc执行信息，可以用于检测rpc运行情况的工具
`rpcinfo` 
- 另外，还需要查看系统的iptables、/etc/hosts.allow、/etc/hosts.deny是否设置了正确的NFS访问规则；

- 客户端操作和配置
- showmout命令对于NFS的操作和查错有很大的帮助，所以我们先来看一下showmount的用法 

```shell
showmout 
-a #这个参数是一般在NFS SERVER上使用，是用来显示已经mount上本机nfs目录的cline机器。 
-e #显示指定的NFS SERVER上export出来的目录。 

#例如： 
showmount -e 192.168.0.30 
Export list for localhost: 
/tmp         * 
/home/linux *.linux.org 
/home/public (everyone) 
/home/test    192.168.0.100 
#客户端运行以下命令MOUNT NFS文件系统
mount -t nfs 192.168.70.50:/opt /mnt/disk1
```

- mount nfs目录的方法： 

```shell
mount -t nfs hostname(orIP):/directory /mount/point 
具体例子： 
 #Linux: 
mount -t nfs 192.168.0.1:/tmp /mnt/nfs 
```
- mount nfs的其它可选参数： 
  - HARD mount和SOFT MOUNT： 
   - HARD: NFS CLIENT会不断的尝试与SERVER的连接（在后台，不会给出任何提示信息,在LINUX下有的版本仍然会给出一些提示），直到MOUNT上。 
   - SOFT:会在前台尝试与SERVER的连接，是默认的连接方式。当收到错误信息后终止mount尝试，并给出相关信息。 
   - 例如：mount -F nfs -o hard 192.168.0.10:/nfs /nfs 
 
- rsize和wsize： 
- 文件传输尺寸设定：wsize 来进行设定。这两个参数的设定对于NFS的执行效能有较大的影响 
  - bg：在执行mount时如果无法顺利mount上时，系统会将mount的操作转移到后台并继续尝试mount，直到mount成功为止。（通常在设定/etc/fstab文件时都应该使用bg，以避免可能的mount不上而影响启动速度） 
  - fg：和bg正好相反，是默认的参数 
- nfsvers＝n:设定要使用的NFS版本，默认是使用2，这个选项的设定还要取决于server端是否支持NFS VER 3 
 - mountport：设定mount的端口 
  - port：根据server端export出的端口设定，例如如果server使用5555端口输出NFS,那客户端就需要使用这个参数进行同样的设定 
- timeo=n:设置超时时间，当数据传输遇到问题时，会根据这个参数尝试进行重新传输。默认值是7/10妙（0.7秒）。如果网络连接不是很稳定的话就要加大这个数值，并且推荐使用HARD MOUNT方式，同时最好也加上INTR参数，这样你就可以终止任何挂起的文件访问。 
- intr 允许通知中断一个NFS调用。当服务器没有应答需要放弃的时候有用处。 
- udp：使用udp作为nfs的传输协议（NFS V2只支持UDP) 
- tcp：使用tcp作为nfs的传输协议 
- namlen=n：设定远程服务器所允许的最长文件名。这个值的默认是255 
- acregmin=n：设定最小的在文件更新之前cache时间，默认是3 
- acregmax=n：设定最大的在文件更新之前cache时间，默认是60 
- acdirmin=n：设定最小的在目录更新之前cache时间，默认是30 
- acdirmax=n：设定最大的在目录更新之前cache时间，默认是60 
- actimeo=n：将acregmin、acregmax、acdirmin、acdirmax设定为同一个数值，默认是没有启用。 
- retry=n：设定当网络传输出现故障的时候，尝试重新连接多少时间后不再尝试。默认的数值是10000 minutes 
- noac:关闭cache机制。 
- 同时使用多个参数的方法：
`mount -t nfs -o timeo=3,udp,hard 192.168.0.30:/tmp /nfs`
- **请注意，NFS客户机和服务器的选项并不一定完全相同，而且有的时候会有冲突。比如说服务器以只读的方式导出，客户端却以可写的方式mount,虽然可以成功mount上，但尝试写入的时候就会发生错误。一般服务器和客户端配置冲突的时候，会以服务器的配置为准。**

## /etc/fstab的设定方法 
- /etc/fstab的格式如下： 
  - fs_spec:该字段定义希望加载的文件系统所在的设备或远程文件系统,对于nfs这个参数一般设置为这样：192.168.0.1:/NFS 
  - fs_file:本地的挂载点 
  - fs_type：对于NFS来说这个字段只要设置成nfs就可以了 
  - fs_options:挂载的参数，可以使用的参数可以参考上面的mount参数。 
  - fs_dump　-　该选项被"dump"命令使用来检查一个文件系统应该以多快频率进行转储，若不需要转储就设置该字段为0 
  - fs_pass　-　该字段被fsck命令用来决定在启动时需要被扫描的文件系统的顺序，根文件系统"/"对应该字段的值应该为1，其他文件系统应该为2。若该文件系统无需在启动时扫描则设置该字段为0 。 

## 可能出现的问题
- rpc超时问题
  - 在服务器上的hosts文件加上了客户机的ip地址解析。

## NFS安全 
- NFS的不安全性主要体现于以下4个方面: 
  - 新手对NFS的访问控制机制难于做到得心应手,控制目标的精确性难以实现 
  - NFS没有真正的用户验证机制,而只有对RPC/Mount请求的过程验证机制 
  - 较早的NFS可以使未授权用户获得有效的文件句柄 
  - 在RPC远程调用中,一个SUID的程序就具有超级用户权限. 
- 加强NFS安全的方法： 
- 合理的设定/etc/exports中共享出去的目录，最好能使用anonuid，anongid以使MOUNT到NFS SERVER的CLIENT仅仅有最小的权限，最好不要使用root_squash。 
- 使用IPTABLE防火墙限制能够连接到NFS SERVER的机器范围 

```shell
iptables -A INPUT -i eth0 -p TCP -s 192.168.0.0/24   --dport 111 -j ACCEPT 
iptables -A INPUT -i eth0 -p UDP -s 192.168.0.0/24   --dport 111 -j ACCEPT 
iptables -A INPUT -i eth0 -p TCP -s 140.0.0.0/8    --dport 111 -j ACCEPT 
iptables -A INPUT -i eth0 -p UDP -s 140.0.0.0/8    --dport 111 -j ACCEPT 
#如果我们的NFS服务器在防火墙后边，则需要在防火强策略中加入如下策略：
iptables -A INPUT -p tcp -m state --state NEW -m multiport --dport 111,2049,4001,32764:32767 -j ACCEPT
iptables -A INPUT -p udp -m state --state NEW -m multiport --dport 111,2049,4001,32764:32767 -j ACCEPT
```
<br>
<br>

- 为了防止可能的Dos攻击，需要合理设定NFSD 的COPY数目。 
- 使用 /etc/hosts.allow和/etc/hosts.deny 控制客户端的访问

```shell
/etc/hosts.allow 
portmap: 192.168.0.0/255.255.255.0 : allow 
portmap: 140.116.44.125         : allow 
/etc/hosts.deny 
portmap: ALL : deny 
```

<br>
<br>

- 改变默认的NFS 端口 
  - NFS默认使用的是111端口，但同时你也可以使用port参数来改变这个端口，这样就可以在一定程度上增强安全性。 

<br>
<br>
- 使用Kerberos V5作为登陆验证系统


- 相同目录不同权限NFS权限问题优先级
   - 配置如下
   ```shell
   vim /etc/exports
   /tmp/share *(rw)
   /tmp/share 192.168.1.0/24(ro)
   #如出现以上情况，只有第一条会生效，也就是说允许越大的那一条NFS条目生效。
   ```