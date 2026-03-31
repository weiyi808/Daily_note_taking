---
type: concept
aliases: [状态对齐, State Alignment]
---

# State Alignment

## 定义
通过关键点轨迹误差衡量生成视频的状态演化是否与真值一致。

## 数学形式
$$
\mathrm{SA}=\frac{1}{KT}\sum_{k,t}\frac{1}{|\Theta|}\sum_{\delta\in\Theta}
\mathbf{1}(\|\hat{\mathbf{p}}_{k,t}-\mathbf{p}_{k,t}\|_2\le \delta)
$$

## 核心要点
1. 兼顾时间一致性与空间误差。
2. 对长期漂移更敏感。

## 代表工作
- [[WildWorld]]: 在交互世界模型中提出并验证。

## 相关概念
- [[Skeleton Tracking]]
- [[Action Following]]
