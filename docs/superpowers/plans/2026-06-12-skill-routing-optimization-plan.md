---
title: Skill 路由系统优化 实施计划
description: 为 19 个全局技能增加 category 分层字段，重写 description，改造 skill-manager 安装脚本自动分类
tags: [skill-routing, optimization, implementation-plan]
---

# Skill 路由系统优化 实施计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 为 19 个全局技能增加 category 分层字段 + 重写 description（触发场景 + 负样本格式）+ 改造 skill-manager 安装脚本自动分类。

**Architecture:** 所有改动限于 `~/.claude/skills/` 目录内。安装脚本增加交互式分类选择。每个 SKILL.md 只改 frontmatter，body 不改。

**Tech Stack:** Python 3（安装脚本）、YAML frontmatter（技能元数据）

---

### Task 1: 改造 install_skill.py — 增加分类交互

**Files:**
- Modify: `~/.claude/skills/skill-manager/scripts/install_skill.py`

CATEGORIES = {
    "1": ("开发流程", "brainstorming / writing-plans / TDD / writing-skills"),
    "2": ("执行引擎", "subagent-driven / dispatching / executing-plans"),
    "3": ("质量保障", "debugging / code-review / verification"),
    "4": ("工作流工具", "worktrees / finishing-branch / superpowers"),
    "5": ("领域工具", "douyin / pdf / kaipan / info-cross-verify"),
    "6": ("元管理", "skill-manager"),
}

- [ ] **Step 1: 在文件开头新增 CATEGORIES 常量和 prompt_category 函数**

在 `SKILLS_DIR` 定义之后，`ensure_cache()` 之前，插入以下代码：

```python
CATEGORIES = {
    "1": ("开发流程", "brainstorming / writing-plans / TDD / writing-skills"),
    "2": ("执行引擎", "subagent-driven / dispatching / executing-plans"),
    "3": ("质量保障", "debugging / code-review / verification"),
    "4": ("工作流工具", "worktrees / finishing-branch / superpowers"),
    "5": ("领域工具", "douyin / pdf / kaipan / info-cross-verify"),
    "6": ("元管理", "skill-manager 自身"),
}

def prompt_category(skill_name, dest):
    """安装后让用户选择分类，写入 SKILL.md frontmatter"""
    import re

    if not sys.stdin.isatty():
        print()
        print(f"⚠️  非交互式终端，跳过分类选择")
        print(f"   手动运行: python ~/.claude/skills/skill-manager/scripts/check_categories.py")
        return

    print()
    print(f"📂 请为新技能「{skill_name}」选择分类：")
    print()
    for key in sorted(CATEGORIES.keys(), key=int):
        name, desc = CATEGORIES[key]
        print(f"  {key}. {name}  ({desc})")
    print()

    while True:
        try:
            choice = input("输入序号 (1-6): ").strip()
            if choice in CATEGORIES:
                break
            print("  请输入 1-6 之间的数字")
        except (EOFError, KeyboardInterrupt):
            print("\n  已跳过分类选择")
            return

    category_name = CATEGORIES[choice][0]
    skill_md = os.path.join(dest, "SKILL.md")

    with open(skill_md, "r", encoding="utf-8") as f:
        content = f.read()

    # 在 name: 行后插入 category:
    new_content = re.sub(
        r"(^name:\s*.+$)",
        rf"\1\ncategory: {category_name}",
        content,
        count=1,
        flags=re.MULTILINE,
    )

    with open(skill_md, "w", encoding="utf-8") as f:
        f.write(new_content)

    print()
    print(f"✅ 已将「{skill_name}」归类到「{category_name}」")
```

- [ ] **Step 2: 在 install_skill() 返回前调用 prompt_category**

在 `install_skill()` 函数的 `return True` 之前（打印安装成功之后），插入一行：

```python
    prompt_category(actual_name, dest)

    return True
```

- [ ] **Step 3: 验证脚本语法**

```bash
python -c "import sys; sys.path.insert(0, os.path.expanduser('~/.claude/skills/skill-manager/scripts')); compile(open(os.path.expanduser('~/.claude/skills/skill-manager/scripts/install_skill.py')).read(), 'install_skill.py', 'exec'); print('OK')"
```

Expected: `OK`

- [ ] **Step 4: Commit**

```bash
cd ~/.claude/skills/skill-manager/scripts
git add install_skill.py
git commit -m "feat: add interactive category selection after skill install"
```

