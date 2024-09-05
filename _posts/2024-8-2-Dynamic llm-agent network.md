---
layout: post
title: Dynamic LLM-Agent Network
tags: 学习笔记
---

在综述里看到的，介绍 Agents Communication 的结构 “Layered” 里提到的文章。

https://arxiv.org/abs/2310.02170

https://github.com/SALT-NLP/DyLAN.

## Abstract

集成多个 LLM agent 可以进一步提高性能。文章提出了一个智能体决策团队，基于 query 在 **动态交互** 的架构中交流。构建了一个  **Dynamic LLM-Agent Network (DyLAN) **，用于 agent 在复杂任务上的协作。agent 在动态架构中进行 **多轮交互**，有 inferencetime agent selection and an early-stopping mechanism 来提高表现和效率。设计了一个基于 Agent Importance Score

 的智能体团队自动优化算法，**基于每个智能体各自的贡献来选择最佳智能体集合**。

## Introduction

现有的：在一个团队中组合不同的 agent 来处理同一个询问，来解决复杂的任务，往往是静态的为每个 agent 设置预定义好的角色。

存在的问题：

- 手动设置智能体的角色，使它很难推广于其他领域和任务；
- 智能体以固定顺序生成答案，可能导致 sensitivity；
- 需要很强的人类经验，而且可能和实际情况不符。

DyLAN：用前馈网络来制定每个任务的协作过程，把不同 time step 的 LLM 视为 **节点**，不同 time step 的消息交换作为 **边**，这样把多轮 agent 协作组织成了一个多层网络。用了一个 LLM-empowered ranker 对不同的 agent 排序，并 **在之后的交互中停用表现不佳的 agent**（inference-time agent selection），从而构建了一个动态的交互结构。**同一层的 agent 达成共识时终止推理过程**（early-stopping mechanism），保证效率。提出了不需要人工的自动优化方法，为了选择 top-k 智能体，有三步：每个智能体对它前面的解决方案打分（propagation）；汇总它们后继的打分，量化评分（aggregation），总结所有 time step 的评分，为每个智能体生成一个 Agent Importance Score；根据这个得分来选择（selection）。

## Related Work

咕咕咕

##  Dynamic LLM-Agent Network

### FORMULATION

![image-20240802110630126](./../images/2024-8-2-Dynamic%20llm-agent%20network/image-20240802110630126.png)

- 节点：节点是在特定 time step 的智能体，智能体接收其它智能体在上一个 time step 生成的内容，把它作为输入，并生成响应。$a_{t,i}$ 表示第 i 个智能体在第 t 个 time step，可以看作是一个输入是 prompt, q, 前一个时间节点的所有前驱的输出集合的函数。

- 边：多智能体写作过程中，每个节点之间的通信。边有向，且只能从 t-1 到 t。
- 消息传递：引导信息流通过前馈网络中的节点和边。两种类型的消息传递：前向传递，跨 time step 向特定的智能体传递消息，系统生成最终答案；反向传递，这个 Agent Importance Score，是沿着反向边来传递分数。

### CONSTRUCTION OF DYLAN

- 在 middle time step 中增加  inference-time agent selection
- 在适当的层级终止推理

层：同一个 time step 上工作的一组 agent。在每一层中，每个节点接收 **来自前一层（上一个 time step）的所有节点** 的响应。这种通信在层之间形成边，比如图 1，询问被发送到第一层的节点。

**inference-time agent selection**：在第 L 层选择 top-m 响应来前馈，用一个 LLM-Ranker 来分析前一层的响应并且打分评价，被识别为无用的智能体在之后的层被停用。

**early-stopping mechanism**：在单层中超过 2/3 的智能体答案一致时，推理过程被终止（达到最大 time step 的时候也终止），把这个机制用在了每一层。

![image-20240802124151939](./../images/2024-8-2-Dynamic%20llm-agent%20network/image-20240802124151939.png)

### AGENT TEAM OPTIMIZATION

把图简化成一个更小的图。

1. Propagation：要求每个节点对前一个节点的响应打分
2. 每个节点汇总他的后继节点对他的评价（反向传递），在不同的 time step 里量化他自己的贡献
3. 总结同一个智能体在不同步骤的得分，根据这些得分，提取出贡献最大的 top-k 智能体

![image-20240802130605523](./../images/2024-8-2-Dynamic%20llm-agent%20network/image-20240802130605523.png)

$a_{t-1,j}\rightarrow a_{t,i}$，计算 $a_{t-1,j}$ 的贡献，先找他的后继，求它的后继的贡献与这个后继对他的打分的乘积，求总和。（应该做了归一化）

在最后一层初始化贡献，然后反向传递。