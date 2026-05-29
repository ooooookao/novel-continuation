---
step: 2
title: 并行提取
---

# Step 2: 并行提取（子 agent）

## 执行清单

- [ ] 子 agent A → 风格分析 → `style-profile.md`
- [ ] 子 agent B → 人物提取 → `characters/*.txt`（含对话样本）
- [ ] 子 agent C → 世界观提取 → `world-facts.txt`
- [ ] 子 agent D → 大纲提取 → `outline-original.md`
- [ ] 全部完成后汇总，向用户展示要点
- [ ] 用户确认后进入 Step 3

原文读取完成后，启动 **4 个子 agent 并行执行**。每个子 agent 独立读取原文（≤5万字）或 `summaries/` 中的摘要（>5万字），输出结果到指定文件，互不干扰。

## 子 agent 任务分配

**子 agent A — 风格分析**
- 读取原文或 `summaries/batch-*.md`
- 从语言、叙事、氛围三个维度分析写作风格
- 输出到 `style-profile.md`
- 参考：[references/style-analysis.md](../references/style-analysis.md) + [assets/templates/style-profile.md](../assets/templates/style-profile.md)

**子 agent B — 人物提取 + 对话样本**
- 读取原文或 `summaries/batch-*.md`
- 识别主要角色，为每人创建角色文件
- 从原文中采集 2-3 段典型对话（覆盖不同情绪状态），存入角色文件的【对话样本】区块
- 输出到 `characters/[角色名].txt`
- 参考：[references/character-file-system.md](../references/character-file-system.md) + [assets/templates/character-card.txt](../assets/templates/character-card.txt)

**子 agent C — 世界观提取**
- 读取原文或 `summaries/batch-*.md`
- 提取地点、规则、物品、时间线等世界观设定
- 输出到 `world-facts.txt`
- 参考：[assets/templates/world-facts.txt](../assets/templates/world-facts.txt)

**子 agent D — 大纲提取**
- 读取原文或 `summaries/batch-*.md`
- 提取每章/每段的情节脉络、主线、支线、伏笔
- 输出到 `outline-original.md`
- 长原文（>20万字）时，大纲分层输出：
  - `outline-original.md` — 顶层：主线/支线/伏笔总览
  - `summaries/arc-XX.md` — 按故事弧分文件的章节摘要
- 参考：[assets/templates/outline-original.md](../assets/templates/outline-original.md)

## 汇总

4 个子 agent 全部完成后，主 agent 汇总结果，向用户展示：
- 风格档案要点
- 识别到的主要角色列表
- 世界观核心设定
- 原文大纲概要

用户确认无误后，进入 Step 3。
