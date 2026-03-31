---
title: "LeWorldModel: Stable End-to-End Joint-Embedding Predictive Architecture from Pixels"
method_name: "LeWM"
authors: [Lucas Maes, Quentin Le Lidec, Damien Scieur, Yann LeCun, Randall Balestriero]
year: 2026
venue: arXiv
tags: [world-model, jepa, latent-dynamics, offline-rl, model-predictive-control]
zotero_collection: _待整理
image_source: online
arxiv_html: https://arxiv.org/html/2603.19312v2
created: 2026-04-01
---

# 论文笔记：LeWorldModel

## 元信息

| 项目 | 内容 |
|---|---|
| 论文 | LeWorldModel: Stable End-to-End Joint-Embedding Predictive Architecture from Pixels |
| 方法名 | LeWM |
| 任务 | 基于像素的[[世界模型]]学习与[[模型预测控制]] |
| 场景 | 2D/3D 控制任务：Push-T、Reacher、Two-Room、OGBench-Cube |
| 代码 | https://github.com/lucas-maes/le-wm |
| 项目页 | https://le-wm.github.io/ |
| arXiv | https://arxiv.org/abs/2603.19312 |

## 一句话总结

> LeWM 用“预测损失 + 高斯正则(SIGReg)”两项损失稳定训练端到端 JEPA 世界模型，在少参数与低算力下实现接近 SOTA 的规划效果与显著推理提速。

## 核心贡献

1. 提出首个无需 EMA/stop-grad/预训练编码器的稳定端到端 JEPA 世界模型。
2. 将损失项从复杂多项压缩为两项，并把关键可调超参基本收敛到一个（$\lambda$）。
3. 在离线、无奖励、纯像素条件下，规划速度相对 foundation-based world model 最高约 48x。
4. 通过 probing 与 violation-of-expectation 评估，展示潜变量包含可解释的物理结构。

## 问题背景

### 要解决的问题

现有[[联合嵌入预测架构]]（JEPA）在世界模型中常发生表示塌缩：编码器把不同观测映射为近常量向量，导致预测看似“容易”但失去可控语义。

### 现有方法局限

- 依赖[[指数滑动平均]]、stop-gradient、额外监督或预训练基础视觉模型。
- 端到端方案（如多项 VICReg 风格目标）超参多、训练曲线噪声大、迁移不稳。
- foundation-based 方案虽然稳，但计算与规划代价高，且牺牲端到端表示学习自由度。

### 本文动机

作者希望在“可证明反塌缩 + 端到端 + 低复杂度”之间取得平衡：只保留任务必需目标（未来嵌入预测）并用统计正则约束潜变量分布。

## 方法详解

### 模型架构

LeWM 包含两个核心模块：

- **Encoder**：将观测图像 $\mathbf{o}_t$ 编码为潜变量 $\mathbf{z}_t$。
- **Predictor**：在动作 $\mathbf{a}_t$ 条件下预测下一时刻潜变量 $\hat{\mathbf{z}}_{t+1}$。

具体实现要点：

- 编码器为 ViT-tiny（约 5M 参数），以 `[CLS]` 表征投影到训练潜空间。
- 预测器为 6 层 Transformer（约 10M 参数），使用[[自适应层归一化]]注入动作。
- 总参数量约 15M，可单卡数小时训练完成。
- 训练不使用重建损失，不使用 reward，不使用外部状态标签。

### 训练流程（offline, reward-free）

1. 从离线轨迹采样序列 $(\mathbf{o}_{1:T}, \mathbf{a}_{1:T})$。
2. 编码得到 $\mathbf{z}_{1:T}$，预测器自回归生成 $\hat{\mathbf{z}}_{2:T}$。
3. 计算一步预测损失与 SIGReg 正则。
4. 端到端反向传播优化编码器与预测器全部参数。

### 规划流程（latent MPC）

