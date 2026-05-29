---
name: novel-continuation
description: >-
  续写小说的专业 skill。根据用户提供的小说原文，分析写作风格、情节脉络、人物性格、
  语调主题，然后生成风格无缝衔接的续写内容。支持中文和英文小说。用户可指定续写长度、
  情节走向、焦点角色。当用户提到"续写"、"接着写"、"写下一章"、"继续写"、
  "continue the story"、"write the next chapter"、"keep writing"等关键词时触发。
  也适用于用户提供一段小说内容并希望 agent 基于此继续创作的场景。
  即使用户没有明确说"续写"，只要他们粘贴了一段小说内容并表达出想要继续的意图，
  也应触发此 skill。
---

# Novel Continuation

专业的小说续写工具。核心理念：**先分析，再规划，后写作，定期反思**。

---

## ⚠️ 执行纪律

**两层文件结构：**

| 文件 | 用途 | 何时读 |
|------|------|--------|
| `SKILL.md`（本文件） | 全局规则、策略、故障处理、工作流导航 | skill 加载时自动读 |
| `steps/stepN-*.md` | 每步的详细操作指南 + 执行清单 | 执行到该步时才读 |

- **不要凭记忆做事**：执行某步前，先读对应的 step 文件
- **每步完成后**：对照 step 文件顶部的执行清单打勾，确认全部完成再进下一步
- **清单是强制的**：每个 `- [ ]` 都必须完成，跳过任何一个都要回头补

---

## When to Use

Use this skill when the user:
- 粘贴了一段小说内容，要求续写
- 说"接着写"、"续写"、"继续写"、"写下一章"
- 提供章节内容并询问"接下来会发生什么"
- 要求基于现有内容扩展或延伸故事
- 说 "continue the story"、"write the next chapter"、"keep going"

## Do NOT Use

- 用户要求从零开始写新小说（使用 novel-writer skill）
- 用户要求修改已有内容而非续写
- 非虚构写作、剧本、诗歌等非小说体裁

---

## 整体工作流

每步的 **✓** 是最低要求，不可跳过。详细清单见对应 step 文件。

```
Step 1 → steps/step1-read-original.md
  ✓ 接收原文  ✓ 确认章数/字数  ✓ >5万字则选阅读策略  ✓ 生成摘要
    ↓
Step 2 → steps/step2-parallel-extraction.md
  ✓ 4个子agent并行提取（风格/人物/世界观/大纲）  ✓ 汇总展示给用户
    ↓
Step 3 → steps/step3-confirm-params.md
  ✓ 章数  ✓ 字数  ✓ 方向  ✓ 角色  ✓ 风格  ✓ 计算总字数
    ↓
Step 4 → steps/step4-outline-generation.md
  ✓ 生成大纲  ✓ 详细程度匹配字数  ✓ >20章则按弧拆分
    ↓
Step 5 → steps/step5-user-review.md
  ✓ 交给用户  ✓ 等待反馈  ✓ 修改直到确认
    ↓
Step 6 → steps/step6-outline-review.md（两轮：粗调→修→细调）
  ✓ 第1轮粗调：9维扫描，只报严重问题  ✓ 修改大纲  ✓ 第2轮细调：9维扫描，报所有问题  ✓ 最终确认
    ↓
Step 7 → steps/step7-writing.md（核心循环，每章重复）
  ✓ 准备上下文  ✓ 写作子agent  ✓ 字数校验(≥90%)  ✓ 质量审查(5维+反AI)
  ✓ 保存+更新(delta/progress/threads/角色/大纲)  ✓ arc末章则生成checkpoint
    ↓
Step 8 → steps/step8-reflection.md（arc完成或≥15章触发）
  ✓ 10维反思  ✓ 保存报告  ✓ 展示发现  ✓ 根据报告调整
    ↓
Step 9 → steps/step9-export.md
  ✓ 拼接章节  ✓ 生成统计  ✓ 保存.txt和.md
```

**断点续写**：读取 `progress.md` → 跳过 Step 1-6 → 从 Step 7 接续。恢复步骤见 `steps/step7-writing.md`。

---

## 子 agent 策略

用子 agent 隔离上下文、并行执行。**主 agent** 协调+汇总+更新状态+交互，**子 agent** 执行具体任务。

### 子 agent 故障处理

