# CodeRAG 项目复盘与重审

日期：2026-03-26

## 一句话判断

这个项目不是“失败的半成品”，而是一个方向对、主链路已打通、但工程化明显停在 MVP 阶段的个人项目。它最有价值的部分是：

- 选题成立：`VS Code 扩展 + 本地索引后端 + 代码检索/解释`
- 主闭环成立：`打包工作区 -> 上传 -> chunk -> 入库 -> search/explain/agent`
- 架构有边界：前端扩展、API、索引、检索、LLM 适配已经分层

它最需要的不是“推倒重来”，而是一次有节制的整理：先收敛范围、再补工程骨架、最后再谈功能升级。

---

## 项目定位

从 [README.md](/Users/eridani/Work/code-rag-agent-new/code-rag/README.md) 和代码实现看，这个项目的目标是做一个面向 VS Code 的本地 Code RAG 工具：

- 扩展负责触发搜索、解释、Agent、构建索引
- 后端负责上传 zip、入队任务、检索、调用 LLM
- 索引侧用 Tree-sitter 做 AST-aware chunking
- 存储侧用 Chroma 做向量检索
- 任务侧用 Redis + RQ 做异步构建

这个方向本身是合理的，而且对个人项目来说，拆分也不算乱。

---

## 我这次实际检查了什么

我没有假装“完整跑通”整个系统，而是做了更适合当前阶段的项目体检：

- 阅读了核心说明、依赖、Docker 配置
- 串读了 API、索引、检索、Agent、VS Code 扩展主链路
- 做了 Python 语法级检查：`python3 -m compileall api ai indexer retriever eval`
- 检查了仓库结构、Git 状态、空文件、配置文件现状

当前可确认：

- Python 代码层面没有语法错误
- 仓库是独立 Git 仓库，当前只有 `.DS_Store` 脏改动
- VS Code 扩展依赖未安装，所以我没有宣称前端构建已验证

---

## 当前架构评价

### 1. 结构是清楚的

这个项目最好的地方，是代码虽然简略，但职责边界已经有了：

- [api/main.py](/Users/eridani/Work/code-rag-agent-new/code-rag/api/main.py) 负责 HTTP 入口
- [indexer/chunker.py](/Users/eridani/Work/code-rag-agent-new/code-rag/indexer/chunker.py) 负责代码切块
- [indexer/embed_ingest.py](/Users/eridani/Work/code-rag-agent-new/code-rag/indexer/embed_ingest.py) 负责写入 Chroma
- [retriever/hybrid_search.py](/Users/eridani/Work/code-rag-agent-new/code-rag/retriever/hybrid_search.py) 负责检索和重排
- [ai/llm.py](/Users/eridani/Work/code-rag-agent-new/code-rag/ai/llm.py) 负责模型适配
- [clients/vscode/src/extension.ts](/Users/eridani/Work/code-rag-agent-new/code-rag/clients/vscode/src/extension.ts) 负责 VS Code 入口

这说明你去年其实已经把“产品骨架”想明白了。

### 2. 主链路是通的

从实现上看，索引主流程是成立的：

1. 扩展把 workspace 打包上传
2. 后端保存 zip 并入队
3. worker 解压后切 chunk
4. chunk 写入 Chroma collection
5. `/search`、`/explain`、`/agent/explain` 再按 workspace 查询

这一点很重要，因为很多个人项目死在“只有 demo，没有真正通路”。这个项目不是。

### 3. 但工程成熟度明显停在 MVP

项目现在更像“做出来了第一版闭环”，而不是“可持续迭代的产品仓库”。

表现为：

- 文档有了，但和实现不完全一致
- 配置层存在，但很多没真正接入
- 测试层几乎空白
- 依赖、License、空文件这些基础卫生还没收口

---

## 这个项目现在最值得保留的部分

### AST-aware chunking 思路值得保留