---

### Task 2: 新增 check_categories.py — 分类完整性检测脚本

**Files:**
- Create: `~/.claude/skills/skill-manager/scripts/check_categories.py`

- [ ] **Step 1: 编写检测脚本**

```python
#!/usr/bin/env python3
"""扫描 ~/.claude/skills/ 下所有 SKILL.md，检测缺失 category 字段的技能"""
import os
import sys
import re
from pathlib import Path

SKILLS_DIR = os.path.expanduser("~/.claude/skills")
CATEGORY_ORDER = ["开发流程", "执行引擎", "质量保障", "工作流工具", "领域工具", "元管理"]

def check_skills():
    missing = []
    found = {}

    for skill_dir in sorted(Path(SKILLS_DIR).iterdir()):
        if not skill_dir.is_dir():
            continue
        skill_md = skill_dir / "SKILL.md"
        if not skill_md.exists():
            continue

        content = skill_md.read_text(encoding="utf-8")
        name_match = re.search(r"^name:\s*(.+)$", content, re.MULTILINE)
        category_match = re.search(r"^category:\s*(.+)$", content, re.MULTILINE)

        name = name_match.group(1).strip() if name_match else skill_dir.name

        if category_match:
            cat = category_match.group(1).strip()
            found.setdefault(cat, []).append(name)
        else:
            missing.append(name)

    has_issues = False

    # 按分类展示已分类技能
    print("=" * 60)
    print("📊 技能分类统计")
    print()
    for cat in CATEGORY_ORDER:
        skills = found.get(cat, [])
        print(f"  {cat} ({len(skills)}): {', '.join(skills) if skills else '(空)'}")
    print()

    uncategorized = [s for s in found if s not in CATEGORY_ORDER]
    if uncategorized:
        has_issues = True
        print(f"⚠️  未识别的分类: {', '.join(uncategorized)}")

    if missing:
        has_issues = True
        print(f"❌ 缺失 category ({len(missing)}):")
        for s in missing:
            print(f"     - {s}")
    else:
        print("✅ 所有技能已分类")

    return has_issues

if __name__ == "__main__":
    has_issues = check_skills()
    sys.exit(1 if has_issues else 0)
```

- [ ] **Step 2: 验证脚本可运行**

```bash
python ~/.claude/skills/skill-manager/scripts/check_categories.py
```

Expected: 当前列出所有技能缺失 category（因为还没改 description）

- [ ] **Step 3: Commit**

```bash
cd ~/.claude/skills/skill-manager/scripts
git add check_categories.py
git commit -m "feat: add category completeness checker script"
```

---

### Task 3: 重写 开发流程 类技能描述（4个）

**Files:**
- Modify: `~/.claude/skills/brainstorming/SKILL.md:1-4`
- Modify: `~/.claude/skills/writing-plans/SKILL.md:1-4`
- Modify: `~/.claude/skills/test-driven-development/SKILL.md:1-4`
- Modify: `~/.claude/skills/writing-skills/SKILL.md:1-4`

- [ ] **Step 1: 修改 brainstorming/SKILL.md frontmatter**

将现有 frontmatter：
```yaml
---
name: brainstorming
description: "You MUST use this before any creative work - creating features, building components, adding functionality, or modifying behavior. Explores user intent, requirements and design before implementation."
---
```

替换为：
```yaml
---
name: brainstorming
category: 开发流程
description: >
  触发于任何创造性工作（新功能、新组件、修改行为）前。探索意图与需求，输出设计方案。
  ⛔ 不用于纯提问、简单查询、已有明确方案的任务。
---
```

- [ ] **Step 2: 修改 writing-plans/SKILL.md frontmatter**

将现有 frontmatter：
```yaml
---
name: writing-plans
description: Use when you have a spec or requirements for a multi-step task, before touching code
---
```

替换为：
```yaml
---
name: writing-plans
category: 开发流程
description: >
  触发于有设计 spec 后、编码前，需要生成多步骤实施计划时。输出 bite-sized 任务清单和计划文档。
  ⛔ 不用于单文件小改动、无 spec 的探索性开发。← 跟在 brainstorming 之后。
---
```

- [ ] **Step 3: 修改 test-driven-development/SKILL.md frontmatter**

将现有 frontmatter：
```yaml
---
name: test-driven-development
description: Use when implementing any feature or bugfix, before writing implementation code
---
```

