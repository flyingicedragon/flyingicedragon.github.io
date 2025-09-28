+++
title = "git merge driver简介"

date = 2025-09-26

[taxonomies]
tags = ["git"]
categories = ["technology"]
+++
Git 的合并驱动（merge driver）是一种高级功能，允许你自定义 Git 处理文件合并冲突的方式。默认情况下，Git 会尝试自动合并文件，若冲突则标记冲突内容让用户手动解决。而合并驱动能让你编写规则，告诉 Git 如何智能处理特定文件的合并。


### 一、合并驱动的核心原理
1. 当 Git 合并两个分支时，会检查 `.gitattributes` 文件，查看是否有文件配置了 `merge=xxx` 规则
2. 若有，则调用名为 `xxx` 的合并驱动处理该文件
3. 驱动可以是简单的命令（如 `true`），也可以是复杂的脚本，决定最终保留哪个版本的文件


### 二、基础配置步骤（必做）
#### 步骤1：定义文件与驱动的关联（`.gitattributes`）
在项目根目录创建 `.gitattributes` 文件，指定哪些文件使用哪个驱动：
```bash
# 格式：[文件路径] merge=[驱动名称]
config/personal.env merge=ignore-personal  # 个人配置文件用自定义驱动
*.json merge=keep-ours  # 所有JSON文件用另一个驱动
```

#### 步骤2：配置驱动的具体行为（`git config`）
通过 `git config` 定义驱动的执行逻辑，格式：
```bash
git config [--global] merge.[驱动名称].driver "[处理命令]"
```
- `--global` 表示全局生效（所有项目），不加则仅当前项目生效


### 三、合并驱动示例（从简单到复杂）

#### 示例1：最简单的驱动（`true`，保留当前分支版本）
这是最常用的基础驱动：
```bash
# 配置驱动：merge.ours.driver
git config merge.ours.driver true
```
- 原理：`true` 是一个 Unix 命令，永远返回"成功"（退出码0）
- 效果：合并时**无条件保留当前分支的文件版本**，忽略源分支的变更
- 适用场景：个人配置文件（如 `personal.env`），合并到公共分支时始终用公共版本

在 `.gitattributes` 中配合使用：
```bash
config/personal.env merge=ours
```


#### 示例2：保留源分支版本（覆盖当前分支）
与示例1相反，强制使用源分支的文件版本：
```bash
# 定义 "take-theirs" 驱动
git config merge.take-theirs.driver "cp %B %A"
```
- 命令解释：
  - `%B`：表示源分支（被合并的分支，如 `dev-personal`）的文件版本
  - `%A`：表示当前分支（合并到的分支，如 `dev`）的文件版本
  - `cp %B %A`：将源分支的文件复制到当前分支，覆盖当前版本
- 适用场景：某些必须以个人分支版本为准的配置文件

在 `.gitattributes` 中使用：
```bash
config/force-personal.json merge=take-theirs
```


#### 示例3：自定义脚本驱动（条件合并）
更灵活的方式是编写脚本，根据文件内容决定合并策略。例如：合并时保留双方的注释，但以当前分支的配置值为准。

##### 步骤1：创建合并脚本（`merge-config.sh`）
```bash
#!/bin/bash

# 参数说明：
# $1: 当前分支的文件路径（%A）
# $2: 共同祖先的文件路径（%O）
# $3: 源分支的文件路径（%B）

# 保留当前分支的配置值，但合并源分支的注释（假设注释以#开头）
# 1. 提取当前分支的配置（非注释行）
grep -v '^#' "$1" > "$1.tmp"
# 2. 追加源分支的注释（只保留注释行）
grep '^#' "$3" >> "$1.tmp"
# 3. 替换原文件
mv "$1.tmp" "$1"

# 必须返回0表示成功
exit 0
```

##### 步骤2：赋予执行权限
```bash
chmod +x merge-config.sh
```

##### 步骤3：配置驱动关联脚本
```bash
# 注意脚本路径需使用绝对路径
git config merge.smart-config.driver \
  "$(pwd)/merge-config.sh %A %O %B"
```

##### 步骤4：在 `.gitattributes` 中应用
```bash
config/mixed.conf merge=smart-config
```
- 效果：合并时自动保留当前分支的配置值，同时合并源分支的注释
- 适用场景：需要部分保留双方内容的复杂配置文件


#### 示例4：冲突时提示手动处理（增强版默认行为）
默认情况下，Git 会在冲突文件中插入 `<<<<<<<` 标记。可以自定义提示信息：
```bash
git config merge.prompt-conflict.driver \
  'if ! diff -q %A %B >/dev/null; then echo "⚠️ 请手动合并 %A"; exit 1; else exit 0; fi'
```
- 命令解释：
  - `diff -q %A %B`：检查两个版本是否有差异
  - 若有差异（冲突），则输出提示并返回非0（让 Git 标记为冲突）
  - 若无差异，直接成功
- 适用场景：重要配置文件，不允许自动合并，必须人工确认


### 四、查看与删除已配置的驱动
```bash
# 查看所有合并驱动
git config --list | grep merge.

# 删除某个驱动
git config --unset merge.ours.driver
```


### 五、总结
合并驱动的核心价值是**自动化处理特定文件的合并规则**，避免重复的手动操作：
- 简单场景用 `true` 或 `cp %B %A` 即可满足需求
- 复杂场景（如条件合并、内容提取）可以通过脚本实现
- 配合 `.gitattributes` 精确控制哪些文件应用规则
