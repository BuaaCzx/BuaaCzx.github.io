---
layout: post
title: MetaGPT
categories: [文献阅读, Multi-Agent]
tags: [LLM, 学习笔记]
slug: metaGPT
---

在综述里看到的，介绍 Agents Communication 的结构 “Shared Message Pool” 里提到的文章。

https://arxiv.org/abs/2308.00352

https://github.com/geekan/MetaGPT

## Introduction

![image-20240802183056274](./../images/2024-8-2-MetaGPT%20Meta%20Programming%20for%20A%20Multi-Agent%20Collaborative%20Framework/image-20240802183056274.png)

通过广泛的实践，人类已经开发出被广泛接受的各种领域的标准化操作流程（SOP），这些 SOP 对于解构任务、协调合作方面有关键作用，概述了每个小组成员的责任。

基于 SOPs 构建了一个  GPT-based Meta-Programming framework：MetaGPT。

MetaGPT 要求智能体 **生成结构化的输出**，如需求文档，流程图，接口规范等。

## Related Work

咕咕咕

##  METAGPT: A META-PROGRAMMING FRAMEWORK

![image-20240802183239910](./../images/2024-8-2-MetaGPT%20Meta%20Programming%20for%20A%20Multi-Agent%20Collaborative%20Framework/image-20240802183239910.png)

### AGENTS IN STANDARD OPERATING PROCEDURES

- Specialization of Roles

  定义了五种角色：Product Manager, Architect, Project Manager, Engineer, and QA Engineer。每个智能体监视 message poll，来观察接收其他智能体的消息。

- Workflow across Agents

  按照 SOP 来，所有智能体按顺序工作。

###  COMMUNICATION PROTOCOL

- Structured Communication Interfaces

  目前大部分多智能体框架都用不受约束的自然语言作为通信接口。我们为每个角色设置一个 format，要求每个人根据自己的角色提供必要的输出。智能体之间通过文档和图表（结构化的输出）而不是简单的文本对话进行沟通。

- Publish-Subscribe Mechanism

  如果每次一对一交流信息，导致通信效率低下。把信息存储在 message pool 中，智能体在 pool 中发布消息，并访问其他消息。每个智能体都可以直接从 pool 中检索需要的信息，不需要等待其他智能体响应。

  **订阅机制**，智能体利用自己角色特定的兴趣（而不是对话）来提取信息，根据 profile 来选择关注信息，实际实现中，智能体接收到一定依赖项之后，才激活他自己的操作。

###  ITERATIVE PROGRAMMING WITH EXECUTABLE FEEDBACK

让智能体根据需求，在完成功能性代码的同时写相应的测试，测试通过之后再进行后续任务，否则继续调试，直到测试通过或者达到 3 次为止。