替换为：
```yaml
---
name: test-driven-development
category: 开发流程
description: >
  触发于实现任何功能或修复 bug 时，写实现代码之前。先写测试 → 观察失败 → 写最小实现。
  ⛔ 不用于一次性原型、自动生成的代码、纯配置文件。
---
```

- [ ] **Step 4: 修改 writing-skills/SKILL.md frontmatter**

将现有 frontmatter：
```yaml
---
name: writing-skills
description: Use when creating new skills, editing existing skills, or verifying skills work before deployment
---
```

替换为：
```yaml
---
name: writing-skills
category: 开发流程
description: >
  触发于创建新 skill、编辑现有 skill、或部署前验证 skill 时。用 TDD 方法验证 skill 有效性。
  ⛔ 不用于使用 skill（用其他技能干活），不用于修改非 skill 的普通代码。
---
```

- [ ] **Step 5: 验证** 

```bash
python ~/.claude/skills/skill-manager/scripts/check_categories.py
```

Expected: 开发流程 (4) 已列出，缺失数减少

- [ ] **Step 6: Commit**

```bash
cd ~/.claude/skills
git add brainstorming/SKILL.md writing-plans/SKILL.md test-driven-development/SKILL.md writing-skills/SKILL.md
git commit -m "refactor: add category and rewrite descriptions for Process skills"
```

---

### Task 4: 重写 执行引擎 类技能描述（3个）

**Files:**
- Modify: `~/.claude/skills/subagent-driven-development/SKILL.md:1-4`
- Modify: `~/.claude/skills/dispatching-parallel-agents/SKILL.md:1-4`
- Modify: `~/.claude/skills/executing-plans/SKILL.md:1-4`

- [ ] **Step 1: 修改 subagent-driven-development/SKILL.md frontmatter**

将现有 frontmatter：
```yaml
---
name: subagent-driven-development
description: Use when executing implementation plans with independent tasks in the current session
---
```

替换为：
```yaml
---
name: subagent-driven-development
category: 执行引擎
description: >
  触发于有书面实施计划、任务间独立、想在当前 session 顺序执行时。每任务独立 subagent + 两阶段 review。
  ⛔ 不用于并行独立任务（用 dispatching-parallel-agents）、跨 session 执行（用 executing-plans）。
---
```

- [ ] **Step 2: 修改 dispatching-parallel-agents/SKILL.md frontmatter**

将现有 frontmatter：
```yaml
---
name: dispatching-parallel-agents
description: Use when facing 2+ independent tasks that can be worked on without shared state or sequential dependencies
---
```

替换为：
```yaml
---
name: dispatching-parallel-agents
category: 执行引擎
description: >
  触发于有 2+ 个完全独立的任务（不同子系统、不同 bug），可同时执行时。并行派发 subagent 各管各的。
  ⛔ 不用于顺序执行计划任务（用 subagent-driven-development）、任务间有依赖关系。
---
```

- [ ] **Step 3: 修改 executing-plans/SKILL.md frontmatter**

将现有 frontmatter：
```yaml
---
name: executing-plans
description: Use when you have a written implementation plan to execute in a separate session with review checkpoints
---
```

替换为：
```yaml
---
name: executing-plans
category: 执行引擎
description: >
  触发于有书面实施计划、需要在独立 session 中分批执行时。加载计划 → review → 逐个执行 → 汇报。
  ⛔ 不用于当前 session 内执行（用 subagent-driven-development）。
---
```

- [ ] **Step 4: 验证**

```bash
python ~/.claude/skills/skill-manager/scripts/check_categories.py
```

- [ ] **Step 5: Commit**

```bash
cd ~/.claude/skills
git add subagent-driven-development/SKILL.md dispatching-parallel-agents/SKILL.md executing-plans/SKILL.md
git commit -m "refactor: add category and rewrite descriptions for Execution skills"
```

---

### Task 5: 重写 质量保障 类技能描述（4个）

**Files:**
- Modify: `~/.claude/skills/systematic-debugging/SKILL.md:1-4`
- Modify: `~/.claude/skills/requesting-code-review/SKILL.md:1-4`
- Modify: `~/.claude/skills/receiving-code-review/SKILL.md:1-4`
- Modify: `~/.claude/skills/verification-before-completion/SKILL.md:1-4`

- [ ] **Step 1: 修改 systematic-debugging/SKILL.md frontmatter**

将现有 frontmatter：
```yaml
---
name: systematic-debugging
description: Use when encountering any bug, test failure, or unexpected behavior, before proposing fixes
---
```

