---
layout:     post
title:      "3k 个外周血单个核细胞的预处理与聚类"
subtitle:   " \"开始学习数据处理\""
date:       2026-04-10 23:30:00
author:     "Zemiss"
header-img: "img/bg-weeklyplan.jpg"
catalog: true
tags:
  - Study
---

# 3k 个外周血单个核细胞的预处理与聚类（传统工作流）——Scanpy
## 目录
- 预处理
- 主成分分析
- 计算近邻图
- 近邻图降维可视化
- 近邻图聚类
- 寻找标记基因

```python
from __future__ import annotations
import matplotlib.pyplot as plt
import pandas as pd
import scanpy as sc
```

2017年5月，本文档最初用于演示**Scanpy**可复现Seurat官方引导聚类教程的大部分分析流程（Satija 等人，2015）。
在此衷心感谢Seurat的作者们提供该教程！此后我们对流程做了少量增删调整。

数据集来自**健康供体的3k个PBMC（外周血单个核细胞）**，可从10x Genomics官网免费获取（链接见文档对应位置）。
在类Unix系统中，可取消注释并运行以下代码下载并解压数据，最后一行用于创建存储处理后数据的目录。

```bash
%%bash
mkdir -p data write
cd data
test -f pbmc3k_filtered_gene_bc_matrices.tar.gz || curl https://cf.10xgenomics.com/ | tar -xzf pbmc3k_filtered_gene_bc_matrices.tar.gz
```

---
### 说明
点击**Edit on GitHub**按钮可下载本笔记本。在GitHub页面中，可右键**Raw**按钮并选择“链接另存为”下载；也可直接下载整个scanpy-tutorial仓库。

### 提示
在Jupyter笔记本与Jupyter Lab中，按**Shift + Tab**可查看Python函数的文档，按两次可展开完整视图。

```python
sc.settings.verbosity = 3  # 日志等级：错误(0)、警告(1)、信息(2)、提示(3)
sc.set_figure_params(dpi=80, facecolor="white")
sc.logging.print_header()
results_file = "write/pbmc3k.h5ad"  # 存储分析结果的文件
```

将计数矩阵读入**AnnData**对象，该对象可存储多种注释信息与数据的不同表示形式，自带基于HDF5的文件格式：**.h5ad**。

```python
adata = sc.read_10x_mtx(
    var_names="gene_symbols",  # 使用基因符号作为变量名
    "data/filtered_gene_bc_matrices/hg19/",  # 存放.mtx文件的目录
    cache=True,  # 写入缓存文件，加快后续读取速度
)
```

... 从缓存文件 cache/data-filtered_gene_bc_matrices-hg19-matrix.h5ad 中读取

```python
adata
```

AnnData 对象，细胞数×基因数 = 2700 × 32738
var: 'gene_ids'

```python
adata.var_names_make_unique()  # 若使用 var_names='gene_ids'，则无需此步
# 若要匹配R语言结果，执行：adata.X = adata.X.astype("int32")
```

```python
adata
```

AnnData 对象，细胞数×基因数 = 2700 × 32738
var: 'gene_ids'

