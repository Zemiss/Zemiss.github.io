---
layout:     post
title:      "3k 个外周血单个核细胞的预处理与聚类"
subtitle:   " \"开始学习数据处理\""
date:       2026-04-11 23:00:00
author:     "Zemiss"
header-img: "img/bg-weeklyplan.jpg"
catalog: true
tags:
  - Study
---

# 预处理之前

## 目录
- 预处理
- 主成分分析
- 计算邻近图
- 邻近图降维可视化
- 邻近图聚类
- 寻找标记基因

```python
from __future__ import annotations
import matplotlib.pyplot as plt
import pandas as pd
import scanpy as sc
```

2017年5月，本文档最初用于演示：**Scanpy 能够复现 Seurat 官方引导式聚类教程的大部分分析结果**（Satija 等人，2015）。
在此衷心感谢 Seurat 的作者们提供该教程！在此期间，我们对流程做了一些增删调整。



### 数据说明
数据集来自**1位健康捐献者的3k个PBMC细胞**，可从10x Genomics官网免费获取（链接见原文）。
在类Unix系统中，可取消注释并运行以下代码下载并解压数据。最后一行用于创建存放处理后数据的目录。

```bash
%%bash
mkdir -p data write
cd data
test -f pbmc3k_filtered_gene_bc_matrices.tar.gz || curl https://cf.10xgenomics.com/ | tar -xzf pbmc3k_filtered_gene_bc_matrices.tar.gz
```



### 提示
- 点击页面上的**Edit on GitHub**按钮可下载该笔记本。在GitHub页面，可右键点击**Raw**按钮，选择“链接另存为”下载。也可直接下载整个 scanpy-tutorial 仓库。
- 在Jupyter Notebook与Jupyter Lab中，按 **Shift + Tab** 可查看Python函数的文档；按两次可展开完整说明。

---

```python
# 日志等级：错误(0)、警告(1)、信息(2)、提示(3)
sc.settings.verbosity = 3
# 设置绘图参数：分辨率80，背景白色
sc.set_figure_params(dpi=80, facecolor="white")
# 打印Scanpy版本等头部信息
sc.logging.print_header()

# 定义用于保存分析结果的文件路径
results_file = "write/pbmc3k.h5ad"
```



## 读取数据
将计数矩阵读入为**AnnData对象**。该对象可存储多种注释信息与数据的不同表示形式，自带基于HDF5的文件格式：`.h5ad`。

```python
adata = sc.read_10x_mtx(
    "data/filtered_gene_bc_matrices/hg19/",  # 存放.mtx文件的目录
    var_names="gene_symbols",       # 使用基因符号作为变量名
    cache=True,                     # 写入缓存文件，加快后续读取
)
```

输出：
`... reading from cache file cache/data-filtered_gene_bc_matrices-hg19-matrix.h5ad`

```python
adata
```

输出：
`AnnData object with n_obs × n_vars = 2700 × 32738`
`var: 'gene_ids'`



### 提示
如需更全面了解AnnData，可查看官方文档**Getting started with anndata**。

```python
# 确保基因名唯一（若使用var_names='gene_ids'则无需此步）
adata.var_names_make_unique()

# 若要与R语言结果完全匹配，执行：
# adata.X = adata.X.astype("int32")

adata
```

输出：
`AnnData object with n_obs × n_vars = 2700 × 32738`
`var: 'gene_ids'`



## 预处理
展示**所有细胞中，单个细胞内计数占比最高**的前20个基因。
```python
sc.pl.highest_expr_genes(adata, n_top=20)
```
对每个细胞的计数进行标准化
完成（耗时0:00:01）

### 基础过滤
```python
sc.pp.filter_cells(adata, min_genes=200)  # 本案例中无实际效果
sc.pp.filter_genes(adata, min_cells=3)
```

```python
adata
```
过滤掉**在少于3个细胞中表达**的19024个基因
AnnData 对象：细胞数×基因数 = 2700 × 13714
obs: 'n_genes'
var: 'gene_ids', 'n_cells'

接下来标注**线粒体基因**，这对质控至关重要。

引用自《Simple Single Cell》工作流（Lun等，2016）：
线粒体基因比例过高，通常提示**细胞质量差**，可能是细胞膜破损导致胞质RNA流失。原因是线粒体比转录本分子大，更不容易从膜破损处漏出。

使用 `pp.calculate_qc_metrics` 可高效计算多项质控指标。
```python
# 将以 MT- 开头的基因标注为线粒体基因
adata.var["mt"] = adata.var_names.str.startswith("MT-")
sc.pp.calculate_qc_metrics(adata, qc_vars=["mt"], percent_top=None, log1p=False, inplace=True)
```