替换为：
```yaml
---
name: systematic-debugging
category: 质量保障
description: >
  触发于遇到任何 bug、测试失败、异常行为时，提修复方案之前。输出根因分析，禁止跳过根因直接修。
  ⛔ 不用于症状修复、猜测性改代码。修复必须在根因定位之后。
---
```

- [ ] **Step 2: 修改 requesting-code-review/SKILL.md frontmatter**

将现有 frontmatter：
```yaml
---
name: requesting-code-review
description: Use when completing tasks, implementing major features, or before merging to verify work meets requirements
---
```

替换为：
```yaml
---
name: requesting-code-review
category: 质量保障
description: >
  触发于完成重要功能或合并前，需要外部代码审查时。派发 code-reviewer subagent 获取审查意见。
  ⛔ 不用于自我验证（用 verification-before-completion）、小改动确认。
---
```

- [ ] **Step 3: 修改 receiving-code-review/SKILL.md frontmatter**

将现有 frontmatter：
```yaml
---
name: receiving-code-review
description: Use when receiving code review feedback, before implementing suggestions, especially if feedback seems unclear or technically questionable - requires technical rigor and verification, not performative agreement or blind implementation
---
```

替换为：
```yaml
---
name: receiving-code-review
category: 质量保障
description: >
  触发于收到代码审查反馈后、实施修改建议前。逐条技术评估 → 验证 → 实施，禁止盲目接受。
  ⛔ 不用于自我审查、提出审查请求（用 requesting-code-review）。
---
```

- [ ] **Step 4: 修改 verification-before-completion/SKILL.md frontmatter**

将现有 frontmatter：
```yaml
---
name: verification-before-completion
description: Use when about to claim work is complete, fixed, or passing, before committing or creating PRs - requires running verification commands and confirming output before making any success claims; evidence before assertions always
---
```

替换为：
```yaml
---
name: verification-before-completion
category: 质量保障
description: >
  触发于声称工作完成、修复完成、测试通过之前。运行验证命令 + 展示输出证据，禁止凭记忆断言。
  ⛔ 不用于外部审查（用 requesting-code-review）、没有可验证标准的情况。
---
```

- [ ] **Step 5: 验证**

```bash
python ~/.claude/skills/skill-manager/scripts/check_categories.py
```

- [ ] **Step 6: Commit**

```bash
cd ~/.claude/skills
git add systematic-debugging/SKILL.md requesting-code-review/SKILL.md receiving-code-review/SKILL.md verification-before-completion/SKILL.md
git commit -m "refactor: add category and rewrite descriptions for Quality skills"
```

---

### Task 6: 重写 工作流工具 类技能描述（3个）

**Files:**
- Modify: `~/.claude/skills/using-git-worktrees/SKILL.md:1-4`
- Modify: `~/.claude/skills/finishing-a-development-branch/SKILL.md:1-4`
- Modify: `~/.claude/skills/using-superpowers/SKILL.md:1-4`

- [ ] **Step 1: 修改 using-git-worktrees/SKILL.md frontmatter**

将现有 frontmatter：
```yaml
---
name: using-git-worktrees
description: Use when starting feature work that needs isolation from current workspace or before executing implementation plans - ensures an isolated workspace exists via native tools or git worktree fallback
---
```

替换为：
```yaml
---
name: using-git-worktrees
category: 工作流工具
description: >
  触发于需要隔离工作区开始新功能、或在执行实施计划前。检测现有隔离 → 创建或复用工作区。
  ⛔ 不用于已在隔离环境中时、不需要隔离的单文件小改动。
---
```

- [ ] **Step 2: 修改 finishing-a-development-branch/SKILL.md frontmatter**

将现有 frontmatter：
```yaml
---
name: finishing-a-development-branch
description: Use when implementation is complete, all tests pass, and you need to decide how to integrate the work - guides completion of development work by presenting structured options for merge, PR, or cleanup
---
```

替换为：
```yaml
---
name: finishing-a-development-branch
category: 工作流工具
description: >
  触发于所有任务完成、测试通过后，需要决定如何收尾时。验证测试 → 展示合并/PR/清理选项 → 执行。
  ⛔ 不用于实现未完成时、测试未通过时。
---
```

- [ ] **Step 3: 修改 using-superpowers/SKILL.md frontmatter**

