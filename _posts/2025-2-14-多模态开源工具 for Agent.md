---
layout: post
title: 多模态开源工具 for Agent
categories: [video agent]
tags: [LLM, 学习笔记, LLM tools]
slug: multimodal_tool
description: 多模态开源工具，用于构建Agent
---

近期的工作目标是自己构建一个 Agent，Agent 的关键是把各种工具串联起来，构建一个端到端的工作流。重点可能不像提出新算法那样去关注在评估集上的特定指标提升，主要是提出一个“好玩”的问题，做出一个有意思的东西。

在前期，主要是寻找并运行各种开源的生成工具和模型，积累 Agent 的“工具库”，搭建环境并体验尝试使用，并在这个过程中慢慢寻找潜在的 idea。

## ConsisID

Identity-preserving text-to-video.

https://huggingface.co/papers/2411.17440

给定参考人物图像，以及文字指示，生成单个特定人物的视频。

- [x] 已完成环境配置，制作环境镜像。但是模型下载比较慢。

**使用时遇到的问题**

- 如果上传的人像由于光线原因，面部不同部位色彩有差异，生成出来的图像也会继承这一面部特征，但由于背景改变，这一特征变得不合理。也就是说生成效果会很依赖于人面部图像的质量，可以接一个 I2I 的模型去对上传的人像进行优化，或者是修改。
- 你给什么角度的照片，它就吐这个人这个角度的视频。有局限性。

## MagicTime

metamorphic video generation.

https://huggingface.co/spaces/BestWishYsh/MagicTime

https://github.com/PKU-YuanGroup/MagicTime

文生图，主要是让物体的变化符合物理规律。

## IP-Adapter

https://github.com/tencent-ailab/IP-Adapter

一个适用于生成特定 IP image 的 adapter，可以结合一些模型完成各种生成特定 IP 图像的任务。

## Ingredients

https://github.com/feizc/Ingredients

Based on ConsisID，上传两个特定人物图像，生成视频。 

## UniPortrait

https://github.com/junjiehe96/UniPortrait

生成特定 ID 的图像。

## Rotate-and-Render

https://github.com/Hangz-nju-cuhk/Rotate-and-Render

Recover a frontal face image of the same person from a single face image under any poses.

生成人物正面照。

模型地址过期了。。。

## PhotoMaker

https://photo-maker.github.io/

人像编辑，可以用 prompt 改变人像角度。

> prompt: woman img, a highly detailed and realistic portrait of a young woman with delicate facial features, natural skin texture, and warm lighting. The image is captured from a front view perspective with a medium shot framing, showing the subject’s full head, including completely full hair. The background is softly blurred to enhance depth, with professional portrait photography and cinematic lighting.

> neg prompt: nsfw, lowres, bad anatomy, bad hands, text, error, missing fingers, extra digit, fewer digits, cropped, worst quality, low quality, normal quality, jpeg artifacts, signature, watermark, username, blurry, cropped head, missing hair, head cut off, no hair, partial head, incomplete head, upper body only, close-up without hair

## PuLID

https://github.com/ToTheBeginning/PuLID

time_step那个参数设2效果比较好

> prompt: woman , front view perspective with a medium shot framing, showing the subject’s full head, including completely full hair.

