# 京言 AI 导购 Agent

> 基于 Dify 搭建的 Multi-Agent 电商导购 Chatflow，15 节点 / 19 连线，25 条评测用例驱动 V1.0→V1.1 迭代，通过率 60%→92%。

## 项目简介

面向京东 3C 家电（电视品类）的 AI 导购 Agent，采用 Master-Worker 多智能体架构：

- **Master（意图分类）**：Question Classifier 做 6 类意图确定性路由
- **4 个专业子 Agent**：商品参数、比价优惠、履约服务、售后客服
- **2 个辅助 Agent**：需求澄清（多轮追问）、兜底回复（超范围/闲聊）
- **合规审核**：独立 Reflection 节点（temperature=0），5 条红线自检
- **RAG 知识库**：4 个业务域知识库按意图动态检索，Token 降低 90%

## 架构图

```
用户消息 → 意图分类（6类）
  ├─ 商品咨询 → 商品知识检索 → 商品参数Agent ─┐
  ├─ 价格优惠 → 促销规则检索 → 比价优惠Agent ─┤
  ├─ 安装履约 → 服务承诺检索 → 履约服务Agent ─┼→ 结果汇聚 → 合规审核(Reflection) → 最终回复
  ├─ 售后服务 → 售后规则检索 → 售后客服Agent ─┤
  ├─ 需求模糊 → 需求澄清Agent ────────────────┤
  └─ 其他/闲聊 → 兜底回复Agent ───────────────┘
```

## 目录结构

```
├── dsl/                          # Dify Chatflow DSL（可直接导入）
│   └── dify-chatflow-京言AI导购.yml
├── knowledge-base/               # 4 个 RAG 知识库源文件
│   ├── 01-商品参数库.md           # 4 款 75 寸电视详细参数
│   ├── 02-促销规则库.md           # 满减/PLUS/以旧换新/国补
│   ├── 03-服务承诺库.md           # 配送/安装/入户政策
│   └── 04-售后规则库.md           # 退换货/保修/转人工
├── evaluation/                   # 评测脚本与结果
│   ├── run_eval_v11.py           # 评测脚本（调 Dify API 批量跑用例）
│   ├── eval-results-v1.0.md      # V1.0 评测结果
│   ├── eval-results-v1.1.md      # V1.1 评测结果
│   ├── eval-results-v1.0.json    # V1.0 原始数据
│   └── eval-results-v1.1.json    # V1.1 原始数据
└── screenshots/                  # 工作流截图
    └── dify-workflow.png
```

## 快速开始

1. 在 [Dify](https://cloud.dify.ai) 中创建 Chatflow 应用
2. 导入 `dsl/dify-chatflow-京言AI导购.yml`
3. 创建 4 个知识库，分别上传 `knowledge-base/` 下的 Markdown 文件
4. 将 4 个知识检索节点关联到对应知识库
5. 配置模型（推荐 deepseek-chat），发布即可对话

## 评测结果

### V1.0 基线

| 指标 | 结果 |
|------|------|
| 意图识别准确率 | 96%（24/25） |
| 回答通过率 | 60%（15/25） |
| 人工评分 | 3.68/5 |
| 平均 Token/条 | 2479 |
| 免责标注率 | 100% |
| 多轮指代理解 | 3/3 |

**4 类 Bad Case：**
1. 幻觉：推荐知识库外型号（雷鸟、Redmi 等）
2. 事实错误：国补和以旧换新规则说反
3. 退货未转人工：询问订单号、解释流程
4. 超范围：兜底回复提及其他家电

### V1.1 迭代

| 指标 | V1.0 | V1.1 |
|------|------|------|
| 意图识别准确率 | 96% | **100%** |
| 回答通过率 | 60% | **92%** |
| 人工评分 | 3.68/5 | **4.44/5** |
| 幻觉（库外型号） | 2 例 | **0 例** |
| 退货转人工 | 1/2 | **2/2** |
| 平均 Token/条 | 2479 | 2423 |

**修复手段（仅改 Prompt，不动架构）：**
- 商品 Agent：硬编码 4 款型号白名单 +「绝对禁止」段
- 比价 Agent：修正国补/以旧换新可叠加规则
- 售后 Agent：退货/退款/换货立即转人工，不询问不解释
- 所有 Agent：末尾加「只服务电视品类」

## 技术栈

- **编排平台**：Dify Cloud
- **模型**：deepseek-chat（意图分类温度 0，Agent 温度 0.3）
- **Rerank**：qwen3-rerank（top_k=4, score_threshold=0.5）
- **评测**：Python 脚本调 Dify Chat API 批量跑用例

## 作者

杨惠雯 — AI 产品经理候选人
- GitHub: [@Alaraby527](https://github.com/Alaraby527)
