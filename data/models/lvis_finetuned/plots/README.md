# 训练结果图对比（四个模型）

按模型区分存放，每个目录含完整官方图：`results.png`（训练曲线）、`confusion_matrix(_normalized).png`（混淆矩阵）、`Box{PR,F1,P,R}_curve.png`（PR/F1/P/R 曲线）、`results.csv`（原始指标）。

## 目录说明

| 目录 | 模型 | 轮数 | mAP50 | mAP50-95 | 说明 |
|------|------|:---:|:---:|:---:|------|
| `yolo26n/` | YOLO26n | 73 | 0.203 | 0.139 | 早期 nano 模型，精度基线最低 |
| `yolo26m_run1/` | YOLO26m Run1 | 47 | 0.272 | 0.185 | Ep1-24 默认增强(峰值0.194)，Ep25起激进增强导致负优化 |
| `yolo26m_run2_final/` | YOLO26m Run2 | 100(完成) | **0.308** | **0.216** | 从头训练+中等增强，峰值 Ep82，全程最优 |
| `yolo26l/` | YOLO26l Run3 | 11(暂停) | 0.260 | 0.181 | imgsz896 大模型，训练至 Ep11 暂停的阶段性备份，尚未收敛 |

> **yolo26l 为阶段性快照**：训练至 Ep11 手动暂停，配置 batch16/imgsz896/lr0=0.01/cos_lr/warmup3/patience30，增强 mosaic0.5/mixup0.1/cutmix0.2/erasing0.2/close_mosaic15。当前 mAP50-95=0.181 仍在爬升期（未达 26m 水平），断点 `last.pt`(Ep11) 保留可随时 resume 续训到 Ep50。发布模型 `../yolo26l_lvis_best.pt`(已 strip 至 54MB)。

> **Run2 已完整训练 100 轮**：mAP50-95 在 Ep82 达峰值 0.2159，之后 cosine LR 尾段轻微回落至 Ep100=0.2085。`best.pt`(Ep82) 为最优权重，对应 `../yolo26m_lvis_run2_best.pt`。`results.png` 为完整 100 轮训练曲线；val 曲线基于 best.pt(Ep82)。

## 怎么看训练结果（推荐顺序）

1. **`results.png`（最直观）**：一图看全部指标随 epoch 变化——loss 下降、mAP 上升、是否过拟合。
   直接对比三个模型的 results.png 即可看出精度演进：0.139 → 0.185 → 0.216。
2. **`BoxPR_curve.png`**：PR 曲线，越靠右上越好，曲线下面积≈mAP50。
3. **`BoxF1_curve.png`**：F1 峰值点对应最佳置信度阈值，部署设阈值用。
4. `confusion_matrix_normalized.png`：看每类召回率（300类，关注对角线亮度）。

## 关键结论

- **Run2（yolo26m_run2_final）为最优**：峰值 mAP50-95=0.2159(Ep82)，比 Run1 峰值(0.194)高 +0.022，且训练曲线健康单调上升。
- Run1 的 results.png 可清晰看到 Ep25 后激进增强导致的曲线掉头（负优化教训）。
- 对应模型权重见上级目录：`yolo26m_lvis_run2_best.pt`（Run2 Ep82 峰值）、`yolo26m_lvis_ep24_original.pt`（Run1 Ep24）。
