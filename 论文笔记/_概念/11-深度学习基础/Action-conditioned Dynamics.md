---
type: concept
aliases: [动作条件动力学, Action-conditioned Dynamics]
---

# Action-conditioned Dynamics

## 定义
由动作驱动的状态转移过程，是交互世界模型的核心动力学假设。

## 数学形式
$$
s_{t+1}=f_\theta(s_t,a_t,\epsilon_t)
$$

## 核心要点
1. 需区分像素变化与真实状态变化。
2. 对长时程误差传播高度敏感。

## 代表工作
- [[WildWorld]]: 用显式状态监督该动力学学习。

## 相关概念
- [[World Model]]
- [[State Representation]]
