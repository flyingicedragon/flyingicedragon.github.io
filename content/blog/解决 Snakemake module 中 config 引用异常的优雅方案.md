+++
title = "解决Snakemake module 中config引用异常的优雅方案"

date = 2025-10-30

[taxonomies]
tags = ["snakemake"]
categories = ["technology"]
+++
## 解决Snakemake module 中config引用异常的优雅方案

在使用Snakemake的module功能时，你可能会遇到一个隐蔽的问题：在module的shell代码块中直接使用`config[key]`时，实际调用的是全局配置而非module指定的局部配置。这是Snakemake的一个已知bug（[#1688](https://github.com/snakemake/snakemake/issues/1688)），本文将介绍其解决思路及完整测试案例。


### 问题根源与现有方案局限

当在module中通过`config: config["x"]`指定局部配置后，若在规则的`shell`部分直接使用`{config[key]}`，会发现其仍引用全局config。这是因为module的shell解析器无法正确识别局部config。

现有方案多通过`params`传递配置（如`params: k=config['k']`再用`{params.k}`），但会产生大量重复代码，不够简洁。


### 更优雅的解决方法

核心思路：避免在module的`shell`块中直接引用`config`。通过`run`块结合`shell()`函数，可间接使用局部config——`run`块能正确识别module的局部配置，再通过`shell()`执行命令即可。


### 完整测试用例

#### 1. 项目结构
```
snakemake/
├── Snakefile       ## 主工作流
├── config.yaml     ## 配置文件
└── m.smk           ## 模块文件
```


#### 2. 文件内容

##### （1）config.yaml（全局与模块配置）
```yaml
x: x_global  ## 全局配置

m:           ## 模块m的局部配置
  x: x_module
  y: y_module
```

##### （2）Snakefile（主工作流）
```python
configfile: "config.yaml"

## 总规则，依赖两个输出文件
rule all:
    input: "a.out", "m/a.out"

## 全局规则：直接在shell中使用config[x]（引用全局配置）
rule a:
    output: "a.out"
    shell: "echo {config[x]} > {output}"

## 引入模块m，指定使用config["m"]作为局部配置
module m:
    snakefile: "m.smk"
    prefix: "m"  ## 输出文件会放在m/目录下
    config: config["m"]

## 使用模块m中的所有规则，命名为m_*
use rule * from m as m_*
```

##### （3）m.smk（模块文件）
```python
## 验证模块能读取局部config[y]
print(config["y"])  ## 运行时会输出"y_module"

## 模块内规则：用run+shell()使用局部config[x]
rule a:
    output: "a.out"
    run:
        ## 在run块中读取局部config[x]，通过shell()执行命令
        shell('echo %s > {output}' % config['x'])
```


#### 3. 运行与验证

##### 执行命令
```bash
snakemake --cores 1
```

##### 预期结果
- 全局规则`a`生成`a.out`，内容为`x_global`（引用全局config）；
- 模块规则`m_a`生成`m/a.out`，内容为`x_module`（引用模块局部config）；
- 运行日志中会打印`y_module`，证明模块成功读取局部配置。


### 总结

通过`run`块+`shell()`函数的组合，既能规避module中shell解析config的bug，又能减少冗余代码，是处理Snakemake模块配置问题的实用技巧。
