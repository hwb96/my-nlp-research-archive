# My NLP Research Archive

这是一个个人档案库，用于收集和整理我平时认为有价值的研究论文、项目和其他资源。主要关注自然语言处理（NLP）领域的论文，但也包括一些相关的研究和项目，以增强我的知识和理解。


## 概述

在这个仓库中，我会定期更新我阅读的论文和项目，包括论文的名称、地址以及我认为重要的地方。这些资源将帮助我更好地理解NLP及其相关领域的最新进展。

## 论文列表

### 大模型

**论文名称**: [CharPoet: A Chinese Classical Poetry Generation System Based on Token-free LLM](https://arxiv.org/abs/2401.03512)

   - **重要部分**: 
    这篇非常不错，因为我们平时让模型输出“不少于300字”，“不少于200字”，由于分词器的固有逻辑，肯定是无法准确遵循的。这篇给了一个新方法。
    CharPoet的中文古典诗歌生成系统，它使用了一种“无词元”（token-free）的方法。这意味着该系统在生成过程中不使用常规的词元，而是直接在字符或字节的层面上进行操作。
    
    具体操作示例
    去除长词元：在常规的词元化过程中，文本会被分解为较大的单位（如词或短语）。例如，“我爱你”可能被视为一个词元。如果使用CharPoet，则可能将其分解为“我”、“爱”、“你”三个独立的字符。
    
    保留字符级和字节级词元：在该系统中，仅保留字符级（每个汉字作为一个词元）和字节级（每个字节作为一个词元）而去除其他长词元。这意味着模型只能处理单个汉字或字节，而不会使用完整的词组。
    
    在诗歌数据集上微调：虽然CharPoet的基础是从现有的基于词元的模型中修剪而来，但它会在特定的诗歌数据集上进行微调，使其能够生成符合古典诗歌格式和内容的作品。
    
    分词器已经去除了长词元（long tokens），但在输出阶段依然需要将这些长词元的 logits 设置为一个较大的负数，原因如下：
    
    模型架构的固定性：虽然你在分词器中去除了长词元，但在语言模型的头部（output head）中，依旧保留了这些词元的“槽位”或者“通道”。这些长词元虽然不会在输入中被访问到，但它们在模型内部仍然是潜在的输出候选项。为了确保这些词元不被错误地生成，模型必须在输出时人为将其概率设为零。
    
    保证一致性：尽管分词器已删除长词元，但模型的其他部分（如 transformer 和输出头部）在设计时可能依赖于完整的词表。如果在这些位置直接删除词元，可能会破坏模型的架构或影响现有的权重和参数。通过将 logit 设为负值，而不是物理上删除词元，可以在不改变架构的情况下实现这种控制。
    
    将长词元的 logits 设置为非常大的负数，在 前向传播 中的输出阶段实现
    
    实际嵌入过程：
    嵌入矩阵的构造：假设嵌入矩阵的大小是 (65024, 4096)，这表示词表有 65024 个词元，每个词元的嵌入维度是 4096。长词元的嵌入向量也包含在这个矩阵中。
    
    字符级和字节级词元的选择：分词器在处理输入时，只会将文本分解为字符级或字节级词元，并且生成这些词元的索引。例如，假设字符 "大" 对应索引 100，字节 "模" 对应索引 200，而一个长词元 "大模型" 可能对应索引 5000。
    
    只访问有效词元：当模型执行嵌入操作时，它只会根据分词后的索引（例如字符 "大" 和 "模" 的索引）从嵌入矩阵中提取对应的嵌入向量。因为分词器已经去除了长词元，所以模型不会访问索引为 5000 的嵌入向量，即长词元的嵌入向量不会被用到。
    
    总结：
    分词后，长词元不会被访问：分词器在将文本分解为字符级或字节级词元后，嵌入操作只会访问这些词元对应的嵌入向量，长词元的嵌入向量不会被使用。
    
    长词元的嵌入向量可以不变，也可以人为设置为全 0：虽然长词元的嵌入向量保存在嵌入矩阵中，但在模型前向传播中它们不会被使用，你可以选择让这些向量保持不变，或者将它们初始化为全 0。



**论文名称**: [TableRAG: Million-Token Table Understanding with Language Models](https://arxiv.org/abs/2410.04739v1)

   - **重要部分**: 

    这篇论文提出了一个名为TableRAG的框架，旨在解决大型表格数据理解中的一些关键挑战。具体来说，它试图解决以下问题：
    
    - 大规模表格数据的处理：传统的语言模型（LMs）在处理整个大型表格时面临上下文长度限制、计算成本高昂和推理能力下降的问题。
    
    - 信息定位的效率：在大型表格中，关键信息可能分散在不同的行和列中，直接处理整个表格以找到这些信息非常低效。
    
    - 减少提示长度和信息丢失：当表格很大时，直接将整个表格作为输入提示给语言模型会导致提示长度过长，可能丢失关键信息。
    
    - 提高表格理解的可扩展性：现有的表格理解方法在扩展到更大的表格时面临挑战，需要一种可扩展的解决方案来有效处理大型表格。
    
    为了解决这些问题，TableRAG采用了一种结合了检索增强生成（RAG）的方法，通过查询扩展、模式检索和单元格检索来精确地定位关键信息，然后将这些信息提供给语言模型进行处理。这种方法提高了数据编码的效率和检索的准确性，显著减少了提示长度，缓解了信息丢失问题，并提高了在大规模表格理解中的性能。


**论文名称**: [Never Lost in the Middle: Improving Large Language Models via Attention Strengthening Question Answering](https://arxiv.org/abs/2311.09198)

   - **重要部分**: 

     - 微调多文档问答能力。

     - 正样本构造：将复杂的多文档问答（Multi-doc QA）任务分解为三个子任务，分别是问题重复（Question Repetition, QR）、索引预测（Index Prediction, IP）和答案总结（Answer Summarization, AS），下边这个例子能很形象说明：

       ![image-20240711190228310](image/image-20240711190228310.png)

     - 负样本构造：

       - 70%基于检索的负样本：从文档集合中检索相关性较高的负样本，提高负样本的挑战性，增强模型的鲁棒性。假设问题是“什么是新冠病毒的主要症状？”正样本可能是一个详细描述新冠病毒症状的文档。基于检索的负样本可能是描述流感症状的文档，这些文档在某些方面与新冠病毒症状有关，但并不是正确答案。
       - 30%随机负样本：从负样本候选集合中随机抽取，提供负样本多样性，保持训练数据的代表性。

**论文名称**: [Improving Portfolio Optimization Results with Bandit Networks](https://papers.cool/arxiv/2410.04217)

   - **重要部分**: 

        - 这篇论文试图解决的主要问题是多臂赌博机（Multi-Armed Bandit, MAB）在非平稳环境中的应用局限性。在现实世界的应用场景中，如推荐系统、医疗保健和金融等领域，环境往往是动态变化的，这导致传统的MAB算法（这些算法通常假设奖励分布是静态的）在这些场景中的效果受限。为了克服这一局限性，论文提出了新的MAB算法，专门针对非平稳环境进行优化。
        
        - 具体来说，论文的主要贡献包括：
     
       
   1 提出自适应折扣汤普森采样（Adaptive Discounted Thompson Sampling, ADTS）算法：该算法通过放宽折扣因子和滑动窗口机制来提高适应性，从而更好地响应奖励分布的变化。
       
   2 引入组合自适应折扣汤普森采样（Combinatorial Adaptive Discounted Thompson Sampling, CADTS）算法：该算法是ADTS算法在组合赌博机框架内的扩展，旨在解决组合赌博机中的计算挑战，并改善动态资产配置。
       
   3 提出新型架构——赌博机网络（Bandit Networks）：这种架构整合了ADTS和CADTS算法的输出，以减轻股票选择过程中的计算限制。
       
       4 通过使用真实金融市场数据的广泛实验，展示了这些算法和架构在适应动态环境和优化决策过程方面的潜力。例如，与经典的投资组合优化方法（如资本资产定价模型、等权重、风险平价和马尔科维茨模型）相比，所提出的赌博机网络在样本外夏普比率上高出20%。

2. **论文名称**: [Another Paper Title](http://example.com)

   - **重要部分**:
     - 



## 项目

1. **项目名称**: [Example Project Title](http://example.com)
   - **描述**: 

## 资源

- **资源名称**: [Example Resource Title](http://example.com)
  - **描述**: 