绘制小提琴图，展示以下质控指标：
- 计数矩阵中每个细胞表达的基因数
- 每个细胞的总计数
- 线粒体基因计数占比

```python
sc.pl.violin(
    adata,
    ["n_genes_by_counts", "total_counts", "pct_counts_mt"],
    jitter=0.4, multi_panel=True,
)
```

剔除**线粒体基因表达过高**或**总计数异常**的细胞：
```python
fig, axs = plt.subplots(1, 2, figsize=(10, 4), layout="constrained")
sc.pl.scatter(adata, x="total_counts", y="pct_counts_mt", show=False, ax=axs[0])
sc.pl.scatter(adata, x="total_counts", y="n_genes_by_counts", show=False, ax=axs[1])
```

通过切片AnnData对象执行实际过滤：
```python
adata = adata[
    (adata.obs.n_genes_by_counts < 2500) & (adata.obs.n_genes_by_counts > 200) & (adata.obs.pct_counts_mt < 5)
].copy()
adata.layers["counts"] = adata.X.copy()
```

```python
adata
```
AnnData 对象：细胞数×基因数 = 2638 × 13714
obs: 'n_genes', 'n_genes_by_counts', 'total_counts', 'total_counts_mt', 'pct_counts_mt'
var: 'gene_ids', 'n_cells', 'mt', 'n_cells_by_counts', 'mean_counts', 'pct_dropout'
layers: 'counts'

对数据进行**总计数标准化**（文库大小校正），使每个细胞统一为10000条读数，确保细胞间计数可比。
```python
sc.pp.normalize_total(adata, target_sum=1e4)
```
对每个细胞的计数进行标准化
完成（耗时0:00:00）

### 对数化处理
```python
sc.pp.log1p(adata)
```

### 筛选高变异基因
不计并列情况，结果与Seurat教程完全一致。
```python
sc.pp.highly_variable_genes(
    adata,
    layer="counts",
    n_top_genes=2000, min_mean=0.0125,
    max_mean=3,
    min_disp=0.5,
    flavor="seurat_v3",
)
```
提取高变异基因
→ 添加：
'highly_variable'：布尔向量（adata.var）
'highly_variable_rank'：浮点向量（adata.var）
'means'：浮点向量（adata.var）
'variances'：浮点向量（adata.var）
'variances_norm'：浮点向量（adata.var）

```python
sc.pl.highly_variable_genes(adata)
```

### 注意
高变异基因筛选结果会保存在 `.var["highly_variable"]` 注释中，PCA、`sc.pp.neighbors` 及后续流形/图工具会自动识别，因此**无需再手动过滤基因**。

对每个基因**缩放至单位方差**，截断超过10倍标准差的数值。
若要匹配Seurat的PBMC3k教程，可仅回归校正 `pct_counts_mt`，是否子集为高变异基因为可选步骤。
```python
adata.layers["scaled"] = adata.X.toarray()
sc.pp.regress_out(adata, ["total_counts", "pct_counts_mt"], layer="scaled")
sc.pp.scale(adata, max_value=10, layer="scaled")
```
回归校正 ['total_counts', 'pct_counts_mt']
完成（耗时0:00:02）



## 主成分分析
通过**主成分分析（PCA）** 对数据降维，揭示主要变异轴并去除噪声。
```python
sc.pp.pca(adata, layer="scaled", svd_solver="arpack")
```
计算PCA，主成分数=50
完成（耗时0:00:01）

可绘制PCA坐标散点图，后续分析暂不使用。
```python
sc.pl.pca(adata, annotate_var_explained=True, color="CST3")
```

查看**单个主成分对总方差的贡献**，用于确定计算细胞近邻关系时需保留的主成分数（如聚类 `sc.tl.louvain()`、tSNE `sc.tl.tsne()`）。经验表明，大致估算即可。
```python
sc.pl.pca_variance_ratio(adata, n_pcs=20)
```

尽管算法不同导致成分略有差异，但主成分排名与Seurat教程一致（仅符号可能相反）。
```python
sc.pl.pca_loadings(adata, components=(1, 2), include_lowest=True)
```



## 计算近邻图
基于PCA结果计算**细胞近邻图**，可直接使用默认参数；为复现Seurat结果，使用以下参数。
```python
sc.pp.neighbors(adata, n_neighbors=10, n_pcs=40)
```
基于 'X_pca' 计算近邻，主成分数=40
完成：添加至 `.uns['neighbors']`
`.obsp['connectivities']`：加权邻接矩阵（耗时0:00:04）
`.obsp['distances']`：每对近邻的距离



