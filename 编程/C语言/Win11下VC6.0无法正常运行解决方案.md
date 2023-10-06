[toc]



# Win11下VC6.0无法正常运行解决方案







## 1.

解压结束了之后会有Microsoft Visual Studio文件  进入Microsoft Visual Studio文件夹



![image-20220513135532089](.图片存放/image-20220513135532089.png)

## 2. 

找到Common 进入Common 找到MSDev98进入MSDev98 找到bin 进入bin

![image-20220513135646638](.图片存放/image-20220513135646638.png)

![image-20220513135702668](.图片存放/image-20220513135702668.png)

![image-20220513135722507](.图片存放/image-20220513135722507.png)

## 3. 

找到MSDEV.EXE，将MSDEV.EXE改为MSDEVL.EXE

![image-20220513135807383](.图片存放/image-20220513135807383.png)

## 4. 

反键进入属性找到兼容性，以兼容模式运行这个程序 选择windows xp (service pack 2) , 以勾选以管理员身份运行此程序 

![image-20220513135857395](.图片存放/image-20220513135857395.png)

## 5. 

选择更改所有用户的兼容性，以兼容模式运行这个程序 选择windows xp (service pack 2) , 以勾选以管理员身份运行此程序。全部无脑确定，然后运行程序

![image-20220513135926046](.图片存放/image-20220513135926046.png)

## 6.

运行成功

![image-20220513140007612](.图片存放/image-20220513140007612.png)