1. 由起始观测与目标观测编码得到 $\hat{\mathbf{z}}_1,\mathbf{z}_g$。
2. 用 CEM 采样候选动作序列并在潜空间 rollout。
3. 以终点潜变量到目标潜变量的距离作代价，迭代更新动作分布。
4. 按 MPC 仅执行前 $K$ 步并重规划，降低长 horizon 误差累积。

## 关键公式

### 公式1: [[下一嵌入预测损失|未来潜变量一步预测]]

$$
\mathcal{L}_{\mathrm{pred}}
=
\left\lVert
\hat{\mathbf{z}}_{t+1}-\mathbf{z}_{t+1}
\right\rVert_2^2,
\qquad
\hat{\mathbf{z}}_{t+1}=\mathrm{pred}_{\phi}(\mathbf{z}_t,\mathbf{a}_t).
$$

**含义**：要求模型在动作条件下预测下一帧潜变量，逼近真实编码潜变量。

**符号说明**：
- $\mathbf{z}_t$：时刻 $t$ 的潜变量编码。
- $\hat{\mathbf{z}}_{t+1}$：预测器输出的下一时刻潜变量。
- $\mathbf{a}_t$：动作输入。

### 公式2: [[SIGReg|随机投影高斯分布正则]]

$$
\mathrm{SIGReg}(\mathbf{Z})
=
\frac{1}{M}\sum_{m=1}^{M} T\!\left(\mathbf{h}^{(m)}\right),
\quad
\mathbf{h}^{(m)}=\mathbf{Z}\mathbf{u}^{(m)},
\ \mathbf{u}^{(m)}\in\mathbb{S}^{d-1}.
$$

**含义**：把高维潜变量随机投影到多个 1D 方向，用 Epps-Pulley 统计量约束其接近标准高斯，抑制塌缩并维持多样性。

**符号说明**：
- $\mathbf{Z}\in\mathbb{R}^{N\times B\times d}$：序列与批次上的潜变量集合。
- $M$：随机投影数量（文中默认 1024）。
- $T(\cdot)$：单变量正态性统计量。

### 公式3: [[LeWM损失|总训练目标]]

$$
\mathcal{L}_{\mathrm{LeWM}}
=
\mathcal{L}_{\mathrm{pred}}
\lambda\cdot \mathrm{SIGReg}(\mathbf{Z}).
$$

**含义**：在预测能力与表示分布约束之间做平衡；实验表明主要只需调 $\lambda$。

### 公式4: [[潜空间目标匹配|规划终端代价]]

$$
\mathcal{C}(\hat{\mathbf{z}}_{H})
=
\left\lVert
\hat{\mathbf{z}}_{H}-\mathbf{z}_{g}
\right\rVert_2^2,
\qquad
\mathbf{z}_g=\mathrm{enc}_{\theta}(\mathbf{o}_g).
$$

**含义**：衡量 rollout 终点与目标观测潜表示的距离，是 CEM 规划优化目标。

## 关键图表

### Figure 1: 规划速度对比