## 近邻图降维可视化
推荐使用**UMAP**将图降至二维（McInnes等，2018）。相比tSNE，UMAP能更好保留流形全局结构与轨迹信息。
偶尔会出现聚类不连通等结构异常，通常可通过以下代码修复：
```python
sc.pl.paga(adata, plot=False)  # 去掉 plot=False 可查看粗粒度图
sc.tl.paga(adata)
sc.tl.umap(adata, init_pos='paga')
```

```python
sc.tl.umap(adata)
```
计算UMAP
完成：添加 'X_umap'（UMAP坐标，存于adata.obsm）、'umap'（UMAP参数，存于adata.uns）
耗时0:00:02

```python
sc.pl.umap(adata, color=["CST3", "NKG7", "PPBP"])
```

由于已将 `adata.X` 设为标准化数据，上述图展示**标准化基因表达量**；也可直接绘制原始计数，或“scaled_hvg”（标准化→对数化→缩放）数据。
```python
sc.pl.umap(adata, color=["CST3", "NKG7", "PPBP"], layer="counts")
sc.pl.umap(adata, color=["CST3", "NKG7", "PPBP"], layer="scaled")
```



## 近邻图聚类
与Seurat等框架一致，推荐使用**Leiden图聚类算法**（基于模块度优化的社区发现算法，Traag等，2019）。该算法直接对已构建的细胞近邻图聚类。
```python
sc.tl.leiden(
    adata,
    resolution=0.7,
    random_state=0,
    flavor="igraph",
    n_iterations=2,
    directed=False,
)
adata.obs["leiden"] = adata.obs["leiden"].copy()
adata.uns["leiden"] = adata.uns["leiden"].copy()
adata.obsm["X_umap"] = adata.obsm["X_umap"].copy()
```
运行Leiden聚类
完成：得到8个聚类，添加 'leiden'（聚类标签，存于adata.obs，分类变量）
耗时0:00:00

绘制聚类结果，与Seurat结果高度吻合。
```python
sc.pl.umap(adata, color=["leiden", "CD14", "NKG7"])
```



## 寻找标记基因
计算**每个聚类的差异表达基因排名**，最简单快速的方法是t检验。
```python
sc.tl.rank_genes_groups(adata, "leiden", mask_var="highly_variable", method="t-test")
sc.pl.rank_genes_groups(adata, n_genes=25, sharey=False)
```
基因排名
完成：添加至 `.uns['rank_genes_groups']`
'names'：按分组索引的排序后数组
'scores'：按分组索引的排序后数组
'logfoldchanges'：按分组索引的排序后数组
'pvals'：按分组索引的排序后数组
'pvals_adj'：按分组索引的排序后数组
耗时0:00:00

```python
sc.settings.verbosity = 2  # 降低日志等级
```

**Wilcoxon秩和检验（Mann-Whitney-U检验）** 结果与t检验接近，**发表文章推荐使用**（参考Soneson & Robinson，2018）。
也可使用更强大的差异分析工具，如MAST、limma、DESeq2，以及Python的diffxpy。
```python
sc.tl.rank_genes_groups(adata, "leiden", mask_var="highly_variable", method="wilcoxon")
sc.pl.rank_genes_groups(adata, n_genes=25, sharey=False)
```
基因排名
完成（耗时0:00:03）

保存结果：
```python
adata.write(results_file)
```

另一种方法是**逻辑回归**（Ntranos等，2019）。核心区别：逻辑回归为**多变量分析**，传统差异检验为单变量分析（详见Clark等，2014）。
```python
sc.tl.rank_genes_groups(adata, "leiden", mask_var="highly_variable", method="logreg")
sc.pl.rank_genes_groups(adata, n_genes=25, sharey=False)
```
基因排名
完成（耗时0:00:03）

这些基因大多为高表达基因，少数例外如CD8A（见下方点图）。

| Leiden分组 | 标记基因 | 细胞类型 |
| --- | --- | --- |
| 0 | IL7R | CD4+ T细胞 |
| 1 | CD14、LYZ | CD14+ 单核细胞 |
| 2 | MS4A1 | B细胞 |
| 3 | CD8A | CD8+ T细胞 |
| 4 | GNLY、NKG7 | NK细胞 |
| 5 | FCGR3A、MS4A7 | FCGR3A+ 单核细胞 |
| 6 | FCER1A、CST3 | 树突状细胞 |
| 7 | PPBP | 巨核细胞 |

定义标记基因列表，供后续使用：
```python
marker_genes = [
    *["IL7R", "CD79A", "MS4A1", "CD8A", "CD8B", "LYZ", "CD14"],
    *["LGALS3", "S100A8", "GNLY", "NKG7", "KLRB1"],
    *["FCGR3A", "MS4A7", "FCER1A", "CST3", "PPBP"],
]
```

```python
adata = sc.read(results_file)
```

