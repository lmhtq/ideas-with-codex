# LLM 记忆系统演化调研：从 RAG 到 Memory OS、参数内化与 Neural Memory

调研日期：2026-06-03  
范围：以 ChatGPT 发布后的 LLM 记忆系统演化为主，简要补充 ChatGPT 前史。  
结论先行：2026 年之前没有单一路线“解决记忆”。现实可落地的方向是混合记忆系统：短期上下文压缩 + 可审计文本/结构化记忆 + 检索/图谱层 + 必要时的参数化适配 + 严格的写入、冲突、遗忘、溯源和评估机制。

---

## 0. 执行摘要

LLM 的“记忆”在 2022-2026 年经历了五次重要迁移：

1. **从无状态上下文到显式上下文管理**：ChatGPT 初期主要依赖当前对话窗口，工程上用 rolling summary、conversation buffer、profile note 和 prompt 压缩补洞。
2. **从知识检索到 RAG 记忆层**：RAG 解决“外部知识可查”，但检索命中不等于系统真的记住。它强在可审计、可删除、低风险更新，弱在行为连续性、冲突处理和长期人格/偏好演化。
3. **从被动检索到 agent memory**：Generative Agents、MemoryBank、MemGPT/Letta、A-Mem、LightMem、MemRL、MemEvolve 等把记忆放进 agent 循环，出现 write-manage-read、reflection、self-editing、sleep-time update、runtime RL 等机制。
4. **从 memory layer 到 Memory OS**：MemOS、EverMind/EverOS/EverMemOS、MemoryOS、Mem0、Zep/Graphiti、Memori、TencentDB Agent Memory 等把记忆变成系统资源，开始强调生命周期、层级、调度、迁移、版本、治理、benchmark 和 observability。
5. **从外部文本到参数/latent/tensor 内化**：LoRA、model editing、MemoryLLM、MemLLM、Gist Tokens、ICAE、Titans、MIRAS、MSA 等试图把记忆从“可读文本”压缩为权重、soft prompt、memory token、KV/activation 或 neural memory。潜力高，但可控删除、解释性和产品成熟度仍落后于外部记忆。

一个关键区分：**RAG 是 retrieval memory，产品 memory 是 personalization memory，agent memory 是 behavior memory，参数/latent memory 是 internalized memory**。把它们都叫“记忆”会掩盖重要差异。

---

## 1. 领域综述与 Awesome List 反向校验

为了避免只按概念分类导致漏项，本调研先用综述论文和 curated list 做反向校验。

### 1.1 综述论文给出的主流框架

近两年的综述把 agent memory 从“存储”提升到完整生命周期：