![Figure 1](https://le-wm.github.io/static/images/planning/planning_time.png)

说明：项目页给出的固定设置下规划耗时对比，LeWM 通过低 token 潜表示显著缩短规划时间。

### Figure 2: Push-T 固定 FLOPs 结果

![Figure 2](https://le-wm.github.io/static/images/planning/ctrl-pusht_fixed_flops.png)

说明：同算力预算下，LeWM 在 Push-T 上优于对照模型，体现“效率-效果”平衡。

### Figure 3: OGBench-Cube 固定 FLOPs 结果

![Figure 3](https://le-wm.github.io/static/images/planning/ctrl-ogb-cube_fixed_flops.png)

说明：在更复杂 3D 任务中依然保持竞争力，但对强预训练视觉先验方法仍有差距。

## 实验结果

### 任务与基线

- 任务：Two-Room、Reacher、Push-T、OGBench-Cube。
- 基线：DINO-WM、PLDM、GCBC、GCIVL、GCIQL（文中不同小节使用）。
- 设置：离线、无奖励、纯视觉输入（多数比较不使用 proprioception）。

### 主要结论（论文报告）

1. **速度**：规划阶段较 DINO-WM 最高约 **48x** 提升，完整规划约 1 秒量级。
2. **性能**：在 Push-T 与 Reacher 上，LeWM 超过 PLDM，并在 Push-T 超过部分 DINO-WM 配置。
3. **稳定性**：两项损失训练曲线更平滑，明显优于多项损失方案的噪声与不稳定性。
4. **物理表征**：latent probing 指标整体优于 PLDM，接近 DINO-WM。
5. **VoE**：对“物理不连续（如 teleport）”扰动的 surprise 显著上升（文中报告 paired t-test 显著）。

### Table 1: Push-T 物理量 probing（节选）

| Property | Model | Linear MSE↓ | Linear r↑ | MLP MSE↓ | MLP r↑ |
|---|---:|---:|---:|---:|---:|
| Agent Location | DINO-WM | 1.888 | 0.977 | 0.003 | 0.999 |
| Agent Location | PLDM | 0.090 | 0.955 | 0.014 | 0.993 |
| Agent Location | **LeWM** | **0.052** | 0.974 | 0.004 | 0.998 |
| Block Location | DINO-WM | **0.006** | **0.997** | 0.002 | 0.999 |
| Block Location | PLDM | 0.122 | 0.938 | 0.011 | 0.994 |
| Block Location | **LeWM** | 0.029 | 0.986 | **0.001** | 0.999 |
| Block Angle | DINO-WM | **0.050** | **0.979** | **0.009** | **0.995** |
| Block Angle | PLDM | 0.446 | 0.745 | 0.056 | 0.972 |
| Block Angle | LeWM | 0.187 | 0.902 | 0.021 | 0.990 |

### 失败案例与局限

- 在简单低维环境 Two-Room 上，LeWM 表现可能不及对照。
- 论文解释为：高维潜空间上的高斯先验与低内在维度数据可能存在张力。
- 长时域规划仍受模型偏差累积影响，MPC 只能部分缓解。

## 复现与工程要点

- 训练资源：单卡 GPU 可完成（论文强调可复现性门槛较低）。
- 超参搜索：核心集中在 $\lambda$，可用对数复杂度思路进行高效搜索。
- 规划器：CEM 样本数/迭代数直接影响实时性与性能折中。
- 编码器维度：达到一定阈值后性能趋于饱和，不需极端大模型。

## 批判性思考

### 优点

1. 训练目标简化而非“堆技巧”，方法论清晰。
2. 在保持端到端学习前提下兼顾训练稳定与推理效率。
3. 提供 probing 与 VoE 两种角度验证 latent 语义，不只看任务成功率。

### 局限性

1. 复杂 3D 视觉任务上仍可能被强预训练视觉 backbone 压制。
2. 分布正则目标固定为各向同性高斯，可能对低复杂度场景不匹配。
3. 评估侧重离线控制，泛化到在线闭环与跨域迁移还需更多证据。

### 后续可做

1. 分层世界模型 + 分层规划缓解长时域误差累积。
2. 自适应先验（非各向同性）替代固定高斯先验。
3. 与视频大模型预训练结合，提升复杂视觉场景鲁棒性。

## 关联概念与论文

- [[联合嵌入预测架构]]：LeWM 的基础范式。
- [[世界模型]]：潜空间动力学建模与规划。
- [[模型预测控制]]：测试时在线优化动作序列。
- [[交叉熵法]]：CEM 采样优化器。
- [[表示塌缩]]：JEPA 训练核心风险。
- [[SIGReg]]：本文关键抗塌缩正则。
- [[离线强化学习]]：无环境交互数据设定。

## 速查卡片

> [!summary] LeWorldModel (LeWM)
> - 核心思想：JEPA 预测 + 随机投影高斯正则，最小化训练技巧依赖。
> - 关键收益：端到端稳定训练、参数轻量、规划显著提速。
> - 代表指标：最高约 48x 规划速度提升；Push-T/部分任务性能具竞争力。
> - 适用场景：离线、纯视觉、目标驱动的潜空间规划问题。

*笔记创建时间: 2026-04-01*