以数据框形式展示**每个聚类（0-7）排名前10的基因**，取前5行：
|  | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | LTB | CD74 | LYZ | NKG7 | CCL5 | LST1 | HLA-DPA1 | PF4 |
| 1 | IL32 | CD79A | S100A9 | GNLY | NKG7 | FCER1G | HLA-DPB1 | SDPR |
| 2 | IL7R | HLA-DRA | S100A8 | GZMB | CST7 | AIF1 | HLA-DRB1 | GNG11 |
| 3 | MALAT1 | CD79B | TYROBP | CTSW | GZMA | FCGR3A | HLA-DRA | PPBP |
| 4 | CD2 | HLA-DPB1 | FTL | PRF1 | B2M | COTL1 | CD74 | NRGN |

获取包含分数与分组的表格：
```python
result = adata.uns["rank_genes_groups"]
groups = result["names"].dtype.names
pd.DataFrame({f"{group}_{key[:1]}": result[key][group] for group in groups for key in ["names", "pvals"]})
```

|  | 0_n | 0_p | 1_n | 1_p | 2_n | 2_p | 3_n | 3_p | 4_n | 4_p |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | LTB | 4.523933e-135 | CD74 | 1.972305e-184 | LYZ | 5.016654e-251 | NKG7 | 4.220384e-90 | CCL5 | 1.8572e-132 |
| 1 | IL32 | 1.391499e-112 | CD79A | 1.882747e-169 | S100A9 | 1.126831e-248 | GNLY | 9.274173e-87 | NKG7 | 6.1677e-112 |
| 2 | IL7R | 2.756677e-110 | HLA-DRA | 2.840747e-168 | S100A8 | 6.457752e-238 | GZMB | 1.164234e-85 | CST7 | 2.2454e-88 |
| 3 | MALAT1 | 4.433395e-88 | CD79B | 1.868216e-154 | TYROBP | 1.259221e-223 | CTSW | 5.115917e-81 | GZMA | 6.8968e-83 |
| 4 | CD2 | 4.504927e-54 | HLA-DPB1 | 3.333383e-148 | FTL | 1.887168e-213 | PRF1 | 3.618719e-79 | B2M | 3.7051e-82 |

### 单聚类对比分析
```python
sc.tl.rank_genes_groups(
    adata, "leiden",
    mask_var="highly_variable",
    groups=["0"],
    reference="1", method="wilcoxon",
)
sc.pl.rank_genes_groups(adata, groups=["0"], n_genes=20)
```
完成（耗时0:00:00）
基因排名

若需查看某一分组的详细结果，使用 `sc.pl.rank_genes_groups_violin`：
```python
sc.pl.rank_genes_groups_violin(adata, groups="0", n_genes=8)
```

重新读取已计算差异表达的对象（与其他组对比得到的差异基因）：
```python
adata = sc.read(results_file)
sc.pl.rank_genes_groups_violin(adata, groups="0", n_genes=8)
```

若需**跨分组对比特定基因**，使用以下代码：
```python
sc.pl.violin(adata, ["CST3", "NKG7", "PPBP"], groupby="leiden")
```

### 标注细胞类型
```python
new_cluster_names = [
    "CD4 T", "CD14+ 单核细胞",
    "B", "CD8 T",
    "NK", "FCGR3A+ 单核细胞",
    "树突状细胞", "巨核细胞",
]
adata.rename_categories("leiden", new_cluster_names)
```

```python
sc.pl.umap(adata, color="leiden", legend_loc="on data", title="", frameon=False)
```
警告：图片已保存至文件 figures/umap.pdf

标注细胞类型后，可视化标记基因：
```python
sc.pl.dotplot(adata, marker_genes, groupby="leiden")
```

也可使用更紧凑的堆叠小提琴图：
```python
sc.pl.stacked_violin(adata, marker_genes, groupby="leiden")
```

分析过程中，AnnData对象累计存储了以下注释信息：
```python
adata
```
AnnData 对象：细胞数×基因数 = 2638 × 13714
obs: 'n_genes', 'n_genes_by_counts', 'total_counts', 'total_counts_mt', 'pct_counts_mt'
var: 'gene_ids', 'n_cells', 'mt', 'n_cells_by_counts', 'mean_counts', 'pct_dropout'
uns: 'hvg', 'leiden', 'leiden_colors', 'log1p', 'neighbors', 'pca', 'rank_genes_groups'
obsm: 'X_pca', 'X_umap'
varm: 'PCs'
layers: 'counts', 'scaled'
obsp: 'connectivities', 'distances'

```python
# compression='gzip' 可节省磁盘空间，但会略微减慢读写速度
adata.write(results_file, compression="gzip")
```

可使用 `h5ls` 快速查看文件概览（支持多种参数），详情见官方文档。
该文件格式未来可能进一步优化，但所有读取函数将保持向后兼容。