将现有 frontmatter：
```yaml
---
name: using-superpowers
description: Use when starting any conversation - establishes how to find and use skills, requiring Skill tool invocation before ANY response including clarifying questions
---
```

替换为：
```yaml
---
name: using-superpowers
category: 工作流工具
description: >
  触发于每次会话开始。确保 skill 查找和使用规范生效，强制 Skill 工具调用优先于任何回复。
  ⛔ 不用于 subagent 内部调用（subagent 应跳过此 skill）。
---
```

- [ ] **Step 4: 验证**

```bash
python ~/.claude/skills/skill-manager/scripts/check_categories.py
```

- [ ] **Step 5: Commit**

```bash
cd ~/.claude/skills
git add using-git-worktrees/SKILL.md finishing-a-development-branch/SKILL.md using-superpowers/SKILL.md
git commit -m "refactor: add category and rewrite descriptions for Workflow skills"
```

---

### Task 7: 重写 领域工具 类技能描述（4个）

**Files:**
- Modify: `~/.claude/skills/douyin-extract/SKILL.md:1-4`
- Modify: `~/.claude/skills/pdf/SKILL.md:1-6`
- Modify: `~/.claude/skills/kaipan-voice-gen/SKILL.md:1-5`
- Modify: `~/.claude/skills/info-cross-verify/SKILL.md:1-5`

注：`pdf`、`kaipan-voice-gen`、`info-cross-verify` 的 description 使用多行 YAML 格式（`>` 或原始值），替换时需保持格式一致。

- [ ] **Step 1: 修改 douyin-extract/SKILL.md frontmatter**

将现有 frontmatter：
```yaml
---
name: douyin-extract
description: 提取抖音视频的完整信息——视频描述、作者数据、互动指标 + ASR 语音转文字获取口播文案 + 评论抓取
---
```

替换为：
```yaml
---
name: douyin-extract
category: 领域工具
description: >
  触发于用户提供抖音链接时。提取视频描述、作者信息、互动数据，支持 ASR 语音转口播文案全文和评论抓取。
  ⛔ 不用于非抖音平台的链接、私密视频。
---
```

- [ ] **Step 2: 修改 pdf/SKILL.md frontmatter**

将现有 frontmatter：
```yaml
---
name: pdf
description: Comprehensive PDF manipulation toolkit for extracting text and tables, creating new PDFs, merging/splitting documents, and handling forms. When Claude needs to fill in a PDF form or programmatically process, generate, or analyze PDF documents at scale.
license: Proprietary. LICENSE.txt has complete terms
---
```

替换为：
```yaml
---
name: pdf
category: 领域工具
description: >
  触发于 PDF 操作——提取文字/表格、创建/合并/拆分文档、填写表单。支持 pypdf 和命令行工具。
  ⛔ 不用于非 PDF 文档格式（Word、Excel）。
license: Proprietary. LICENSE.txt has complete terms
---
```

- [ ] **Step 3: 修改 kaipan-voice-gen/SKILL.md frontmatter**

将现有 frontmatter：
```yaml
---
name: kaipan-voice-gen
description: >
  开盘电话 TTS 语音批量生成。使用 ChatTTS 本地生成，确保同一批次音色一致、数字正确发音。
  触发场景：(1) 生成/更新小区语音包 (2) 修改话术文本后重新生成 (3) 用户反馈语音问题需要修复 (4) 新增小区话术。
  核心原则：一个批次一个音色，阿拉伯数字必须转中文数字，"吗"字不能丢。
---
```

替换为：
```yaml
---
name: kaipan-voice-gen
category: 领域工具
description: >
  触发于生成/更新开盘电话 TTS 语音、修改话术文本后重新生成。用 ChatTTS 本地批量生成，统一批次音色一致。
  ⛔ 不用于非开盘话术的通用 TTS 需求。
---
```

- [ ] **Step 4: 修改 info-cross-verify/SKILL.md frontmatter**

将现有 frontmatter：
```yaml
---
name: info-cross-verify
description: >
  信息交叉验证技能。在搜集和汇总上海小区信息时，对所有"可能对客户产生影响的
  事实性描述"进行多源交叉验证，区分信息来源可信度，剥离营销语言，标注不确定信息。
  触发场景：(1) 进行小区深度分析的信息采集阶段 (2) 搜集竞品对比数据时
  (3) 用户要求验证某条具体信息的真实性时 (4) 任何需要将网络信息写入分析报告的场景。
  核心原则：不确定的事宁可不说，不可编造。经纪人点评不可作为独立信息源。
---
```

