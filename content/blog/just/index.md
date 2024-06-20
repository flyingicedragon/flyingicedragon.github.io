+++
title = "Just 和 Justfile 简介与 fzf 配置"

date = 2024-06-20

[taxonomies]
tags = ["Linux", "Rust", "Makefile"]
categories = ["technology"]
+++

`Just`是一个使用`Rust`开发的类似`Make`的保存和运行命令工具。其语法与`Make`高度类似。同于由于并非构建系统，所以语法相对简单。

`Just`拥有不错的中英文文档，相关内容不在此赘述。

[英文文档](https://just.systems/man/en/)  
[中文文档](https://just.systems/man/zh/)时效性略低，建议只作为快速了解使用。后期查询详细功能还是直接看英文文档。

## 以交互方式选择配方

`Just`支持`--choose`参数来唤起一个选择器来选择配方。默认选择器是`fzf`，同时也可以使用`--chooser`参数来设置一个选择器。

[Doc](https://just.systems/man/en/chapter_53.html)

## 在`zsh`中设置快捷键调用`fzf`

添加以下内容至`.zshrc`：

```bash
bindkey -s "^[OP" "just --choose^M"
bindkey -s "^[OQ" "just -g --choose^M"
```

`bindkey`命令用于在`zsh`中绑定快捷键。`^[OP`和`^[OQ`分别对应`F1`和`F2`。可以在`zsh`中输入`cat`和`Enter`后再键入相关按键查看相关symbols。

这里使用`F1`选择当前项目中的命令，使用`F2`选择全局命令。