[indexer/chunker.py](/Users/eridani/Work/code-rag-agent-new/code-rag/indexer/chunker.py) 不是简单按字符切片，而是围绕函数、类、方法、箭头函数做 chunk，这个方向是对的。它至少已经考虑了：

- 结构化代码单元
- 注释拼接
- 导出信息
- AST 路径
- 去重 ID

这部分完全可以作为后续重构的核心保留资产。

### workspace 隔离意识是对的

[api/main.py:77](/Users/eridani/Work/code-rag-agent-new/code-rag/api/main.py#L77) 的 collection 命名和 [clients/vscode/src/api.ts:97](/Users/eridani/Work/code-rag-agent-new/code-rag/clients/vscode/src/api.ts#L97) 的 `x-workspace-id` 设计，说明你当时已经意识到“多项目隔离”是代码检索工具的核心问题。这点很成熟。

### 扩展 + 本地后端的产品形态也值得保留

不是做一个网页 demo，而是直接嵌进 VS Code，这让它更接近真实使用场景。对个人项目来说，这是加分项。

---

## 主要问题与风险

下面这些不是吹毛求疵，而是我认为最值得优先处理的结构性问题。

### 1. 产品能力声明和真实能力有偏差

最明显的是语言支持。

扩展打包时允许很多文件类型进入 zip，例如 `.py`、`.java`、`.go`、`.rs`、`.c`、`.cpp`、`.md`、`.json` 等，见 [clients/vscode/src/indexer.ts:25](/Users/eridani/Work/code-rag-agent-new/code-rag/clients/vscode/src/indexer.ts#L25)。

但后端 chunker 实际只支持 JS/TS 相关扩展，见 [indexer/chunker.py:24](/Users/eridani/Work/code-rag-agent-new/code-rag/indexer/chunker.py#L24)。

这会导致两个问题：

- 用户感知上像是“支持很多语言”
- 实际索引阶段只会处理 JS/TS，别的文件大多只是被上传但不会进入 AST chunk

这类不一致比“功能还没做”更伤体验，因为它会制造误解。

### 2. Agent 路径没有真正接入统一的 LLM 配置体系

`/explain` 走的是按请求解析 provider/api key 的新逻辑，见 [api/main.py:305](/Users/eridani/Work/code-rag-agent-new/code-rag/api/main.py#L305) 和 [ai/llm.py](/Users/eridani/Work/code-rag-agent-new/code-rag/ai/llm.py)。

但 Agent 里仍然直接用环境变量创建 OpenAI client，见 [ai/agent.py:31](/Users/eridani/Work/code-rag-agent-new/code-rag/ai/agent.py#L31)。

后果是：

- 扩展里传的 `x-llm-provider`、`x-api-key` 对 Agent 不一定生效
- `ctx` 在 `/agent/explain` 里基本没被用上，见 [api/main.py:628](/Users/eridani/Work/code-rag-agent-new/code-rag/api/main.py#L628)
- Agent 实际上仍偏“只支持 OpenAI”

这和 README 里“Provider 可切换”的整体叙述不完全一致。

### 3. 搜索结果体验有个隐性缺口：`text_preview` 实际拿不到内容

`/search` 调用检索时传的是 `include_documents=False`，见 [api/main.py:183](/Users/eridani/Work/code-rag-agent-new/code-rag/api/main.py#L183)。

而 [retriever/hybrid_search.py:73](/Users/eridani/Work/code-rag-agent-new/code-rag/retriever/hybrid_search.py#L73) 在这种情况下会把 `docs` 置成 `None`，于是 [retriever/hybrid_search.py:89](/Users/eridani/Work/code-rag-agent-new/code-rag/retriever/hybrid_search.py#L89) 生成的 `text_preview` 也基本为空。

最终 [clients/vscode/src/searchView.ts:39](/Users/eridani/Work/code-rag-agent-new/code-rag/clients/vscode/src/searchView.ts#L39) 的 tooltip 价值会很有限。

这不是大 bug，但说明有些功能链路“看起来在，实际没喂够数据”。

### 4. 依赖面过大，明显超过当前项目真实需求

从代码导入看，实际核心依赖集中在：

- `fastapi`
- `pydantic`
- `redis`
- `rq`
- `chromadb`
- `tree-sitter`
- `tree-sitter-languages`
- `openai`
- `python-dotenv`
- `requests`

但 [requirements.txt](/Users/eridani/Work/code-rag-agent-new/code-rag/requirements.txt) 里有大量当前代码并未直接使用的包，而且体量很大。这通常意味着它更像一次环境冻结，而不是项目级依赖声明。

后果很现实：

- Docker build 更慢
- 依赖冲突概率更高
- 未来恢复这个项目的成本更高

好消息是你已经有了一个更接近真实需求的 [requirements-mvp.txt](/Users/eridani/Work/code-rag-agent-new/code-rag/requirements-mvp.txt)，这说明收敛依赖并不难。

### 5. 配置层存在感不强，很多地方仍然硬编码

仓库里虽然有配置目录，但目前：

- [configs/embedding.yaml](/Users/eridani/Work/code-rag-agent-new/code-rag/configs/embedding.yaml) 是空文件
- [configs/retrieval.yaml](/Users/eridani/Work/code-rag-agent-new/code-rag/configs/retrieval.yaml) 是空文件

同时大量参数仍散落在代码和环境变量里，例如：

- 检索分数逻辑
- chunk 限制
- zip 限制
- 默认 top_k
- 模型名

这会让后续调参和多环境管理比较痛苦。

### 6. 仓库完成度信号偏弱

目前有几个非常直观的“未收尾”信号：

- [LICENSE](/Users/eridani/Work/code-rag-agent-new/code-rag/LICENSE) 是空文件
- [test.py](/Users/eridani/Work/code-rag-agent-new/code-rag/test.py) 是空文件
- `configs/*.yaml` 为空

这些东西本身不一定影响运行，但会直接影响你重新接手时对仓库的信任感，也会影响别人看这个项目时的专业印象。

---

## 按模块点评

### 后端 API

[api/main.py](/Users/eridani/Work/code-rag-agent-new/code-rag/api/main.py) 的特点是“功能集中、可读性尚可、但过胖”。

优点：

- 路由齐全
- 逻辑比较直白
- explain 和 stream explain 都有
- workspace 隔离已经接上

问题：

- 单文件承担太多职责
- 搜索、解释、索引、Agent、状态查询都堆在一个文件里
- prompt、fallback、context 构造和 route 混在一起

建议以后拆成：

- `api/routes/search.py`
- `api/routes/indexing.py`
- `api/routes/agent.py`
- `services/explain_service.py`
- `services/index_service.py`

### 索引器

[indexer/chunker.py](/Users/eridani/Work/code-rag-agent-new/code-rag/indexer/chunker.py) 是目前最像“核心资产”的一块。

优点：

- 逻辑相对独立
- 输入输出比较明确
- 已经有一定语义信息提取

问题：

- 只支持 JS/TS
- query 规则仍比较基础
- 没有配套测试样本验证 chunk 质量
- 注释抽取和 chunk 选择逻辑仍偏启发式

### 检索器

[retriever/hybrid_search.py](/Users/eridani/Work/code-rag-agent-new/code-rag/retriever/hybrid_search.py) 目前属于“够用但非常轻”。

优点：

- 简单直接
- 有 runtime code 优先意识
- symbol boost 思路是对的

问题：

- 重排规则全靠硬编码
- 评分体系比较原始
- 没有独立评测闭环来校准参数

### LLM/Agent

[ai/llm.py](/Users/eridani/Work/code-rag-agent-new/code-rag/ai/llm.py) 比 [ai/agent.py](/Users/eridani/Work/code-rag-agent-new/code-rag/ai/agent.py) 明显更成熟。

我对它们的判断是：

- `llm.py` 体现了你后来已经开始往“统一 provider 适配层”走
- `agent.py` 还保留着更早期的 MVP 风格

也就是说，这里不是“完全不会做”，而是“重构做了一半停住了”。

### VS Code 扩展

扩展部分总体可读，而且命令入口清楚。

优点：

- 命令链路清楚
- 配置项也有基本暴露
- 结果视图和 Explain 面板都已实现

问题：

- 多工作区支持很弱，默认只取第一个 workspace，见 [clients/vscode/src/indexer.ts:47](/Users/eridani/Work/code-rag-agent-new/code-rag/clients/vscode/src/indexer.ts#L47)
- 索引文件类型和后端支持不一致
- Explain WebView 的 markdown 渲染是手写简化版，后面很容易出显示问题

---

## 为什么我不建议你直接重写

如果今天从零重写，短期会很爽，但你会丢掉几个已经很值钱的东西：

- 你去年已经想明白的产品边界
- 你已经走通的索引链路
- 你已经试过的 VS Code 交互形态
- 你对“本地优先、workspace 隔离、AST-aware”这几个关键判断

更合适的做法是：

1. 先保留现有仓库
2. 做一次“工程化清理 + 范围收敛”
3. 在此基础上再决定是否抽第二版

---

## 我建议的重整优先级

### 第一阶段：先把项目重新变得可信

目标：让仓库重新具备“我敢继续在这里开发”的感觉。

建议先做：

- 清理 `requirements.txt`，收敛到真实运行依赖
- 补全 `LICENSE`
- 删除或填充空文件
- 在 README 顶部明确“当前只支持 JS/TS”
- 对齐扩展允许上传的文件类型和后端真实支持范围
- 补一个最小 smoke test 文档或脚本

### 第二阶段：把实现和架构对齐

目标：减少“文档说一套、代码半套”的状态。

建议做：

- 统一 explain 和 agent 的 LLM 配置入口
- 把 API 路由拆文件
- 把 chunk / retrieval / provider 的关键配置外置
- 让 search 返回可用 preview
- 给索引流程加更明确的错误信息和完成摘要

### 第三阶段：再谈能力升级

目标：让项目从“可演示”走向“可持续使用”。

建议方向：

- 增强 chunk 质量和召回评估
- 增加更多语言支持
- 增加增量索引而不是每次整包上传
- 加真正的 streaming explain UI
- 做更可信的 Agent 调度和证据展示

---

## 如果把它当成你的个人作品集项目

这个项目是有作品集潜力的，但前提是你把“半成品感”去掉。

对外展示时，最重要的不是再加 5 个功能，而是把以下几点做扎实：

- 问题定义清楚
- 架构图清楚
- 支持范围明确
- demo 路径稳定
- 仓库基础卫生完整

换句话说，这个项目现在缺的不是想法，而是“第二次整理”。

---

## 我建议你接下来最值得做的事

如果你愿意继续，我建议下一步直接从下面这组动作开始，而不是继续空想：

1. 先做一次仓库清理和文档校正
2. 再统一 LLM 配置链路
3. 再修索引文件类型和 preview 这两个低成本体验问题
4. 最后再决定要不要做第二版架构拆分

这条路线投入小，但回报很高。

---

## 结论

这是一个“方向靠谱、骨架已成、但停在 MVP 工程化门口”的项目。

我对它的总体评价是：

- 选题：好
- 架构意识：比一般个人项目强
- 核心链路：成立
- 工程完成度：偏低
- 是否值得继续：值得，而且不需要推倒重来

如果你要，我下一步最适合做的是二选一：

- 直接帮你做第一轮“仓库整理 + 文档校正 + 依赖收敛计划”
- 或者直接开始改代码，先把我上面点到的 2 到 3 个高价值问题修掉

