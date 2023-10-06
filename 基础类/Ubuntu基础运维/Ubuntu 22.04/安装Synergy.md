# 安装Synergy



```shell
vim rock-core-ubuntu-qt4-bionic.list 
deb https://ppa.launchpadcontent.net/rock-core/qt4/ubuntu/ focal main
# deb-src https://ppa.launchpadcontent.net/rock-core/qt4/ubuntu/ focal main
sudo apt updata 
sudo apt install libqtcore4 libqtgui4 libqt4-network libavahi-compat-libdnssd1 
wget http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.0g-2ubuntu4_amd64.deb
sudo dpkg -i openssl/libssl1.1_1.1.0g-2ubuntu4_amd64.deb 
sudo dpkg -i synergy_1.13.0-stable.bdb8f767_ubuntu20_amd64.deb
```

