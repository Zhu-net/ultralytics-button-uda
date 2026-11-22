# Ultralytics UDA 实验说明与结果分析

远程来源：`/mnt/yihao/codes/buttonDet/ultralytics-uda`  
归档版本：分支 `button-uda`，提交 `9fe8fde4`  
本文中的路径均相对于当前 `ultralytics-uda/` 归档目录。

## 实验关系

```text
监督源模型：../ultralytics/runs/elevator/1010/train_qwen_adv_150/weights/best.pt
  ├── 1010：伪标签阈值和训练长度迭代（v1 → v2 → v3）
  ├── 1021：显式目标域数据
  ├── 1023_test：分辨率与数据增强消融
  └── 1026：更新后的源域/目标域数据配置（最终版本）
```

所有 UDA 实验默认 `unsup_ratio=0.25`、`unsup_weight=0.25`，使用 EMA teacher-student 伪标签训练。
训练目录保留 best/last 权重，但周期性 `epoch*.pt` 已排除。只有
`runs/elevator/1026/test_unsup_150` 保留完整 `visualizations/` 和 `fail_cases/` 图片。

## 训练目录与参数

| 目录 | 关键变量 | 实际轮数 | 源域 best epoch | 源域 best mAP50-95 | 最后一轮 mAP50-95 |
| --- | --- | ---: | ---: | ---: | ---: |
| `runs/elevator/1010/train_qwen_adv_unsup_150` | 高阈值：conf 0.9、IoU 0.7；patience 50 | 51 | 1 | 0.8442 | 0.8232 |
| `runs/elevator/1010/train_qwen_adv_unsup_150_v2` | 与 v1 同阈值，patience 0，完整 150 轮 | 150 | 1 | 0.8442 | 0.8139 |
| `runs/elevator/1010/train_qwen_adv_unsup_150_v3` | conf/IoU 0.6，楼层/灯光阈值 0.7 | 150 | 1 | 0.8442 | 0.8227 |
| `runs/elevator/1021/train_adv_unsup_150` | v3 阈值，显式 `elevator-target.yaml` | 150 | 1 | 0.8439 | 0.8025 |
| `runs/elevator/1023_test/train_flips0_mosaic0.5_closemosaic30` | 640；fliplr 0、mosaic 0.5、close_mosaic 30 | 150 | 3 | 0.8440 | 0.8098 |
| `runs/elevator/1023_test/train_imgsz960` | 960；默认 fliplr 0.5、mosaic 1.0 | 150 | 11 | 0.8391 | 0.8256 |
| `runs/elevator/1023_test/train_flips0_mosaic0.5_closemosaic30_imgsz960` | 960 加增强消融 | 150 | 13 | 0.8407 | 0.8231 |
| `runs/elevator/1026/train_unsup_150` | 更新为 `*_1026.yaml` 源域和目标域数据 | 150 | 1 | 0.8437 | 0.7846 |

这里的 best epoch 只由训练过程中使用的源域验证 mAP50-95 决定，并不等价于目标域最优轮次。

## 1010：伪标签阈值迭代结果

| 目录 | 权重/测试集 | mAP50 | mAP50-95 | 楼层完全准确率 | 灯光 F1 |
| --- | --- | ---: | ---: | ---: | ---: |
| `runs/elevator/1010/test_qwen_adv_unsup_150_best` | v1 best，标准测试 | 0.9692 | 0.8447 | 0.9653 | 0.8729 |
| `runs/elevator/1010/test_qwen_adv_unsup_150_last` | v1 last，标准测试 | 0.9691 | 0.8226 | 0.9810 | 0.9771 |
| `runs/elevator/1010/test_qwen_adv_unsup_150_v2_last` | v2 last，标准测试 | 0.9694 | 0.8160 | 0.9828 | 0.9840 |
| `runs/elevator/1010/test_qwen_adv_unsup_150_v2_u2v10929-1020` | v2，U2V 时间窗 | 0.9430 | 0.8437 | 0.9283 | 0.8750 |
| `runs/elevator/1010/test_qwen_adv_unsup_150_v3_last` | v3 last，标准测试 | 0.9765 | 0.8237 | 0.9833 | 0.9839 |
| `runs/elevator/1010/test_qwen_adv_unsup_150_v3_u2v10929-1020` | v3，U2V 时间窗 | 0.9684 | 0.8792 | 0.9299 | 0.8590 |

## 1021：显式目标域结果

