---
title: LLM-based Agent综述阅读笔记
published: 2023-11-24
description: 论文<A Survey on Large Language Model based Autonomous Agents>阅读笔记
image: ''
tags: [NLP, LLM]
category: Papers
draft: false 
---

论文链接：https://arxiv.org/pdf/2308.11432.pdf

本文的作者是来自人民大学高瓴人工智能学院的，先前从事的工作大多都是推荐系统。在发布这篇survey之前，该组还有一篇RecAgent论文，也是和agent相关的工作。



## Introduction

**Autonomous agent的定义**：指的是**能够自主规划并采取相应的行动来完成任务**的智能系统。

*An autonomous agent is a system situated within and a part of an environment that senses that environment and acts on it, over time, in pursuit of its own agenda and so as to effect what it senses in the future.*

agent的概念之前应该是在**强化学习**领域中常用到的，是强化学习算法适用的对象。但是在强化学习领域中，agent是在**隔离环境**中学习的，agent的行动是基于**简单的****启发式算法**进行的，只能在特定的任务中工作，在**不受限的开放环境中**的表现并不好。在近年的LLM取得发展后，autonomous agent的发展也取得了进展，任务的核心在于使LLM具备接近人类的记忆和规划能力。

这篇survey从三个方面介绍了基于LLM的autonomous agents：

- 结构（**construction**）
  - **硬件基础**：如何设计一个能够更好地利用LLM的agent结构
  - **软件资源**：如何增强或激发agent处理各项任务的能力
- 应用（**application**）：介绍了agent在社会科学、自然科学和工程方面的各种应用
- 评估（**evaluation**）：介绍了各种agent的评估策略，包括主观和客观两类



## Construction

Autonomous agent需要运用LLM与人类相似的能力（记忆、规划）来应对各种各样的任务，因此在设计相应的结构时，就需要考虑到以下两个方面：

- 结构的设计需要更好地利用LLM
- 在已有的结构下，需要使agent掌握实现对应任务的能力

将**Agent**类比为传统的**机器学习**，**Agent的结构**就对应于**网络的结构**，**Agent学习到相应的能力**就对应于**网络学习对应的参数**。



### Agent Architecture Design

本文提出了一种统一的框架，概括了目前Agent方向上所有工作的结构设计：每个Agent系统由四个模块组成，分别是profiling module、memory module、planning module和action module。

简单来说，**profiling module**的作用是**定义agent的身份**，**memory module**用于**回顾和记录过去的状态**，**planning module**用于**规划未来的行动**，**action module**用于将agent的决定**转为实际的行动**。

![img](framework.png)

#### Profiling Module

Agents通常会在执行任务时被分配一个特定的身份或角色，而profiling module的作用就是向agent指示其应该担任的职责。这种指示通常会以prompt的形式来交给LLM，包含各方面的信息，比如年龄、性别、职业这些基础信息，也有其他心理相关的信息如性格、社交方面的信息（和其他agent的关系）等。需要囊括的信息视任务而定，譬如想要研究人类认知，agent的心理学信息就很重要。

目前有三种设置profile的策略：

