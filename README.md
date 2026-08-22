# 电视选购 Copilot — Multi-Agent 导购系统

> 从零自研 Python 代码的 Multi-Agent 电视导购系统，零第三方依赖，clone 即跑。
> 融合京言 AI 导购（Dify Multi-Agent 架构）与 DeepGuide（自研 Agent 循环）的核心能力。

## 项目简介

本项目是两个项目的合并产物：
- **京言 AI 导购**：基于 Dify 的 Multi-Agent Chatflow，15 节点，Master-Worker 架构，4 个 RAG 知识库
- **DeepGuide**：从零用 Python 实现的单 Agent，自研 RAG、需求解析、记忆系统、Replanner 约束检查

合并后保留了 DeepGuide 的**零依赖自研代码**，吸收了京言的**Multi-Agent 架构、4 个知识库、确定性路由、合规审核独立节点、25 条评测集**。

## 技术栈

- **语言**：Python 3.10+（零第三方依赖，仅标准库）
- **前端**：原生 HTML/CSS/JS（单文件）
- **LLM**：OpenAI-compatible API（可选，无 Key 时降级确定性模板）
- **存储**：JSON 文件（知识库 + 长期记忆）

## 架构设计

```
用户消息
  │
  ├─ NeedParser（需求解析：预算/尺寸/距离/用途/刷新率）
  │
  ├─ Master Router（确定性6类意图分类）
  │   ├─ product → 商品参数 Agent（RAG检索 + 结构化筛选 + 加权打分）
  │   ├─ promotion → 比价优惠 Agent（促销规则 + 叠加计算）
  │   ├─ fulfillment → 履约服务 Agent（配送/安装/入户）
  │   ├─ aftersales → 售后客服 Agent（强制转人工）
  │   ├─ clarify → 需求澄清 Agent（多轮追问，每次一个问题）
  │   └─ fallback → 兜底回复
  │
  ├─ Replanner（硬约束二次检查：预算/刷新率/库存/越权）
  │
  └─ Compliance（合规审核 Reflection，5条红线，修正循环最大2次）
      │
      └─ 最终回答 + 完整执行轨迹
```

### 四层终止条件

| 类型 | 实现 |
|---|---|
| 完成 | Worker 成功 + 合规通过 |
| 失败 | 无候选且无降级 / 合规2次修正失败 |
| 中断 | 售后高风险 / 用户要求人工 |
| 防重复 | 澄清≤3次 / 合规修正≤2次 / 主循环≤8轮 |

## 核心模块

| 模块 | 文件 | 说明 |
|---|---|---|
| LLM 客户端 | `core/llm_client.py` | 3次指数退避重试，模型分级（强/弱模型） |
| 记忆系统 | `core/memory.py` | 短期记忆（会话槽位）+ 长期记忆（用户授权后本地存储） |
| 需求解析 | `core/parser.py` | 支持中文数字/3k缩写/2米5口语/否定句/4K8K排除 |
| RAG 检索 | `core/rag.py` | 自研 token overlap 检索 + 结构化商品筛选打分 |
| 约束检查 | `core/replanner.py` | 预算/刷新率/库存/越权检查，无候选拒绝编造 |
| Master 路由 | `agents/master.py` | 确定性6类分类，对比/可行性/高端意向特殊放行 |
| 商品 Agent | `agents/product.py` | 推荐 + 型号对比 + 引用标注 |
| 优惠 Agent | `agents/promotion.py` | 促销检索 + 叠加规则计算 + 到手价估算 |
| 履约 Agent | `agents/fulfillment.py` | 配送/安装/入户政策查询 + 兜底 |
| 售后 Agent | `agents/aftersales.py` | 问题分类 + 强制转人工 |
| 澄清 Agent | `agents/clarify.py` | 多轮追问，带选项，最多3次 |
| 合规审核 | `agents/compliance.py` | 5条红线 + 修正循环 |
| 编排引擎 | `app.py` | ReAct 风格主循环 + HTTP 服务 |

## 快速开始

### 环境要求
- Python 3.10+
- 无需安装任何依赖

### 启动

```bash
cd tv-shopping-copilot
python app.py
```

访问 http://localhost:8765

### 配置 LLM（可选）

设置环境变量后启用 LLM 润色推荐话术：

```bash
set AI_API_KEY=your_api_key
set AI_BASE_URL=https://api.openai.com/v1
set AI_MODEL=gpt-4o-mini
```

不设置时自动降级为确定性模板模式，架构和执行轨迹完整可用。

### 运行评测

```bash
python eval.py
```

## 节点 PRD（六要素）

每个 Worker Agent 均附带完整节点 PRD，包含：
- 节点名称与描述
- 输入字段（名称/类型/是否必填/来源）
- 输出格式
- 权重规则
- 异常处理
- 枚举值

详见各 Agent 文件中的 `prd` 类属性。

## 评测结果

**V1.0 评测：25/25 = 100%**

| 类别 | 通过率 | 达标线 |
|---|---|---|
| 正常 Case（15条） | 100% | ≥90% |
| 边界 Case（7条） | 100% | ≥50% |
| 异常 Case（3条） | 100% | ≤1%异常率 |

详细评测报告见 [EVALUATION_REPORT.md](EVALUATION_REPORT.md)。

## 知识库

`data/knowledge.json` 包含 4 个知识库：
- **商品库**：10 款电视（覆盖 43-75 寸，1299-7999 元）
- **知识文档**：5 篇选购指南（尺寸/面板/游戏/明亮客厅/通用建议）
- **促销规则**：5 条（以旧换新/满减/直降/价保/优惠券）
- **履约政策**：4 条（配送/安装/入户/运费）
- **售后规则**：4 条（七天无理由/保修/质量问题/发票）

## 成本优化

- **模型分级**：简单分类用规则/弱模型，复杂推荐用强模型
- **结果缓存**：高频查询结果缓存24小时
- **上下文压缩**：每个 Agent 只加载所需上下文，不全量塞入
- **并行节点**：商品/优惠/履约可并行调用（架构支持）

## 项目文件结构

```
tv-shopping-copilot/
├── app.py                  # 编排引擎 + HTTP服务
├── eval.py                 # 自动化评测脚本
├── smoke_test.py           # 冒烟测试
├── CONTEXT.md              # 项目上下文记录
├── README.md
├── EVALUATION_REPORT.md    # 评测报告
├── AI_PRD.md               # AI产品需求文档
├── core/                   # 核心模块
│   ├── llm_client.py
│   ├── memory.py
│   ├── parser.py
│   ├── rag.py
│   └── replanner.py
├── agents/                 # Agent模块
│   ├── base.py
│   ├── master.py
│   ├── product.py
│   ├── promotion.py
│   ├── fulfillment.py
│   ├── aftersales.py
│   ├── clarify.py
│   └── compliance.py
├── data/
│   ├── knowledge.json      # 4个知识库
│   ├── eval_cases.json     # 25条评测集
│   └── eval_report.json    # 评测结果
├── templates/
│   └── index.html          # 前端界面
└── memory/                 # 长期记忆（运行时生成）
```

## License

MIT
