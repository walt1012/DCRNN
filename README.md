# DCRNN模型

DCRNN主要是用来进行区域范围的交通速度预测的模型。首先需要获取到区域范围内的传感器数据，并使用k-way算法对其进行建模分区；再利用历史交通数据对各个分区进行学习训练，分区重叠的节点会进行优化；最终得到完整区域的交通预测模型。


![Diffusion Convolutional Recurrent Neural Network](figures/model_architecture.jpg "Model Architecture")

参考文献：[Diffusion Convolutional Recurrent Neural Network: Data-Driven Traffic Forecasting](https://arxiv.org/abs/1707.01926)

## 安装依赖
```bash
pip install -r requirements.txt
```

## 数据集
- 代码中使用了METR-LA数据集作为案例，原始数据在 `data/metr-la.h5`，需要使用`scripts/generate_training_data.py`对原始数据集进行处理，按照(7:1:2)生成训练集、验证集和测试集。
```bash
python -m scripts.generate_training_data --output_dir=data/METR-LA --traffic_df_filename=data/metr-la.h5
```
- 输入(每五分钟每个传感器的速度数据:mile/h):

|                     | sensor_0 | sensor_1 | sensor_2 | sensor_n |
|:-------------------:|:--------:|:--------:|:--------:|:--------:|
| 2018/01/01 00:00:00 |   60.0   |   65.0   |   70.0   |    ...   | 
| 2018/01/01 00:05:00 |   61.0   |   64.0   |   65.0   |    ...   |
| 2018/01/01 00:10:00 |   63.0   |   65.0   |   60.0   |    ...   |
|         ...         |    ...   |    ...   |    ...   |    ...   |
- 输出(速度的预测值和真实值:mile/h):

| predictions | groundtruth | 
|:--------:|:--------:|
|   70.0   |    63.5   |
|   65.0   |    57.75  | 
|   60.0   |    66.75  |
|    ...   |    ...    |

## 区域建模
需要用到传感器的数据（传感器编号，位置信息等）,`data/sensor/graph/graph_sensor_ids.csv`和`data/sensor_graph/graph_sensor_locations.csv`中记录了传感器的编号和经纬度。
```bash
# 生成传感器邻接矩阵
python -m scripts.gen_adj_mx  --sensor_ids_filename=data/sensor_graph/graph_sensor_ids.txt --normalized_k=0.1 --output_pkl_filename=data/sensor_graph/adj_mx.pkl
```

## 模型训练
```bash
python dcrnn_train.py --config_filename=data/model/dcrnn_la.yaml
```

## 模型评估
```bash
python -m scripts.eval_baseline_methods --traffic_reading_filename=data/metr-la.h5
```
## 模型预测
```bash
python run_demo.py --config_filename=data/model/dcrnn_la.yaml
```