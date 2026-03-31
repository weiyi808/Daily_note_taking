---
title: "WildWorld: A Large-Scale Dataset for Dynamic World Modeling with Actions and Explicit State toward Generative ARPG"
method_name: "WildWorld"
authors: [Zhen Li, Zian Meng, Shuwei Shi, Wenshuo Peng, Yuwei Wu, Bo Zheng, Chuanhao Li, Kaipeng Zhang]
year: 2026
venue: arXiv
tags: [world-model, dataset, interactive-video-generation, state-conditioning, benchmark]
zotero_collection: _待整理
image_source: online
arxiv_html: "https://arxiv.org/html/2603.23497v1"
created: 2026-04-01
---

# 论文笔记：WildWorld

## 元信息

| 项目 | 内容 |
|------|------|
| 论文类型 | 数据集 + 基准 + 基线实验 |
| 核心领域 | [[World Model]] / [[Interactive Video Generation]] |
| 游戏来源 | Monster Hunter: Wilds |
| 数据规模 | 108M+ frames |
| 动作规模 | 450+ 动作类别（角色侧） |
| 标注类型 | 动作、状态、骨架、相机位姿、深度、分层描述 |
| 基准名称 | [[WildBench]] |
| 代码 | [GitHub](https://github.com/ShandaAI/WildWorld) |
| 项目页 | [Project](https://shandaai.github.io/wildworld-project/) |

---

## 一句话总结

> WildWorld 把“动作-状态-视觉”三者对齐到同一帧级时序里，构建了一个面向长时程交互世界建模的数据与评测闭环。

---

## 核心贡献

1. **大规模显式状态数据集**：提出 WildWorld，提供 108M+ 帧、119 列帧级标注，并覆盖多实体交互场景。  
2. **语义丰富动作空间**：角色动作包含 455 motion IDs 与多 bank 组合，避免“仅相机平移/旋转”式弱动作监督。  
3. **统一评测基准 WildBench**：从视频质量、相机控制、动作跟随、状态对齐四个维度衡量交互世界模型。  
4. **状态条件生成基线**：给出 CamCtrl、SkelCtrl、StateCtrl、StateCtrl-AR，对比展示“显式状态”价值与局限。  

---

## 问题背景

### 要解决的问题

现有交互视频数据集常把动作影响直接等同于像素变化，缺少可解释的中间状态，导致模型在长时程预测中难以保持一致性。  

### 现有方法局限

- 动作语义弱：常仅有移动、镜头控制等粗粒度指令。  
- 状态不可见：例如弹药、体力、硬直等关键变量不在标签里。  
- 评测不充分：很多基准偏重感知质量，不能直接检验动作执行与状态演化是否正确。  

### 本文动机

作者希望把交互生成问题重新表述为“[[Action-conditioned Dynamics]] + [[State Representation]] + 视觉投影”的组合建模问题，使模型能学到可组合、可追踪、可评估的世界演化机制。  

---

## 方法与数据构建详解

### 1) 数据采集平台（Data Acquisition）

- 在引擎 tick 级别记录结构化动作与状态：角色/怪物位置、旋转、速度、动画 ID、血量、资源条等。  
- 同步记录观测：RGB、深度、相机内外参，并移除 HUD 提升训练可用性。  
- 用时间戳贯穿多源流，给后续对齐与过滤提供基准。  

### 2) 自动化游戏流水线（Automated Gameplay）

- 自动菜单导航，随机采样任务与 NPC 组合。  
- 利用行为树驱动 NPC 协同战斗，减少人工录制成本。  
- 通过 target-lock 相机机制保持主体可见，形成更稳定的镜头运动分布。  

### 3) 处理与标注（Processing + Annotation）

- 时长过滤：短于 81 帧样本剔除。  
- 连续性过滤：相邻帧时间间隔超过 1.5 倍目标帧间隔（约 50ms@30FPS）剔除。  
- 亮度过滤：长时间极亮/极暗片段剔除。  
- 遮挡过滤：利用相机弹簧臂异常与骨架重叠阈值剔除严重遮挡样本。  
- 分层字幕：动作级 caption + 样本级 summary，增强文本条件可控性。  

### 4) 数据统计（Statistics）

- 29 种怪物、4 个角色、4 类武器、5 个开放世界场景。  
- 约 66% 战斗，34% 跑图/移动。  
- 样本时长多数在 4k~28k 帧，存在 >40k 帧长程片段。  
- 动作分布呈长尾，利于研究稀有动作与组合行为。  

---

## 关键公式

### 公式1：[[Action Following|动作跟随评分]]

$$
\mathrm{AF} = \frac{1}{N}\sum_{i=1}^{N}\mathbf{1}\!\left(\hat{y}_i = y_i\right)
$$

**含义**：将每个动作片段作为二分类一致性判断（生成片段与 GT 片段动作是否一致），取全体片段平均。  

**符号说明**：
- $N$：动作片段总数；  
- $y_i$：第 $i$ 段的真实动作语义判定；  
- $\hat{y}_i$：第 $i$ 段的模型动作语义判定；  
- $\mathbf{1}(\cdot)$：指示函数。  

### 公式2：[[State Alignment|状态对齐分数]]

$$
\mathrm{SA} = \frac{1}{K}\sum_{k=1}^{K}\frac{1}{T}\sum_{t=1}^{T}
\frac{1}{|\Theta|}\sum_{\delta\in\Theta}
\mathbf{1}\!\left(\lVert \hat{\mathbf{p}}_{k,t}-\mathbf{p}_{k,t}\rVert_2 \le \delta\right),
\quad \Theta=\{4,8,16,32\}
$$

**含义**：对每个关键点在多阈值像素半径下统计命中率，再对时间与关键点取平均，衡量状态轨迹一致性。  

**符号说明**：
- $K$：关键点数；$T$：序列帧数；  
- $\mathbf{p}_{k,t}$ 与 $\hat{\mathbf{p}}_{k,t}$：GT 与预测的 2D 关键点坐标；  
- $\Theta$：像素误差阈值集合。  

### 公式3：[[State-conditioned Generation|状态条件生成目标]]

$$
\mathcal{L}_{\text{total}}
= \mathcal{L}_{\text{gen}}
\lambda_d \mathcal{L}_{\text{decoder}}
\lambda_p \mathcal{L}_{\text{predictor}}
$$

**含义**：在视频生成损失之外，增加状态解码监督与下一帧状态预测监督，提升状态表征可恢复性和时序可预测性。  

**符号说明**：
- $\mathcal{L}_{\text{gen}}$：主生成目标（扩散/去噪框架）；  
- $\mathcal{L}_{\text{decoder}}$：状态重建损失；  
- $\mathcal{L}_{\text{predictor}}$：状态转移预测损失；  
- $\lambda_d,\lambda_p$：权重系数。  

---

## 关键图表

### Figure 1: 数据集概览

![Figure 1](https://cdn.jsdelivr.net/gh/ShandaAI/wildworld-project@0.1.0/static/images/teaser.png)

说明：展示 WildWorld 的任务定义与标注维度，强调“动作/状态/观测”三路对齐。  

### Figure 2: 数据构建流程

![Figure 2](https://cdn.jsdelivr.net/gh/ShandaAI/wildworld-project@0.1.0/static/images/framework-arxiv.png)

说明：对应论文中数据流水线，从自动采集到过滤与 caption 生成的完整闭环。  

### Figure 3: 数据统计

![Figure 3](https://cdn.jsdelivr.net/gh/ShandaAI/wildworld-project@0.1.0/static/images/dataset_overview_figure.png)

说明：覆盖实体分布、时长分布、动作长尾分布，解释为何适用于长时程交互建模。  

### Figure 4: 定量结果可视化

![Figure 4](https://cdn.jsdelivr.net/gh/ShandaAI/wildworld-project@0.1.0/static/images/table1.png)

说明：不同条件控制方式在 WildBench 各指标上的整体对比。  

### Figure 5: 定性对比

![Figure 5](https://cdn.jsdelivr.net/gh/ShandaAI/wildworld-project@0.1.0/static/images/qualitative_comparison_v2.png)

说明：展示 CamCtrl / SkelCtrl / StateCtrl 在动作执行、遮挡建模、主体清晰度上的差异。  

---

## 实验设置

### 比较方法

- **Baseline**：Wan2.2-TI2V-5B，无显式交互控制。  
- **CamCtrl**：以每帧相机位姿作为控制输入。  
- **SkelCtrl**：以骨架视频作为控制输入。  
- **StateCtrl**：融合离散状态 + 连续状态，注入 DiT 中间层。  
- **StateCtrl-AR**：仅给首帧真值状态，后续状态自回归预测。  

### 训练细节

- 分辨率：$544\times960$  
- 采样帧数：81 帧  
- 帧率：16 FPS  
- 学习率：$1\times10^{-5}$  
- 迭代：250k  
- 优化器：Adam  
- 推理步数：50  

---

## 实验结果

### Table 1: WildBench 主结果（论文原表）

| Method | MS | DD | AQ | IQ | ATE(↓) | RPE(↓) | Action Following | State Alignment |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| Baseline | 96.38 | 99.00 | 50.81 | 65.62 | 4.63 | 0.18 | 53.77 | 11.29 |
| CamCtrl | 97.85 | 97.00 | 48.29 | 62.88 | 2.02 | 0.13 | 83.46 | 15.18 |
| SkelCtrl | 97.85 | 95.00 | 47.92 | 62.43 | 2.55 | 0.10 | 92.81 | 22.03 |
| StateCtrl | 97.45 | 99.00 | 50.86 | 67.78 | 0.94 | 0.07 | 85.66 | 16.06 |
| StateCtrl-AR | 97.43 | 99.00 | 50.90 | 67.76 | 1.01 | 0.08 | 74.66 | 16.13 |

### 结果解读

1. **交互相关指标普遍提升**：所有控制方法都显著优于 Baseline。  
2. **SkelCtrl 强动作对齐**：Action Following 与 State Alignment 最好，但 AQ/IQ 下降。  
3. **StateCtrl 相机控制最强**：ATE/RPE 最优，说明显式状态对时序几何一致性有帮助。  
4. **StateCtrl-AR 暴露累积误差**：状态自回归在长序列上动作跟随退化明显。  
5. **VBench 饱和现象**：MS/DD 指标接近天花板，难区分真正的交互可控性。  

### 指标可靠性

- Action Following 与人工标注一致率约 85%。  
- State Alignment 在 GT 视频跟踪验证下可提供稳定的相对比较信号。  

---

## 批判性分析

### 优点

1. 数据规模与标注粒度都达到当前交互世界建模的高标准。  
2. 显式状态让“可控生成”与“可评估生成”形成闭环。  
3. WildBench 补足了纯感知质量指标对交互能力刻画不足的问题。  

### 局限

1. 数据来自单一 AAA 游戏域，跨游戏泛化仍未知。  
2. 动作评估部分依赖 VLM 判别，仍可能受提示词与模型偏差影响。  
3. 状态代理主要依赖骨架关键点，未覆盖全部隐变量（如复杂物理交互）。  

### 未来改进方向

1. 跨引擎/跨游戏统一状态语义本体。  
2. 引入多主体博弈与任务成功率等更高层评测。  
3. 将显式状态与潜变量世界模型做层次化融合。  

---

## 术语与关联

- [[World Model]]：核心任务范式。  
- [[State Representation]]：显式状态建模基础。  
- [[Action-conditioned Dynamics]]：交互驱动时序演化。  
- [[State Alignment]]：本文核心新评测维度之一。  
- [[Action Following]]：本文核心新评测维度之一。  
- [[Camera Control]]：交互视频中镜头可控性的关键。  
- [[Skeleton Tracking]]：状态代理提取手段。  
- [[Diffusion Transformer]]：StateCtrl 的生成主干。  
- [[Benchmark Design]]：WildBench 的方法学价值。  

---

## 复现清单

- [x] 数据规模与统计信息已公开到论文与项目页  
- [x] 基准指标定义清晰（含阈值与流程）  
- [x] 基线训练配置给出关键超参数  
- [ ] 数据下载与完整训练脚本是否全部开放（以仓库更新为准）  

---

## 速查卡片

> [!summary] WildWorld
> - 核心：显式状态标注的大规模交互世界建模数据集。  
> - 方法：动作/状态/观测三路对齐 + WildBench 四维评测。  
> - 结果：StateCtrl 在相机控制最优，SkelCtrl 在动作与状态对齐最强。  
> - 启发：仅靠感知指标不足，必须显式评估动作执行与状态一致性。  

---

*笔记创建时间: 2026-04-01*
