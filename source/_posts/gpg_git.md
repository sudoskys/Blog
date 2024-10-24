---
title: Gpg&Git Linux 配置指南
date: 2022-7-6 12:28:10
tags: 
  - Git
  - 配置
---




## 前言
本文针对Linux系统上的 Git 相关一套配置。


## 准备
- 安装
安装 Git 套件工具，一般都自带
安装 gpg2 ，一般自带


--------------

## 初始配置

- 配置账户
```
git config --global user.name "John Doe"
git config --global user.email johndoe@example.com
```

- 生成 ssh 密钥
```
ssh-keygen -t rsa -C '邮箱'
```

- 添加至服务商
打开 .ssh/xxxx.pub，接着打开 git 网站，右上角用户头像，点击 settings，左侧菜单 SSH KEYS，将文件内容复制到 key 里 添加就可以了（复制公钥时不要复制多余的空格）


- 添加 SSH 到 ssh-agent 中
`eval "$(ssh-agent -s)"`


- 多账户 Git
在 .ssh 新建 `config` 文件（不带后缀）

按照如下模板配置 对应信息。

 ```
Host blog.dianas.cyou.例子.com
    HostName blog
    User AKEN
    IdentityFile ~/.ssh/git
  
Host gitlab.com
    HostName gitlab
    User Bob
    IdentityFile ~/.ssh/gitlab
 ```
 
PS：如果你使用 ssh 服务器链接出现指纹改变的情况，只需要去 `.ssh/known_hosts` 删除对应记录即可消除警告！


---------


## 配置 GPG

- 创建密钥
```
gpg2 --full-gen-key

```
接下来遵循指导和需求生成即可。不过注意，如果没有需求，这里提示的真实姓名最好不要填写户口本的姓名，不要自爆。

如果不相信自己的记忆力，设置密码时请将密码记在纸上或密码管理器中。

```
pub   rsa3072 2021-08-31 [SC] [expires: 2021-09-02]
      C3DE08A5665gpg密钥ID605BD95861C17D

```




- 列出密钥
```gpg --list-keys```


- 导出公钥
```
gpg --armor --export 邮箱
或者导入文件
gpg --export --armor 邮箱 > somename_pubkey.asc

```

将导出的 **公钥** 上传 github 个人设置的 gpg 设置中。

它应该以此开头

```
-----BEGIN PGP PUBLIC KEY BLOCK-----
```



- 配置到 Git
```
gpg --list-key
git config --global user.signingkey YOUR_KEYID

git config --global commit.gpgSign true
git config --global tag.forceSignAnnotated true


```



最后检查一下你的配置


`git config -l`

-------

## 常用使用

Git 有很多值得注意的使用方法。
```
git add .
git commit -m "备注信息"
git push
git pull
```
push是推送，pull是拉取。

-----

Gpg 广泛用于身份认证和消息加密中，你可以使用以下命令加密消息

- 加密消息
```
gpg -e -a -r 'designer@domain.com' message.txt

```


- 撤销密钥
```
生成证书
gpg [-a] [-o <output-file>] --gen-revoke <key-id>
```

通过导入撤销证书撤销密钥

```
gpg --import [<input-file>]
```

发布撤销密钥
如果之前使用了密钥服务器，则发布到密钥服务器即可

```
gpg --send-keys <key-id>
```

如果是手动导出的方式，则再次导出并发送给其他人导入即可

```
gpg --export-keys <key-id>
```

注意： 密钥一旦撤销将无法恢复。



推荐阅读：[GPG 使用指南](https://www.cnblogs.com/val3344/p/15611590.html)

## 引用

 CSDN：[使用GPG加密通讯，设置git提交验证密钥](https://blog.csdn.net/qq_43520820/article/details/120129566)
 
 git-scm.com：[GIT-BOOK](https://git-scm.com/book/zh/v2/)

[gpg非对称和简单的CA创建、签署、吊销](https://www.cnblogs.com/zy2271/p/13619605.html)