- **手动设置**：人工编写相应的prompt交给LLM，优点在于profile的设计可以**非常灵活**，但相应地会比较**耗费人力**
  - **Generative Agent**（*[Generative Agents: Interactive Simulacra of Human Behavior](https://dl.acm.org/doi/pdf/10.1145/3586183.3606763)*）：
    - 这篇文章构造了一个小的沙盒世界Smallville，观察了25个基于LLM的生成式agent在其中的各种表现，包括日常生活、社交等。研究发现，这些agent在模拟人类行为上具有非常大的潜力，在彼此之间能够传递信息、相互协作。
    - 文章的最大创新在于提出了一种适用于agent的记忆结构memory stream，让agent能够基于memory来做出相应的决策。基于这个memory stream，agent能够决定自己接下来采取的action，还会定期进行反思、总结和规划。
    - 在这篇文章的实验中，25个agent的身份信息都是手工编写的：
    - > John Lin is a pharmacy shopkeeper at the Willow Market and Pharmacy who loves to help people. He is always looking for ways to make the process of getting medication easier for his customers; John Lin is living with his wife, Mei Lin, who is a college professor, and son, Eddy Lin, who is a student studying music theory; John Lin loves his family very much; John Lin has known the old couple next-door, Sam Moore and Jennifer Moore, for a few years; John Lin thinks Sam Moore is a kind and nice man; John Lin knows his neighbor, Yuriko Yamamoto, well; John Lin knows of his neighbors, Tamara Taylor and Carmen Ortiz, but has not met them before; John Lin and Tom Moreno are colleagues at The Willows Market and Pharmacy; John Lin and Tom Moreno are friends and like to discuss local politics together; John Lin knows the Moreno family somewhat well — the husband Tom Moreno and the wife Jane Moreno.
- **LLM****生成**：向LLM指示profile的生成规则和包含内容，让LLM帮忙生成新的profiles，优点在于**省时省力**，但缺点是**结果不可控**（缺乏直接的控制）
  - **RecAgent**（*[When Large Language Model based Agent Meets User Behavior Analysis: A Novel User Simulation Paradigm](https://arxiv.org/pdf/2306.02552.pdf)*）：
    - 这篇文章和本survey的作者相同。在研究中，作者尝试借助agent模拟人类行为，构建了一个推荐系统的模拟环境，并在其中放置了最高达1000个agent，模拟用户在推荐系统中的浏览、搜索、观看等行为；agent之间还会直接地、或是以社交平台发贴的方式相互影响，模拟出推荐相关的社交行为。
    - 从这篇文章中可以看出作者所在组对于agent研究的一个逻辑雏形。文章中同样提到了agent的结构设计，和本survey中提出的统一框架非常相似；在讲述如何设计profiling module时，也同样介绍了本survey中详细列出的这三种方法。
    - 研究团队借助了LLM来生成这1000个agent的档案。作者将下图所示的表格交给LLM，再给予其几个seed profile作为样例，要求基于已有的设计来填满表格，从而自动地构造了大量的profile。
    - ![img](RecAgent.png)
- **和数据集关联的生成方法**：特指基于现实世界数据集构造profile的方法，这种构造方法的优点在于比较符合现实状况中的数据分布情况，能够**更好地还原现实世界中的情景**
  - **Out of One, Many**（*[Out of One, Many: Using Language Models to Simulate Human Samples](https://arxiv.org/pdf/2209.06899.pdf)*）：
    - 这篇论文的研究主体不能算是agent，研究的重点其实是能否将LLM运用在社会学的相关调研中。作者研究了LLM对于一些人类群体的bias，发现可以通过身份、性格等资料调整一个LLM，让它来模拟特定群体的想法和评论。这种思想和为agent分配profile从而影响其行为的逻辑是一致的。
    - 在这篇文章的实验中，作者使用了ANES（美国国民选举研究）数据集中的真实选民数据，包括种族、性别、年龄、自我认同（党性强弱）等，将其交给GPT-3，让其扮演该选民投出选票。下表比较了GPT-3和真实的选民投票之间的关联性，其中tetra.指代tetrachoric correlation，prop. agree指代proportion agreement，两个数值都是反应两个变量之间的关联性的，越接近1则说明关联性越强。从结果可以看出GPT-3在被分配真实的profile之后，能够非常逼真地模拟真实人类的反应。
    - ![img](Out_of_one.png)

目前的工作都是独立分开地使用上述方法，本文的作者猜想可能结合使用会带来更好的效果。假设需要使用agent模拟社会发展，可以基于真实数据集来生成一部分profile，模拟现在的情况，再手动地构造一些未来可能会出现的身份的profile，用于模拟日后的社会发展状况。

#### Memory Module

Memory module的作用是存储从环境中获得的信息，让agents能够基于这些已记录的信息来计划后续的行为。本文从三个方面介绍了memory module：

- Memory structures：LLM-based agents的memory分为长期和短期两种，其中的短期记忆指**受Transformer结构限制的上下文窗口内的信息**，长期记忆则**使用额外的向量数据库或其他方法的方式来记录信息**，使agent能够长期、快速地反复查询回溯相关记忆。 基于这两种memory类型，目前的工作中有两种常用的记忆结构：
  - **Unified Memory**（单一/统一记忆）：只使用短期记忆的结构，通常以**in-context learning**的方式实现，相关的记忆信息会写成prompt的形式交给agent。使用短期记忆的方法能够非常直接地增强agent**对上下文相关的行为的感知能力**。
    - **RLP**（*[Reflective Linguistic Programming (RLP): A Stepping Stone in Socially-Aware AGI (SocialAGI)](https://arxiv.org/pdf/2305.12647.pdf)*）一文中，指出目前的LLM虽然对话能力很强，但是会缺乏一点“人的气质”；因此，该文章的作者为这个对话agent设置了内部状态，以prompt的形式提醒该agent，在聊天时应当遵循怎样的心理活动，从而增强其扮演对话角色的丰富和逼真程度。
    - ![img](RLP.png)
  - **Hybrid Memory**（混合记忆）：同时使用了长短期记忆，短期记忆用来**增强暂时的感知能力**，长期记忆用来**持久地巩固重要信息**。结合使用这两种方法能够增强agent的**长期推理和经验积累**的能力。
    - **Generative Agent**：在这篇文章中，agent会将自己观察到的、自己采取的行为、其他agent或环境对其造成的影响，以**长期的记忆流**的形式储存下来；短期记忆只关心agent的当前状态。agent的规划、反思行为都依赖于这个记忆流，甚至在agent每次采取行动时，都需要从记忆流中抽取信息来决定接下来的行为。
    - ![img](GenerativeAgent.png)
  - 需要注意的是，很少会有工作**只**使用长期记忆，因为agent所处的环境是动态、复杂的，每个需要执行的行为之间都会存在**很强的相关性**，所以上下文之间的关联性很强，几乎不太能脱离短期记忆而独立工作。
- Memory Formats：在agent中存储的memory会有很多种不同的存储格式，不同的格式有不同的特点，适用于不同的任务。作者在survey中列举出了几种比较常见的存储形式，实际上的存储形式非常多种多样，并不仅限于接下来列举的几种。
  - **Natural Language**：直接使用自然语言描述记忆相关的信息。优点在于能够以**非常灵活**、**容易理解**的方法表达记忆信息，而且包含**丰富的语义信息**，能够给与agent全面的指示。
    - **Reflexion**（*[Reflexion: Language agents with verbal reinforcement learning](https://openreview.net/pdf?id=vAElhFcKW6)*）：
    - 这篇文章提出了使用自然语言的方式来帮助模型从失败的经验中进行学习。具体来说，agent内部有Actor、Evaluator、Self-reflection和Memory四个结构，其中的Actor会在t时刻基于policy执行具体操作at，得到来自环境的反馈oi；Evaluator会对Actor的输出进行评估，输出一个具体的分数；Self-reflection会基于Evaluator的打分结果，对Actor的决策生成自然语言的反馈信息，并将这个反馈信息存储在memory中，作为长期记忆存储；Memory结构同样包含长短期记忆，属于hybrid memory结构，其中的短期记忆负责记录agent的决策，长期记忆用来存储self-reflection的输出结果
    - Actor与环境交互后做出决策，Evaluator对Actor的决策给出打分，Self-Reflection基于打分结果和决策给出反馈信息并存入memory，后续的决策结果便会受到该memory的影响。比起直接使用Evaluator的打分结果，使用自然语言能够给LLM的信息会更多。
    - ![img](Reflexion.png)
  - **Embeddings**：将记忆信息编码成向量的形式，从而**增强查询、读取记忆的效率**。
    - **MemoryBank**（*[MemoryBank: Enhancing Large Language Models with Long-Term Memory](https://arxiv.org/pdf/2305.10250.pdf)*）：
      - 这篇文章提出了一种让LLM的具备长期记忆的方法，使LLM具备了存储、回忆、更新自身memory的能力。LLM会将对话的内容和时间一并存储，并且基于该对话生成summary和用户肖像，这些存储下来的对话或summary都会以向量的形式存储。LLM需要查询记忆时，query会被编码成向量，再基于一定的搜索算法来返回与其最相似的向量。
      - 本文比较具有独创性的一点是，在设计记忆模块时，受到了人类的遗忘曲线启发，为LLM也设计了相似的遗忘机制，使其能够选择性地遗忘或者反复强化一些特定的记忆内容。
      - ![img](MemoryBank.png)
  - **Databases**：将记忆信息存储在数据库内，方便agent**有效、全面地对记忆进行操作**。
    - **ChatDB**（*[Chatdb:](https://arxiv.org/pdf/2306.03901.pdf)* *[Augmenting LLMs with databases as their symbolic memory](https://arxiv.org/pdf/2306.03901.pdf)*）：
      - 这篇文章和MemoryBank的关注点类似，指出LLM在长期记忆方面存在欠缺；虽然有些LLM尝试借助神经网络来提升自己的记忆能力，但是在需要长期推理的任务中表现不佳。作者认为，这是因为它们都没有以结构化的形式来存储历史信息，而且在操作记忆时的方法不够精确（没有采取符号化的操作方法）。
      - 文章提出的ChatDB是一种优化LLM的记忆能力的方法，总体结构包括一个LLM Controller和一个基于数据库的memory模块。LLM Controller负责控制记忆模块，在需要更新或读取记忆时生成SQL语句读写数据库。对用户的每个输入，如果不需要访问memory即可回答，那么就由LLM直接回答；若需要查询记忆或更新记忆，则会先将用户的输入分解成一系列对数据库的操作，再依次执行。
    - ![img](ChatDB.png)
  - **Structured Lists**：以结构化的列表存储记忆信息，能够以**有效简洁的方式传递语义信息**。
    - **GITM**（*[Ghost in the Minecraft: Generally Capable Agents for Open-World Environments via Large Language Models with Text-based Knowledge and Memory](https://arxiv.org/pdf/2305.17144.pdf)*）：
      - 这篇paper中，作者研发出了一个能够在Minecraft这个以探索为主要玩法的游戏中自主学习、探索、掌握新技能的agent，展现出agent有能力在开放世界中处理突发状况和长期、复杂的任务。
      - 有不少agent研究都基于MC这个游戏，可能是因为这个游戏能够在一定程度上模拟比较接近真实世界的环境，能够初步地考察agent和人类相似的一些能力，比如基于环境调整目标，记住一些自己需要掌握的技能等。
      - GITM借助了LLM能够分解任务的能力，将agent所遭遇的每个目标都递归地分解成子任务，显著提升了agent在任务中的成功率。另外，由于LLM在决策和规划方面的能力出众，与此同时又不擅长做底层的控制，所以GITM设计了很多结构化的动作，从而让LLM在规划过程中能够做出比较清晰的决策。更重要的是，这个agent具有反思和总结的能力；任务执行失败时会反思失败的原因，执行成功时的方案则会被记录在memory中，在后续遇到相似目标时作为参考。
      - 对agent的每个目标，结构中的LLM分解器会递归地将其分解为多个sub-goal，直到无法分解为止。分解的结果会以树的结构存储，然后以后序遍历的顺序来依次完成每个sub-goal。
    - ![img](GITM.png)
  - 还存在譬如代码的存储格式（Voyager），且这些存储格式之间并不互斥，GITM就使用了多种的存储格式。
    - **Voyager**（*[Voyager: An open-ended embodied agent with large language](https://arxiv.org/pdf/2305.16291.pdf)* *[models](https://arxiv.org/pdf/2305.16291.pdf)*）：
      - 这篇文章同样将agent所处的环境设置在在开放式的探索游戏Minecraft中，和GITM的主题比较类似，但侧重点有所不同；GITM更强调agent在任务完成度上的表现，而Voyager则强调agent的终身学习能力。
      - Voyager为agent设计了一个尽可能探索更多事物的要求，让它在探索过程中掌握新的技能，而和GITM不同的是，GITM设计了一系列结构化动作来帮助LLM做出决策，Voyager则直接让LLM编写可执行的代码来完成任务。Agent可能无法一次就生成正确的代码，因此LLM会迭代地检查agent所处的环境及代码报错来修正，直到能够成功执行为止。在查找需要用到的技能时，会先要求LLM生成一段query的描述，再基于该描述查询与之最接近的技能。
    - ![img](Voyager.png)
- Memory Operations：agent和环境的交互是基于和memory的交互进行的，memory让agent能够从环境中学习、积累、利用相关的知识。agent有三种针对memory的操作：**读**、**写**和**反思**
  - **Memory Reading**：指的是从已经存储的memory中抽取出有效信息，从而增强agent的行为能力。
    - 以**GITM**一文为例，当agent遇到和先前成功实现的目标相类似的新目标时，就会从记忆中查找先前成功时的执行方法。
    - 抽取信息的关键在于**抽取有效信息**的方法，这一点在Generative Agent一文中给出了方案，从recency、relevance、importance三个维度进行衡量。
      - **Recency**指的是记忆信息的时效性，越新存储的记忆越重要，得分越高。在Generative Agent中使用的是指数衰减函数来衡量。
      - **Importance**衡量信息的重要程度，在Generative Agent一文中直接让LLM在1~10之间打分；在记忆生成时，importance得分就会随之生成。
      - **Relevance**评价信息的相关性，文章中使用余弦相似度计算。
    - Generative Agent给出的方案启发了本文提出的方案，也就是所示的公式。其中的alpha、beta、gamma均是系数（超参数），在不同工作中取值不同，代表着不同的研究对信息的关注角度也不同。许多工作只在意信息的相关性（**GITM、Voyager**），有些工作则同等地注重所有方面的衡量（**Generative Agent**）。
    - ![img](GenerativeAgent_1.png)
  - **Memory Writing**：写操作是为了将agent从环境中感知到的信息存入记忆中。在这一操作内，需要解决两个问题：**遇到和已有记忆相似的新记忆时该如何处理**、**记忆存储达到上限时该如何处理**，也就是Memory Duplicate和Memory Overflow两个问题。
    - Memory Duplicate：这一问题关注的是**如何处理跟已有的memory相似度高或重复的记忆信息**，解决的思想也非常直觉，就是**将重复或冗余信息合并**。
      - 在**GITM**中，提到每个sub-goal都会记录成功的执行方案，相同sub-goal下的成功方案大体相同，但可能存在细微差异；在文章中，这些成功方案的数量一旦达到N（N=5），就会被总结成一个统一的方案，减少冗余存储。
    - Memory Overflow：这一问题则关注**如何在记忆容量达到上限时移除信息**，与之对应的解决方法是**设计比较合理的删除规则**。
      - 在**Reflexion**中，长期记忆使用一个滑动窗口存放。由于滑动窗口的大小依然有限，所以一旦出现overflow的问题，就会以FIFO的方式，删除最早存储的记忆。
  - **Memory Reflection**：指agent使用接近人类的能力来评估自己的认知进程，从而让agent具备独立概括、推理的能力，使其能够获得更加复杂或抽象或更高层次的信息。
    - 以**Generative Agents**一文为例，其中的agent就具备定期反思、总结其memory的能力。在Smallville环境中的一天时间下，每个agent都会进行三次reflection行为。在需要reflection时，首先会将该agent的memory中最近的100条记录交给LLM，要求LLM基于这些内容，想出3个比较高层次的问题（QG）；然后基于LLM生成的三个问题来查询memory中的相关内容，对每个问题及其对应的memory内容，要求LLM总结出一些新的洞见或观点。这些被总结出的新结论即为reflection的结果，最终也会被存入memory中。
    - ![img](GenerativeAgent_2.png)

    - 再举例如刚才提到的**GITM**，在处理重复或冗余记忆时，实质上也是对已有经验做出的总结，是将现有的方案抽象成一个更具泛化性的结论的过程。

#### Planning Module

Planning module使得agent也具备了和人类一样，将复杂的任务分解为更简单的子任务并加以解决的能力。本文基于agent能否在规划过程中得到反馈信息，将目前的规划策略分为了两类：

- Planning without Feedback：指在agent的规划过程中不会受到外界反馈的影响。这种规划方法的缺陷非常明显，比较适用于推理步数相对较少、相对简单的环境或任务中（相较于agent所处的动态复杂环境）。
  - ![img](planning.png)

  - **Single-path Reasoning**：最终目标会被分解成多个中间步骤，但是每个步骤后面只有一个步骤（单条路径）。CoT、Zero-shot CoT都采用的是这种规划策略。ReWOO、HuggingGPT同样使用单条路径的规划方法，但是和CoT的方案不同，会在每个步骤结束后访问LLM。
    - **ReWOO**（*[Rewoo: Decoupling reasoning from observations for efficient augmented language models](https://arxiv.org/pdf/2305.18323.pdf)*）：
      - 这篇文章的研究主体其实是Augmented Language Model（ALM），指的是借助外部工具来完成任务的LLM，如ReAct、AutoGPT等。文章指出，传统的ALM模型会在反复查询中产生非常大的计算开销和冗余问题，所以提出了解耦合的方法。
      - 当ALM遇到无法一次性解决的问题时，就会进行反复的查询，每一轮得到的结果都会拼接到前一轮结果上，导致输入变得越来越大，计算复杂度越来越高；而在本文中，每个问题会先被Planner分解成多个子问题和解决步骤，再交给Worker依次执行这些解决步骤（调用api或LLM），最后由Solver将得到的答案进行合并、总结。
  - **Multi-Path Reasoning**：对应于单路径的推理过程，多路径的每个分解会产生多个后续的结果。CoT-SC、ToT、AoT、GoT、LMZSP、RAP都采用这种推理方法，不过ToT、LMZSP、RAP在每次生成推理步骤时都需要访问一次LLM。
    - **LMZSP**（*[Language models as](https://proceedings.mlr.press/v162/huang22a/huang22a.pdf)* *[zero-shot planners: Extracting actionable knowledge for embodied agents](https://proceedings.mlr.press/v162/huang22a/huang22a.pdf)*）：
      - 这篇文章指出，LLM虽然能够在帮助下将任务分解成很多步子任务，但在子任务中给出的action容易产生语义上的歧义，或很难在仿真环境中执行。因此，文章提出了一套自定义的动作集合，在LLM给出action后，基于语义从中寻找最接近的动作执行。
      - 文章中使用的方法是，先向LLM给出示例，让其基于示例分解任务；对每个分解结果，使用预训练的masked LLM将其转化为可执行的action（在定义的可执行action空间中找到与之语义最接近的动作）。这里每一步生成的备选子方案都可能有很多，但是最终只会选取最可执行的一个。
      - ![img](LMZSP.png)
  - External Planner：借助外部的规划器来帮助agent进行任务规划，如LLM+P、LLM-DP、CO-LLM。
    - **LLM+P**（*[LLM+P: Empowering large language models with optimal planning proficiency](https://arxiv.org/pdf/2304.11477.pdf)*）：
      - 这篇文章认为LLM在做长期规划时的表现很差，所以采用了一些传统的规划方法来和LLM结合使用，提升LLM在这方面的能力。
      - LLM首先会接收到关于某个任务的描述，然后将其转化成Planning Domain Definition Languages（PDDL，一种用于描述规划问题的特殊语言，具有特定的格式和内容要求），再使用可以接收这种语言作为输入的规划器（如Fast Downward，使用一种叫“因果图”的数据结构来帮忙搜索解决方案）；外部的规划器会接收PDDL描述作为输入，基于各种各样的搜索算法得到PDDL方案，再经由LLM转化回原本的自然语言，即得到最终的方案。
      - 与之类似的还有**LLM-DP**（*[Dynamic Planning with a LLM](https://arxiv.org/pdf/2308.06391.pdf)*），同样是借助PDDL和外部规划器帮忙分解任务的。
    - ![img](LLM+P.png)
- Planning with Feedback：在规划过程中不借助反馈的方法不一定可行，因为很难从任务开始时就生成一个完善的计划，且计划的执行过程中可能会遇到很多无法预料的问题。模仿人类基于实际情况不断修正方案的这种模式，很多agent的planning module采用的都是能够接收反馈信息的设计，基于反馈信息的类型和来源对其进行分类。这种计划方式更适合用在长程、复杂的推理任务中。
  - **Environmental** **Feedback**：从客观世界或虚拟环境中得到反馈信息，可能是任务完成或失败的信号或行动采取后观察到的内容。
    - 在**GITM**一文中，给定agent一个目标，LLM会为其提供初步的解决方案；若agent没能成功达成该目标，那么任务失败的讯息和agent所处的环境的物理信息会作为feedback重新交给LLM，随之生成新的解决方案。
    - ![img](GITM_1.png)

    - Voyager一文中接受的feedback也与之类似。
  - **Human Feedback**：通过和人类的直接交互得到反馈信息，能够让agent学习到人类的价值观和偏好，帮助消除一些幻觉问题。
    - **Inner Monologue**（*[Inner monologue: Embodied](https://arxiv.org/pdf/2207.05608.pdf)* *[reasoning through planning with language models](https://arxiv.org/pdf/2207.05608.pdf)*）：
      - 这篇文章研究的对象是具身智能，即LLM+机器人的组合。作者通过使用多种不同的文本反馈信息（如上图所示，不局限于human feedback，还有environmental feedback），在未经额外训练的few-shot条件下，能够形成一种类似人类的内心独白（inner monologue）的反思机制，从而调整其接下来的行为，提升任务的成功率。
    - ![img](InnerMonologue.png)
  - **Model Feedback**：来自agent内部的一些反馈信号，这种信息通常来自预训练的模型。
    - **Reflexion**中，agent会得到evaluator给予其输出的评价，基于evaluator的这一评价，self-reflection结构会生成相应的自然语言反馈信息，再交给LLM，令其调整之前的输出。
    - **Self-Refine**（*[Self-Refine: Iterative Refinement with Self-Feedback](https://arxiv.org/pdf/2303.17651.pdf)*）一文的设计思路和Reflexion非常相似，模型同样会迭代地对自己的输出结果进行反馈和订正。具体来说，通过prompt让LLM对自己先前的输出结果生成feedback，然后基于该feedback再生成新版本的答案。这种迭代循环过程会一直持续到LLM对自己的结果满意，或是达到其他的停止条件为止。
    - ![img](SelfRefine.png)

#### Action Module

Action Module能够将agent的决策转变成实际的输出或操作，一般位于**最下游**的位置，能够**直接和环境交互**。

- Action Goal：agent通过采取行动希望达成的目的或效果，一般分为三种，完成任务、进行沟通或环境探索。
  - **Task Completion**：agent采取行动来实现特定的目标；agent做出的每个动作都有其明确的目标，且都是为了最终目标的实现而做的。
    - 举例来说，在**GITM**这篇文章中，agent采取行动的目的就是为了达成一些游戏内设置的任务目标，譬如制造特定的道具、采取素材等
  - **Environment** **Exploration**：希望让agent来探索未知的环境，扩大它的认知能力。
    - **Voyager**一文中的agent就是一个终身学习的agent，在环境探索的过程中来寻找其尚未掌握技能，或者是完善其已经掌握的技能。
  - **Communication**：agent采取的行动是用来和其他agent或人类交流、合作或交换信息的。
    - **ChatDev**（*[Communicative Agents for Software Development](https://arxiv.org/pdf/2307.07924.pdf)*）：
      - 这篇文章提出了一种基于交流的软件开发框架。该框架将软件开发的流程分解为设计、编程、测试、编写文档四个阶段，在每个阶段中设置多个agent并为它们分配不同的身份，譬如程序员、测试工程师等。这些agent之间通过聊天交流协作，在软件开发过程中达成一定的共识，从而完成整体的功能开发。
      - 具体来说，对一个具体的子任务，两个角色之间会进行多轮讨论，提出、验证解决方案，讨论的内容包括软件使用的语言、开发的形式、代码debug、测试等诸多方面。ChatDev的提出让软件开发的成本变得非常低，所需的时间成本也降得很低，大大提升了生产效率和成本。
      - ![img](ChatDev.png)
- Action Production：agent采取行动的逻辑和LLM不同，LLM的输入和输出通常是由较强的直接联系的，agent则需要基于不同的策略来生成下一步的行动。通常有两种策略：**基于当前任务，从memory中抽取信息**；**基于提前生成的计划**。
  - Action via Memory Recollection：agent从memory中抽取信息来生成下一步的行动。
    - 在**Generative Agent**一文中，agent每次采取行动前，都会查询一遍自身的记忆流，按照抽取memory的规则抽出得分最高的一件事。
    - **ChatDev**一文中，agent之间两两对话的记录都会被存进agent的memory中，影响其后续采取的行为。
  - Action via Plan Following：agent会基于其事先制定好的计划来采取行动。
    - 在**GITM**中，agent会通过将每个目标分解成多个小目标的方法来制订全局的计划，然后再根据该计划依次实现小目标，从而达到最终目标的效果。
  - 个人认为两种生成方法并不冲突，譬如GITM在执行每个sub-goal时，又会借助memory recollection的方法生成行动。
- Action Space：指的是agent能够执行的行为构成的集合，即行为空间，通常分为两类：**借助外部工具扩展的新能力**，或**LLM****自身就具备的能力**。
  - External Tools：在面对某些专业领域内的任务时，agent需要借助一些相应的外部工具来处理这些任务，这些外部工具可以是API、外部数据库或其他模型。
    - API：通过调用外部的API来扩充agent自身的行动空间。
      - **HuggingGPT**（*[HuggingGPT: Solving AI Tasks with ChatGPT and its Friends in Hugging Face](https://arxiv.org/pdf/2303.17580.pdf)*）：
        - 这篇文章是一个借助外部工具来为LLM扩展行为空间的典型例子。
        - 在该框架下，ChatGPT会作为一个中心控制器，在用户以自然语言向LLM发起具体的任务request之后，LLM会先将任务分解成一个序列（Task Planning），识别任务之间的执行顺序和依赖关系。为了提升任务规划的性能，框架还系统地定义了一套任务模板，方便模型做解析。然后，LLM会将任务和模型进行匹配，基于用户的query和HuggingFace社区中各种模型的描述来查询能够胜任当前任务的模型。在这里会对模型根据下载量做一个排序，选取topK作为候选模型（Model Selection）。选取模型后，就会运行这些模型来完成用户的任务；在任务完成后，LLM会将前三个阶段的内容summarize成一段话，一起返回给模型，包括分解出来的任务类型、使用的模型和模型的运行结果。
        - ![img](HuggingGPT.png)
    - Databases & Knowledge Bases：将agent与外部知识库链接来生成更多实际可执行的行动
    - External Models：使用其他的模型来解决问题，比起API的处理能力更强
      - **ChemCrow**（*[ChemCrow: Augmenting large-language models with chemistry tools](https://arxiv.org/pdf/2304.05376.pdf)*）：
        - 本文同样借助了外部工具来扩展模型本身的可执行空间，集中关注了化学领域上的自动化生产过程。
        - 由于LLM本身在化学方面的能力并不好，甚至会在一些基础化合物的化学式上发生错误，ChemCrow在LLM的基础上融入了18种不同的化学工具，成功减轻了LLM在化学领域的幻觉问题，允许化学研究者便捷地借助该工具来执行化学实验。ChemCrow能够给出某一化合物的详细且合理的合成步骤，在这一方面的表现甚至比GPT-4更佳。
        - ![img](ChemCrow.png)
  - Internal Knowledge：仅使用LLM自身的能力来解决问题，比如其规划能力、沟通能力和常识理解能力
    - Planning Capability：指的是LLM将大任务分解成子任务的能力。
      - 在**GITM**和**Voyager**中，借助LLM的这一能力，允许agent由下至上、递归地实现最终目标。
    - Conversation Capability：LLM生成高质量对话的能力。
      - **ChatDev**是基于对话建立的多agent系统，agent之间的交流协作全部依靠LLM本身的沟通能力；**RLP**是专门用于对话的agent，沟通能力自然很重要。
    - Common Sense Understanding Capability：LLM具备的理解人类常识的能力，让agent能够模拟人类的行为和日常生活。
      - 在**Generative Agents**和**RecAgent**这些文章中，依靠agent自身的常识理解能力，能够非常好地模拟出人类在虚拟环境中的行为。
- Action Impact：指的是行为可能会带来的影响。实际工作中，agent的行为可能会产生的影响非常多，本survey只详细列举了其中几项。
  - **Changing Environments**：通过行为来直接改变其所处的环境的状态，具体来说包括**改变位置、拿起物品、建造东西**等。在GITM和Voyager中的采集行为就会产生这样的效果。
  - **Altering Internal State**s：采取行为会影响到agent自身的内部状态，譬如影响其**memory的内容、生成新的计划、使其掌握新的知识**等。在Generative Agent中，agent采取行动后，memory也会随之更新。
  - **Triggering New Actions**：agent采取的一个行动可能会**引发下一个行动**。譬如在Voyager中，为了实现某个较大的目标，可能会在实现一个sub-goal之后立刻执行下一个sub-goal。

### Capability Acquisition

Capability Acquisition指的是**使agent具备胜任特定任务所需的技能**的这一过程。作者将agent进行能力学习的策略分为了三类：**fine-tuning**、**prompt engineering**、**mechanism engineering**，并基于此总结了（他们认为的）模型的能力学习策略的演化过程（见下图）：

![img](Capability.png)

个人认为作者提出的三种能力学习策略也对应着这张图上的三个阶段。微调方法对应于机器学习时代下的参数学习的方法，设计prompt使模型掌握能力的方法则是LLM时代的新学习方法，在此提出的mechanism engineering对应于agent时代，是更新的学习策略。

- **Fine-tuning**：在对应特定任务的数据集上进行微调，从而使agent具备相应能力的方法。作者在此又基于fine-tuning使用的数据集的构造方法进行了细分，分为人工标注数据集、LLM生成数据集及来自现实世界的数据集三类，但实际上使用的方法都是微调，所以不详细展开。
  - **ToolLLM**（*[ToolLLM: Facilitating Large Language Models to Master 16000+ Real-world APIs](https://arxiv.org/pdf/2307.16789.pdf)*）：
    - 这篇文章就使用了微调的方法来使LLM具备了调用外部API的能力，在LLaMA的基础上微调出了ToolLLaMA。
    - 作者首先在RapidAPI这个比较主流的API市场上收集了大量的API。这些API具备非常强的层级性，从上到下按照category、tool、API的层级排列而成。然后使用了ChatGPT，要求其基于给出的API生成调用的指令。在指令生成这一步中，作者关注了API的多样性和多工具使用的情况。因为在现实情况下，同时使用多个API实现复杂的功能是非常常见的要求。这里的多工具使用的采样就是基于API的category进行采样，因为作者发现属于同一category下的API之间的联系性会更强。
    - 同时，作者还会要求ChatGPT对给出的每条指令，生成一个可行的动作序列。每个动作序列都是ChatGPT和API response之间的一次多轮对话。作者为这个动作序列定义了两种收尾的标志，一种就是成功调用API得到回应的结尾，另一种则是因为失败而给出give up的讯号。另外，在ChatGPT生成行动序列的过程中，作者还使用了DFSDT（深度优先搜索决策树）来提升模型的推理性能。
    - 基于这种方法，作者得到了微调用的数据集ToolBench，然后使用ToolBench微调LLaMA，得到ToolLLaMA。ToolLLaMA的表现好于常规方法（ChatGPT+ReAct）。
    - ![img](ToolLLaMA.png)
- **Prompting Engineering**：通过设计合理的prompt来提升agent的表现。先前介绍的**RLP**一文中，在每一轮对话时使用prompt为agent设置与人类相似的心理活动，从而使agent做出的回应更加逼真，提升了模型性能。前面提及的**Self-refine**、**Reflexion**也采用了类似prompting engineering的能力学习方法。
  - **Retroformer**（*[Retroformer: Retrospective Large Language Agents with Policy Gradient Optimization](https://arxiv.org/pdf/2308.02151.pdf)*）：
    - 这篇文章使用的方法也是prompting engineering，思路上和**Self-refine**、**Reflexion**两篇文章很类似。本文的作者认为，使用自然语言让LLM为自己指出错误以提升性能的这种方法不够完善，模型很有可能意识不到问题具体出在哪一环节，因此在本文中提出了一种借助强化学习策略来增强prompt的方法。
    - 文章提出了一种框架，将一个回顾性的模型与LLM相结合；该回顾性模型将会使用例如policy gradient的强化学习策略进行训练，使其具备能够基于环境反馈而自动调整prompt的能力，从而增强prompt对LLM的指导能力。这个回顾性模型会将用户输入的prompt、来自环境的反馈及reward都作为输入，从agent的动作序列中检测出具体是哪一个动作出了问题，再将该信息融入prompt交给Actor LM，起到指导作用。
    - ![img](Retroformer.png)
- **Mechanism Engineering**：指的是设计合适的进化机制，概念非常广泛，包括设计特别的模块、引入新的工作规则等各种各样从设计层面增强agent能力的策略。文章中列举了四种比较有代表性的mechanism engineering策略：
  - **Trial-and-error**：试错法，具体指先让agent采取行动，再使用critic对其行动进行评价，最后基于该评价来校正agent的行为模式。和前述的self-reflection的区别可能在于是否会借助prompt进行校正行为，这里的试错法也许不借助prompt就能进行自我纠正。
    - **RAH**（*[RAH! RecSys-Assistant-Human: A Human-Centered Recommendation Framework with LLM Agents](https://arxiv.org/pdf/2308.09904.pdf)*）：
      - 这篇文章提出了一个基于agent辅助的、以用户为中心的推荐系统。文章的作者认为，传统的以推荐系统为中心的设计方法面临非常多的困境，譬如在切换到新的领域时会遇到的“冷启动”问题，平衡用户隐私和推荐内容的关系；解决这些问题需要从用户角度入手，考虑以用户为中心的方法，提出了RAH框架，使用agent和LLM的能力来协助用户。
      - 在该推荐系统中，多个agent将会共同构成一个assistant，这个assistant会充当用户和推荐系统之间的中介。具体来说，当系统为用户推荐内容时，assistant会基于其学习到的用户偏好为其进一步过滤；用户选取感兴趣或不感兴趣的内容时，assistant会先接收到用户反馈，基于该反馈学习用户的特点，最后生成assistant的反馈信息，选择性地将用户偏好交给推荐系统。
      - ![img](RAH.png)

      - assistant由5个agent组成，perceive agent会首先接收、理解来自用户的信息，并增强文本的内容和特征；learn agent接收perceive的输出、用户的反馈，分析用户喜欢和讨厌的内容；act agent会在接收到某一内容时，基于学习到的用户偏好来预测用户是否对该内容感兴趣。这一步中使用了CoT，先让agent猜想用户喜欢或讨厌该内容的原因，再让其分析用户对该内容的态度，并预测用户可能给出的评价，最终预测用户的态度；critic agent会对act agent的预测结果给出评判，在这一步中会将act agent的结果和用户的实际态度进行比较，提出两个结果的区别，分析agent犯错的具体原因，给予修正建议，从而督促learn agent重新学习用户的偏好。这就是本文使用的Learn-Act-Critic循环，也就是试错法的具体表现；reflect agent会在成功学习到用户特征之后，将原本的用户特征结合分析，查找是否存在重复或冲突的内容，保证用户偏好的一致性。
      - ![img](RAH_1.png)
  - **Crowd-Sourcing**：指让多个agent各自提出观点，再通过多轮讨论使agent之间相互理解，从而合并观点，达成最终的一致的结论。这种方法适用的场景是多agent的系统，但是在agent相关的研究中很少出现。在这里引用的这篇文章可能是唯一一篇提出这种策略的相关研究。
    - *[Improving Factuality and Reasoning in Language Models through Multiagent Debate](https://arxiv.org/pdf/2305.14325.pdf)*：
      - 在这篇文章中，作者提出了一种让多个LLM之间相互讨论的方法：对于每个问题，先让每个LLM各自生成答案，然后让它们阅读、评价其他LLM生成的答案，再基于此更新自己的答案；重复这一步骤多轮，直到系统得出最终答案。这种方法和其他的模型性能提升方法是可以兼容的。
      - 通过研究，作者发现这种多轮讨论会使每个agent修改自己不太确定的答案，消除回答中的错误内容，从而实现推理正确率的提升。在后续研究中，作者还发现讨论轮数、参与讨论的agent个数的增多会使得这种多轮讨论的准确率提升更加明显。
      - ![img](CrowdSourcinng.png)
  - **Experience Accumulation**：指的是agent基于自身过去成功执行的案例来学习新能力的方法。这种方法比较常见，先前介绍的很多agent相关研究都用到了这一策略，且基本上都需要借助memory module，通过读写memory的方式实现。
    - 在**GITM**一文中，agent在成功达到某一目标后，就会将该目标和与之对应的解决方案存储到memory结构中。在后续再次遭遇到相似或相同的目标时，就会基于先前存储下来的成功方案采取行动。
    - 与之类似的，**Voyager**一文中，agent会将自己成功掌握的技能以可执行代码的形式存入skill library中，并且在执行过程中不断完善该代码。在后续的探索中，agent就可以不断地使用这些已掌握的技能来应对各种情形。
    - **MemPrompt**（*[Memory-assisted prompt editing to improve GPT-3 after deployment](https://arxiv.org/pdf/2201.06009.pdf)*）：
      - 这篇文章关注的是LLM对用户提问产生的误解。举例来说，当用户询问“什么词语和good很相似”时，存在理解上的歧义，LLM可能会误以为用户在询问发音类似的词语，而用户实际上询问的是近义词。为解决这种误解，文章提出一种基于用户反馈的增强方法，让用户对模型的误解给出相应的回馈，模型会将该回馈存储下来，并且长期地基于该回馈信息纠正自己的行为，防止再犯类似的错误。
      - 在模型回答用户的问题时，它会同时给出自己对该问题的理解。譬如当用户询问“什么词语和good很相似”时，模型会在答案前加上类似“和good发音类似的词语是”的内容，帮助用户察觉模型在理解问题上产生的偏差，从而给出反馈。
      - ![img](MemPrompt.png)
  - **Self-driven Evolution**：由agent自身驱动的进化，具体指的可能是agent自动地为自己设立目标、探索环境、接受反馈的过程。
    - **LMA3**（*[Augmenting Autotelic Agents with Large Language Models](https://arxiv.org/pdf/2305.12487.pdf)*）：
      - 在这篇文章中，作者设计了一个基于LLM的框架，使agent具备了自主设定目标、学习新技能的能力。
      - 该框架分为Goal Generator、Relabeler和Reward Function三部分。Goal Generator会基于agent已经掌握的内容和环境中可以进行的操作，生成更high-level的可行目标，同时还会将其分解为几个子目标，方便agent实现；agent基于一定的学习策略来实现生成的目标，relabeler就会对这一过程中掌握的内容进行重新标注，将这些学习到的内容视作agent的新能力；reward function会对其学习成果进行评判，确认agent掌握了哪些新能力。在这样的框架下，agent就会自主地为自己设计目标、学习新技能、增长能力，自我驱动地实现进化。
      - 个人感觉LMA3的思想和Voyager、GITM这种同样能够“自主学习”的agent的区别就是它agent的目标空间是无界的，是纯粹由agent自行定义的。
      - ![img](LMA3.png)

## Application

![img](Application.png)

Agent的应用这一章讲的内容比较少，作者分了社会科学、自然科学及工程三个方面，简要介绍了agent在各领域内的相关工作。

- **Social Science**：
  - 在**心理学**领域上，agent通常用来进行模拟实验、提供心理健康方面的帮助等。但与此同时，agent可能会在模拟实验中表现得“过于完美”，影响下游任务。
  - **政治经济学**方面，agent的主要作用就是模拟实验。在先前介绍的**Out of one, many**一文中，研究团队就探究agent在模拟人类的政治立场上的能力，发现可以通过使用特定的身份特点来模拟特定人群的政治观点。
  - **社会模拟**是agent广泛运用的领域。通过使用agent，大幅降低了社会研究所需的高昂成本。**Generative Agents**一文就构建了一个由多agent模拟的人类社会，研究了agent的社会行为模式。此外，还有一些工作借助模拟课堂情况来研究了学生的基础认知技能、借助社会模拟来研究信息的传播模式等。
  - **司法**和**研究助理**这两个方面也可以借助agent开展工作。agent能够帮助在司法审判中做出更加合理的决策，也能够用来进行一些社会科学研究的辅助工作，譬如生成文章摘要、抽取关键词、提出社会研究的框架等。
- **Natural Science**：
  - 在自然科学领域中，agent可以用来帮助进行**文档与数据管理**，或是起到**实验助理**的作用**。**例如设计实验流程、查询数据、使用代码进行简易计算等步骤都可以借助agent自动化进行。**ChemCrow**就是在自然科学方面功能非常完备的工作，该研究借助agent了和外部工具来设计化学实验。
  - 另外，agent还可以用来进行**学科教育**，它能够用来帮助学生了解一些科研方法、实验及分析思路，也可以作为一种教育工具使用。
  - **CodeHelp**（*[CodeHelp: Using Large Language Models with Guardrails for Scalable Support in Programming Classes](https://arxiv.org/pdf/2308.06921.pdf)*）：
    - 本文将agent运用在了编程教育上，提出了一个能够即时为学生提供指导的编程学习辅助系统。这个系统不会直接为学生提供答案，而是给出适当提示、引导学生自行解决问题，很好地避免了学生过度依赖这一工具；另外，从教师角度出发，CodeHelp系统还提供了一系列观察、检测、评估学生学习情况的工具。
    - 系统会要求学生给出代码的语言、遇到问题的代码片段、问题描述及报错信息，让LLM基于这些request内容给予学生即时回应。LLM会先判断学生给出的信息是否充分，不充分则要求学生重新完善request内容，充分则进入推理过程；推理过程中，LLM本身会被要求不要给出示范代码和纠错后的代码，同时还支持教师屏蔽某些关键词的查询，而后生成两个response，从中选取得分最高的一个作为返回信息；在返回前，系统会再一次检查response内容中是否含有代码块，若含有则要求LLM删除代码块并重写response。
    - ![img](CodeHelp.png)
  - **EduChat**（*[EduChat: A Large-Scale Language Model-based Chatbot System for Intelligent Education](https://arxiv.org/pdf/2308.02773.pdf)*）：
    - 本文将agent系统运用在了教育领域上，通过在人工标注的教育相关指令集及相关书籍上训练和微调LLM，使得LLM具备了教育相关的知识和技能。此外，为了解决LLM的幻觉问题，作者还设计了一种针对查询增强的开放域QA，将互联网上最新的相关语料作为LLM外部的知识库。
    - ![img](EduChat.png)
- **Engineering**：
  - 在工程领域中，agent可以使用在**土木工程**领域，帮助进行建筑结构的设计与优化；还可以使用在**工业自动化**方向上，用来智能地规划、控制工业生产过程。
  - **计算机科学****和****软件工程**领域，agent可以用来进行自动化的开发，包括编程、测试、debug、写文档一系列完整流程，还能作为多功能的生产力工具。先前介绍的**ChatDev**就利用了多agent协同来自动化软件开发过程；**ToolLLM**将agent与外部工具融合，使其具备了多方面的完善能力，与之类似的还有**HuggingGPT**。
  - 在**机器人**（或**具身化****AI**）方向上，agent与机器人相关技术融合，开始在实际环境中解决具体的问题。在先前介绍的**Inner Monologue**一文中，机器人就搭载了agent系统，提升了其规划、解决问题的能力。
  - **MetaGPT**（*[MetaGPT: Meta Programming for A Multi-Agent Collaborative Framework](https://arxiv.org/pdf/2308.00352.pdf)*）：
    - 这篇文章和**ChatDev**的工作类似，都是构建了一个多个agent相互协作的软件开发框架。相较于ChatDev，MetaGPT对软件开发的过程、agent之间的交互都提出了比较标准明确的规范，在生成代码的可执行性上取得了比较大的提升。
    - 在本文中，作者使用了SOP（Standardlized Operating Procedures，标准操作流程）这一概念，为agent分配了不同的角色和职责，定义了工作流程。另外，本文规定agent之间使用结构化的文档或规范格式进行交流，防止自然语言的对话产生歧义或其他问题；在debug方面，作者优化了debug的迭代流程，让engineer自己负责迭代地修改、优化代码。
    - 比较具有独创性的是，MetaGPT在系统中设计了一个公用的信息池，解决了agent需要一个一个通信的低效率问题；每个agent都会订阅自己需要的内容，在新内容发布时，只有订阅的agent会接收到相关的信息。
    - ![img](MetaGPT.png)

## Evaluation

围绕agent的评估，作者在survey中分为了主观、客观两类评估方法：

![img](Evaluation.png)

### **Subjective Evaluation**

主观的评估方法指的是通过人类的评判来评估agent某一方面的能力。在缺少评估用指标，或在评估某一难以量化的能力时，通常会使用主观的评估方法。

这种方法能够反映人类的态度和想法，使agent与人类的价值观相匹配，但缺点是耗时耗力，且可能会引入偏见。在一些工作中，会使用其他的LLM代替人类进行打分。

主观的评估方法分为两种：

- **Human Annotation**：由人工标注者直接对agent的表现进行打分。
  - **Generative Agents**采用的评估方法就是直接打分。研究团队想要对他们提出的memory stream机制的有效性进行评估，所以询问了agent五个问题，检查了agent对自身的认知（**self-knowledge**）、有关其他人的记忆（**memory**）、对未来行动的规划（**plan**）、在假定情境下的反应（**reaction**）及更高层次的认知问题（**reflection**）。然后，要求标注者们了解每个agent的具体信息后，对其做出的回答进行打分。
- **Turing Test**：字面意思是图灵测试，具体指的是让人类区分agent的输出和实际的人类回答两者之间的区别。如果难以区分，则能够反映出agent在这一方面的表现能够到达人类水平。
  - **Out of one, many**一文中，为了对agent模拟人类的水平进行评估，研究团队让人类尝试区分了agent获取个人特征等信息后生成的response与来自真人的response，从而证实了agent具备模拟社会研究的能力。

### **Objective Evaluation**

客观的评估方法指的是使用数量指标进行评估。与主观的评估方法相比，这种客观的评估方法更加具体、可测量。

关注客观的评估方法时，主要分三个方面讨论：metrics、protocols、benchmarks。

- **Metrics**：指的是衡量能力用的指标，基于现有的agent工作，一般分为以下几类：
  - **Task success**：用于衡量agent在完成任务或实现目标方面的能力，包括**success rate**（**LLM****+P**、**Reflexion**、**MemoryBank**），**reward/score**，**coverage**（**GITM**），**accuracy**（**ChatDB**）。
  - **Human similarity**：衡量agent的行为与人类的相似程度，包括**trajectory/location accuracy**、**dialogue similarities**、**mimicry of human responses**（**Out of one, many**）。
  - **Efficiency**：评估agent的效率，不局限于运行效率，还包括开发成本等方面。具体有**planning length**、**development cost**（**ChatDev**）、**inference speed**（**GITM**、**Voyager**）、**number of clarification dialogues**。
- **Protocols**：指的应该是指标的考察重点（或使用方法），按照agent的使用环境可以分为几类：
  - **Real-world simulation**：agent通常会设置在游戏或其他可交互的环境中，被要求自动执行一些行动来达成一些目标。在这些情境中，通常使用任务成功率或与人类的相似度来衡量agent的表现，主要考查的是agent在拟真环境下的实践能力。
  - **Social evaluation**：这些情景中，agent会被设置在一些模拟的社会或社交网络中，被要求进行一些团队协作或讨论的任务。这种方法考察的是agent在合作、交流、相互理解以及模仿人类的社会行为等几个方面的能力。
  - **Multi-task evaluation**：这个场景比较宽泛，就是指要求agent完成一系列不同领域和类型的任务的考察情境，考察的主要是agent在开放域环境中的泛化能力。
  - **Software testing**：要求agent执行一些软件开发相关的任务，包括生成测试案例、debug、使用外部工具等，考察的就是agent在编程方面的能力。
- **Benchmarks**：具体的评测数据集，作者在这里没有进行系统性的分类，所以选了几个比较有代表性的benchmark介绍一下：
  - **AgentBench**（*[AgentBench: Evaluating LLMs as Agents](https://arxiv.org/pdf/2308.03688.pdf)*）：
    - 本文是第一篇对agent进行系统性评估的工作，文中提出的benchmark考察了agent在多个领域中的能力，涉及了操作系统、数据库、知识图谱、卡牌游戏、解谜游戏、模拟家庭环境、网络购物与网络浏览这八个不同的环境以及与其对应的任务，评估了共计27个基于LLM的agent的性能。研究发现，GPT-4能够胜任上述各种符合现实情境的任务，而开源的LLM性能普遍差于非开源的商业LLM。
    - ![img](AgentBench.png)
  - **GentBench**（*[Gentopia: A Collaborative Platform for Tool-Augmented LLMs](https://arxiv.org/pdf/2308.04030.pdf)*）：
    - 这篇论文的主要贡献实际上是提出了一个叫做Gentopia的框架，支持以非常简单灵活的方式定制agent，将各种LLM、任务与插件集成到了统一的框架下。在此基础上，文章中还给出了一个benchmark，也就是这里的GentBench，用来全面地评估用户设计出来的agent在推理、知识水平、安全性、跨语言能力及效率这几方面的性能。在构建数据集时，作者还借助gpt-3.5-turbo来筛除难度太低的数据。
    - ![img](GentBench.png)

## Challenges

作者在survey的最后列举了几条agent研究将会遭遇的挑战和困难：

- **Role-playing Capability**：对于大部分的agent系统来说，都需要为agent分配特定的身份来使其履行职责，在LLM预训练过程中缺失的那部分角色信息会使其在相关研究中成为阻碍；另外，特别在心理学研究方向上，LLM缺少自觉和心理特征，可能需要人为地为其设计prompt或微调，才能使其具备该方向上的工作能力。
- **Generalized Human Alignment**：和数据伦理等内容关联较大。作者认为，如果需要模拟真实的人类社会，那么就有必要使agent具备一些不太正面、更加泛化的品质，因为模拟人类社会的最终目的还是发现并解决问题。这个方法可能会使agent变得具有危害、有悖伦理。
- **Prompt Robustness**：有相关研究关注过prompt的稳定性，发现只需要在prompt上做一点修改，就会使模型的结果变得截然不同。作者认为，在agent研究中，囊括的prompt数量不少，甚至会有prompt框架出现，一个prompt可能会影响到其他的prompt，导致模型的表现不够稳定（没懂这部分）。
- **Hallucination**
- **Knowledge Boundary**：作者认为，有必要在一些研究中控制agent或LLM的知识边界。譬如在模仿人类行为时，借助agent研究其对某部电影的偏好或厌恶；基于实际，agent在判断好恶前应当是没有任何前置知识、仅凭猜想的，但可能在预训练过程中agent已经掌握了相关知识，导致模拟并不完全真实。
- **Efficiency**