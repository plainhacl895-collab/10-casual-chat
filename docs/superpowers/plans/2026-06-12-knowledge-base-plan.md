# 房产AI知识库搭建 实施计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 在 E 盘创建三层知识库目录结构 + 模块化 CLAUDE.md 配置 + 打通 Obsidian 和 VS Code Claude Code。

**Architecture:** 纯文件系统方案——Obsidian 和 VS Code Claude Code 共享 `e:\房产AI知识库\` 文件夹，文件夹本身就是桥接器。配置拆分为 6 个模块文件，各司其职。

**Tech Stack:** Obsidian（桌面端）、VS Code Claude Code 插件（已安装）、文件系统（Markdown）

---

### Task 1: 安装 Obsidian

**Files:** 无（下载安装程序）

- [ ] **Step 1: 下载 Obsidian 安装程序**

访问 https://obsidian.md/download 下载 Windows 版安装程序。

- [ ] **Step 2: 安装到 D 盘**

安装时选择目标路径 `D:\Obsidian\`，不占用 C 盘空间。

- [ ] **Step 3: 验证安装**

启动 Obsidian，确认能正常打开。

---

### Task 2: 创建文件夹结构

**Files:**
- Create: `e:\房产AI知识库\0-收集箱\临时收藏\`
- Create: `e:\房产AI知识库\0-收集箱\待整理\`
- Create: `e:\房产AI知识库\1-整理中\房产业务\房源产品\`
- Create: `e:\房产AI知识库\1-整理中\房产业务\谈判\`
- Create: `e:\房产AI知识库\1-整理中\房产业务\销售\`
- Create: `e:\房产AI知识库\1-整理中\AI技术\`
- Create: `e:\房产AI知识库\2-沉淀\`
- Create: `e:\房产AI知识库\_配置\`

- [ ] **Step 1: 创建所有目录**

```bash
mkdir -p "e:/房产AI知识库/0-收集箱/临时收藏"
mkdir -p "e:/房产AI知识库/0-收集箱/待整理"
mkdir -p "e:/房产AI知识库/1-整理中/房产业务/房源产品"
mkdir -p "e:/房产AI知识库/1-整理中/房产业务/谈判"
mkdir -p "e:/房产AI知识库/1-整理中/房产业务/销售"
mkdir -p "e:/房产AI知识库/1-整理中/AI技术"
mkdir -p "e:/房产AI知识库/2-沉淀"
mkdir -p "e:/房产AI知识库/_配置"
```

- [ ] **Step 2: 验证目录结构**

```bash
find "e:/房产AI知识库" -type d | sort
```

预期输出：所有 8 个目录均存在。

---

### Task 3: 写入 _配置/CLAUDE.md（主入口）

**Files:**
- Create: `e:\房产AI知识库\_配置\CLAUDE.md`

- [ ] **Step 1: 写入文件**

```bash
cat > "e:/房产AI知识库/_配置/CLAUDE.md" << 'CLAUDE'
# 房产AI知识库

请先阅读 `_配置/` 下的所有文件，了解我和我的知识库结构。
CLAUDE
```

- [ ] **Step 2: 验证**

```bash
cat "e:/房产AI知识库/_配置/CLAUDE.md"
```

---

### Task 4: 写入 _配置/profile.md

**Files:**
- Create: `e:\房产AI知识库\_配置\profile.md`

- [ ] **Step 1: 写入文件**

```bash
cat > "e:/房产AI知识库/_配置/profile.md" << 'PROFILE'
# 我是谁

- 一线房产经纪人，10年+从业经验
- 业务核心三块：房源产品、谈判、销售
- 同时自学 AI 技术，偏实操落地
- 学习时间：碎片时间为主，每天约1小时固定
PROFILE
```

- [ ] **Step 2: 验证**

```bash
cat "e:/房产AI知识库/_配置/profile.md"
```

---

### Task 5: 写入 _配置/rules.md

**Files:**
- Create: `e:\房产AI知识库\_配置\rules.md`

- [ ] **Step 1: 写入文件**

```bash
cat > "e:/房产AI知识库/_配置/rules.md" << 'RULES'
# 整理标准与笔记格式

## 房产业务整理维度
- 房源产品：关键参数（户型/面积/价格段）、卖点、适合客群、竞品对比
- 谈判：具体话术、对方出招→我方应对话术、心理博弈逻辑
- 销售：拓客渠道、跟进节奏、逼单时机、客户分类方法

## 笔记格式
- 顶部标注：来源、时间、标签
- 关键结论用 `###` 标题突出
- 我的思考用 `>` 引用块标注

## 不做的
- 不过度分类，阶段一保持简洁
- 不帮我写我没要的东西
- 不清楚的地方主动问
RULES
```

- [ ] **Step 2: 验证**

```bash
cat "e:/房产AI知识库/_配置/rules.md"
```

---

### Task 6: 写入 _配置/tasks.md

**Files:**
- Create: `e:\房产AI知识库\_配置\tasks.md`

- [ ] **Step 1: 写入文件**

```bash
cat > "e:/房产AI知识库/_配置/tasks.md" << 'TASKS'
# AI 任务

