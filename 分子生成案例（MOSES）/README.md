分子生成案例 (MOSES - charRNN)
文献来源：Polykovskiy, D., Zhebrak, A., Sanchez-Lengeling, B., Golovanov, S., Tatanov, O., Belyaev, S., ... & Zhavoronkov, A. (2020). Molecular sets (MOSES): a benchmarking platform for molecular generation models. Frontiers in pharmacology, 11, 565644.

本项目是一个基于 MOSES 框架，使用字符级循环神经网络（charRNN）进行从头分子生成的教学案例。案例完整地展示了从数据预处理、模型训练、分子生成到性能评估的全过程。
目录
项目简介
项目结构
实验流程
如何使用
结果与讨论
拓展思考
主要依赖
项目简介
在现代药物研发中，利用AI设计具有理想化学特性的新分子已成为重要方向。本项目旨在：
教学员如何利用 MOSES 框架训练一个 charRNN 模型。
掌握从大型化学数据库（CHEMBL）获取和预处理分子数据的方法。
学习使用 MOSES 提供的标准化指标全面评估生成模型。
深入理解各项评估指标背后的化学与模型性能意义。
项目结构
Generated code
分子生成案例 (MOSES)/
├── data/
│   ├── generate_mol.csv     # 模型生成的分子
│   ├── test.csv             # 测试集
│   ├── test_stats.npz       # 测试集的预计算统计信息，用于加速评估
│   └── train.csv            # 训练集
│
└── moses/
    ├── char_rnn/
    │   ├── model.py         # charRNN 模型网络结构定义
    │   └── trainer.py       # 模型训练脚本
    │
    ├── dataset/
    │   └── dataset.py       # 数据集加载与处理
    │
    └── metrics/
        ├── metrics.py       # 计算所有评估指标的主脚本
        ├── NP_Score/        # Natural Product-likeness score 计算模块
        ├── SA_Score/        # Synthetic Accessibility score 计算模块
        └── mcf.csv          # 药物化学过滤器相关数据
Use code with caution.
data/: 存放所有与数据相关的文件，包括原始的训练/测试集和模型生成的结果。
moses/: MOSES 框架的核心代码。
char_rnn/: 包含 charRNN 模型的定义和训练逻辑。
dataset/: 负责数据的加载、预处理和特征化。
metrics/: 包含了用于评估生成分子质量的所有脚本和依赖。
实验流程
数据收集与预处理: 从 CHEMBL 数据库下载约45万个活性小分子。经过SMILES标准化、去重，并筛选出 Ki值小于1nM 的高活性分子，最终得到约25万个分子。
数据集划分: 将数据集按7:3的比例随机划分为训练集 (train.csv) 和测试集 (test.csv)。
模型训练: 使用 moses/char_rnn 中的模型，在训练集上进行训练。
特征化: SMILES字符串首先被转换为整数索引序列，再通过一个嵌入层（Embedding Layer）转换为模型能够处理的、包含化学语义的特征向量。
训练: 模型在训练集上被训练了 10个轮次 (Epochs)。
分子生成: 使用训练好的模型进行采样，生成 1000个 新分子，并保存到 data/generate_mol.csv。
性能评估: 使用 moses/metrics/metrics.py 脚本，将生成的分子与测试集进行比较，计算一系列标准化指标。
如何使用
1. 模型训练
使用 trainer.py 脚本和准备好的训练数据来训练模型。
注意: 上述命令为示例，请根据实际 trainer.py 的参数进行调整。训练好的模型权重将保存在 checkpoints/ 目录下。
2. 分子采样
使用训练好的模型权重生成新的分子。
3. 性能评估
最后，使用 metrics.py 脚本评估生成的分子的质量。
注意: 评估脚本需要提前计算好测试集的骨架和统计信息。
结果与讨论
模型生成1000个分子后的评估结果如下：
指标 (Metric)	结果	解释与意义
valid (有效率)	0.865	生成的SMILES中，能被化学软件成功解析为有效化学结构的比率。衡量模型是否学会了SMILES的基本语法。
unique@1000 (独特性)	0.993	生成的1000个分子中的非重复分子比例。高独特性表明模型生成的内容丰富多样，未陷入“模式崩溃”。
Novelty (新颖性)	0.787	生成的有效分子中，未在训练集中出现过的比例。这是衡量模型创造力的关键，表明模型在“创新”而非“记忆”。
FCD/Test (Fréchet ChemNet Distance)	1.913	衡量生成分子集与测试集在化学特征空间中的分布相似度。值越低，代表生成分子的整体化学特性越接近真实数据分布。
SNN/Test (最近邻相似度)	0.630	计算每个生成分子与测试集中最相似分子的平均Tanimoto相似度。高分值意味着生成分子与真实分子的化学类型和结构特征高度相关。
Frag/Test (片段相似度)	0.992	比较生成分子与测试集在化学片段（如官能团）上的分布相似度。接近1表示模型很好地复现了真实分子中常见的化学构建模块。
Scaf/Test (骨架相似度)	0.357	比较分子核心骨架（Scaffold）的分布相似度。此值偏低，可能表明模型在学习和生成多样化、复杂的环状核心结构方面存在挑战。
IntDiv (内部多样性)	0.875	生成分子集合内部成员间的平均不相似度。高分值表示生成的分子库本身结构多样，化学空间覆盖更广。
Filters (药物化学过滤器)	0.843	通过一组标准药物化学过滤器（如PAINS）的分子比例。高通过率意味着生成的分子更具“类药性”和开发潜力。
理化性质 (logP, SA, QED等)	< 0.1	衡量生成分子与测试集在关键理化性质分布上的差异。值越接近0，说明模型能准确复现真实分子的性质分布。

