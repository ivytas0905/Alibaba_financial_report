本项目旨在构建一个基于 RAG（Retrieval-Augmented Generation）的智能分析引擎，对阿里巴巴 FY2025 财报进行深度研判，特别围绕投资机构三维度（核心驱动力与护城河、商业模式与市场卡位、潜在风险与红旗警报）进行结构化分析，输出可直接用于投研参考的分析结果。
核心目标：
- 构建可扩展的 **RAG 检索引擎**
- 利用 LLM + Prompt 设计引导深度思考与关联分析
- 以专业投研格式输出结果（结论 / 理由 / 原文证据）

技术以及流程：
文档解析：pypdf 读取 PDF，python-docx 解析 Word
Chunking：RecursiveCharacterTextSplitter 按 1000 字符分块，200 字符重叠，保证语义连续性
Embedding 向量化：OpenAI Embeddings (text-embedding-3-small)
向量存储与检索：FAISS 本地向量数据库，支持相似度搜索
LLM 分析：GPT-4o-mini（高性价比推理能力）结合 RAG 检索上下文

Prompt 构造：针对任务三设计了专业投研格式 Prompt
Prompt格式明确要求输出结构为：结论，理由，原文证据，这样方便分析结果直观呈现。
多关键词检索：
护城河：["全球拓展", "用户基础", "商家生态", "平台技术"]
收入占比：["云智能集团 收入", "阿里国际数字商业集团 收入", "FY2025 总收入"]
风险：["宏观经济 风险", "市场竞争 风险", "数据安全", "法规监管", "合规"]

推理链引导（Chain of Thought）：
明确提示模型：先列出逻辑，再给结论
对计算类问题：要求展示公式和计算过程
对风险问题：如果原文不足，必须标注“原文未直接给出，以下为推断”

RAG 流程
文档预处理：读取财报 → 切分文本 → 向量化
数据库构建：向量存储至 FAISS，保存 chunk 元信息（页码、段落来源）
语义检索：针对查询和关键词多轮检索，合并结果去重
结果生成：构造 Prompt，调用 GPT 模型生成结论、理由、证据三段式分析
输出管理：结果输出为 Markdown，方便直接用作研报附件

遇到的困难/挑战：
从财报表格中提取的数值异常，多个业务分部收入显示为相同值（如 "31"）
EBITA 部分数据缺失（如 Cloud_Intelligence EBITA 为空）
年度指标（metrics）和风险列表（risk_factors）为空，导致任务三无法直接依赖任务一产出进行精确计算
护城河、风险信息分散在多个段落，单次检索可能无法覆盖
财报对“数据安全与法规监管”表述模糊

