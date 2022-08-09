---
title: Android系统数据存储在Androlua中的体现
date: 2020-10-20 02:58:39
tags: Android
cover: false
---

Android系统有五种数据存储形式，分别是文件存储、SP存储、数据库存储、contentprovider 内容提供者、网络存储。其中，前四个是本地存储。存储的类型包括简单文本、窗口状态存储、音频视频数据、XML注册文件的各种数据。各种存储形式的特点不尽相同，因此对于不同的数据类型有着固定的存储形式，本文为演示方便给出的案例基本相同，都是是采用账号登录来演示数据存储，保存账号和密码信息，下次登录时记住账号和密码。重在说明各种存储形式的原理。

文件存储:

    以I/O流的形式把数据存入手机内存或SD卡，可以存储大数据，如音乐、图片或视频等。对于手机内存来说系统会根据每个应用的包名创建一个/data/data/包名/的文件夹，访问自己包名下的目录是不需要权限的，并且 Android 已经提供了非常简便的 API 可以直接去访问该文件夹。访问时可以用getFilesDir()和getCacheDir(),两个的区别是系统会自动清理后者中的内容。

    SD卡中的文件通常位于mnt/sdcard目录下，不同生产商生产的手机这个路径可能不同。操作sd卡的时通常要判断下sd卡是否可用以及剩余空间是否足够，因为部分手机的SD卡可卸载，SD卡处于非挂载状态时，无法进行读写操作。另外一点，对SD卡的读取和写入操作均需要相应的权限，否则无法完成。获取SD卡路径的方法是Environment.getExternalStorageDirectory()，其余操作与文件存储基本类似。

文件存储位置：data/data/包名

SP存储：

     SP存储本质上是一个XML文件，以键值对的形式存入手机内存中。常用于存储简单的参数设置，如登陆账号密码的存储，窗口功能状态的存储等，该存储文件位于：data/data/包名/shared_prefs文件夹中。使用的时候，首先需要通过context.getSharedPrefrences(String name,int mode)获取SharedPrefrences的实例对象，存储数据时，用SharedPrefrences的实例对象得到SharedPrefrences文件的编辑器，在编辑器中用putXxx()添加数据，之后务必用commit提交数据，否则无法获取数据。取数据时，直接用getXxx()方法。

sp存储自动生成xml文件，其的路径如下：data/data/包名/shared_prefs