# wechat 安装



## 需要有docker作为底层支持



## 脚本编写：  

```shell
vim /usr/local/bin/wechat

case $1 in
start)
        xhost +
        sudo docker restart wechat > /dev/null
        echo "wechat start successful !"
        ;;
stop)
        sudo docker stop wechat > /dev/null
        echo "wechat stop successful !"
        ;;
create)
        xhost +
        sudo docker run -d --name wechat --device /dev/snd --ipc=host  -v /tmp/.X11-unix:/tmp/.X11-unix  -v $HOME/WeChatFiles:/WeChatFiles  -e DISPLAY=unix$DISPLAY  -e XMODIFIERS=@im=fcitx  -e QT_IM_MODULE=fcitx  -e GTK_IM_MODULE=fcitx  -e AUDIO_GID=`getent group audio | cut -d: -f3`  -e GID=`id -g`  -e UID=`id -u`  bestwu/wechat > /dev/null
        echo "wechat create successful !"
        ;;
show)
        sudo docker ps -a | grep wechat
        ;;
console)
        sudo docker exec -it wechat /bin/bash
        ;;
delete)
        sudo docker stop wechat > /dev/null
        sudo docker rm wechat > /dev/null
        echo "wechat delete successful !"
        ;;
*)
        echo -e "\033[31mwechat Error !\033[0m"
        ;;
esac
```







