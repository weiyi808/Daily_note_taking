---
type: concept
aliases: [世界模型, World Model]
---

# World Model

## 定义
通过学习环境状态转移与观测生成规律，对未来进行预测并支持规划的模型。

## 数学形式
$$
s_{t+1}\sim p_\theta(s_{t+1}\mid s_t,a_t),\quad
o_t\sim p_\theta(o_t\mid s_t)
$$

## 核心要点
1. 同时关注动力学可预测性与观测可生成性。
2. 可用于模型预测控制、想象轨迹与长时程决策。

## 代表工作
- [[WildWorld]]: 显式状态标注支持交互世界建模评测。

## 相关概念
- [[State Representation]]
- [[Action-conditioned Dynamics]]
