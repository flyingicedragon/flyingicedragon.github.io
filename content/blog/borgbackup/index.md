+++
title = "borgbackup 超快速入门"

date = 2024-06-17

[taxonomies]
tags = ["Linux"]
categories = ["technology"]
+++

`Borg`是一个用于备份的工具，本文节选并翻译自[官方快速入门](https://borgbackup.readthedocs.io/en/stable/quickstart.html)。

官方文档编写得十分翔实，本文建议只作为快速了解所用。

## 安装

安装方法众多，包括`apt`、`pip`等。

[Installation](https://borgbackup.readthedocs.io/en/stable/installation.html)

## 新建repository

```bash
borg init --encryption=repokey /path/to/repo
```

`Borg`也支持利用`ssh`等远程位置进行备份。  
可以设置`--encryption`参数为`none`关闭加密。

## 备份目录

```bash
borg create /path/to/repo::Monday ~/src ~/Documents
```

“Monday”是备份（archive）的名称，需要为一个唯一的名称，并且得是一个有效的目录名。  
可以使用`{now}`等占位符方便生成唯一名称。

```bash
borg create /path/to/repo::src_{now} ~/src
```

## 在第二天建立一个新的备份

```bash
borg create --stats /path/to/repo::Tuesday ~/src ~/Documents
```

由于采用增量备份的原因，第二次备份的速度会快得多。

## 列出所有备份

```bash
$ borg list /path/to/repo
drwxr-xr-x user   group          0 Mon, 2016-02-15 18:22:30 home/user/Documents
-rw-r--r-- user   group       7961 Mon, 2016-02-15 18:22:30 home/user/Documents/Important.doc
...
```

## 恢复“Monday”备份

```bash
borg extract /path/to/repo::Monday
```

## 删除“Monday”备份

```bash
borg delete /path/to/repo::Monday
```

注意：仅仅删除不会释放硬盘空间

## 释放硬盘空间

```bash
borg compact /path/to/repo
```
