---
title: FewRel训练流程
date: 2020-09-27 16:38:34
mathjax: true
categories:
- deep learning
tags:
- few-shot learning
- FewRel
---

介绍FewRel数据集的格式，模型的训练流程、输入输出、loss定义。

<!--more-->

FewRel数据集为json格式，用python的json包载入后，结构如下：

```python
{'class_id_1':[{'tokens': ['word_1', ... , 'word_n'], 'h': ['head_word', 'Q1331049', [[16]]], 't': ['tail_word', 'Q3056359', [[13, 14]]]}, ...],
'class_id_2':[{}, ...],
...}
```

# 数据集处理

`FewRelDataset`中一个样本为：

```python
"""
tuple(support_set, query_set, query_label)

support_set: dict{'word': list[], 'pos1': list[], 'pos2': list[], 'mask': list[] }
query_set: dict{'word': list[], 'pos1': list[], 'pos2': list[], 'mask': list[] }

'word', 'pos1', 'pos2', 'mask'的list中都含有N * K个元素,
每个元素都是torch.tensor, shape = [max_length], dtype=torch.int64, 其label与query_label对应

query_label: list[int]
[0, 0, ..., 0, 1, 1, ..., 1, 2, 2, ..., 2, ..., N - 1, N - 1, ..., N - 1]
"""
```

`data_loader.get_loader`方法返回`iter(data_loader)`

`next(data_loader)`得到的数据为：`tuple(batch_support, batch_query, batch_label)`

```python
"""
batch_support, batch_query: dict{'word': tensor, 'pos1': tensor, 'pos2': tensor, 'mask': tensor}
    
torch.Size([batch_size * N * K, maxlen])
batch_label: torch.Size([batch_size * N * K])
"""
```

# 模型训练

```shell
python train_demo.py --test val_nyt --model proto --encoder cnn
```

## 指定数据集

| 参数    | 默认值     | 备注 |
| ------- | ---------- | ---- |
| --train | train_wiki |      |
| --val   | val_wiki   |      |
| --test  | test_wiki  |      |

## 任务设置

| 参数     | 默认值 | 备注                     |
| -------- | ------ | ------------------------ |
| --trainN | 10     | 训练时的N                |
| --N      | 5      | N-way                    |
| --K      | 5      | K-shot                   |
| --Q      | 5      | 查询集中每个类别样本数量 |

## 训练设置

| 参数           | 默认值 | 备注                             |
| -------------- | ------ | -------------------------------- |
| --batch_size   | 4      |                                  |
| --train_iter   | 30000  | 训练迭代次数                     |
| --val_iter     | 1000   | 验证迭代次数                     |
| --test_iter    | 10000  | 测试迭代次数                     |
| --val_step     | 2000   | 训练val_step步之后，进行一次验证 |
| --lr           | -1     | 学习速率                         |
| --weight_decay | 1e-5   | weight decay                     |
| --only_test    | false  | 只进行测试                       |

每次迭代从data_loader获取一个batch，而每个batch使用的类别与样本是随机选取的。

## 模型设置
| 参数          | 默认值 | 备注                      |
| ------------- | ------ | ------------------------- |
| --model       | proto  | 模型名字                  |
| --encoder     | cnn    | 编码器：cnn或bert         |
| --max_length  | 128    | 句子最大长度              |
| --hidden_size | 230    | 隐藏层单元数量            |
| --dropout     | 0.0    |                           |
| --grad_iter   | 1      | 累积grad_iter次迭代的梯度 |
| --optim       | sgd    | sgd / adam / adamw        |

## 保存/载入断点
| 参数        | 默认值 | 备注                             |
| ----------- | ------ | -------------------------------- |
| --load_ckpt | None   | 断点文件路径，                   |
| --save_ckpt | None   | 若不为None，将替换默认生成的名字 |
| --ckpt_name | ''     | 断点名字，加在默认生成的名字后   |

