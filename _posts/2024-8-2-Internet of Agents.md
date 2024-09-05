---
layout: post
title: Internet of Agents
tags: 学习笔记
---

Internet of Agents: Weaving a Web of Heterogeneous Agents for Collaborative Intelligence

https://arxiv.org/abs/2407.07061

https://github.com/OpenBMB/IoA

## Introduction

创建一个平台来促进智能体之间的自动协作，IoA，一个受互联网启发的智能体通信与协作的框架。

解决的问题：

- Ecosystem Isolation：大部分框架只考虑在自己生态系统里的智能体；
- Single-Device Simulation：几乎所有多智能体框架都在单个设备上模拟多智能体，但在现实生活中，智能体可以在多个设备上；
- Rigid Communication and Coordination：如何根据任务决定团队成员。

## FRAMEWORK DESIGN AND KEY MECHANISMS OF IOA

### OVERVIEW OF IOA

- 支持跨设备、多位置分布
- **自主团队形成，动态调整协作**
- 集成异构智能体

### ARCHITECTURE OF IOA

- **交互层**：负责团队组建和智能体通信，包括智能体查询、群组设置和消息路由。
- **数据层**：管理与智能体、群组和任务相关的信息，包括智能体注册、会话管理和任务管理。
- **基础层**：提供智能体集成、数据管理和网络通信的基础设施。

###  KEY MECHANISMS

#### AGENT REGISTRATION AND DISCOVERY

咕咕咕

#### AUTONOMOUS NESTED TEAM FORMATION

- 智能体注册和发现
  - 每个智能体在加入 IoA 时注册，提供其能力和领域描述。
  - 智能体可以通过查询服务发现其他具有所需能力和特征的智能体。
- 自主嵌套团队组建
  - 当智能体被分配任务时，它会根据任务描述和已发现的智能体信息自主决定是否需要其他合作者。
  - 智能体可以使用“搜索智能体”工具寻找合适的合作者，并使用“启动群组聊天”工具创建新的群组。
  - 对于复杂的任务，智能体可以创建嵌套子群组，形成树状结构，每个子群组负责特定的子任务。
- 自主对话流程控制
  - 智能体之间的对话遵循顺序发言机制，确保清晰有序的沟通。
  - 对话流程被抽象为有限状态机，包括讨论、同步任务分配、异步任务分配、暂停和触发、结论等状态。
  - **每个智能体的 LLM** 负责根据对话历史和当前状态自主决定状态转换和下一个发言者。