子 agent 调用可能遇到两类问题：**调用失败**（超时/报错/无响应）和**质量熔断**（反复重写仍不达标）。

#### 调用失败降级策略

```
第1次失败 → 等待 5 秒，重试（同参数）
第2次失败 → 等待 15 秒，重试（同参数）
第3次失败 → 降级：主 agent 自己执行该任务
```

**各任务类型的降级处理：**

| 任务 | 降级方式 |
|------|----------|
| 写作 | 主 agent 直接写，写完后立即保存到文件 |
| 质量审查 | 主 agent 自审，评分标准不变 |
| 扩写 | 主 agent 直接扩写 |
| 提取（风格/人物/世界观/大纲） | 主 agent 逐个执行 |
| 反思检查 | 主 agent 做简化版（只查最近 5 章） |
| 大纲自审 | 主 agent 自审，只查前 4 个维度 |

降级执行的结果仍需通过正常校验/审查。在 `progress.md` 中记录降级事件。

#### 质量熔断机制

```
第1次不达标 → 退回重写
第2次不达标 → 退回重写（上限）
第3次不达标 → 熔断：暂停，向用户报告
```

**熔断报告内容：**
```
⚠️ 写作熔断 — 第X章已重写3次仍未通过质量审查。

第1次：X.X/10 — [问题A]
第2次：X.X/10 — [问题B]
第3次：X.X/10 — [问题C]

请选择：
1. 降低质量标准（接受 7.0+）
2. 降低字数要求
3. 修改大纲
4. 手动提供内容
5. 跳过，标记待补写
```

扩写同样 3 次熔断。大纲自审/反思失败不影响主流程，降级或跳过。

### 步骤切换协议

每步完成后、进入下一步前：

1. 对照当前 step 文件顶部的执行清单，确认所有 checkbox 都已完成
2. 如有遗漏，立即补完
3. 读取下一步的 step 文件，再开始执行

---

## Step 3: 确认续写参数 → `steps/step3-confirm-params.md`

## Step 5: 用户审阅大纲 → `steps/step5-user-review.md`

## Step 9: 导出完整小说 → `steps/step9-export.md`

---

## 进度追踪

### progress.md — 项目状态文件

项目的"记忆中枢"，新会话时第一个读取的文件。包含：项目信息、当前进度、Arc 追踪、强制提示词、大纲更新记录、待处理反馈、下一步行动。

模板：[assets/templates/progress.md](assets/templates/progress.md)

### 每章写完后的更新

见 `steps/step7-writing.md` 的【保存与更新】。

### 用户反馈处理

1. 记录到 `progress.md` 的【待处理反馈】
2. 分析影响范围（单章 / 后续走向 / 角色设定）→ 对应修改
3. 处理完后从【待处理反馈】中移除

### 进度查看

用户问"写到哪了"时，读取 `progress.md` 汇报：已完成章数、当前章节/Arc、距下次反思还有几章、待处理反馈、预计剩余工作量。

---

## Language Handling

- **中文原文 → 中文续写**: 保持原文语言风格（简体/繁体、文言/白话）
- **英文原文 → 英文续写**: 保持原文 register（formal/informal, American/British）
- **混合语言**: 续写保持同样的混用模式
- **不要翻译**: 除非用户明确要求

---

### 状态增量系统

每章写完后产出 delta，每个 arc 完成时生成 checkpoint。模板：
- [assets/templates/delta.md](assets/templates/delta.md)
- [assets/templates/checkpoint.md](assets/templates/checkpoint.md)

### threads.md

伏笔/情节线程账本，每章写完后必须更新。模板：[assets/templates/threads.md](assets/templates/threads.md)

---

## 角色日志截断机制

续写超过 30 章后，角色文件的【续写更新日志】归档（`characters/archive/`），只保留最近 20 章活跃日志。

---

## Edge Cases

- **原文太短** (< 500字): 询问用户是否能提供更多内容
- **原文是对话体**: 续写保持对话为主
- **原文有明显错误**: 不要"纠正"原文的风格选择
- **用户要求改变风格**: 告知会破坏一致性，但尊重选择
- **角色太多** (>8个): 建议用户确认核心角色
- **原文为 .docx**: 先用 python-docx 提取纯文本
- **扩写 3 次仍不达标**: 停止循环，向用户报告
