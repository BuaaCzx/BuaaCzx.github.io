---
layout: post
title: Self-Evaluation Guided Beam Search for Reasoning
tags: 学习笔记
---

## Abstract

将问题分解为中间步骤在大型语言模型（LLM）的推理中表现出色。然而，**推理链的增长会带来不确定性和错误积累**，从而难以获得准确的最终结果。为了解决多步推理中的不确定性挑战，我们引入了一种 **逐步自我评估机制**，以指导和校准LLM的推理过程。

这种自我评估机制能够定位逻辑错误，导致更高的推理一致性和稳定性。

代码公开在https://guideddecoding.github.io/。