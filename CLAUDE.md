# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目定位

cheat-on-content 是一个 **Claude Code 方法论 Skill 系统**（v0.1.0），不是传统软件项目。它将内容创作变为可校准预测循环：**打分 → 盲预测 → 发布 → T+3d 复盘 → 进化 rubric**。13 个子 skill 通过 `SKILL.md` 主文件路由，安装后以 `/cheat-*` 斜杠命令方式在 Claude Code 中调用。

## 核心原则（不可妥协）

1. **盲预测 immutable** — 预测必须在见数据前写完，`## 预测` 段一旦写入禁止修改，只能往 `## 复盘` 段追加。harness 层由 `hooks/prediction-immutability.sh` 强制执行。
2. **Bump = 全量重打** — rubric 升级必须用新公式重打所有有实绩的校准样本，≥4/5 排序一致才通过，且需跨模型独立审核。
3. **Rubric 是工作台不是博物馆** — 被推翻或已吸收的观察直接删除，不留考古层，git history 才是档案。

## 安装与卸载

```bash
# 安装（symlink 子 skill 到 ~/.claude/skills/）
bash install.sh

# 卸载
bash uninstall.sh

# 仅重装 hooks（git pull 后）
bash install.sh --reinstall-hooks <目标项目目录>
```

## 架构概览

```
SKILL.md                        ← 总协议 + 路由器（触发词 → 子 skill 映射）
├── skills/cheat-*/SKILL.md     ← 13 个子 skill（每个一个文件）
│   ├── cheat-init              ← 首次 onboarding + 脚手架创建
│   ├── cheat-learn-from        ← 对标账号导入
│   ├── cheat-seed              ← 选题 brainstorm
│   ├── cheat-score             ← 单稿打分（只输出，不写文件）
│   ├── cheat-predict           ← 核心：打分 + 写 immutable 预测
│   ├── cheat-shoot             ← 拍摄登记（buffer +1）
│   ├── cheat-publish           ← 发布登记（buffer -1）
│   ├── cheat-retro             ← T+3d 复盘
│   ├── cheat-bump              ← rubric 升级（含跨模型审计）
│   ├── cheat-recommend         ← 候选池排序推荐
│   ├── cheat-trends            ← 热点抓取
│   ├── cheat-status            ← 状态看板
│   └── cheat-migrate           ← schema 版本迁移
├── shared-references/          ← 跨 skill 协议定义（所有 skill 引用）
│   ├── blind-prediction-protocol.md
│   ├── bump-validation-protocol.md
│   ├── cadence-protocol.md     ← buffer 警戒系统
│   ├── candidate-schema.md
│   ├── migration-protocol.md
│   ├── observation-lifecycle.md
│   ├── prediction-anatomy.md
│   └── state-management.md     ← .cheat-state.json 完整 schema
├── templates/                  ← cheat-init 写入用户项目的模板（10 个文件）
├── hooks/                      ← Claude Code harness 钩子
│   ├── prediction-immutability.sh/.json  ← PreToolUse 拦截（阻塞写预测段）
│   ├── session-start.sh/.json            ← 进会话自动渲染状态看板
│   └── log-event.sh/.json                ← 被动使用日志记录
├── migrations/                 ← schema 版本迁移
│   ├── registry.md             ← LATEST_SCHEMA 标记 + 版本链
│   └── 1.0-to-1.1.md           ← 迁移步
├── adapters/                   ← 数据源适配器
│   ├── perf-data/douyin-session/  ← 抖音数据爬虫（Python/Playwright）
│   ├── script-extraction/whisper/ ← Whisper 语音转文字
│   └── trend-sources/             ← 热点源定义（weibo-hot, zhihu-hot）
├── starter-rubrics/            ← 预置 rubric
│   ├── opinion-video-zero.md   ← v0 等权重占位版（冷启动用）
│   └── opinion-video.md        ← v2 校准版（25+ 样本拟合）
└── tools/score-curve.py        ← 预测准确率收敛曲线图（matplotlib）
```

## 关键约定

### 状态文件
用户项目根目录下的 `.cheat-state.json` 是单一状态来源（43 字段），**绝不能**放在全局 `~/.claude/` 或本 repo 内。schema 版本管理通过 `migrations/` 系统执行 `/cheat-migrate`。

### 文件模板
`templates/` 下的文件是 `cheat-init` 写入用户项目的骨架。修改后需同步验证 `cheat-init` 的写入逻辑。

### Hooks 开发
三个 hook 脚本各有一对 `.sh`（执行逻辑）+ `.json`（hook 声明），放在 `hooks/` 目录。`install.sh` 负责部署到用户项目的 `.cheat-hooks/`。新增 hook 要：写 `.sh` → 写 `.json` → 更新 `install.sh`。

### 路由表
向 `SKILL.md` 的路由表（第 38-55 行）追加新的触发词，而不是修改现有的。拒绝请求列表（第 66-77 行）也在此文件维护。

### 开发新增子 skill
1. 在 `skills/cheat-<name>/SKILL.md` 写 skill（必须包含：前置条件、执行步骤、Refusals 段）
2. 在 `SKILL.md` 路由表加行
3. 在 `install.sh` 的 `SKILLS` 列表加条目
4. 在 `uninstall.sh` 同步

## 工作流速查

```
cold-start: init → learn-from（可选）→ seed → [score → predict → shoot → publish → retro] 循环
calibration: [predict → shoot → publish → retro] 循环 → bump → [predict ...]
```