- [Memory for Autonomous LLM Agents: Mechanisms, Evaluation, and Emerging Frontiers](https://arxiv.org/abs/2603.07670) 将 agent memory 形式化为 write-manage-read loop，并把机制归为 context-resident compression、retrieval-augmented stores、reflective self-improvement、hierarchical virtual context、policy-learned management。
- [From Storage to Experience](https://arxiv.org/abs/2605.06716) 将演化分为 Storage、Reflection、Experience 三阶段：先保存轨迹，再从轨迹中反思，最后抽象可迁移经验。
- [From Human Memory to AI Memory](https://arxiv.org/abs/2504.15965) 用人类记忆映射 AI memory，强调 object、form、time 三个维度。
- [AI Meets Brain](https://arxiv.org/abs/2512.23343) 从认知神经科学连接 agent memory，强调生命周期、benchmark、安全和 skill acquisition。
- [Human-inspired Perspectives](https://arxiv.org/abs/2411.00489) 把长期记忆放在人类启发式视角下讨论。
- [LLM Agent Memory: A Survey from a Unified Representation-Management Perspective](https://openreview.net/forum?id=KPs1EgGKcT) 强调 representation 与 management 的统一：记忆不只是存储格式，也包括写入、更新、压缩、删除、冲突与检索策略。

这些综述共同指向一个结论：记忆系统的难点已经从“有没有地方存”转向“怎么写、怎么整合、怎么忘、怎么验证、怎么避免记忆腐化”。

### 1.2 Awesome list 用于发现遗漏，不用于直接定论

反查的 curated list：

- [Awesome-Agent-Memory](https://github.com/TeleAI-UAGI/Awesome-Agent-Memory)
- [awesome-long-term-memory](https://github.com/xiaowu0162/awesome-long-term-memory)
- [awesome-ai-memory](https://github.com/topoteretes/awesome-ai-memory)
- [awesome-rag](https://github.com/coree/awesome-rag)

这些列表确认了几个需要补入主线的方向：

- **自进化记忆**：MemRL、MemEvolve、Evo-Memory、TAME。
- **轻量 MAG 与 sleep-time update**：LightMem。
- **Memory OS 生态**：MemOS、MemoryOS、EverMind/EverOS/EverMemOS。
- **Benchmark 扩展**：LoCoMo、LongMemEval、BEAM、MemBench、MemoryAgentBench、AgentMemoryBench、EverMemBench、Minerva、LoCoMo-Plus。
- **工程生态**：Mem0、Zep/Graphiti、Letta、Memori、TierMem、SuperLocalMemory、TencentDB Agent Memory、Cognee、Basic Memory、OpenMemory MCP、LangMem、Supermemory。

### 1.3 项目可信度分层

后文对项目采用三层收录规则：

| 层级 | 标准 | 处理方式 |
|---|---|---|
| 一级重点 | 有论文、代码、benchmark、官方文档或产品落地证据 | 进入主线分析 |
| 二级生态 | GitHub 活跃或工程实现清晰，但证据主要来自 README、博客、自报 benchmark | 进入项目索引，标注证据边界 |
| 三级观察 | Reddit、社区发布、营销页、未充分复现或争议较大 | 进入生态雷达，不作为核心论据 |

所有 benchmark 数字都需要标注来源：论文、官方 repo、自报 README、第三方复现、社区讨论不能混为一谈。

---

## 2. 前史：记忆作为知识增强

ChatGPT 之前的“记忆”主线不是个性化长期记忆，而是**让模型访问外部知识或长距离上下文**。

### 2.1 关键节点

- **Memory Networks / Neural Turing Machine / Differentiable Neural Computer**：早期把 memory 当成可读写外部结构，但并未成为 LLM 产品架构。
- **kNN-LM / cache LM**：用最近邻或连续 cache 增强语言模型预测，属于非参数知识补充。
- **REALM**：训练时把检索与语言模型结合，开创 retrieval-augmented pretraining 路线。
- **RAG**：把检索文档作为生成条件，成为后续知识增强和企业问答的基础范式。
- **RETRO**：DeepMind 的 retrieval-enhanced transformer，把外部检索纳入模型生成路径。
- **Transformer-XL / Compressive Transformer**：解决固定上下文窗口问题，为后来的 recurrent memory、compressed context 打基础。

### 2.2 与今天长期记忆的区别

早期系统主要回答“模型不知道的知识如何补进去”。今天的 LLM memory 还要回答：

- 是否能跨会话持续影响行为？
- 是否能记住用户偏好、事件、关系、任务状态和失败经验？
- 是否能更新、纠错、合并、删除和溯源？
- 是否能处理时间变化和冲突事实？
- 是否能在低延迟、低 token 成本下稳定运行？

所以，RAG 是后续长期记忆的重要基础，但不等价于长期记忆。

---

## 3. ChatGPT 后第一代记忆：上下文、摘要与文本压缩

2022 年 11 月 ChatGPT 出现时，产品体验强烈暗示“它在对话里记得我刚说过的话”，但这种记忆主要来自当前上下文窗口。跨会话记忆并不存在或非常有限。

### 3.1 第一代工程补丁

典型实现包括：

- **Conversation buffer**：把历史消息原样塞回 prompt。简单、忠实，但成本线性增长。
- **Rolling summary**：超过窗口后把旧对话摘要化。成本低，但会产生摘要漂移和事实丢失。
- **Profile note**：抽取用户偏好、身份、项目背景，形成短文本画像。
- **Task/session replay**：在 coding agent 中保存关键操作、失败、约束、文件状态，下次恢复。
- **Declarative memory files**：如 `CLAUDE.md`、`AGENTS.md`、`MEMORY.md` 一类项目级上下文文件，用纯文本维持长期背景。

这类方案的优点是可读、可编辑、可审计；缺点是写入标准粗糙、压缩损失不可控、容易把过期信息长期污染 prompt。

### 3.2 纯文本与 prompt 压缩

代表方法：

- [LLMLingua](https://arxiv.org/abs/2310.05736)：用小模型或打分机制压缩 prompt，减少 token 成本。
- [LongLLMLingua](https://arxiv.org/abs/2310.06839)：面向 long-context 与 RAG 场景，试图缓解 lost-in-the-middle。
- Selective Context：删除低信息量 token，保留任务相关上下文。
- [Gist Tokens](https://arxiv.org/abs/2304.08467)：训练模型把 prompt 压缩成可缓存的 gist token。
- [ICAE](https://arxiv.org/abs/2307.06945)：把长上下文压缩为短 memory slots，属于文本压缩和 latent memory 的过渡。

趋势判断：文本压缩仍然是最现实的基础设施，尤其在 agent 长任务中很重要。但它不是完整记忆系统；它缺少事实生命周期、版本、冲突处理和主动检索。

### 3.3 2025-2026：从 prompt compression 到 agent context compaction

2025 以后，压缩问题从“把一个 prompt 变短”扩展为“长时间运行的 agent 如何持续整理上下文”。这比传统 prompt compression 更难，因为 agent 的上下文包含用户目标、工具调用、错误分支、中间推理、文件状态、外部环境反馈和历史决策。

代表方向：

- [Prompt Compression for Large Language Models: A Survey](https://aclanthology.org/2025.naacl-long.368.pdf)：系统整理 hard prompt compression、soft prompt compression、example selection 等路线，说明压缩已经成为独立研究问题。
- [An Empirical Study on Prompt Compression for Large Language Models](https://arxiv.org/abs/2505.00019)：从生成质量、幻觉、多模态、词省略等维度比较不同压缩方法，提醒压缩不是单纯 token 变少。
- [ACON: Optimizing Context Compression for Long-horizon LLM Agents](https://arxiv.org/abs/2510.00615)：针对长程 agent，将环境观察和交互历史压缩为更短的自然语言 condensation，并用失败案例反推压缩规则。
- [Active Context Compression](https://arxiv.org/abs/2601.07190)：把 context compression 作为 agent 的自主 memory management 行为。
- [Memento](https://www.microsoft.com/en-us/research/articles/memento-teaching-llms-to-manage-their-own-context/)：让模型在完成一个 context block 后产出 memento，即高密度结论、关键中间值和策略决定，并用 block masking 降低无效历史的注意力成本。
- [Lossless Prompt Compression via Dictionary-Encoding and In-Context Learning](https://arxiv.org/abs/2604.13066)：探索字典编码和 in-context decoding，适合重复结构化数据，但依赖模型正确理解编码协议。

这一阶段的经验是：

- **摘要不是中立操作**：摘要会改变证据权重，也会丢失异常、反例、约束和未完成事项。
- **压缩链会累积损失**：一次摘要可接受，多次 auto-compact 后容易出现“记忆腐化”。
- **原文仍需可回溯**：高质量 compaction 应保留 raw log 或 evidence pointer，不能只留下不可验证摘要。
- **压缩应按任务边界触发**：工具调用、子任务完成、错误恢复、用户确认点比固定 token 阈值更适合作为 compaction 时机。
- **纯文本压缩最适合作为第一层记忆**：它便宜、可读、可编辑，但应与检索层、事件日志和结构化事实层配合。

---

## 4. RAG 与图谱记忆：外部化记忆的主干

RAG 的本质是把知识外部化：模型不需要把所有事实写进参数，而是在需要时检索相关证据。

### 4.1 RAG 作为 memory 的价值

RAG 强在：

- 可解释：能给出来源文档。
- 可更新：改数据库比改模型权重容易。
- 可删除：合规和隐私更可控。
- 可扩展：适合企业知识库、文档问答、客服、代码库检索。
- 可组合：能与 rerank、query rewrite、metadata filter、graph traversal 结合。

RAG 弱在：

- 检索到了不代表模型会用对。
- 相似度检索容易混淆时间、角色、版本和否定信息。
- 长期个人记忆需要更新和冲突合并，不能只靠 chunk retrieval。
- 多轮 agent 行为中的“经验”常常不是一段可检索文档，而是策略、失败模式、工作流和偏好。

### 4.2 从 Vector RAG 到 Graph Memory

RAG 记忆层从 flat vector store 逐步发展为混合结构：

- **Vector + BM25 hybrid search**：解决 dense embedding 漏召和关键词精确匹配问题。
- **Rerank**：用 cross-encoder 或 LLM reranker 提高上下文质量。
- **Metadata/time-aware retrieval**：用时间、用户、项目、权限过滤。
- **GraphRAG / LightRAG / HippoRAG**：引入实体、关系、路径和图遍历。
- **Zep/Graphiti**：Zep 官方将 Graphiti 定位为 open-source temporal Context Graph framework，支持 temporal edges、hybrid retrieval 和动态失效旧事实；Zep 页面还明确把 LoCoMo 和 LongMemEval 作为 long-running agent memory benchmark 进行展示。

图谱记忆的关键贡献是：把“事实”从孤立文本片段提升为有时间、有主体、有关系、有状态变化的结构。这对用户画像、业务实体、项目状态和多 agent 共享记忆尤其重要。

### 4.3 From RAG to Memory

[From RAG to Memory: Non-Parametric Continual Learning for Large Language Models](https://arxiv.org/abs/2502.14802) 一类工作把 RAG 看成非参数持续学习的基础：模型权重冻结，记忆系统不断接收新事实、整理索引、优化召回。这条路线的核心不是“检索更多”，而是让外部记忆具备持续学习能力。

---

## 5. Agent Memory：从日志检索到主动记忆管理

Agent memory 的关键变化是：记忆不再只是 QA 前检索文档，而是进入 agent 的 perception-action loop。

### 5.1 代表系统

| 系统 | 时间 | 核心机制 | 意义 |
|---|---:|---|---|
| [Generative Agents](https://arxiv.org/abs/2304.03442) | 2023 | memory stream + reflection + planning | 把观察、反思、计划结合，展示“人格连续性” |
| [MemoryBank](https://arxiv.org/abs/2305.10250) | 2023 | 长期对话记忆、用户画像、遗忘曲线 | 明确面向长期聊天和个性化 |
| [LongMem](https://arxiv.org/abs/2306.07174) | 2023 | memory bank + retrieval module | 把 long history 作为可检索 memory |
| [MemGPT](https://arxiv.org/abs/2310.08560) / [Letta](https://docs.letta.com/guides/agents/architectures/memgpt) | 2023-2024 | OS-style virtual context、self-editing memory | 提出 LLM as OS 的虚拟上下文管理 |
| [A-Mem](https://arxiv.org/abs/2502.12110) | 2025 | 动态组织 memory nodes 和 links | 从静态存储转向 agentic 组织 |
| [MIRIX](https://arxiv.org/abs/2507.07957) | 2025 | modular multi-agent memory | 面向多 agent 的 memory system |
| [LightMem](https://arxiv.org/abs/2510.18866) | 2025/2026 | sensory/short-term/long-term 三阶段，sleep-time update | 强调效率和离线巩固 |
| [MemRL](https://arxiv.org/abs/2601.03192) | 2026 | episodic memory 上的 runtime reinforcement learning | 让经验影响策略选择 |
| [MemEvolve](https://arxiv.org/abs/2512.18746) | 2025/2026 | 共同演化经验和记忆架构 | 从 agent 自进化走向 memory architecture 自进化 |

### 5.2 Agent memory 的生命周期

成熟的 agent memory 至少包括：

1. **写入过滤**：什么值得记？事实、偏好、决策、失败、工具结果、用户反馈、环境状态，不能全量写。
2. **结构化编码**：raw log、atomic fact、episode、entity、scene、profile、procedure、skill。
3. **管理与合并**：去重、压缩、版本、过期、冲突、置信度、来源。
4. **检索与重构**：按任务目标构造必要上下文，而非简单 top-k。
5. **反馈更新**：根据成功/失败更新记忆权重或策略效用。
6. **遗忘与撤销**：删除、隐私隔离、错误记忆回滚。

从这个角度看，大量“记忆项目”其实只实现了存储和检索，尚未实现完整生命周期。

---

## 6. Memory OS 与系统化记忆层

2025 之后，一个明显趋势是把记忆提升为“系统资源”或“平台层”。这和传统 OS 管理内存、磁盘、缓存、权限、生命周期有相似性。

### 6.1 MemGPT/Letta：虚拟上下文管理

MemGPT 的贡献在于把有限 context window 类比为主存，把外部 archival memory 类比为磁盘，让 LLM 通过工具调用进行 self-editing 和 paging。Letta 继承这一思想，发展为 stateful-agent platform。

优势：抽象清晰，适合多会话 agent。  
风险：LLM 主动管理 memory 的可靠性取决于工具设计、提示约束和模型能力，成本与延迟可能偏高。

### 6.2 MemOS：把 plaintext、activation、parametric memory 统一成资源

MemOS 有两个重要版本：

- [MemOS: An Operating System for Memory-Augmented Generation](https://arxiv.org/abs/2505.22101)
- [MemOS: A Memory OS for AI System](https://arxiv.org/abs/2507.03724)

MemOS 的关键贡献是把记忆分为三类：

- **Plaintext memory**：外部文本、文档、结构化知识、RAG store。
- **Activation memory**：上下文状态、KV/activation、运行时中间状态。
- **Parametric memory**：模型权重、LoRA、adapter、被内化的知识。

其核心抽象 MemCube 试图封装内容、元数据、来源、版本、迁移和融合能力。重要性在于：它不是单纯 memory layer，而是试图建立跨存储形态的统一治理接口。

### 6.3 EverMind/EverOS/EverMemOS：self-evolving agent memory 生态

EverMind 需要按生态处理：

- [EverOS](https://github.com/EverMind-AI/EverOS)：开源框架，组织 use cases、methods、benchmarks；README 中列出 EverCore、HyperMem、EverMemBench、EvoAgentBench 等组件。
- [EverMemOS](https://arxiv.org/abs/2601.02163)：自组织 Memory OS，提出 MemCell、MemScene、profile、Foresight 等生命周期机制。
- EverCore：本地长期记忆系统。
- HyperMem：hypergraph-based hierarchical memory。
- EverMemBench：三层记忆质量评估，覆盖 factual recall、applied reasoning、personalized generalization。
- MSA / Memory Sparse Attention：偏 latent-memory 与长上下文模型结构方向。

EverMind 的定位更接近“长期记忆生态 + benchmark + agent/coding 集成”，而不是单一算法。

### 6.4 Mem0、Zep/Graphiti、Memori、TencentDB Agent Memory

- [Mem0](https://arxiv.org/abs/2504.19413)：生产级 memory layer，动态抽取、巩固、检索显著信息，并提供 graph memory 版本；论文用 LoCoMo 与多类 baseline 比较。
- [Zep/Graphiti](https://www.getzep.com/platform/graphiti/)：temporal context graph，强调动态事实、时间边、hybrid retrieval、本地开源 Graphiti 与商业 Zep Context Lake 的边界。
- [Memori](https://github.com/memorilabs/memori)：agent-native memory infrastructure，自报 LoCoMo 成绩和低 token 注入；应作为工程生态重点，但 benchmark 数字按自报处理。
- [TencentDB Agent Memory](https://github.com/TencentCloud/TencentDB-Agent-Memory)：本地优先，强调 symbolic short-term memory 与 layered long-term memory，包含 L0 conversation、L1 atom、L2 scenario、L3 persona 以及 Mermaid canvas/context offload。
- [MemoryOS](https://aclanthology.org/2025.emnlp-main.1318.pdf)：作为 Memory OS 路线的论文/代码代表之一，强调 agent 长期记忆管理。
- [TierMem](https://arxiv.org/abs/2602.17913)：provenance-aware tiered memory，把检索看成 inference-time evidence allocation，强调可验证证据链。
- [SuperLocalMemory](https://arxiv.org/abs/2603.14588)：local-first / zero-LLM 记忆方向，强调信息几何、生命周期、矛盾检测和数据主权；需注意其 benchmark 多为项目自报或论文自报。

### 6.5 这类系统真正解决的问题

Memory OS / memory layer 的意义不是“存更多”，而是：

- 统一多种 memory substrate。
- 给每条记忆来源、版本、权限、生命周期。
- 允许跨 agent、跨模型、跨平台迁移。
- 支持可观测和调试。
- 控制写入质量，防止 hallucination 被持久化。
- 把 memory 从 prompt engineering 上移到基础设施层。

---

## 7. LoRA、参数化记忆与模型编辑

参数化记忆是把知识或行为写进模型权重或可插拔参数。

### 7.1 为什么有吸引力

优势：

- 推理时低延迟，不需要外部检索。
- 行为更稳定，适合风格、格式、领域技能。
- 可用 LoRA/adapter 做租户、任务、个人化插件。
- 对反复使用的知识或 procedure，长期成本可能低于每次检索。

参数化记忆的直觉是：如果某些知识、风格或技能会长期高频使用，与其每次都检索并塞进上下文，不如把它变成模型行为的一部分。但这同时牺牲了外部记忆最重要的三个优点：可审计、可删除、可局部纠错。

### 7.2 主要路线

- **Fine-tuning / SFT**：把领域数据写进模型。
- **LoRA / PEFT / Adapter**：低成本可插拔参数更新。
- **Model editing**：ROME、MEMIT、MEND 等针对事实修改。
- **Continual learning**：持续注入新数据，但面临灾难性遗忘。
- **Sleep consolidation**：社区和研究原型把记忆先外部存储，再在“睡眠期”训练 LoRA 或执行权重编辑。

可以进一步拆成五类：

| 路线 | 写入粒度 | 典型目标 | 记忆含义 | 主要风险 |
|---|---|---|---|---|
| SFT / continued pretraining | 数据集级 | 领域适配、风格、能力 | 统计行为迁移 | 成本高、难回滚、遗忘 |
| LoRA / adapter | 插件级 | 租户、任务、风格、领域技能 | 可插拔行为记忆 | adapter 冲突、组合不可预测 |
| Model editing | 事实级或小批量事实 | 修改具体知识 | 局部参数记忆 | 泛化、邻近事实破坏、可验证性 |
| Continual learning | 时间序列任务 | 持续适应新知识 | 长期参数更新 | catastrophic forgetting |
| Sleep consolidation | 离线巩固 | 把稳定外部记忆转为参数 | 外部到内部迁移 | 错误记忆被固化 |

### 7.3 代表研究与近期信号

- [Knowledge Editing for Large Language Models: A Survey](https://arxiv.org/abs/2310.16218) 系统整理 ROME、MEMIT、MEND、SERAC、GRACE 等模型编辑路线。它们证明参数局部修改可行，但也暴露 locality、portability、generalization 和 multi-edit interference 问题。
- [Continual Learning in Large Language Models](https://arxiv.org/abs/2603.12658) 将 LLM continual learning 分为 continual pre-training、continual fine-tuning、continual alignment，并强调遗忘缓解机制。
- [Continual Learning via Sparse Memory Finetuning](https://arxiv.org/abs/2510.15103) 直接把 sparse memory layers 作为持续学习路径，与 full fine-tuning 和 LoRA 对比，说明“只更新与输入相关的记忆参数”可能比普通 LoRA 更接近长期记忆。
- [STABLE](https://arxiv.org/abs/2510.16089) 使用 gated continual self-editing 和 LoRA 约束顺序更新中的遗忘。
- [How LoRA Remembers?](https://huggingface.co/papers/2605.30260) 将 LoRA 作为探针研究 parametric memory law，说明 LoRA 记忆能力可以量化，但这仍主要是“可控记忆写入”的研究信号，不等于产品级可删记忆。

### 7.4 适用边界

适合写进参数的内容：

- 稳定风格和格式偏好。
- 高频技能和 procedure。
- 领域术语、模板、分类规则。
- 已验证且长期稳定的事实。

不适合写进参数的内容：

- 高频变化事实。
- 法务、医疗、金融等需要来源可追溯的事实。
- 用户隐私和可删除信息。
- 尚未验证的 agent 推断。

核心判断：LoRA 是“内化行为/技能”的好工具，但不是个人记忆数据库的替代品。

### 7.5 与 RAG / Memory OS 的关系

参数化记忆不应和 RAG 对立，而应处在 memory lifecycle 的后端：

1. 新事实先进入事件日志或 plaintext memory。
2. 经过去重、验证、冲突处理后进入结构化 memory。
3. 只有稳定、高频、低隐私风险、低变更频率的内容，才考虑进入 LoRA/adapter 或模型编辑。
4. 参数化更新必须保留外部 provenance：知道它来自哪些事实、何时写入、如何回滚。

这也是 MemOS 这类框架把 parametric、activation、plaintext memory 放在同一体系里的原因：参数记忆应是记忆系统的一层，而不是绕过治理的黑盒。

---

## 8. Tensor、Latent 与 Neural Memory

这条路线试图把记忆从可读文本压缩成模型可直接消费的隐空间表示。

### 8.1 Context compression 到 latent memory

代表方法：

- [Gist Tokens](https://arxiv.org/abs/2304.08467)：把 prompt 压缩为少量可缓存 token。
- [ICAE](https://arxiv.org/abs/2307.06945)：把长上下文编码成 memory slots。
- [LongMem](https://arxiv.org/abs/2306.07174)：用 memory bank 让模型利用长历史。
- [MemoryLLM](https://arxiv.org/abs/2402.04624)：面向自更新模型，将文本知识转为可记忆结构。
- [MemLLM](https://arxiv.org/abs/2404.11672)：训练 LLM 使用显式 read-write memory。
- M+、MemoryLLM 新变体：继续探索自更新、可解释 FFN memory、插拔式 memory。

这类方法的共同目标是降低 token 和检索成本，让长期信息以更短、更模型友好的形式进入推理。

可以把 latent/tensor memory 分成四类：

| 类别 | 代表 | 本质 | 是否可读 | 成熟度 |
|---|---|---|---|---|
| Soft prompt / memory token | Gist Tokens、ICAE、AutoCompressors | 把长文本压成少量向量/token | 低 | 研究原型到局部可用 |
| Activation / KV memory | KV cache、PagedAttention、context caching | 运行时状态和推理缓存 | 低 | 工程成熟，但不是长期记忆 |
| FFN / parameter memory | MemoryLLM 2026、model editing | 把 FFN 或权重视为可访问 memory | 很低到中等 | 研究前沿 |
| Test-time neural memory | Titans、MIRAS、MSA | 推理时更新或检索 neural memory | 低 | 高潜力，工程早期 |

### 8.2 Soft compression：从自然语言摘要到隐空间摘要

纯文本摘要是 hard compression：压缩结果仍是自然语言。Gist Tokens、ICAE、AutoCompressors 属于 soft compression：压缩结果是模型内部可消费的 token/向量，不一定能被人读懂。

优势：

- 压缩率更高。
- 可缓存并重复使用。
- 更接近模型内部表示，可能减少“摘要风格”带来的语义偏差。

风险：

- 难审计，难人工修复。
- 跨模型迁移困难。
- 一旦压错，用户很难定位是哪条事实丢失。
- 对隐私删除和合规解释不友好。

### 8.3 KV cache 与推理显存管理不是同一类“记忆”

vLLM/PagedAttention、KV cache compression、context caching 常被误称为 memory。它们确实管理“内存”，但主要是推理系统的显存和吞吐优化，不等同于 agent 的长期认知记忆。

区别：

- KV cache 记的是当前/近前推理状态。
- Agent memory 记的是跨会话事实、经验和偏好。
- Neural memory 试图让模型在 test time 写入长期可用状态。

### 8.4 FFN memory：把 Transformer 内部模块看成记忆

[MemoryLLM: Plug-n-Play Interpretable Feed-Forward Memory for Transformers](https://arxiv.org/abs/2602.00398) 代表了另一种内化方向：把 FFN 从 self-attention 中解耦，研究 token 如何访问 FFN 参数中的 memory locations。它进一步提出 token-wise lookup，使 FFN memory 可预计算并在存储和显存之间按需迁移。

这一方向的重要性在于：

- 它把“模型参数里存了知识”从隐喻推进到更可分析的结构。
- 它为权重/FFN offloading、模块化部署和可解释参数记忆提供了入口。
- 它也说明参数记忆不一定只能通过 LoRA 或 editing 讨论，Transformer 结构本身可能就存在可重构的 memory substrate。

但它距离产品记忆仍有明显距离：用户级事实删除、跨任务冲突处理、权限隔离、来源追踪都不是 FFN memory 本身能解决的。

### 8.5 Neural long-term memory 与 test-time memory

代表方向：

- [Titans](https://research.google/pubs/titans-learning-to-memorize-at-test-time/)：Google 提出的 test-time neural memory，把 attention 视为短期记忆，把 neural memory 作为长期记忆模块。
- [MIRAS / Titans blog](https://research.google/blog/titans-miras-helping-ai-have-long-term-memory/)：将 neural long-term memory 用于长程信息保留。
- [Memory Sparse Attention / MSA](https://arxiv.org/abs/2603.23516)：在模型结构中引入 latent memory、sparse routing、KV compression 和 memory-parallel inference，目标是把检索和生成放进统一可训练 latent state。

趋势判断：这是研究前沿，但目前产品级可控性不足。对于企业和个人助手，外部可审计记忆仍更现实；对于超长上下文、视频/多模态、持续学习模型，neural memory 的潜力更大。

### 8.6 Tensor 内化的边界判断

“Tensor 内化”适合解决：

- 超长上下文的计算和带宽瓶颈。
- 重复上下文的高压缩表示。
- 模型结构层面的长期依赖。
- 多模态或连续信号的长期状态。

它不适合单独解决：

- 用户要求“忘记我说过的 X”。
- 企业要求审计某条答案引用了哪个文件。
- 多租户权限隔离。
- 错误记忆的人工修复。

因此，tensor/neural memory 更可能成为底层加速和能力增强模块，而不是替代外部 memory governance。

---

## 9. 产品级记忆：ChatGPT、Gemini、Claude

产品级记忆关注的不是论文 benchmark，而是用户控制、隐私、体验连续性和企业边界。

### 9.1 ChatGPT Memory

OpenAI 于 2024-02-13 发布 ChatGPT memory 测试，后续更新包括：

- 2024-09-05：Memory 面向 Free、Plus、Team、Enterprise 更广泛可用。
- 2025-04-10：Memory 更全面，分为 saved memories 和 chat history reference。
- 2025-06-03：免费用户开始获得轻量 memory improvements。

OpenAI 官方说明中强调用户可以显式要求记住、询问记住了什么、要求忘记，也可以在设置中管理；同时指出删除聊天不自动删除 saved memory，需要单独删除。

产品意义：ChatGPT Memory 把“跨会话个性化”产品化，但它不是完整 agent memory OS。它更像用户画像、偏好、近期会话 insight 和安全/隐私控制的组合。

### 9.2 Gemini recall past chats

Google 于 2025-02-13 宣布 Gemini app 可 recall past chats。官方说明：Gemini 可以使用相关历史聊天来回答问题或总结过去对话，用户可以查看、删除或决定聊天历史保留时间，也可以关闭 Gemini Apps Activity。

产品意义：Gemini 选择把“历史聊天可回忆”作为连续工作流能力，而不是强调显式 saved memory。

### 9.3 Claude memory

Anthropic 于 2025-09-11 发布 Claude memory，重点面向 Team 和 Enterprise 工作场景。Claude memory 强调专业上下文、团队项目和偏好；并提供 Incognito chats、组织管理员控制、导入/导出等工作流。

产品意义：Claude 把 memory 明确绑定到 work context 和企业治理，和 ChatGPT 的个人助手记忆侧重点不同。

### 9.4 产品级记忆的共同限制

- 用户通常看不到完整写入和检索过程。
- 记忆来源和使用解释仍不完整。
- 对敏感信息、错误记忆、过期事实的处理仍是核心风险。
- 产品 memory 常服务于 personalization，不一定服务于复杂 agent 的长期任务执行。

---

## 10. Benchmark 与评估陷阱

记忆系统评估正在快速变化。早期 benchmark 多测 recall，现在逐步扩展到 temporal reasoning、conflict resolution、test-time learning、selective forgetting 和 long-horizon task success。

### 10.1 主要 benchmark

| Benchmark | 时间 | 测什么 | 局限 |
|---|---:|---|---|
| [LoCoMo](https://arxiv.org/abs/2402.17753) | 2024 | 多 session 长对话 QA、事件总结、多模态对话 | 数据规模和 judge 有争议；容易被 prompt/context 策略影响 |
| [LongMemEval](https://arxiv.org/abs/2410.10813) | 2024/2025 | 500 问题，信息抽取、多 session、时间推理、知识更新、拒答 | 部分版本可被现代长上下文直接覆盖，可能测到 context management 而非长期记忆 |
| [BEAM](https://arxiv.org/abs/2510.27246) | 2025 | up to 10M tokens，100 conversations，2000 validated questions，10 类 memory abilities | 更接近超长记忆，但生成式数据和成本较高 |
| [MemBench](https://aclanthology.org/2025.findings-acl.989.pdf) | 2025 | factual + reflective、多场景、多指标、10k/100k token | 仍需看任务是否贴近真实 agent |
| [MemoryAgentBench](https://github.com/HUST-AI-HYZ/MemoryAgentBench) | 2026 | Accurate Retrieval、Test-Time Learning、Long-Range Understanding、Conflict Resolution | 新 benchmark，生态复现仍在早期 |
| [AgentMemoryBench](https://github.com/s010m00n/AgentMemoryBench) | 2026 | system memory + personal memory，online/offline/replay/transfer/repair | 更复杂，横向比较成本高 |
| [EverMemBench](https://github.com/EverMind-AI/EverOS) | 2026 | factual recall、applied reasoning、personalized generalization | 与 EverMind 生态绑定较深，需关注第三方复现 |
| [LoCoMo-Plus](https://arxiv.org/abs/2602.10715) | 2026 | beyond-factual cognitive memory，隐式约束保持 | 新方向，方法和数据仍需时间验证 |
| [LongMemEval-V2](https://arxiv.org/abs/2605.12493) | 2026 | web agents 的环境经验、工作流、gotchas | 更贴近 agent 工作，但仍是新发布 |

### 10.2 应该测哪些能力

一个 memory benchmark 至少应区分：

- **Recall**：能不能找回事实。
- **Temporal reasoning**：能不能理解时间变化。
- **Multi-hop reasoning**：能不能跨多条记忆组合。
- **Conflict resolution**：新旧事实冲突时如何处理。
- **Selective forgetting**：能不能删除和忽略过期/错误信息。
- **Test-time learning**：能不能从交互反馈中改进。
- **Procedural memory**：能不能记住成功/失败工作流。
- **Long-horizon task success**：记忆是否真的提高任务完成率。
- **Efficiency**：token、延迟、API call、存储和维护成本。
- **Trustworthiness**：溯源、权限、隐私、记忆投毒防御。

### 10.3 Benchmark 陷阱

1. **LLM-as-Judge 偏差**：judge 可能接受错误答案，也可能对不同系统上下文不公平。
2. **答案键错误**：长期对话数据容易出现事实标注错误。
3. **只测检索，不测生命周期**：高 Recall@5 不代表系统能写入、合并、忘记、纠错。
4. **上下文窗口作弊**：如果 corpus 能塞进现代长上下文，benchmark 可能退化为“谁能塞更多”。
5. **自报成绩不可直接等同 SOTA**：README、营销页、博客数字必须和论文、代码、复现实验分开。
6. **任务不等价**：聊天记忆、代码 agent、web agent、游戏 agent、企业知识库的记忆需求不同。

---

## 11. 时间线：代表论文、产品与项目

| 年份 | 关键节点 | 代表 |
|---:|---|---|
| 2020 前 | 可微外部记忆、cache LM、长上下文早期结构 | Memory Networks、NTM、DNC、kNN-LM、Transformer-XL |
| 2020-2021 | 检索增强预训练与生成 | REALM、RAG、RETRO |
| 2022 | ChatGPT 出现，无状态对话产品化 | 当前上下文窗口、聊天历史 |
| 2023 H1 | Agent memory 兴起 | Generative Agents、MemoryBank、Gist Tokens |
| 2023 H2 | 长期记忆和 OS 类比 | LongMem、MemGPT、LLMLingua、LongLLMLingua |
| 2024 | 产品 memory 与 long-term benchmark | ChatGPT Memory、LoCoMo、LongMemEval、ICAE、MemoryLLM、MemLLM |
| 2025 H1 | 生产级 memory layer | Mem0、A-Mem、From RAG to Memory、MemOS MAG |
| 2025 H2 | Memory OS、图谱、轻量 MAG、自进化 | MemOS AI System、Zep/Graphiti、LightMem、MIRIX、MemEvolve、BEAM |
| 2026 H1 | 评估深化、self-evolving、local-first、provenance | EverMemOS、MemRL、TierMem、TAME、MemoryAgentBench、AgentMemoryBench、TencentDB Agent Memory、LongMemEval-V2 |

---

## 12. 项目索引与分层

### 12.1 一级重点

| 项目/论文 | 类型 | 核心价值 |
|---|---|---|
| Generative Agents | agent architecture | memory + reflection + planning |
| MemoryBank | long-term dialogue memory | 用户画像、遗忘曲线 |
| MemGPT/Letta | OS-style agent framework | 虚拟上下文、自管理 memory |
| Mem0 | production memory layer | 抽取、巩固、检索、graph memory |
| Zep/Graphiti | temporal graph memory | 时间图谱、混合检索、动态事实 |
| MemOS | memory OS | plaintext/activation/parametric 统一抽象 |
| EverMind/EverOS/EverMemOS | memory OS ecosystem | self-organizing memory、benchmark、agent 集成 |
| LightMem | efficient MAG | 三阶段记忆和 sleep-time update |
| MemRL | self-evolving agent memory | episodic memory + runtime RL |
| MemEvolve | meta-evolving memory systems | 记忆架构也参与演化 |
| TierMem | provenance-aware memory | tiered evidence allocation 与溯源 |
| TencentDB Agent Memory | local-first engineering memory | symbolic short-term + layered long-term |

### 12.2 二级生态

| 项目 | 观察点 |
|---|---|
| Memori | agent-native memory infra，自报 LoCoMo 成绩，适合作为工程案例 |
| SuperLocalMemory | local-first / zero-LLM 记忆，强调数据主权和数学基础 |
| Cognee | graph/RAG memory 生态项目 |
| Basic Memory | 本地 Markdown 知识/记忆，适合个人和 coding agent |
| OpenMemory MCP / Supermemory | MCP 与跨工具个人记忆生态 |
| LangMem | LangGraph/LangChain 生态 memory |
| Hindsight / Engram / Memento / MemPalace 等 | 社区活跃，需重点看复现和 benchmark 方法 |

### 12.3 三级观察

Reddit、社区发布和项目营销页中不断出现新 memory layer。它们的价值在于暴露真实需求：本地优先、跨工具共享、coding agent 长期状态、可视化调试、memory poisoning 防御。但除非有论文、代码和可复现 benchmark，否则不应作为核心证据。

---

## 13. 综合矩阵：五条路线的取舍

| 路线 | 写入方式 | 读取方式 | 优点 | 风险 | 适用场景 |
|---|---|---|---|---|---|
| 文本压缩 | 摘要、规则、LLM 压缩 | 直接注入 prompt | 简单、可读、低门槛 | 摘要漂移、污染 prompt | coding agent、项目上下文、短中期任务 |
| RAG/图谱 | 文档/事实入库 | 检索、rerank、graph traversal | 可审计、可删、可扩展 | 命中不等于用对，冲突难 | 企业知识库、用户事实、业务实体 |
| Agent memory layer | 抽取、巩固、反思、profile | task-aware retrieval | 跨会话行为连续 | 写入质量、延迟、隐私 | 助手、客服、coding/web agent |
| 参数内化 | Fine-tune、LoRA、model editing | 模型权重直接影响输出 | 低延迟、行为稳定 | 难解释、难删、灾难性遗忘 | 稳定技能、风格、领域模板 |
| Neural/tensor memory | memory token、activation、latent slots | 模型内部 attention/reader | 高压缩、潜在端到端优化 | 不透明、工程早期 | 超长上下文、多模态、研究前沿 |

---

## 14. 2026 年判断：可落地架构

一个务实的 2026 LLM 记忆架构应包含：

1. **短期工作记忆**：当前任务窗口 + 可控 compact，不把无关日志全塞回 prompt。
2. **事件日志**：append-only，保存原始对话、工具调用、关键决策和失败，用于审计与回放。
3. **事实层**：抽取 atomic facts，带来源、时间、置信度、主体、权限。
4. **实体/图谱层**：维护人、项目、任务、文件、组织、关系和时间变化。
5. **画像/偏好层**：稳定个人偏好和工作方式，写入需更谨慎。
6. **程序/技能层**：成功工作流、失败模式、工具使用经验。
7. **检索编排层**：按任务目标选择 raw log、fact、entity、profile、procedure 的组合，而不是统一 top-k。
8. **生命周期治理**：去重、冲突、过期、删除、来源追踪、敏感信息策略。
9. **评估与观测**：LoCoMo/LongMemEval 只是起点，还要做真实任务回放、A/B、延迟/token 成本和用户纠错率。
10. **可选参数化**：把稳定技能或风格通过 LoRA/adapter 内化，但不要把敏感、动态、需删除事实写进权重。

### 14.1 推荐的写入策略

按“先外部、后内化”的原则处理新信息：

| 信息类型 | 首选位置 | 是否进入参数 | 原因 |
|---|---|---:|---|
| 原始对话、工具日志、文件状态 | append-only event log | 否 | 审计与回放需要原文 |
| 用户明确偏好 | profile / plaintext memory | 通常否 | 需要可见、可删、可改 |
| 项目事实、业务实体、关系 | graph / structured store | 否 | 需要时间、权限、来源 |
| agent 失败经验 | episodic/procedural memory | 可选 | 先验证是否可泛化 |
| 稳定输出风格 | adapter / LoRA | 是 | 低隐私、高频、稳定 |
| 领域术语和格式 | LoRA / SFT | 可选 | 适合低频更新领域 |
| 高频动态事实 | RAG / graph | 否 | 参数化会过期且难删 |
| 高敏感个人信息 | encrypted / scoped memory | 否 | 合规和用户控制优先 |

### 14.2 推荐的读取策略

不要把所有记忆统一 top-k 注入。更好的读取方式是按任务组装：

1. 先读系统约束、用户偏好和当前任务目标。
2. 再查实体/图谱，确定人、项目、时间和状态。
3. 对需要证据的问题，查原文或文档 RAG。
4. 对执行类任务，查 procedure、失败案例和工具经验。
5. 对长任务，只注入 compacted state，同时保留 raw evidence pointer。
6. 对不确定或冲突记忆，要求模型显式暴露不确定性，而不是自动合并成单一事实。

### 14.3 推荐的评估策略

实际落地不要只看 LoCoMo 或 LongMemEval 排名。建议同时评估：

- **记忆写入准确率**：该记的是否写入，不该记的是否过滤。
- **回忆准确率**：问到历史事实时是否找对。
- **冲突处理**：用户纠正偏好或事实后是否更新。
- **遗忘执行**：删除请求后是否真正不再使用。
- **任务收益**：记忆是否提高真实任务完成率，而不是只提高 QA。
- **成本曲线**：随着 session、用户、项目增加，token、延迟、存储、API call 如何增长。
- **可观测性**：能否解释“为什么这条记忆被写入/检索/注入”。
- **安全性**：是否能防 memory poisoning、跨用户泄漏、prompt injection 持久化。

---

## 15. 最终结论

LLM 记忆系统的演化可以概括为：

- **2022-2023：上下文即记忆**。通过塞历史、摘要、prompt 压缩维持连续性。
- **2023-2024：检索即记忆**。RAG 和向量库成为默认外部记忆，但逐渐暴露“找得到不等于记得住”。
- **2024-2025：agent 开始管理记忆**。reflection、self-editing、memory stream、graph memory、profile 和 benchmark 进入主流。
- **2025-2026：记忆成为系统层**。MemOS、EverMind、Mem0、Zep、TencentDB Agent Memory 等推动 memory layer / memory OS 化。
- **同时进行：参数和 neural memory 内化**。LoRA、MemoryLLM、ICAE、Titans 等试图把记忆压缩进权重或隐状态，但可控性尚未成熟。

最重要的判断是：**未来不会是 RAG、LoRA、Memory OS、Neural Memory 其中之一胜出，而是混合系统胜出**。RAG 负责可审计事实，文本压缩负责上下文成本，agent memory 负责行为连续，参数化负责稳定技能，neural memory 负责未来的高压缩和长程推理。真正的竞争点会转向 memory lifecycle：写入过滤、冲突解决、遗忘、溯源、安全、低延迟和长期稳定。

---

## 参考来源

### 综述与列表

- [Memory for Autonomous LLM Agents](https://arxiv.org/abs/2603.07670)
- [From Storage to Experience](https://arxiv.org/abs/2605.06716)
- [From Human Memory to AI Memory](https://arxiv.org/abs/2504.15965)
- [AI Meets Brain](https://arxiv.org/abs/2512.23343)
- [Human-inspired Perspectives](https://arxiv.org/abs/2411.00489)
- [LLM Agent Memory Survey](https://openreview.net/forum?id=KPs1EgGKcT)
- [Awesome-Agent-Memory](https://github.com/TeleAI-UAGI/Awesome-Agent-Memory)
- [awesome-long-term-memory](https://github.com/xiaowu0162/awesome-long-term-memory)
- [awesome-ai-memory](https://github.com/topoteretes/awesome-ai-memory)
- [awesome-rag](https://github.com/coree/awesome-rag)

### 核心论文与项目

- [Generative Agents](https://arxiv.org/abs/2304.03442)
- [MemoryBank](https://arxiv.org/abs/2305.10250)
- [LongMem](https://arxiv.org/abs/2306.07174)
- [MemGPT](https://arxiv.org/abs/2310.08560)
- [Letta MemGPT architecture](https://docs.letta.com/guides/agents/architectures/memgpt)
- [LLMLingua](https://arxiv.org/abs/2310.05736)
- [LongLLMLingua](https://arxiv.org/abs/2310.06839)
- [Prompt Compression Survey](https://aclanthology.org/2025.naacl-long.368.pdf)
- [Empirical Study on Prompt Compression](https://arxiv.org/abs/2505.00019)
- [ACON](https://arxiv.org/abs/2510.00615)
- [Active Context Compression](https://arxiv.org/abs/2601.07190)
- [Memento](https://www.microsoft.com/en-us/research/articles/memento-teaching-llms-to-manage-their-own-context/)
- [Lossless Prompt Compression via Dictionary-Encoding](https://arxiv.org/abs/2604.13066)
- [Gist Tokens](https://arxiv.org/abs/2304.08467)
- [ICAE](https://arxiv.org/abs/2307.06945)
- [MemoryLLM](https://arxiv.org/abs/2402.04624)
- [MemLLM](https://arxiv.org/abs/2404.11672)
- [Knowledge Editing Survey](https://arxiv.org/abs/2310.16218)
- [Continual Learning in LLMs](https://arxiv.org/abs/2603.12658)
- [Sparse Memory Finetuning](https://arxiv.org/abs/2510.15103)
- [STABLE](https://arxiv.org/abs/2510.16089)
- [How LoRA Remembers?](https://huggingface.co/papers/2605.30260)
- [A-Mem](https://arxiv.org/abs/2502.12110)
- [From RAG to Memory](https://arxiv.org/abs/2502.14802)
- [Mem0](https://arxiv.org/abs/2504.19413)
- [MemOS MAG](https://arxiv.org/abs/2505.22101)
- [MemOS AI System](https://arxiv.org/abs/2507.03724)
- [MIRIX](https://arxiv.org/abs/2507.07957)
- [LightMem](https://arxiv.org/abs/2510.18866)
- [MemEvolve](https://arxiv.org/abs/2512.18746)
- [EverMemOS](https://arxiv.org/abs/2601.02163)
- [MemRL](https://arxiv.org/abs/2601.03192)
- [TierMem](https://arxiv.org/abs/2602.17913)
- [TAME](https://arxiv.org/abs/2602.03224)
- [MemoryLLM FFN Memory](https://arxiv.org/abs/2602.00398)
- [MSA](https://arxiv.org/abs/2603.23516)
- [SuperLocalMemory](https://arxiv.org/abs/2603.14588)
- [Titans](https://research.google/pubs/titans-learning-to-memorize-at-test-time/)
- [Titans / MIRAS blog](https://research.google/blog/titans-miras-helping-ai-have-long-term-memory/)
- [MemOS GitHub](https://github.com/MemTensor/MemOS)
- [EverOS GitHub](https://github.com/EverMind-AI/EverOS)
- [Zep Graphiti](https://www.getzep.com/platform/graphiti/)
- [TencentDB Agent Memory](https://github.com/TencentCloud/TencentDB-Agent-Memory)
- [Memori](https://github.com/memorilabs/memori)

### 产品与 Benchmark

- [OpenAI ChatGPT Memory](https://openai.com/index/memory-and-new-controls-for-chatgpt/)
- [OpenAI Memory FAQ](https://help.openai.com/en/articles/8590148-memory-faq%3F.class)
- [Gemini recall past chats](https://blog.google/feed/gemini-referencing-past-chats/)
- [Claude memory](https://www.anthropic.com/news/memory?from_blog=true)
- [LoCoMo](https://arxiv.org/abs/2402.17753)
- [LoCoMo code](https://github.com/snap-research/locomo)
- [LongMemEval](https://arxiv.org/abs/2410.10813)
- [BEAM](https://arxiv.org/abs/2510.27246)
- [MemBench](https://aclanthology.org/2025.findings-acl.989.pdf)
- [MemoryAgentBench](https://github.com/HUST-AI-HYZ/MemoryAgentBench)
- [AgentMemoryBench](https://github.com/s010m00n/AgentMemoryBench)
- [mem0 memory-benchmarks](https://github.com/mem0ai/memory-benchmarks)
- [LoCoMo-Plus](https://arxiv.org/abs/2602.10715)
- [LongMemEval-V2](https://arxiv.org/abs/2605.12493)
