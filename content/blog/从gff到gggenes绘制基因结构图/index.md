+++
title = "从 gff 到 gggenes 绘制基因结构图"

date = 2024-06-12

[taxonomies]
tags = ["bioinformatics", "visualization", "R", "Python"]
categories = ["technology"]
+++

`gffutils` 是一个用来解析 gff 文件的 Python 包，可以十分方便地获取 gff 文件中的相关信息。`gggenes` 是 `ggplot2` 的扩展包，用于绘制基因结构图、多物种基因比较图的很好玩的工具。两个工具联用可以实现从 gff 数据获取到基因结构图绘制的全过程。

## 对 gff 原始数据进行处理

### 安装 gff2gggenes

建议使用`pipx`进行安装，也可以使用`pip`。

```shell
# from github
pipx install git+https://github.com/flyingicedragon/gff2gggenes.git
# or from gitee
pipx install git+https://gitee.com/flyingicedragon/gff2gggenes.git
```

### GFF 文件预处理

对 GFF 文件进行预处理，截取仅包含所需基因的 GFF 内容。建议在 Linux 中使用 `sed` 命令完成。

### 程序调用

```shell
gff2gggenes --help
gff2gggenes test.gff
gff2gggenes test.gff --sub
```

不添加 `--sub` 参数，表示将各基因的情况进行输出；添加 `--sub` 参数，表示输出各基因子区域（例如：mRNA、CDS等，与 gff 文件内容有关）。

### 结果输出

CSV 文件输出到工作路径中，文件名结尾是“_gene.csv”或者“_subgene.csv”。

## 利用 R 包 gggenes 进行可视化

gggenes 文档链接：<https://wilkox.org/gggenes/>

### gggene 安装

直接从 CRAN 安装：

```R
install.packages("gggenes")
```

如果使用 rstudio 进行 R 工作，也可以在 package 中安装。
![rstudio.png](rstudio.jpg)

### 启用 ggplot2 和 gggene

可以在 rstudio 的包管理工具中开启，也可以使用以下代码：

```R
library(ggplot2)
library(gggenes)
```

### 导入 python 程序生成的 csv 数据

```R
geneData = read.csv('gene.csv')
```

### 用geom_gene_arrow()画基因箭头

```R
ggplot(geneData, aes(xmin = start, xmax = end, y = molecule, fill = gene)) +
  geom_gene_arrow() +
  facet_wrap(~ molecule, scales = "free", ncol = 1) +
  scale_fill_brewer(palette = "Set3")
```

### （可选）用 `theme_genes` 美化图形

默认的图形不太美观，可以使用自带的 `theme_genes()` 进行美化。

```R
ggplot(geneData, aes(xmin = start, xmax = end, y = molecule, fill = gene)) +
  geom_gene_arrow() +
  facet_wrap(~ molecule, scales = "free", ncol = 1) +
  scale_fill_brewer(palette = "Set3") +
  theme_genes()
```

### 使用 `make_alignment_dummies()` 跨面对齐基因

可以选择一个基因将所有数据进行对齐，在比较基因组时会用到。

```R
dummies <- make_alignment_dummies(
  geneData,
  aes(xmin = start, xmax = end, y = molecule, id = gene),
  on = "geneX"
)

ggplot(geneData, aes(xmin = start, xmax = end, y = molecule, fill = gene)) +
  geom_gene_arrow() +
  geom_blank(data = dummies) +
  facet_wrap(~ molecule, scales = "free", ncol = 1) +
  scale_fill_brewer(palette = "Set3") +
  theme_genes()
```

### 用 `geom_gene_label()` 标记基因

`geom_gene_label()` 可以将标签文本放入基因箭头内，需要把基因名字所在的列名字映射到 `label` 属性。依赖于 `ggfittext` 包。

```R
ggplot(
    geneData,
    aes(xmin = start, xmax = end, y = molecule, fill = gene, label = gene)
  ) +
  geom_gene_arrow(arrowhead_height = unit(3, "mm"), arrowhead_width = unit(1, "mm")) +
  geom_gene_label(align = "left") +
  geom_blank(data = dummies) +
  facet_wrap(~ molecule, scales = "free", ncol = 1) +
  scale_fill_brewer(palette = "Set3") +
  theme_genes()
```

### 查看基因子片段（subgene）

可以使用 `geom_subgene_arrow()` 突出显示子基因片段。此时，需要多调用另一个 csv 文件。

```R
subGeneData = read.csv('subGene.csv')
ggplot(geneData, aes(xmin = start, xmax = end, y = molecule)) +
  facet_wrap(~ molecule, scales = "free", ncol = 1) +
  geom_gene_arrow(fill = "white") +
  geom_subgene_arrow(data = subGeneData,
    aes(xmin = start, xmax = end, y = molecule, fill = gene,
        xsubmin = from, xsubmax = to), color="black", alpha=.7) +
  theme_genes()
```

同时，也可以为基因子片段添加标签。

```R
ggplot(geneData, aes(xmin = start, xmax = end, y = strand)
  ) +
  geom_gene_arrow() +
  geom_gene_label(aes(label = gene)) +
  geom_subgene_arrow(
    data = subGeneData, aes(xsubmin = from, xsubmax = to, fill = subgene)
  ) +
  geom_subgene_label(
    data = subGeneData, aes(xsubmin = from, xsubmax = to, label = subgene),
    min.size = 0
  )
```