| 目录 | 说明 | mAP50 | mAP50-95 | 楼层完全准确率 | 灯光 F1 |
| --- | --- | ---: | ---: | ---: | ---: |
| `runs/elevator/1021/test_adv_unsup_150_last` | 目标域 UDA last，标准测试 | 0.9775 | 0.8036 | 0.9841 | 0.9847 |
| `runs/elevator/1021/test_qwen_adv_150_u2v10929-1020` | 1021 组的 qwen-adv U2V 对照；未保存权重参数 | 0.9713 | 0.8865 | 0.9348 | 0.8235 |

第二个目录缺少 `args.yaml` 和明确权重路径，因此文档只记录其目录命名所表达的对照关系，不把它
强行归因于 `train_adv_unsup_150`。

## 1023：分辨率与增强消融

| 目录 | 设置/数据集 | mAP50 | mAP50-95 | 楼层完全准确率 | 灯光 F1 |
| --- | --- | ---: | ---: | ---: | ---: |
| `runs/elevator/1023_test/val_flips0_mosaic0.5_closemosaic30` | 640 增强消融，val | 0.9774 | 0.8107 | 0.9844 | 0.9823 |
| `runs/elevator/1023_test/test_flips0_mosaic0.5_closemosaic30` | 640 增强消融，test | 0.9640 | 0.8799 | 0.9336 | 0.8366 |
| `runs/elevator/1023_test/val_imgsz960` | 960 默认增强，val | 0.9806 | 0.8273 | 0.9792 | 0.9864 |
| `runs/elevator/1023_test/test_imgsz960` | 960 默认增强，test | 0.9373 | 0.8016 | 0.9138 | 0.7919 |
| `runs/elevator/1023_test/val_flips0_mosaic0.5_closemosaic30_imgsz960` | 960 增强消融，val | 0.9778 | 0.8220 | 0.9841 | 0.9857 |
| `runs/elevator/1023_test/test_flips0_mosaic0.5_closemosaic30_imgsz960` | 960 增强消融，test | 0.9388 | 0.8024 | 0.9128 | 0.8366 |

## 1026：最终数据版本

| 目录 | 数据集 | mAP50 | mAP50-95 | 楼层完全准确率 | 灯光 F1 |
| --- | --- | ---: | ---: | ---: | ---: |
| `runs/elevator/1026/val_unsup_150` | 1026 val | 0.9767 | 0.7851 | 0.9896 | 0.9840 |
| `runs/elevator/1026/test_unsup_150` | 1026 test；完整可视化 | 0.9822 | 0.8541 | 0.9821 | 0.8750 |

## 简单分析

1. **源域 mAP 不适合作为 UDA 的唯一选模指标。** 绝大多数 UDA 训练在第 1 轮就达到最高源域
   mAP50-95，之后逐渐下降；但 last 权重的楼层准确率和灯光 F1 往往显著高于 best。例如 v1 从 best
   到 last，mAP50-95 下降 0.0222，而楼层准确率增加 0.0157、灯光 F1 增加 0.1043。这是典型的
   多任务和域适应目标不一致。
2. **放宽伪标签阈值改善了 U2V 检测。** v3 相比 v2 的 U2V mAP50 从 0.9430 提升到 0.9684，
   mAP50-95 从 0.8437 提升到 0.8792；灯光 F1 小幅下降 0.0160。就目标域检测而言 v3 更好。
3. **960 分辨率没有带来测试收益。** 1023 中 640 增强消融的 test mAP50-95 为 0.8799，而两个
   960 实验仅约 0.802。虽然 960 在 val 上略高，但 val-to-test 落差明显，说明提高分辨率没有解决
   分布差异，还增加了计算成本。
4. **关闭水平翻转并降低 mosaic 对 640 测试最有效。** 该设置取得 1023 test 组最高 mAP50-95、
   楼层准确率和灯光 F1，应作为当前增强策略的优先候选。
5. **1026 在属性任务上很强，但与旧数据版本不能完全横比。** test 的楼层准确率 0.9821、mAP50
   0.9822 均很高；由于源域和目标域 YAML 都已更新，不能把它与 1010/1023 的差值全部归因于算法。
6. **val 与 test 的排序不一致。** 1026 val mAP50-95 为 0.7851、test 为 0.8541；1023 的 960
   实验也表现出明显的 val/test 反转。后续应固定同一数据版本并报告分场景结果。

## 使用建议

- 目标域模型选择应至少联合考察 mAP50-95、楼层完全准确率、灯光 F1 和 U2V 专用测试，不应只取
  源域 `best.pt`。
- 如果需要稳定复现实验，优先使用每个目录中的 `args.yaml`；目录名不足以还原 1021 对照实验的权重。
- 建议后续保存每次评估使用的 `model`、`data`、`split` 和 Git SHA，避免评估目录映射依赖命名推断。
- 数据集本体未归档，`scripts/*.yaml` 中仍是服务器绝对路径。
