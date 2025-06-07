AI在有机化学中的应用——邮寄反应产率预测案例

1. 数据存放在data文件中，“data.csv”每一列包含 base、ligand、additive、aryl halide反应物等信息。
   “model_data.csv”只包含了计算的特征描述符，和需要预测的产率（yield），训练模型使用model_data.csv
2. under_stand_data.ipynb 包含了每个组分计算的特征描述符。  
3. 完整的流程，导入并分割数据、训练模型、预测、解释模型在main.ipynb中。

4. 论文和论文中的SI
