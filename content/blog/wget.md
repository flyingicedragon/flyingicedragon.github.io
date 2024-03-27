+++
title = "Linux下wget下载整个FTP目录(含子目录)"

date = 2020-09-05

[taxonomies]
tags = ["Linux", "网络"]
+++
```bash
wget -nH -m --ftp-user=your_username --ftp-password=your_password ftp://your_ftp_host/*
```

-nH：不创建以主机名命名的目录  
--cut-dirs：后面接希望去掉原来的目录层数，从根目录开始计算。如果想完全保留 FTP 原有的目录结构，则不要加该参数。  
-m：下载所有子目录并且保留目录结构。  
–ftp-user：FTP用户名  
–ftp-password：FTP 密码  
ftp://.../*：FTP 主机地址。最后可以跟目录名来下载指定目录。

例子：

```bash
wget -nH -m --ftp-user=tom --ftp-password=123456 ftp://192.168.19.1/tom/
```

> https://www.cnblogs.com/ftl1012/p/9265699.html
