# WizardLearner🧙‍♂️

Created by crazycth



## 介绍

本项目致力于基于开源Code、Math等相关数据集 from scratch 预训练一个小参数量的 CodeLLM，走完 **预训练** + **退火(annealing)** + **指令微调** 全流程

此外，本项目将额外探索 pretrain-ICL、code dedup 与 Synthetic-data



## 项目流程

* Summarize 当前 CodeLLM 的预训练语料与SFT语料，训练一个参数 500M-1B 的 LLM 并进行代码评估
* 关注LLM退火阶段表现，对比不同退火策略下所产生的动力学现象
* 从Pretrain开始，获得对LLM全流程的视野



## 数据准备

根据 [deepseek-math](https://q47cvaon6v.feishu.cn/docx/CHl5dW0P0oCtqhxoPqncKQymn9g) 结果显示，代码、数学等模型Reason能力具有一定共性，因此在训练code-LLM时可加入一系列 Reasoning 相关语料以提高Reason能力。

本项目使用 pretrain、退火与SFT语料如下，推荐使用 [Huggingface镜像站🤗](https://hf-mirror.com/) 进行下载



### Pretrain

|     |                                                            类别                                                             |                                                                Huggingface                                                                | Arxiv |
|-----|:------------------------------------------------------------------------------------------------------------------------------------:|:----------------------------------------------------------------------------------------------------------------------------------------------:|:---:|
| The Stack V2 |  代码  |  <a href="https://huggingface.co/datasets/bigcode/the-stack-v2">🤗</a>  | <a href="https://huggingface.co/datasets/bigcode/the-stack-v2">🐰</a> |
| OpenWebMath | 数学网页 | <a href="https://huggingface.co/datasets/open-web-math/open-web-math">🤗</a> | <a href="https://huggingface.co/datasets/open-web-math/open-web-math">🐰</a> |



### Decay & SFT

[MiniCPM](https://arxiv.org/abs/2404.06395) 首次提出两阶段的Pretrain策略，在Pretrain阶段使用大规模语料来学习模型基础能力。而[JetMOE](https://arxiv.org/abs/2404.07413)进一步证明了退火策略的有效性。而在退火阶段加入高质量无标注语料与SFT数据。笔者认为这种训练策略优势如下：

1. 退火阶段加入SFT数据使得SFT阶段模型可以获得更好的Alignment能力。对于LLM，一个通俗的说法是“在Pretrain阶段学习知识，而在SFT阶段学习回答模式”，google团队近日的[研究](https://arxiv.org/abs/2405.05904)更加证实了这一点。在退火阶段让模型熟悉SFT语料，有助于减少SFT阶段的 OOD (Out of Distribution) 情况出现，也可以减少SFT需要的学习步数。
2. 由于高质量数据token数极少，在退火阶段使用可以使 LLM 更加关注高质量数据背后的任务，且可以防止在整个预训练过程中该部分语料被重复多次。
3. 猜想退火阶段产生的Loss猛降的独特动力学现象使得模型对此阶段语料“学得更好”，而高质量语料与SFT数据更加贴近使用场景。



基于上述想法，在 Decay & SFT 阶段我们使用高质量的知识与多样化的SFT数据，混入原有预训练数据中。所涉及数据集如下：



|                            |                           描述                            |                         Huggingface                          |                          Arxiv                           |
| -------------------------- | :-------------------------------------------------------: | :----------------------------------------------------------: | :------------------------------------------------------: |
| MagicCoder-OSS-Insruct-75K |      oss-instruct 方式构建的 synthetic code-sft数据       | <a href="https://huggingface.co/Qwen/Qwen-1_8B-Chat-Int4">🤗</a> |     <a href="https://arxiv.org/abs/2312.02120">🐰</a>     |
| Evol-codeAlpaca-V1         |      Evil-instruct 方式构建的 synthetic code-sft数据      | <a href="https://huggingface.co/datasets/theblackcat102/evol-codealpaca-v1">🤗</a> |     <a href="https://arxiv.org/abs/2304.12244">🐰</a>     |
| Code-290K-shareGPT         |          公开的chatGPT对话历史，召回code相关部分          | <a href="https://huggingface.co/datasets/ajibawa-2023/Code-290k-ShareGPT">🤗</a> | <a href="https://github.com/domeccleston/sharegpt">🐰</a> |
| CommitPackFT               |              Github 清洗后的高质量commit数据              | <a href="https://huggingface.co/datasets/bigcode/commitpackft">🤗</a> |     <a href="https://arxiv.org/abs/2308.07124">🐰</a>     |
| WildChat-code-like         |          公开的chatGPT对话历史，召回code相关部分          | <a href="https://huggingface.co/datasets/allenai/WildChat-1M">🤗</a> |     <a href="https://arxiv.org/abs/2405.01470">🐰</a>     |
| WebInstruct-code-like      | 从CommonCrawl构建的synthetic sft数据，Recall code相关部分 | <a href="https://huggingface.co/datasets/TIGER-Lab/WebInstructSub">🤗</a> |     <a href="https://arxiv.org/abs/2405.03548">🐰</a>     |
| UltraChat200k-code-like    |          从高质量UltraChat200k 召回code相关部分           | <a href="https://huggingface.co/datasets/HuggingFaceH4/ultrachat_200k">🤗</a> |     <a href="https://arxiv.org/abs/2310.16944">🐰</a>     |