# 安装WeChat、TIM



## 原理

容器化部署WeChat，需要拉取镜像，在使用之前一定要允许x11桌面运行



```shell
xhost + # 允许所有登录用户访问x11桌面

#部署wechat
docker pull  bestwu/wechat
docker run -d --name wechat --device /dev/snd --ipc=host  -v /tmp/.X11-unix:/tmp/.X11-unix  -v $HOME/WeChatFiles:/WeChatFiles  -e DISPLAY=unix$DISPLAY  -e XMODIFIERS=@im=fcitx  -e QT_IM_MODULE=fcitx  -e GTK_IM_MODULE=fcitx  -e AUDIO_GID=`getent group audio | cut -d: -f3`  -e GID=`id -g`  -e UID=`id -u`  bestwu/wechat


#部署TIM
docker run -d --name tim --device /dev/snd --ipc="host" -v $HOME/Data/docker/TIMFiles:/TencentFiles -v /tmp/.X11-unix:/tmp/.X11-unix -e XMODIFIERS=@im=fcitx -e QT_IM_MODULE=fcitx -e GTK_IM_MODULE=fcitx -e DISPLAY=unix$DISPLAY -e AUDIO_GID=`getent group audio | cut -d: -f3` -e VIDEO_GID=`getent group video | cut -d: -f3` -e GID=`id -g` -e UID=`id -u` bestwu/qq:office

```





## 自动化

```shell
vim wechat 
case $1 in 
start) 
	sudo docker start wechat 
	;; 
stop) 
	sudo docker stop wechat 
	;; 
create) 
	docker run -d --name wechat --device /dev/snd --ipc=host  -v /tmp/.X11-unix:/tmp/.X11-unix  -v $HOME/WeChatFiles:/WeChatFiles  -e DISPLAY=unix$DISPLAY  -e XMODIFIERS=@im=fcitx  -e QT_IM_MODULE=fcitx  -e GTK_IM_MODULE=fcitx  -e AUDIO_GID=`getent group audio | cut -d: -f3`  -e GID=`id -g`  -e UID=`id -u`  bestwu/wechat
	;; 
*)
	echo "wechat start Error!"
esac 

```

