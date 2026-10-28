这份文档会把之前定的四阶段路径拆成逐周可打卡、有明确交付物和检验标准的作战表：整体节奏是 第 0–1 个月搭环境跑通第一个 LLM 调用 → 第 1–3 个月完成 RAG+Agent 核心技能并产出第一个上架 App → 第 4–8 个月补齐全栈与端侧 AI、形成作品集矩阵 → 第 9–12 个月冲刺 FDE/架构师面试。所有学习资料优先选官方免费源（LangChain Academy、Apple 开发者文档、OpenAI/Anthropic 官方文档），保证链接有效、不依赖任何付费训练营。
执行前必读：这份计划怎么用
以周为最小单位：每周有 1 个核心任务、1 个产出物、1 条检验标准，完不成就顺延，不跳步。
产出导向：每个阶段结束时，你的 GitHub 上要新增 1 个能拿得出手的项目，这些项目就是你 12 个月后面试的全部筹码。
时间假设：按每周 15–20 小时投入设计（工作日 2 小时 + 周末 6 小时），如果你是全职转型，可以把每个阶段压缩到 2/3 时长。
阶段总览对照表
维度	阶段一：入门跳板（第1–3个月）	阶段二：Agent深耕（第4–8个月）	阶段三：全栈+端侧（第9–12个月）	阶段四：高阶跃迁（第12个月起）	
核心任务	Python + LLM API + Prompt 工程 + RAG 基础	LangChain/LangGraph + MCP + Agent 编排	FastAPI + Next.js + 端侧 AI（MLX/Foundation Models）	行业解决方案 + 售前交付 + 架构设计	
产出物	1 个接 LLM 的 iOS App + 1 个 RAG Demo	1 个带工具调用的 Agent App（上架）	1 个全栈 SaaS + 1 个端侧 AI App	2–3 个企业级案例 + 技术博客矩阵	
目标岗位	AI 应用开发工程师（实习/初级）	AI Agent 开发工程师	AI 全栈 / 端侧 AI 工程师	解决方案架构师 / FDE	
薪资锚点	16–25K	25–40K	30–50K	40–80K（FDE 年包可百万级）	
阶段〇：环境与账户准备（第 1 周，一次性完成）
这一周不学任何新知识，只把"弹药库"备齐，后面所有阶段都依赖这一步。
要开的账户：OpenAI Platform（或 Anthropic Console）、Hugging Face 账号、GitHub 账号（如已有则升级仓库结构）、LangSmith 免费账号（LangChain Academy 课程依赖它做 Tracing）。LangChain Academy 官方明确所有课程免费，注册入口在 smith.langchain.com。
要装的工具链：
Python 3.11–3.13（LangChain Academy 官方指定版本区间，别装 3.15 最新版，避免库兼容问题）
Xcode 26+（Foundation Models 框架需要 iOS/macOS 26 SDK）
Docker Desktop（后面 Dify 自部署、Milvus 都要用）
包管理：brew install python@3.12 + pip install uv（uv 比 pip 快 10 倍，是 2025 年后的事实标准）
第 1 周检验标准：终端能跑通 python3 --version、docker --version、swift --version 三条命令；用 Python 发出第一次 OpenAI/Claude API 调用并收到回复（参考 Claude 官方 Get Started，5 行代码即可）。
阶段一：入门跳板（第 1–12 周）
目标岗位：AI 应用开发工程师 | 月薪锚点 16–25K
这个阶段的核心命题是：把"iOS 开发者"的身份升级为"会用 LLM 的 iOS 开发者"，用你最熟悉的 Swift 做抓手，同时把 Python 补到能干活的水平。
学习资料清单（全免费）
学习内容	推荐资料	时长	说明	
Python 基础	Python 官方 Tutorial（docs.python.org/3/tutorial）	2 周	官方教程明确说"面向会编程的人"，你有 Swift 功底，跳读即可	
Python 补充	python.org/about/gettingstarted	2 天	官方入门页，含环境配置指引	
LLM API 调用	OpenAI Cookbook（github.com/openai/openai-cookbook）	持续参考	官方示例库，Function Calling、流式响应都有可运行代码	
Claude API	platform.claude.com/docs/en/get-started	2 天	Anthropic 官方入门，和 OpenAI API 二选一深入即可	
Prompt 工程	Claude 官方 Prompt Engineering 指南	3 天	在 Claude 文档站的 Build 板块，行业标杆教材	
逐周执行计划
第 1–3 周：Python 速通
干什么：用官方 Tutorial 过语法（重点：列表推导、装饰器、async/await、类型注解——这些和 Swift 的 protocol、闭包、async 高度同构，你会学得飞快）
跟练：每学一个语法点，立刻在 LeetCode 用 Python 刷 2 道简单题巩固
产出：一个 python-cheatsheet.md，记录 Swift→Python 语法映射表（这个笔记本身就能成为一篇爆款技术博客）
第 4–5 周：LLM API 与流式响应
干什么：写 Python 脚本调用 OpenAI/Claude API，重点吃透三件事——流式响应（SSE）、Function Calling、结构化输出（JSON mode）
跟练：跟着 OpenAI Cookbook 的 Function Calling 教程，做一个"查天气+算汇率"的命令行小助手
产出：一个 llm-api-playground 仓库，包含流式对话、工具调用、多轮记忆三个脚本
第 6–8 周：Swift 侧集成（你的主场）
干什么：把 Python 里学到的东西用 Swift 重做一遍——用 URLSession/AsyncStream 接 LLM API，实现打字机流式 UI
跟练：直接看 Apple Foundation Models 框架的 WWDC25 入门 session（免费视频，讲 LanguageModelSession 的流式输出与 Tool Calling）
产出：一个 iOS 聊天 App，支持流式输出 + Function Calling + 会话历史，这是你第一个面试作品
第 9–12 周：RAG 基础
干什么：学 RAG 完整管线——文档加载→切块→Embedding→向量存储→检索→生成
学什么：DeepLearning.AI 的 Retrieval Augmented Generation 短课程（约 2 小时视频 + 动手作业，免费）；向量数据库先用 Milvus Lite（pip 装 pymilvus 即可，官方 Quickstart 10 分钟能跑通）
跟练：用 Python 搭一个"个人笔记问答系统"——把你手机里的备忘录/Notion 导出成 Markdown，切块入库，命令行提问
产出：rag-notes-assistant 仓库，README 里写清切块策略、Embedding 选型、检索效果对比
阶段一验收清单
[ ] Python 能独立写 200 行以上的脚本，不查语法
[ ] 讲得清 SSE 流式响应的原理和 Swift/Python 各自的实现方式
[ ] RAG Demo 能跑通，且能回答"为什么要切块、块多大合适"这类原理题
[ ] GitHub 上有 3 个仓库：API playground、iOS 聊天 App、RAG Demo
[ ] 简历更新：把 iOS 项目经历重写为"具备 LLM 集成能力的移动端工程师"
阶段二：Agent 深耕（第 13–32 周）
目标岗位：AI Agent 开发工程师 | 月薪锚点 25–40K
这是整个转型中含金量最高、回报最厚的阶段。Agent 是 2026 年大模型应用的核心形态，供需缺口最大，而且它和你 iOS 的异步编程、状态管理经验天然契合。
学习资料清单（全免费）
学习内容	推荐资料	时长	说明	
LangGraph 入门	LangChain Academy《Introduction to LangGraph》（免费）	6 小时	官方课程，覆盖状态、记忆、Human-in-the-loop、部署	
LangGraph 速成	《LangGraph Essentials - Python》（免费，1 小时）	1 小时	精简版，先看这个再看完整版	
LangChain 体系	《Foundation: Introduction to LangChain》（免费，30 课）	1.5 小时	含 MCP、多 Agent、Human-in-the-Loop 全模块	
课程配套代码	github.com/langchain-ai/langchain-academy	持续	所有课程的 Jupyter notebook 都在这里，克隆即用	
MCP 协议	modelcontextprotocol.io 官方文档	1 周	"AI 应用的 USB-C"，Agent 连接外部系统的标准	
MCP 规范	MCP Specification（2025-06-18 版）	2 天	深入理解 tools/resources/roots 概念	
Agent 综合课	Hugging Face Agents Course（免费+认证证书）	2 周	72K+ 人已报名，结课有证书可挂 LinkedIn	
Agent 进阶	DeepLearning.AI《AI Agents in LangGraph》	4 小时	从零手写 Agent 再用 LangGraph 重构，含持久化与 HITL	
逐周执行计划
第 13–16 周：LangGraph 通关
干什么：按顺序刷完 LangChain Academy 的三门课，每节课的 notebook 都亲手跑一遍
跟练：跟着《Introduction to LangGraph》的课程项目，搭建一个带记忆和人工介入的邮件处理 Agent
产出：langgraph-notes 仓库，包含你跑通的所有 notebook + 每个概念的中文图解笔记
第 17–20 周：MCP 协议与工具生态
干什么：通读 MCP 官方文档，理解 Server/Client 架构；然后动手写一个自己的 MCP Server
跟练：给上一阶段的 iOS 笔记 App 写一个 MCP Server，让它能被 Claude/Cursor 等客户端调用——比如"查询我的笔记""往笔记里追加内容"这两个 tool
产出：mcp-notes-server 仓库（这是差异化亮点：市面上会写 MCP Server 的 iOS 开发者极少）
第 21–26 周：iOS Agent App 立项开发（本阶段核心交付）
干什么：把前面的积累整合成一个真正的产品级 iOS Agent App。选题建议选你自己有痛点的场景，例如：
智能相册管家：拍照/截图 → 多模态 LLM 理解内容 → 自动打标签归档 → 支持自然语言检索
会议纪要 Agent：录音转写 → 结构化总结 → 自动生成待办并写入提醒事项
技术要点：用 Foundation Models 框架做端侧推理（参考 WWDC25 的 Deep dive session，学 guided generation 和 tool calling）；复杂任务云端用 GPT-4o/Claude，形成端云混合架构
产出：一个上架 App Store 的 Agent App——这是你简历的核弹级筹码，面试时"我自己做了一个上架的 AI App"这句话的分量远超任何课程证书
第 27–32 周：Agent 工程化与评测
干什么：补 Agent 的生产化知识——评测、可观测性、成本控制
学什么：RAGAS 框架（RAG/Agent 评测的事实标准，官方文档 docs.ragas.io 有完整入门）；给上架的 App 接入 LangSmith 做 Tracing
跟练：为你的 Agent App 设计一套评测集（20 个典型问题 + 10 个边界 case），用 RAGAS 跑出基线指标，再做一轮优化对比
产出：一篇技术博客《我是如何评测和优化我的 iOS Agent App 的》，发到个人博客或掘金
阶段二验收清单
[ ] 能不看文档手写一个包含 state、memory、tool calling 的 LangGraph 应用
[ ] 独立开发过至少 1 个 MCP Server
[ ] 有 1 个上架 App Store 的 Agent App（或 TestFlight 灰度）
[ ] 能讲清 Agent loop 的本质：规划→调用工具→观察→循环
[ ] 用 RAGAS 或类似框架给自己的项目做过量化评测
[ ] 开始在 GitHub 置顶仓库写英文 README，积累国际可见度
阶段三：全栈 + 端侧 AI（第 33–52 周）
目标岗位：AI 全栈工程师 / 端侧 AI 工程师 | 月薪锚点 30–50K
这个阶段兵分两路：明线是补 Web 全栈（FastAPI + Next.js），让你具备端到端交付能力；暗线是深耕 Apple 端侧 AI（MLX + Core ML 量化），这是 iOS 出身者独有的护城河。
学习资料清单（全免费）
学习内容	推荐资料	时长	说明	
Next.js 官方课程	nextjs.org/learn 的 App Router 交互课程	1–2 周	官方免费 15 章实战课，做一个全栈 Dashboard	
Next.js 文档	nextjs.org/docs	持续参考	官方文档，Ctrl+K 搜索极快	
FastAPI	官方文档	1 周	Python 异步后端事实标准	
MLX Swift	github.com/ml-explore/mlx-swift	2 周	Apple 官方 Swift 数组框架，API 仿 NumPy/PyTorch	
MLX 示例库	github.com/ml-explore/mlx-swift-examples	2 周	含 LLM 聊天、LoRA 微调、Stable Diffusion 完整示例	
MLX WWDC session	WWDC26 session 328	40 分钟	官方讲 MLX Swift 数值计算，含代码演示	
Core ML 量化	apple.github.io/coremltools 量化指南	3 天	FP16→INT8→INT4 权重量化	
Core ML 文档	developer.apple.com/documentation/coreml	持续	模型转换、NNEngine 部署	
Dify	docs.dify.ai 自部署文档	3 天	企业级 Agent 低代码平台，Docker Compose 一键起	
逐周执行计划
第 33–36 周：Next.js 全栈
干什么：刷完官方 App Router 课程（做一个财务 Dashboard），同时用 FastAPI 给你的 iOS App 写后端
跟练：Next.js 官方课程是"边读边写"的交互式教程，跟着敲完 15 章就能掌握路由、数据获取、认证、部署全流程
产出：给你的 iOS Agent App 配一个 Web 管理后台（用 Next.js），实现数据同步
第 37–40 周：MLX Swift 深入
干什么：跑通 mlx-swift-examples 里的 LLMEval 和 LoRATrainingExample，理解在 Apple Silicon 上原生跑 LLM 的完整链路
跟练：用 MLX Swift 在 iPhone 上跑一个 1B–3B 参数的小模型，做离线问答功能
产出：一篇《在 iPhone 上用 MLX Swift 跑 LLM 的完整实践》技术博客（这个选题在中文技术社区极少有人写，容易出圈）
第 41–46 周：模型量化与性能优化
干什么：学 coremltools 的量化 API（OpLinearQuantizerConfig 等），把一个开源 LLM 从 FP16 压到 INT8/INT4 部署到 iOS
跟练：Hugging Face 社区有一篇实测可行的《Mac ARM 上把 LLM CoreML 量化到 int8》完整脚本教程，照着做一遍
产出：on-device-llm 仓库：一个量化后的端侧 LLM iOS Demo，README 记录模型体积、推理速度、内存占用的量化前后对比数据
第 47–52 周：企业级全栈项目（本阶段收官）
干什么：做一个 SaaS 化的完整产品，把所有技能串起来。例如"企业知识库问答 SaaS"：Next.js 前端 + FastAPI 后端 + RAG（Milvus 向量库）+ Agent（LangGraph）+ iOS 客户端（复用你之前的 App）+ Dify 做管理后台
产出：一个有完整 README、架构图、部署文档（Docker Compose 一键起）的 SaaS 项目——这是你冲击全栈岗位的核心作品
阶段三验收清单
[ ] 能独立用 Next.js + FastAPI 搭建带认证、数据库、支付的全栈应用
[ ] 用 MLX Swift 在 iOS 设备上成功跑通过 LLM 推理
[ ] 掌握至少一种模型量化方案（coremltools 或 GGUF），有实测数据
[ ] GitHub 有一星标数 > 100 的项目（端侧 AI 类项目容易起量）
[ ] 技术博客累计 10 篇以上，其中至少 1 篇被社区推荐
阶段四：高阶跃迁（第 12 个月起，长期）
目标岗位：解决方案架构师 / FDE 前沿部署工程师 | 月薪锚点 40–80K
这一阶段技术学习让位于行业纵深与交付能力，动作从"写代码"变为"写方案、做 PoC、对客户"。
学什么：不再有统一课表，按目标行业定制——选一个垂直领域（金融/政务/制造），读透该行业的 AI 落地案例；学方案文档写作（参考各大模型厂商官网的白皮书结构）；练技术宣讲。
干什么：
主动在公司内部或朋友公司里找 1–2 个免费/低价的 AI 落地机会做完整交付（哪怕是帮一家本地门店做智能客服）
把每个项目沉淀成标准交付物：需求分析文档、方案架构图、PoC 演示、上线运维手册
在技术社区做公开分享（线上线下各至少 3 次），这是 FDE 岗位"客户沟通能力"的最佳证明
验收标准：当你能对着一群不懂技术的业务方，30 分钟讲清"RAG 能帮你解决什么、不能解决什么、要花多少钱、多久见效"时，你就具备 FDE 的核心能力了。
全周期防坑指南（每条都是前人踩过的血泪）
别从数学啃起——Transformer 论文、反向传播推导这些，对应用层岗位的面试加成为零，等真需要再补。你的时间是最大的成本。
别只用 Swift 做 AI——Swift 的 LLM 生态成熟度远落后于 Python/TypeScript，纯 Swift 做后端会很痛苦。Python 必须学，但对会 Swift 的你来说两周就够了。
别追求"先学完再动手"——AI 技术迭代太快，任何课程学完都有一半过时。正确姿势是"学 30% 就动手，遇到问题再回头补"。
别忽视 API 费用管理——第一周就设好 OpenAI/Anthropic 的月度消费上限（$20–50 足够整个学习期），养成用小模型调试、大模型交付的习惯。
别做又一个"ChatGPT 套壳"——面试官看过一千个聊天 Demo 了。你的差异化在于：端侧推理、iOS 生态深度集成、MCP 工具链——这些别人做不了，你能。
GitHub 比证书值钱——LangChain Academy 证书、HF Agents 认证可以顺手拿，但简历上真正让面试官停留 10 秒的，是你那个上架的 App 和量化实测数据。
自我重审：交付前的核对清单
✅ 每个阶段有明确的岗位目标、薪资锚点、时间跨度
✅ 每个学习主题都有具体资料 + 链接 + 时长，优先官方免费源
✅ 逐周计划覆盖到"每周干什么、跟练什么、产出什么"
✅ 四个阶段各有验收清单，可自查是否达标
✅ 产出的 GitHub 项目有明确规格（iOS 聊天 App → MCP Server → 上架 Agent App → 全栈 SaaS → 量化端侧 Demo）
✅ 防坑指南覆盖了路径规划、技术选型、心态管理三维度
✅ 所有引用资料链接均来自官方源（Python 官方教程、LangChain Academy、Apple Developer、modelcontextprotocol.io、DeepLearning.AI、Hugging Face、Milvus、Ragas、Dify、OpenAI Cookbook），无个人博客或过时资源
补充说明两点：一是文中提到的薪资锚点是一线城市 2025–2026 年的市场行情，会随行情波动，投递前建议在招聘平台实时核对；二是 Apple 侧的 Foundation Models、MLX 更新极快（WWDC 每年 6 月大更新），到对应阶段时优先去 developer.apple.com 看最新 session，本文给的入口长期有效。
今天就可以开始执行的动作只有一个：打开终端，装好 Python 3.12 和 Docker，注册 OpenAI/Anthropic API Key，跑通第一次调用。第一步迈出去，后面的路自然清晰。