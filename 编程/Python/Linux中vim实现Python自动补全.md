[toc]



# Linux中vim实现Python自动补全



## 实现原理：

通过修改vim配置来达到Python自动补全的效果，使用了pydiction插件工具，只能实现基础函数补全，第三方库暂时不能实现



## 操作过程： 

```shell 
mkdir -p ~/.vim/bundle 
cd ~/.vim/bundle/
git clone https://github.com/rkulla/pydiction.git
cp -r ~/.vim/bundle/pydiction/after/ ~/.vim 
cat >> ~/.vimrc << EOF 
filetype plugin on
let g:pydiction_location = '~/.vim/bundle/pydiction/complete-dict'
let g:pydiction_menu_height = 3
EOF
```

