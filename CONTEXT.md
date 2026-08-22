# 电视选购 Copilot — 上下文记录

## 项目目标
以 DeepGuide（Python 零依赖单 Agent）为代码底座，扩展为京言 AI 导购的 Multi-Agent 架构，产出一个完全自研代码、可独立运行、不依赖 Dify 的电视选购 Multi-Agent 系统。

## 来源项目
- **DeepGuide**：自研 RAG（token overlap）、NeedParser（中文数字/口语/否定）、短期+长期记忆、Replanner 硬约束检查、LLM 重试降级
- **京言 AI 导购**：Multi-Agent（Master + 5 Worker + 合规审核）、4个RAG知识库、确定性路由、上下文工程、25条评测集

## 合并策略（方案A）
- 保留 DeepGuide 的：自研 RAG、NeedParser、记忆系统、Replanner、零依赖、HTTP服务、前端执行轨迹
- 吸收京言的：Multi-Agent 分工、4个知识库、确定性路由、上下文工程、25条评测、合规审核独立节点

## 技术约束
- Python 3.10+，零第三方依赖（仅标准库）
- LLM 走 OpenAI-compatible API，无 Key 时降级确定性演示

## 创建时间
2026-08-22
