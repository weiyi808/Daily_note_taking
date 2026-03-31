---
type: concept
aliases: [DiT, Diffusion Transformer]
---

# Diffusion Transformer

## 定义
以 Transformer 作为扩散模型主干，在去噪过程中注入条件控制信号进行生成。

## 数学形式
$$
\epsilon_\theta(\mathbf{x}_t, t, \mathbf{c}) \rightarrow \hat{\epsilon},\quad
\mathcal{L}_{diff}=\|\epsilon-\hat{\epsilon}\|_2^2
$$

## 核心要点
1. 容易融合多模态条件（文本、状态、骨架、相机）。
2. 适用于高维时空生成任务。

## 代表工作
- [[WildWorld]]: StateCtrl 将状态嵌入注入 DiT 中间层。

## 相关概念
- [[State Representation]]
- [[Interactive Video Generation]]
