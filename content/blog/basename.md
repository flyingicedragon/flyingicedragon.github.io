+++
title = "Linux 查看目录下的文件名（不带后缀）"

date = 2024-03-27

[taxonomies]
tags = ["Linux"]
+++

现存在以下文件列表：

``` bash
# ls /path
test1.txt  test2.txt
```

需要只提取出文件名，而不需要后缀名，如：test1、test2

``` bash
# 显示文件名，不包含目录
basename /path/test1.txt
# test1.txt

# 显示文件名，不包含目录与后缀
basename /path/text1.txt .txt
# or
basename -s .txt /path/text1.txt
# text1

# 将多个参数按照顺序输出
basename -a /path/text1.txt /path/text2.txt
# 或者直接利用 ls
basename -a `ls`
# text1.txt
# text2.txt
```
