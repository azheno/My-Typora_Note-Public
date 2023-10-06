# RHCA环境制作手册 



## 1. 新建虚拟机，将课程包和基础包导入到虚拟机中



## 2. 将基础包中的rht-usb-8.x-7.r2021111613gitc7258b1文件赋权，并校验课程包和基础包

```shell
chmod +x rht-usb-8.x-7.r2021111613gitc7258b1 
vim /root/.icrm/config.yml  
./rht-usb-8.x-7.r2021111613gitc7258b1 verify RHCIfoundation-RHEL84-7.r2022011822-ILT-7-en_US.icmf 
```



## 3. 添加一块50G的磁盘



## 4. 将磁盘格式化

```shell
fdisk /dev/sdb 
./rht-usb-8.x-7.r2021111613gitc7258b1  usbformat /dev/sdb1 
```



## 5. 将基础包导入

```shell
# 导入基础包的时候需要修改/root/.icrm/config.yml 文件
./rht-usb-8.x-7.r2021111613gitc7258b1  usbadd RHCIfoundation-RHEL84-7.r2022011822-ILT-7-en_US.icmf  
```



