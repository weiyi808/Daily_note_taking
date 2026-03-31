---
type: concept
aliases: [状态表征, State Representation]
---

# State Representation

## 定义
将环境可观测与隐含变量编码为可学习特征，以支持转移预测与生成控制。

## 数学形式
$$
z_t = g_\phi(x_t),\quad s_t=[z_t, u_t]
$$

## 核心要点
1. 可包含离散状态与连续状态。
2. 表征质量直接影响控制精度与稳定性。

## 代表工作
- [[WildWorld]]: 提供显式状态标签用于监督学习。

## 相关概念
- [[State Alignment]]
- [[Diffusion Transformer]]
