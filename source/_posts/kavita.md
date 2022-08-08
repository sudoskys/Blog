---
title: Kavita搭建记录
date: 2022-1-11 11:12:29
tags: Web Build
cover: https://tva2.sinaimg.cn/large/9bd9b167gy1g4ligc2xvnj21hc0xchdt.jpg
---
![Kavita](https://socialify.git.ci/Kareadita/Kavita/image?font=Bitter&language=1&name=1&owner=1&pattern=Overlapping%20Hexagons&stargazers=1&theme=Dark)



## 前言
Kavita是一个十分轻快便捷的电子阅读器，支持多平台多格式。
[Kareadita/Kavita](https://github.com/Kareadita/Kavita)
[Kavita (kavitareader.com)](https://www.kavitareader.com/#home)
本文主要记录其在centos上使用docker搭建的流程。

## 准备
- 一台VPS或其他
- SSH工具便于链接服务器(such as MobaXterm_Personal   for win)
- 一个域名

## 流程

### 初始化VPS
本人安装的为centos8.x版本，为了避免源失效（，请安装最新版本centos）
初次使用VPS
以下步骤请在root用户下进行。
首先惯例检查
```
yum -y update
```

安装docker(ROOT)
```
yum -y install wget

curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

安装 WEBserver及配套设施
```
wget -c http://mirrors.linuxeye.com/oneinstack-full.tar.gz && tar xzf oneinstack-full.tar.gz && ./oneinstack/install.sh --nginx_option 1 --ssh_port 1717 --reboot

或者

yum -y install wget screen curl python git
wget http://mirrors.linuxeye.com/lnmp-full.tar.gz
tar xzf lnmp-full.tar.gz
cd lnmp
screen -S lnmp
./install.sh
```
这里便于以后```选择使用nginx做端口转发``` 
在安装的时候，先去dns提供商那设置A解析，解析到vps的IP上。
### 启动docker
运行以下命令启动docker
```
systemctl start docker
```

```
重启docker服务  sudo service docker restart

关闭docker service docker stop
```

拉取实例
```
docker run --name kavita -p 5000:5000 \
-v /your/manga/directory:/manga \
-v /kavita/data/directory:/kavita/config \
-v /books:/books \
--restart unless-stopped \
-e TZ=Your/Timezone \
-d kizaing/kavita:latest
```
注意：其中的-v为挂载VPS中的目录，只有其中的书籍文件才能被映射到容器内

打开```http://ip地址:5000``` 查看是否OK

## 配置HTTPS访问
cd到Oneinstack或lnmp文件夹，执行`./vhost.sh`即可进行虚拟主机+ssl证书的配置
```
cd lnmp
./vhost.sh
```
### 创建ssl证书
打开你的cloudflare控制面板，在 SSL/TLs --源服务器 中新建一个证书，记下所有信息。证书/私钥




### 安装证书+手动设置端口转发
```/usr/local/apache/conf/vhost/你的域名.conf```
编辑conf文件。
```
yum install nano
nano /usr/local/nginx/conf/vhost/你设置的域名.conf
```
```
证书目录如下，只需要把你的信息复制进文件即可....或使用nano或使用文件上传都行
/usr/local/nginx/conf/
/usr/local/nginx/conf/ssl/
/usr/local/nginx/conf/vhost/
```
such as "/usr/local/nginx/conf/ssl/books.XXX.XXX.crt"

在include后加入（可以不管缩进）
```
location ~ / {  
 proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  
 proxy_set_header Host $http_host;  
 proxy_redirect off;  
 proxy_pass http://127.0.0.1:5000;  
   
 # 使用本地存储策略，并更改大小为理论最大文件尺寸  
 client_max_body_size 20000m;  
}
```

重启nginx服务
```service nginx restart```

>如果你已经能使用域名打开了，请在你的vps提供商那**关闭端口**，如果配置了防火墙，也一并关闭。如果你在签发证书时在dns处关闭了**强制https**，**经过身份验证的源服务器拉取**等，请重新开启。




## 参考
[docker常规操作——启动、停止、重启容器实例_Michel Liu-CSDN博客_docker启动容器](https://blog.csdn.net/Michel4Liu/article/details/80889977)

[Docker Install | Kavita (kavitareader.com)](https://wiki.kavitareader.com/en/install/docker-install)

[自动安装 - OneinStack](https://oneinstack.com/auto/)

[Kavita自建自托管数字图书馆 ，支持多种文件格式 – 姿势小王子 (zsxwz.com)](https://zsxwz.com/2021/12/10/kavita%E8%87%AA%E5%BB%BA%E8%87%AA%E6%89%98%E7%AE%A1%E6%95%B0%E5%AD%97%E5%9B%BE%E4%B9%A6%E9%A6%86-%EF%BC%8C%E6%94%AF%E6%8C%81%E5%A4%9A%E7%A7%8D%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F/)