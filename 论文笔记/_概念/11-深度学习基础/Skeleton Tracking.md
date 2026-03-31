---
type: concept
aliases: [骨架跟踪, Skeleton Tracking]
---

# Skeleton Tracking

## 定义
在视频中跟踪人体/角色关键点轨迹，用于动作与状态一致性分析。

## 数学形式
$$
\hat{\mathbf{p}}_{k,t+1}=h_\psi(\hat{\mathbf{p}}_{k,t}, I_t, I_{t+1})
$$

## 核心要点
1. 关键点轨迹是状态代理信号。
2. 可与多阈值精度结合构建鲁棒指标。

## 代表工作
- [[WildWorld]]: 使用 TAPNext 跟踪计算 State Alignment。

## 相关概念
- [[State Alignment]]
- [[Action Following]]
