> 感谢雷总 token 大放水。以前看小说太监只能骂作者，现在自己写结局了 _(:3 」∠ )_

# Novel Continuation

小说续写 Skill for Claude Code。根据用户提供的小说原文，分析写作风格、情节脉络、人物性格、语调主题，然后生成风格无缝衔接的续写内容。

## 功能特点

- **风格分析**：自动提取原文的叙事视角、句式节奏、用词偏好、修辞手法等风格特征
- **多维度提取**：并行提取人物档案、情节线索、世界观设定、伏笔线索
- **大纲驱动**：先生成续写大纲，经用户确认后再动笔，避免方向跑偏
- **质量自检**：写完后自动反思评分（风格一致性、人物还原度、情节推进、文学性），不达标则修订
- **中英文支持**：中文和英文小说均可处理

## 适用场景

- 用户提供一段小说内容，希望 agent 继续创作
- 续写下一章、补充情节、扩展故事线
- 用户说"续写"、"接着写"、"写下一章"、"继续写"、"continue the story"等

## 安装

```bash
git clone https://github.com/ooooookao/novel-continuation.git ~/.claude/skills/novel-continuation
```

## 使用方式

在 Claude Code 中直接提供小说原文并表达续写意图：

```
帮我续写这段小说，保持风格一致：
[粘贴原文]
```

```
接着写下一章，大概 3000 字
```

```
这段故事还没写完，帮我继续
```

### 关键词触发

续写、接着写、写下一章、继续写、continue the story、write the next chapter、keep writing 等。

## 工作流程

```
Step 1：阅读原文
  └── 理解故事背景、基调、当前场景
  │
Step 2：并行提取（4 个子 agent 同时工作）
  ├── 风格档案（叙事视角、句式、用词、修辞）
  ├── 人物档案（性格、关系、说话方式、当前状态）
  ├── 情节线索（主线/支线、伏笔、冲突）
  └── 世界观设定（时间地点、规则、已知事实）
  │
Step 3：确认续写参数
  └── 字数、走向、焦点角色、特殊要求
  │
Step 4：大纲生成
  └── 场景级续写大纲（开端→发展→转折→高潮→收尾）
  │
Step 5：用户审阅大纲
  └── 确认方向后再动笔
  │
Step 6：大纲审查
  └── 检查与原文的一致性问题
  │
Step 7：分段写作
  └── 按大纲逐段创作，保持风格一致
  │
Step 8：反思修订
  └── 自动评分（风格/人物/情节/文学性），不达标则修订
  │
Step 9：导出
  └── 输出最终文本 + 世界观更新 + 伏笔追踪
```

## 生成的文件

| 文件 | 说明 |
|------|------|
| `style-profile.md` | 原文风格分析档案 |
| `character-card.txt` | 人物档案 |
| `threads.md` | 情节线索追踪 |
| `world-facts.txt` | 世界观设定 |
| `outline-continuation.md` | 续写大纲 |
| `progress.md` | 写作进度 |
| `reflection-report.md` | 质量反思报告 |
| `checkpoint.md` | 续写检查点记录 |

## 依赖

- Claude Code 环境

## 致谢

本 skill 的设计参考了 [novel-writer](https://github.com/anthropics/skills/tree/main/skills/novel-writer) skill：

- **多维度分析框架**：参考 novel-writer 的风格分析、人物档案、世界观构建等多维度提取方法
- **大纲驱动写作**：参考 novel-writer 先规划后创作的工作流
- **质量自检机制**：参考 novel-writer 的反思修订模式，写完后自动评估并修订
- **模板系统**：参考 novel-writer 的模板化输出格式，确保输出结构一致

在 novel-writer 的基础上，novel-continuation 专注于**续写场景**，增加了：
- 原文风格深度分析与匹配
- 并行提取（4 个子 agent 同时工作提高效率）
- 大纲审查（检查与原文的一致性）
- 伏笔追踪与回收

## License

MIT
