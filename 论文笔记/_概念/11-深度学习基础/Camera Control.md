---
type: concept
aliases: [相机控制, Camera Control]
---

# Camera Control

## 定义
评估生成视频是否遵循目标相机轨迹与视角变化。

## 数学形式
$$
\mathrm{ATE}=\frac{1}{T}\sum_t \| \mathbf{T}^{gt}_t-\mathbf{T}^{pred}_t \|,\quad
\mathrm{RPE}=\frac{1}{T-1}\sum_t \| \Delta\mathbf{T}^{gt}_t-\Delta\mathbf{T}^{pred}_t \|
$$

## 核心要点
1. ATE 衡量全局偏差，RPE 衡量局部漂移。
2. 对交互体验与可观察性有直接影响。

## 代表工作
- [[WildWorld]]: 用 CamCtrl 与 StateCtrl 进行系统对比。

## 相关概念
- [[Interactive Video Generation]]
- [[Benchmark Design]]
