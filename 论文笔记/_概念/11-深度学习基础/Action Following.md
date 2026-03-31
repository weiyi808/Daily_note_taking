---
type: concept
aliases: [动作跟随, Action Following]
---

# Action Following

## 定义
评估生成视频是否按输入动作执行对应行为的指标。

## 数学形式
$$
\mathrm{AF} = \frac{1}{N}\sum_{i=1}^{N}\mathbf{1}(\hat{y}_i=y_i)
$$

## 核心要点
1. 按动作片段粒度评估更细致。
2. 可结合人评或 VLM 判别实现自动化。

## 代表工作
- [[WildWorld]]: 在 WildBench 中系统化定义该指标。

## 相关概念
- [[State Alignment]]
- [[Benchmark Design]]
