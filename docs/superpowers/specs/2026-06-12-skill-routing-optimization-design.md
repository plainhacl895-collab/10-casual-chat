---
title: Skill 路由系统优化设计
description: 对全局 19 个 Claude Code 技能进行路由优化，提出 6 大分类的分层结构及 description 重写规范
tags: [skill-routing, optimization, design, architecture]
---

# Skill 路由系统优化设计

参照视频 "Agent Skill过多？4招提升命中率" 的诊断框架，对全局 19 个 Claude Code 技能进行路由优化。

## 背景

当前 19 个技能 + 约 15 个 slash commands = 约 34 个路由选项，全部平铺。存在三组 description 重叠严重的技能对：
- `dispatching-parallel-agents` vs `subagent-driven-development`
- `requesting-code-review` vs `verification-before-completion`  
- `brainstorming` vs `writing-plans`

无分层结构，无负样本描述。

## 目标

1. 所有技能增加 `category` 字段，建立分层结构
2. 所有技能 description 改为"触发场景 + 负样本"格式
3. 改造 `skill-manager` 安装脚本，安装时自动分类

## 方案一：Skill Tree 分层结构

6 大分类，技能归属如下：

### 开发流程 (Process)
- `brainstorming` — 创造性工作入口，输出设计方案
- `writing-plans` — 有 spec 后，编码前生成实施计划
- `test-driven-development` — 写实现代码前
- `writing-skills` — 创建/编辑 skill

### 执行引擎 (Execution)
- `subagent-driven-development` — 顺序执行计划任务 + 两阶段 review
- `dispatching-parallel-agents` — 并行派发独立任务
- `executing-plans` — 独立 session 执行

### 质量保障 (Quality)
- `systematic-debugging` — 遇到 bug 的第一步
- `requesting-code-review` — 完成后请求外部审查
- `receiving-code-review` — 收到审查反馈后
- `verification-before-completion` — 声称完成前自检

### 工作流工具 (Workflow)
- `using-git-worktrees` — 需要隔离工作区时
- `finishing-a-development-branch` — 开发完成，决定合并方式
- `using-superpowers` — 每次会话启动

### 领域工具 (Domain)
- `douyin-extract` — 抖音链接
- `pdf` — PDF 操作
- `kaipan-voice-gen` — 开盘电话 TTS
- `info-cross-verify` — 小区信息验证

### 元管理 (Meta)
- `skill-manager` — 安装/搜索/管理技能

## 方案二：Description 重写规范

每个技能 frontmatter 新增 `category` 字段。description 格式：

```
触发于 <具体场景>。输出 <核心产物>。⛔ 不用于 <负样本场景>。
```

示例（brainstorming）：

```yaml
name: brainstorming
category: 开发流程
description: >
  触发于任何创造性工作（新功能、新组件、修改行为）前。探索意图与需求，输出设计方案。
  ⛔ 不用于纯提问、简单查询、已有明确方案的任务。
```

## 方案三：skill-manager 安装脚本改造

文件：`~/.claude/skills/skill-manager/scripts/install_skill.py`

改动点：
- `install_skill()` 成功后调用新函数 `prompt_category(skill_name, dest)`
- `prompt_category()` 展示 6 个分类列表，接受用户输入 1-6
- 读取 SKILL.md 的 YAML frontmatter，在 `name:` 下一行插入 `category: <选中分类>`
- 无 TTY 时跳过交互，输出检测命令提示

交互格式：

```
📂 请为新技能「<name>」选择分类：

  1. 开发流程    (brainstorming / writing-plans / TDD ...)
  2. 执行引擎    (subagent-driven / dispatching / executing-plans)
  3. 质量保障    (debugging / code-review / verification)
  4. 工作流工具  (worktrees / finishing-branch / superpowers)
  5. 领域工具    (douyin / pdf / kaipan / info-cross-verify)
  6. 元管理      (skill-manager 自身)

输入序号 (1-6): 
```

## 方案四：分类检测脚本（补充）

新增 `~/.claude/skills/skill-manager/scripts/check_categories.py`

- 扫描 `~/.claude/skills/` 下所有 SKILL.md
- 检查 frontmatter 是否包含 `category:` 字段
- 列出缺失分类的技能
- exit code = 缺失技能数

## 实施顺序

1. 改造 `install_skill.py`，增加分类交互
2. 新增 `check_categories.py`
3. 逐技能重写 description，添加 category
4. 全局验证

## 不做的

- 不修改 Claude Code 的技能加载机制（不受控）
- 不删除任何现有技能
- 不合并技能（分析后确认无真正重复）
