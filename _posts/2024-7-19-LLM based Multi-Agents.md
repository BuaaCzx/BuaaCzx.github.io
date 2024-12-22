---
layout: post
title: 综述：基于 LLM 的多智能体
categories: [文献阅读, Multi-Agent]
tags: [LLM, 学习笔记]
slug: multiagent
---



https://arxiv.org/abs/2402.01680

## Abstract

近年来，基于 LLM 的多智能体系统在复杂问题解决和世界模拟方面取得了很大的进步。

文章目标是为了让读者对下面的问题拥有实质性见解：

- 基于 LLM 的多智能体模拟了哪些环境？
- 这些 agent 是怎么被 profiled（描述）的，他们之间如何交流？
- 什么机制使 agent 的能力提高了？

为有兴趣进行深入研究的人总结了常用的数据集和 baseline。

## Introduction

LLM 在推理和规划能力方面有巨大的潜力，这种能力符合人们对于感知环境、做出决定并采取行动的自主智能体的预期。因此基于 LLM 的智能体被研究并迅速发展。

同时提出了多智能体的方法，来充分利用集体的智慧，以及不同智能体特定的 profile 和 skill。

多智能体系统：

- 把 LLM 专门化为各种具有不同功能的智能体
- 使得不同智能体之间的交互可以模拟现实世界

多个智能体协作参与规划、讨论与决策，和人类群体在解决问题时的合作属性很像。利用了 LLM 的交流能力，充分利用了它们生成文本和响应文本输入的能力。同时还利用了 LLM 在各个领域的知识和专注于特定任务的潜力。基于 LLM 的多智能体在解决各种任务上都取得了可喜的成果。

![image-20240719215652892](./../images/2024-7-19-LLM%20based%20Multi-Agents/image-20240719215652892.png)

https://github.com/taichengguo/LLM_MultiAgents_Survey_Papers

在第二节阐述完背景之后，提出了问题：LLM-MA（LLM-based Multi-Agent）系统怎么和共同解决任务的环境保持一致？为了回答这个问题，在第三节提出了一个用来定位、区分、连接 LLM-MA 的不同层次的方案。讨论了以下问题：

- 智能体如何与任务环境交互
- 智能体如何被描述，从而表现出特定的形式
- 智能体之间如何交换消息和协作
- 智能体如何发展其解决问题的能力

第四节分析应用：解决问题和世界模拟

第五节提出了一个开源实现框架，用来研究 LLAMA，还提出了一些数据集和 benchmark。

第六节讨论未来研究的挑战和机遇，第七节进行结论与总结。

## Background

### Single-Agent System

- **Decision-making thought**：智能体在 prompt 的指导下，把复杂问题解构成小的子问题、有方法地对每个部分进行思考、从过去的经验中学习，从而在复杂任务上做出更好的决策的能力。这种能力增加了 single LLM-based agent 的自主性，提高其解决问题的有效性。
- **Tool-use**：利用外部工具和资源来完成任务，增强其能力，在多样和动态的环境下高效运行。
- **Memory**：智能体学习上下文作为短记忆，或将外部向量数据库作为长记忆，从而长时间保存和检索信息的能力。可以让智能体保持上下文一致性，并增强从交互中学习的能力。

###  Single-Agent VS. Multi-Agent Systems

基于 LLM 的单智能体系统有很好的认知能力，这个体系的建设聚焦于内在机制的制定和与外界环境的交互。相反，LLM-MA 系统注重 **不同的智能体 profile，智能体互动，和共同决策的过程**。每个智能体都配备不同的行为和策略，并相互进行交流，这样的多智能体合作可以解决更复杂多样的问题。

##  Dissecting LLM-MA Systems: Interface, Profiling, Communication, and Capabilities
LLM-MA 系统的接口、profile、交流和能力

###  Agents-Environment Interface

operational environments 指 LLM-MA 系统部署的 **context 和设置**，LLM agent 在环境中感知和行动，这反过来影响他们的行为和决策。

Agents-Environment Interface 是指， agent **和环境交互以及感知环境的方式**。通过这个 interface，agent 才可以理解周围环境，做出决策，并从行动的结果中学习。

我们把 LLM-MA 系统中的 interface 分为三种类型，Sandbox, Phisical, None。