## 每日
1. 检查 `0-收集箱/` 是否有新内容，有就帮我：
   - 判断价值（高/中/低）
   - 提取核心观点（3句话以内）
   - 打标签（#房源产品 #谈判 #销售 #AI #其他）
   - 移到 `1-整理中/` 对应目录

## 每周日
生成"本周知识摘要"：
- 这周进了多少条信息
- 哪些值得再深入
- 推荐一个下周聚焦方向
TASKS
```

- [ ] **Step 2: 验证**

```bash
cat "e:/房产AI知识库/_配置/tasks.md"
```

---

### Task 7: 写入 _配置/标签映射.md

**Files:**
- Create: `e:\房产AI知识库\_配置\标签映射.md`

- [ ] **Step 1: 写入文件**

```bash
cat > "e:/房产AI知识库/_配置/标签映射.md" << 'TAGS'
# 标签 → 目录映射

| 标签 | 目标目录 |
|------|------|
| #房源产品 | `1-整理中/房产业务/房源产品/` |
| #谈判 | `1-整理中/房产业务/谈判/` |
| #销售 | `1-整理中/房产业务/销售/` |
| #AI | `1-整理中/AI技术/` |
| #其他 | `1-整理中/` |

> 新增业务线只需在此表加一行，不改任何规则文件。
TAGS
```

- [ ] **Step 2: 验证**

```bash
cat "e:/房产AI知识库/_配置/标签映射.md"
```

---

### Task 8: 写入 _配置/结构映射.md

**Files:**
- Create: `e:\房产AI知识库\_配置\结构映射.md`

- [ ] **Step 1: 写入文件**

```bash
cat > "e:/房产AI知识库/_配置/结构映射.md" << 'STRUCT'
# 当前完整文件夹结构

```
房产AI知识库/
├── 0-收集箱/
│   ├── 临时收藏/
│   └── 待整理/
├── 1-整理中/
│   ├── 房产业务/
│   │   ├── 房源产品/
│   │   ├── 谈判/
│   │   └── 销售/
│   └── AI技术/
├── 2-沉淀/
└── _配置/
    ├── CLAUDE.md
    ├── profile.md
    ├── rules.md
    ├── tasks.md
    ├── 标签映射.md
    └── 结构映射.md
```

> 此为阶段一初始结构，后续如有变更在此更新。
STRUCT
```

- [ ] **Step 2: 验证**

```bash
cat "e:/房产AI知识库/_配置/结构映射.md"
```

---

### Task 9: 连接工具

**验证清单：** Obsidian 打开知识库为 Vault，VS Code 打开知识库为工作区。

- [ ] **Step 1: Obsidian 打开知识库**

启动 Obsidian → 点击"Open folder as vault" → 选择 `e:\房产AI知识库\`

验证：Obsidian 左侧文件树显示 `0-收集箱`、`1-整理中`、`2-沉淀`、`_配置`

- [ ] **Step 2: VS Code 打开工作区**

VS Code → File → Open Folder → 选择 `e:\房产AI知识库\`

验证：VS Code 左侧文件树显示所有目录和文件。然后对 Claude Code 说"请阅读 _配置/ 下的文件，了解我和我的知识库"，确认 AI 能正确读取。

---

### Task 10: 写入 README.md（使用说明）

**Files:**
- Create: `e:\房产AI知识库\README.md`

- [ ] **Step 1: 写入文件**

```bash
cat > "e:/房产AI知识库/README.md" << 'README'
# 房产AI知识库

## 怎么用

1. **丢素材** — 碎片时间看到有用的，直接拖进 `0-收集箱/`
2. **让 AI 整理** — 在 VS Code 里打开这个文件夹，对 Claude Code 说：
   - "帮我整理收集箱里的新内容"
   - "帮我生成这周的知识摘要"
3. **浏览复习** — 在 Obsidian 里打开这个文件夹浏览整理好的笔记

## 工具

- Obsidian：浏览、编辑
- VS Code Claude Code：AI 整理、分析、输出
- 两者看同一个文件夹，文件夹就是桥
README
```

- [ ] **Step 2: 验证完整结构**

```bash
find "e:/房产AI知识库" -type f -o -type d | sort
```

预期：7 个目录 + 7 个文件（6 个配置 + 1 个 README）。

---

### Task 11: 提交到 git

**Files:** 所有新创建的文件

- [ ] **Step 1: 初始化 git 仓库并提交**

```bash
git -C "e:/房产AI知识库" init
git -C "e:/房产AI知识库" add -A
git -C "e:/房产AI知识库" commit -m "init: 房产AI知识库 - 阶段一极简三层结构"
```

- [ ] **Step 2: 验证提交**

```bash
git -C "e:/房产AI知识库" log --oneline
git -C "e:/房产AI知识库" status
```

预期：clean working tree，一个 commit。

---

### 完成检查

全部完成后验证：
1. ✅ 目录结构完整（7 个目录）
2. ✅ 所有 6 个配置文件内容正确
3. ✅ Obsidian 能打开并显示所有文件
4. ✅ VS Code Claude Code 能读取 `_配置/` 下文件
5. ✅ Git 仓库已初始化
