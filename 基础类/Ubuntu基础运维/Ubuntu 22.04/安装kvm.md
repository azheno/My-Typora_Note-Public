

# 安装kvm



```shell
# 查看是否开启虚拟化
egrep -c '(vmx|svm)' /proc/cpuinfo
# 安装cpu-checker
sudo apt install cpu-checker
kvm-ok
#安装kvm
sudo apt install -y qemu-kvm virt-manager libvirt-daemon-system virtinst libvirt-clients bridge-utils
#libvirtd进程开机自启
sudo systemctl enable --now libvirtd
sudo systemctl start libvirtd
#使当前用户能够创建虚拟机
sudo usermod  -aG kvm $USER 
sudo usermod  -aG libvirt  $USER 
```