- **Sandbox**：一个人类构建的虚拟环境，在这个环境中，agent 可以更加自由的交互，试验各种行为和策略。这种 interface 广泛应用于软件开发、游戏等领域。
- **Physical**：一个现实世界的环境，agent 在里面和物理实体交互，遵循现实世界的约束。在 physical space 里，agent 通常需要采取行动，从而可以产生直接的物理结果。例如，像扫地、整理橱柜等任务中，agent 需要不断地执行动作，观察物理环境，并不断改进其动作。
- **None**：没有任何特定的外部环境，agent 不和任何环境进行交互。例如利用多个 agent 来辩论一个问题，来达成共识性的结果（这里列了三篇讲 debate 的文章，正好是目前正在调研的方向）。这种应用主要聚焦于 agent 之间的交互和通信，而不依赖于外部环境。

![image-20240725150736347](./../images/2024-7-19-LLM%20based%20Multi-Agents/image-20240725150736347.png)

### Agents Profiling

在 LLM-MA 系统中，智能体由他们的 **特征、行为、技能** 来定义，这些特征是为了 **满足特定目标** 而制定的。在不同的系统中，智能体扮演了不同的角色，每个角色都有一个综合描述，包括了特征、能力、行为、约束。例如，在游戏环境中，智能体可能被描述为具有不同角色和技巧的玩家，每个玩家对游戏目标都做出不同的贡献。同样的，在辩论平台中，智能体可以被制定成支持者、反对者或法官，每个智能体都有独特的功能和策略来有效履行角色。profile 非常重要，可以让智能体更好的在它们各自的环境中进行相互交流。

把智能体的 profile 方法分成了三种：Pre-defined, Model-Generated, Data-Derived.

- **Pre-defined**：系统设计者显式的制定 profile。
- **Model-Generated**：用模型来制定 profile。
- **Data-Derived**：基于之前就存在的数据集来构建 profile。

###  Agents Communication

从三个角度来分析：

- **Communication Paradigms**：智能体之间交互的方式方法

  目前主要的交流模式：Cooperative, Debate, and Competitive.

  - **Cooperative**：智能体为了一个或多个共同的目标而共同工作，通常用交换信息的方式，来生成一个更强的解决方案。
  - **Debate**：智能体参与辩论，提出并捍卫自己的解决方案，并且批评其他人的观点，这种方法有利于达成共识或者获得一个更加精确的解决方案。
  - **Competitive**：智能体努力实现自己的目标，这可能会和其他智能体的目标相冲突。

- **Communication Structure**：多智能体系统中通信网络的组织和架构

  <img src="./../images/2024-7-19-LLM%20based%20Multi-Agents/image-20240725162414327.png" alt="image-20240725162414327" style="zoom:67%;" />

  - **Layered**：分层结构，每一层的智能体有不同的角色，并且主要在层内和相邻的层进行交互。[Liu et al., 2023] 这篇论文提出了一个 DyLAN 的框架，他把智能体组织在了一个多层的前馈网络里。
  - **Decentralized**：去中心化，在一个平等的网络上运行，每个智能体直接和其它智能体交流，这是世界模拟常用的结构。
  - **Centralized**：包括一个或一组中心智能体，其它智能体通过这个中心节点来进行交互。
  - **Shared Message Pool**：共享消息池，维护一个共享消息池，智能体根据自己的 profile 来发送和接收订阅相关消息，从而提高通信效率。

- **Communication Content**：智能体间交流的内容

  交流的内容一般是文本形式。具体的内容比较多样，取决于应用场景。

### Agents Capabilities Acquisition

能力获取很关键，它可以让智能体进行动态的学习和进化。

两个重要概念：**feedback, adjustment**。

- **Feedback**：智能体收到的关于其行为结果的关键信息，帮助智能体了解其行为的潜在影响，适应复杂多变的问题。反馈一般是文本形式的。
  - Feedback from Environment：从现实环境或虚拟环境中获得反馈，在 LLM-MA 解决问题的场景中比较普遍。
  -  Feedback from Agents Interactions：从其它智能体的评判或者智能体间的通信中获得反馈。像 debate 这种场景里比较常用，世界模拟的场景里也会用到。
  - Human Feedback：从人类那里直接获得反馈，这对于智能体和人类价值观和偏好保持一致至关重要。
  - None：有时不向智能体提供反馈。主要是在世界模拟工作中，重点在于模拟的结果，而不是智能体的规划能力，因此，反馈不是系统的组成部分。

- **Agents Adjustment to Complex Problems**：三种可以增强智能体能力的方案
  - Memory：大部分 LLM-MA 系统通过一个 memory 模块来调整智能体的行为。智能体把以前的反馈和交互内容都存储在 memory 中，在执行操作的时候，检索相关 memory。
  - Self-Evolution：不仅仅靠历史记录，而是动态地进行自我进化，比如改变初始目标和计划策略，并通过反馈和交流记录来自我训练。
  - Dynamic Generation：系统在运行的过程中动态地生成新的智能体，让系统的扩展能力和适应能力变强。

## Applications

