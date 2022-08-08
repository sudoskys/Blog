---
title: 部署Trilium
date: 2022-7-5 14:28:10
tags: Web Build
copyright_info: 原创编写|材料采集与引用已注明
copyright_author: LuoYing
---


![trilium](https://socialify.git.ci/zadam/trilium/image?description=1&font=KoHo&forks=1&language=1&name=1&owner=1&pattern=Plus&stargazers=1&theme=Dark)



Trilium 是一个超高自由度的个人知识库，文档树、分支和笔记的侧栏展示，数据库整合多媒体和文件管理，富文本加 Markdown的写作模式，定时备份和版本历史，全平台的客户端支持。还支持自建与同步，还有加密笔记。

![界面](https://raw.githubusercontent.com/wiki/zadam/trilium/images/screenshot.png)



此文章发布在 dianas cyou 上。
这里介绍本人搭建 trilium 的经验，转载需要注明本站地址。

项目地址 : https://github.com/zadam/trilium

# 准备服务器

- 链接服务器
```
ssh -p 22 -t 用户名@IP 
```


- 更新 apt 包索引。
```
sudo apt-get update
```

- 安装依赖
```
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg2 \
    software-properties-common
```

## 安装 Docker

- （可选）删除镜像
```
sudo rm -rf /var/lib/docker
```
- （可选）卸载旧版本
```
sudo apt-get remove docker docker-engine docker.io containerd runc
```

- 添加 Docker 的官方 GPG 密钥
```
curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/debian/gpg | sudo apt-key add -
```

- 使用官方安装脚本安装(风险自担)
安装命令
```
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```
也可以使用国内 daocloud 一键安装
```
curl -sSL https://get.daocloud.io/docker | sh
```
或 [手动安装](https://www.runoob.com/docker/debian-docker-install.html)

# 拉取镜像

- (可选) 删除容器
```
docker ps 

docker rm -f ps列出的应用id

cd ~

ls

##请注意！！！！备份数据！！！文章数据以数据库形式保存在此文件夹中
rm -r ls列出的triliumdata文件夹名字

```


- 拉取
```
docker pull zadam/trilium:[VERSION]
```

替换 [VERSION] 为对应版本号，如 ```0.53-latest```

- 运行

启动运行需要大约 20s 时间

```
docker run -d -p 0.0.0.0:8080:8080 -v ~/trilium-data:/home/node/trilium-data zadam/trilium:[VERSION]
```

- 重新运行

```
docker ps
docker restart 应用id
docker status
```

- 附录
```
docker pull zadam/trilium:0.53-latest
```
```
docker run -d -p 0.0.0.0:8080:8080 -v ~/trilium-data:/home/node/trilium-data zadam/trilium:0.53-latest
```

- [查看状态](https://www.runoob.com/docker/docker-container-usage.html)
```
docker ps

dokcer status

```

运行后访问服务器IP+端口号会出现初始页面。

接下来配置 ssl 加密 (https) 与 docker 端口 nginx 转发


# 准备 nginx

这里只介绍 dpkg 安装nginx 。
装完应该能正常运行了。如果之前有装 APACHE 要改下端口。。。或者直接

```
apt-get remove apache2
```

```
sudo apt-get install nginx
systemctl start nginx
systemctl enable nginx
```

安装完以后，输入 ```whereis nginx``` 查看 Nginx 的安装位置，其中的 ```nginx.conf ``` 为 Nginx 的配置文件。

## 配置 Nginx

本人以 cloudflare.com 举例，在侧边找到 ssl--->源服务器，点击创建证书，可以获得证书和密钥。
分别填入 

etc/nginx/trilium.pem; 

etc/nginx/trilium.key;

-------------

切换回 ssh 界面
```
cd /etc/nginx/

ls

vim nginx.conf
```

按下 i 开始输入，如果是Vnc只能照着打。
将以下配置加入 http{} ，如果你需要多配置，请将以下配置写入 /etc/nginx/conf.d/default.conf ```vim default.conf```

```
server {
    listen 80;
    
    listen 443 ssl;
    # ssl证书位置
    ssl_certificate      etc/nginx/trilium.pem; 
    ssl_certificate_key  etc/nginx/trilium.key;
    ssl_session_timeout  5m;
    ssl_ciphers  ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers  on;
    
    # 注意更改域名
    server_name 域名1 域名2;
    
    # location 后跟代理的目录，这里很坑，注意
        location / {
        proxy_pass http://127.0.0.1:8080/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
        
        # set to 0 for unlimited. Default is 1M.
        client_max_body_size 0;

        }
        # 载入 conf.d 的配置
        include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/sites-enabled/*;
}


```

按下 Esc 键，按住 Shift + ： 键来唤出命令，输入 wq 保存退出。

- 刷新 Nginx 配置

输入 ```nginx -t``` 测试配置，没有报错就行。
输入 ```nginx -s reload``` 重载配置文件。


打开域名就可以了。
注意续费和同步，程序数据在 trilium_data

请务必保存好自己的密码，这是不可找回的，加密笔记也会永远丢失。

附快捷收集材料的插件

[Trilium Web Clipper ](https://github.com/zadam/trilium-web-clipper)
https://chrome.google.com/webstore/detail/trilium-web-clipper/dfhgmnfclbebfobmblelddiejjcijbjm


## 同步问题

如果同步失败，你需要考虑

1. 服务器是否可以直连？是否需要设置代理端口访问？
2. CloudFlare 代理流量的影响。
3. 网线插了吗？
4. 时区问题

Passage uploaded to `dianas cyou`


## 引用

https://www.runoob.com/docker/debian-docker-install.html

https://sspai.com/post/59739

官方Wiki
https://github.com/zadam/trilium/blob/master/README-ZH_CN.md

https://github.com/baddate/trilium/wiki/