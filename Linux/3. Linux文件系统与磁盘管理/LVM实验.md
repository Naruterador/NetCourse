#### 实验要求
- 由于文件日渐增多，现在需要在某企业的服务器新添加一块磁盘，同时为了便于管理，在新磁盘的分区上进行LVM逻辑卷配置：
  - 在虚拟机中给系统新添加一块虚拟硬盘大小为20G ，添加完硬盘后，使用fdisk查看硬盘是否添加完成。
  - 对新添加硬盘进行分区，划分一个10G的主分区，分区完成后，命令查看磁盘分区结构。
  - 将刚划分的主分区转化成物理卷，命令查看当前物理卷。
  - 创建卷组vgdata，并将刚才的物理卷加入该卷组，命令查看LVM卷组信息。
   ```shell
   # 创建物理卷
   pvcreate /dev/sdb1

   # 创建卷组并将物理卷加入该卷组
   vgcreate vgdata /dev/sdb1
   ```
  - 从vgdata上分割2G给新的逻辑卷lvdata1，命令显示所有逻辑卷属性。
  ```lvcreate -L 2G -n lvdata1 vgdata```

  
  
  - 在逻辑卷lvdata1上创建ext4文件系统。
  ```mkfs -t ext4 /dev/vgdata/lvdata1```


  - 新建目录/data，将（6）中创建好的ext4文件系统的逻辑卷lvdata1挂载好/data目录。
  ```shell
  # 新建目录 /data
  mkdir /data
  
  # 将 lvdata1 挂载到 /data 目录
  mount /dev/vgdata/lvdata1 /data
  ```
  - 经过一段时间的使用，逻辑卷lvdata1的空间已使用完，通过命令给lvdata1增加3G空间。
  ```lvextend -L +3G /dev/vgdata/lvdata1```

  - 检查分区完整性，并重置分区容量，重新挂载文件系统，并使用df命令显示新的分区挂载界面，确定文件系统是否增加了3G空间。
  ```shell
  # 检查分区完整性
  e2fsck -f /dev/vgdata/lvdata1
  
  # 重置分区容量
  resize2fs /dev/vgdata/lvdata1
  
  # 重新挂载文件系统
  mount /dev/vgdata/lvdata1 /data
  ```