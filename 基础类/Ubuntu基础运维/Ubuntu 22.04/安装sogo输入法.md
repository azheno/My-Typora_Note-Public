# 步骤



## 1.  安装fcitx

```shell
sudo apt update
sudo apt install fcitx   
```



## 2. 

![image-20220731201329228](.图片存放/image-20220731201329228.png)

## 3. 

![image-20220731201353131](.图片存放/image-20220731201353131.png)

## 4.设置fcitx开机自启

```shell
sudo cp /usr/share/applications/fcitx.desktop /etc/xdg/autostart/
```



## 5.卸载ibus框架

```shell
sudo apt purge ibus
```



## 6.安装

```shell
sudo dpkg -i sogo.deb 
```



## 7.安装依赖

```shell
sudo apt install libqt5qml5 libqt5quick5 libqt5quickwidgets5 qml-module-qtquick2
sudo apt install libgsettings-qt1
reboot 
```



## 8.

![查看右上角，可以看到“搜狗”字样，在输入窗口即可且出搜狗输入法](.图片存放/imageurl=%2F_next%2Fstatic%2Fmedia%2Fhelp13.512031f9.png)

## 9.

![3. 没有“搜狗”字样，选择配置，将搜狗加入输入法列表即可](.图片存放/imageurl=%2F_next%2Fstatic%2Fmedia%2Fhelp15.c14aafcd.png)