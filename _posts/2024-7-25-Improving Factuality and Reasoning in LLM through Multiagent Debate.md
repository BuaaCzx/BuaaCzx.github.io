---
layout: post
title: Improving Factuality and Reasoning in LLM through Multiagent Debate 
tags: 学习笔记
---

多智能体辩论提高语言模型的真实性和推理能力

https://arxiv.org/abs/2305.14325

## Abstract

提出了一种改进回答的方法，多个语言模型在多轮次中提出它们自己的回答和推理过程并为此辩论，从而得到一个共同的最终答案。研究结果表明，这种方法在许多任务中都能显著提高模型的数学和策略性推理能力。这种方法提高了生成内容的事实性，减少了现在的模型容易产生的错误答案和模型幻觉。这个方法可以直接用于现在的黑盒模型，用相同的过程和 prompt 来解决所有任务。总得来说，这个研究表明，这种“社会思想”的方法有可能显著提高 LLM 的能力，并为语言生成和理解的进一步突破铺平了道路。

项目网站：https://composable-models.github.io/llm_debate/.

## Introduction