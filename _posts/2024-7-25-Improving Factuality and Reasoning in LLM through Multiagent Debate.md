---
layout: post
title: Improving Factuality and Reasoning in LLM through Multiagent Debate 
tags: 学习笔记
---

多智能体辩论提高语言模型的真实性和推理能力

https://arxiv.org/abs/2305.14325

## Abstract

提出了一种改进回答的方法，多个语言模型 **在多轮次中提出它们自己的回答和推理过程并为此辩论，从而得到一个共同的最终答案**。研究结果表明，这种方法在许多任务中都能显著提高模型的数学和策略性推理能力。这种方法提高了生成内容的事实性，减少了现在的模型容易产生的错误答案和模型幻觉。这个方法可以直接用于现在的 **黑盒模型**，用 **相同的过程和 prompt** 来解决所有任务。总得来说，这个研究表明，这种“社会思想”的方法有可能显著提高 LLM 的能力，并为语言生成和理解的进一步突破铺平了道路。

项目网站：https://composable-models.github.io/llm_debate/.

## Introduction

LLM 在互联网上大量文本语料上训练，质量和准确性无法得到保证。

现有办法：These range from prompting models with few or zero-shot
chain-of-thought demonstrations, use of verification, self-consistency, or intermediate scratchpads.

这些技术都是作用与单个模型上的。受 The Society of Mind 和多智能体的启发，提出了一种方法，让多个模型或智能体提出并共同讨论他们的回答和推理过程，来得出一个共同的答案。给定一个查询，每个模型都生成一个候选答案，然后 **读取并评价其它模型的回答，并更新自己的答案**，重复多轮。在给出最终答案之前，模型都可以维护多个推理链和可能的答案。

baseline： [zero-shot chain of thought](https://arxiv.org/abs/2205.11916)，[reflection](https://export.arxiv.org/abs/2303.11366v1), [reflection2](https://arxiv.org/abs/2303.17651)。总之就是比这两种效果好。

给定一个初始的询问，虽然模型相同，但是每个模型实例提出了不同答案（也研究了混合模型，如 chatGPT 和 Bard）。debate 过后，模型们总是会收敛于一个更准确的答案，答案也不太可能包含模型不确定的错误事实，因为不确定的事实会让不同模型产生分歧，从而不在最终结果中出现。debate **不仅仅是放大一个正确答案**，因为很多时候，模型一开始的答案是错的，但是在辩论进行中慢慢得出了正确答案。

用相同的方法和 prompt 模板，黑盒模型就行，不需要模型内部信息。虽然 debate 的成本更高，要多个模型和多轮辩论，但他的答案得到了显著改善，并可以用于生成一些模型训练数据，让模型可以自我完善。

引入了一个新的 benchmark 和 dataset 来评估事实准确性。现代的语言模型有很高的倾向去生成与事实不相符的人物传记，歪曲相关的日期和机构组织，而且这些信息，不同模型生成出来的一般会不一致。通过要求模型达成共识，这些不一致的事实可能会被删除或者被被纠正。

工作：

1. 提出了一个 debate 的方法
2. 引入了一个关于事实准确性的 benchmark
3. 评估 debate 过程的生成表现

## Language Generation through Multiagent Debate

下图是一个 debate 的例子。

![image-20240726164935835](./../images/2024-7-25-Improving%20Factuality%20and%20Reasoning%20in%20LLM%20through%20Multiagent%20Debate/image-20240726164935835.png)

每一轮的 prompt：Using the solutions from other agents as additional information, can you give an updated response....

### Multiagent Language Generation

每个智能体先产生初始回答，然后把其它智能体的回复连接起来，作为 context 提供给每一个智能体，然后每个智能体又根据这个产生新的回复。每个智能体负责验证其它智能体给出的回复，并且据此改进自己的答案。反复重复这个过程。

先让每个智能体独立解决给定的问题，然后给一个相同的 prompt：

![image-20240726182016198](./../images/2024-7-25-Improving%20Factuality%20and%20Reasoning%20in%20LLM%20through%20Multiagent%20Debate/image-20240726182016198.png)

这个 prompt 可以一直用，把内容换成上一轮的生成内容就行。

这个方法和现有的 prompt 方法 orthogonal （正交的）。（？）

可以用额外的技术来 prompt 模型，引出更详细的回答来改进 debate 过程。

<img src="./../images/2024-7-25-Improving%20Factuality%20and%20Reasoning%20in%20LLM%20through%20Multiagent%20Debate/image-20240726182707589.png" alt="image-20240726182707589" style="zoom:66%;" />

### Consensus in Debates

怎么保证一组模型最后会收敛于一个共识性的答案？根据实验发现一定会收敛。

![image-20240726182849846](./../images/2024-7-25-Improving%20Factuality%20and%20Reasoning%20in%20LLM%20through%20Multiagent%20Debate/image-20240726182849846.png)

可以通过改变模型对自己输出的信任程度来控制辩论时间，用上面 Figure 3 的 prompt 来控制辩论持续时间。鼓励模型坚持于自己的观点的 prompt 会带来更长时间的辩论和更好的解决方案。

## Experiments

评估，回答以下问题：

1. 提高了多少推理能力？
2. 提高了多少事实有效性？
3. 什么设计可以让 debate 提高语言生成的表现？