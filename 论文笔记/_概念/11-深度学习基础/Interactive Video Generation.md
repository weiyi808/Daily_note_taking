---
type: concept
aliases: [交互视频生成, Interactive Video Generation]
---

# Interactive Video Generation

## 定义
在生成视频时接收动作、相机、骨架或状态等控制信号，实现可控时序内容合成。

## 数学形式
$$
\hat{v}_{1:T}\sim p_\theta\!\left(v_{1:T}\mid v_1, c_{1:T}\right)
$$

## 核心要点
1. 控制信号的语义强度决定可控上限。
2. 长时程稳定性是关键难点。

## 代表工作
- [[WildWorld]]: 提供用于训练与评测的交互数据基座。

## 相关概念
- [[Camera Control]]
- [[State Representation]]
