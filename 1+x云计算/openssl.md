[toc]



https://ivanzz1001.github.io/records/post/docker/2018/04/09/docker-harbor-https

# 获取证书

这里假设你的registry主机名为reg.yourdomain.com，并且通过DNS记录能够找到你运行Harbor的主机。首先你应该从CA处获得一个

certificate。该certificate通常包含一个a.crt文件和一个a.key文件，例如：yourdomain.com.crt以及yourdomain.com.key。

在测试或开发环境下，你也许会使用一个自签名证书，而不是从CA那里获取。可以通过如下的命令产生你自己的证书：

## 1.创建自签名根证书

可以通过如下的方式来产生一个私钥及自签名证书:

```shell
openssl req \
-newkey rsa:4096 -nodes -sha256 -keyout ca.key \
-x509 -days 365 -out ca.crt
```



## 2. 产生证书签名请求

假如你使用类似于reg.yourdomain.com的FQDN(Fully Qualified Domain Name)方式来连接registry主机，则你必须使用

reg.yourdomain.com来作为CN(Common Name)。否则，假如你使用IP地址来连接你的registry主机的话，CN可以指定为任何值（例如

指定为你的名字）：

```shell
openssl req \
-newkey rsa:4096 -nodes -sha256 -keyout yourdomain.com.key \
-out yourdomain.com.csr
```





## 3. 为registry主机产生证书

假如你使用类似于reg.yourdomain.com的FQDN(Full Qualified Domain Name)方式来连接registry主机，你可以使用如下的命令来为

registry主机产生证书：

```shell
openssl x509 -req -days 365 -in yourdomain.com.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out yourdomain.com.crt
```



假如你是使用ip的话， 比如使用192.168.1.101来连接registry主机的话，你需要使用如下命令：

```shell
echo subjectAltName = IP:192.168.1.101 > extfile.cnf
openssl x509 -req -days 365 -in yourdomain.com.csr -CA ca.crt -CAkey ca.key -CAcreateserial -extfile extfile.cnf -out yourdomain.com.crt
```




https://blog.ziki.cn/post/docker-harbor/


https://blog.51cto.com/liqingbiao/2431439