替换为：
```yaml
---
name: info-cross-verify
category: 领域工具
description: >
  触发于搜集上海小区信息时，对所有可能影响客户的事实性描述进行多源交叉验证。区分信源可信度，剥离营销语言，标注不确定信息。
  ⛔ 不用于单源引用、经纪人点评作为独立信源。
---
```

- [ ] **Step 5: 验证**

```bash
python ~/.claude/skills/skill-manager/scripts/check_categories.py
```

- [ ] **Step 6: Commit**

```bash
cd ~/.claude/skills
git add douyin-extract/SKILL.md pdf/SKILL.md kaipan-voice-gen/SKILL.md info-cross-verify/SKILL.md
git commit -m "refactor: add category and rewrite descriptions for Domain skills"
```

---

### Task 8: 重写 元管理 类技能描述（1个）

**Files:**
- Modify: `~/.claude/skills/skill-manager/SKILL.md:1-4`

- [ ] **Step 1: 修改 skill-manager/SKILL.md frontmatter**

将现有 frontmatter：
```yaml
---
name: skill-manager
description: Anthropic 官方技能本地安装管家。绕过 /plugin install 市场限制，从 GitHub 开源仓库按需下载、更新、卸载官方技能。触发词："安装 XX 技能""下载 XX skill""装个 PDF 技能""更新技能""列出所有技能""有哪些官方技能""搜索 XX 技能""有没有 XX 方面的技能""我需要处理 XX 该用什么技能""推荐技能""删除 XX 技能""卸载 XX 技能""技能商店""技能分类"。
---
```

替换为：
```yaml
---
name: skill-manager
category: 元管理
description: >
  触发于安装/搜索/更新/卸载技能时。从 GitHub 开源仓库按需管理 Anthropic 官方技能，安装时自动提示选择分类。
  ⛔ 不用于使用技能干活（用对应的领域技能）。
---
```

- [ ] **Step 2: 验证**

```bash
python ~/.claude/skills/skill-manager/scripts/check_categories.py
```

Expected:
```
📊 技能分类统计

  开发流程 (4): brainstorming, writing-plans, test-driven-development, writing-skills
  执行引擎 (3): subagent-driven-development, dispatching-parallel-agents, executing-plans
  质量保障 (4): systematic-debugging, requesting-code-review, receiving-code-review, verification-before-completion
  工作流工具 (3): using-git-worktrees, finishing-a-development-branch, using-superpowers
  领域工具 (4): douyin-extract, pdf, kaipan-voice-gen, info-cross-verify
  元管理 (1): skill-manager

✅ 所有技能已分类
```

- [ ] **Step 3: Commit**

```bash
cd ~/.claude/skills
git add skill-manager/SKILL.md
git commit -m "refactor: add category and rewrite description for Meta skill"
```

---

### Task 9: 全局验证

- [ ] **Step 1: 运行 check_categories.py 确认全部通过**

```bash
python ~/.claude/skills/skill-manager/scripts/check_categories.py
```

Expected: exit code 0，全部 19 个技能已分类

- [ ] **Step 2: 逐一验证 YAML frontmatter 可解析**

```bash
python -c "
import os, re
skills_dir = os.path.expanduser('~/.claude/skills')
for d in sorted(os.listdir(skills_dir)):
    md = os.path.join(skills_dir, d, 'SKILL.md')
    if not os.path.exists(md): continue
    with open(md) as f:
        content = f.read()
    if not content.startswith('---'):
        print(f'FAIL (no frontmatter): {d}')
        continue
    # extract frontmatter
    m = re.match(r'^---\n(.*?)\n---', content, re.DOTALL)
    if not m:
        print(f'FAIL (malformed): {d}')
        continue
    fm = m.group(1)
    has_name = 'name:' in fm
    has_cat = 'category:' in fm
    has_desc = 'description:' in fm
    if has_name and has_cat and has_desc:
        print(f'OK: {d}')
    else:
        print(f'FAIL (missing fields): {d}  name={has_name} cat={has_cat} desc={has_desc}')
print('Done')
"
```

Expected: 19 `OK:` 行，无 FAIL

- [ ] **Step 3: 提交最终验证提交**

```bash
cd ~/.claude/skills
git add -A
git diff --cached --stat
git commit -m "verify: all 19 skills categorized and descriptions rewritten"
```

