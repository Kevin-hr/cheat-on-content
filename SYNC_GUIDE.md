# 跨设备同步指南

## 架构

```
C:\Users\XXX\Documents\GitHub\
├── cheat-on-content/     ← Skill 系统源码（git 管理，只跟踪技能代码）
│   ├── .git/
│   ├── skills/
│   ├── install.sh
│   └── ...
│
└── my-channel/           ← 你的内容项目（git 管理，跟踪视频物料和数据）
    ├── .git/
    ├── videos/           ← 视频交付物
    ├── predictions/      ← 预测日志
    ├── scripts/          ← 创作脚本
    └── project_memory.md ← 项目记忆
```

**核心原则**：`cheat-on-content` 与 `my-channel` 是两个独立 git 仓库，互不污染。

---

## 新电脑首次配置（从零开始）

```powershell
# === 步骤 1：创建目录 ===
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\Documents\GitHub"
cd "$env:USERPROFILE\Documents\GitHub"

# === 步骤 2：克隆 skill 源码（你的 fork）===
git clone https://github.com/Kevin-hr/cheat-on-content.git
cd cheat-on-content

# 添加官方上游（用于同步最新 skill 更新）
git remote add upstream https://github.com/XBuilderLAB/cheat-on-content.git

# === 步骤 3：安装 skill 到 Claude Code ===
bash install.sh
# 或 Windows 下用 Git Bash 运行
# "C:\Program Files\Git\bin\bash.exe" install.sh

# === 步骤 4：克隆你的内容项目 ====
cd ..
git clone https://github.com/Kevin-hr/my-channel.git
cd my-channel

# === 步骤 5：测试 skill 是否生效 ====
# 在 my-channel 目录打开 Claude Code → 说 "状态"
# cheat-status 应该能读取项目文件
```

---

## 每天开工的标准流程

```powershell
# === 1. 同步 skill 源码（获取最新官方更新）===
cd "$env:USERPROFILE\Documents\GitHub\cheat-on-content"
git pull origin main
git pull upstream main          # 拉取官方最新

# 如果有冲突，优先保留本地 skill 修改
# 解决冲突后：
#   git add .
#   git commit -m "merge: upstream sync"
#   git push origin main

# === 2. 同步内容项目 ===
cd "$env:USERPROFILE\Documents\GitHub\my-channel"
git pull origin main

# === 3. 检查状态 ====
# 在 my-channel 目录打开 Claude Code → 说 "状态"
```

---

## 有本地修改时合并远程更新

```powershell
# 情况 A：本地有未提交的修改，需要拉取远程
cd "$env:USERPROFILE\Documents\GitHub\my-channel"

# 暂存本地修改
git stash

# 拉取远程最新
git pull origin main

# 恢复本地修改（自动合并）
git stash pop

# 如果 stash pop 有冲突 → 手动解决冲突文件，然后：
git add .
git stash drop  # 清除 stash
git commit -m "merge: resolve conflicts with remote"
git push origin main
```

```powershell
# 情况 B：本地已有新提交，远程也有新提交
cd "$env:USERPROFILE\Documents\GitHub\my-channel"

# 先拉取
git pull origin main

# 如果有冲突 → 解决冲突文件，然后：
git add .
git commit -m "merge: resolve conflicts"
git push origin main
```

---

## 收工前提交

```powershell
# === 提交内容项目 ===
cd "$env:USERPROFILE\Documents\GitHub\my-channel"

git add .
git commit -m "feat: 当日工作内容摘要"
git push origin main

# === 同步 skill 源码（如有本地修改）===
cd "$env:USERPROFILE\Documents\GitHub\cheat-on-content"

git status
# 如有修改：
git add .
git commit -m "fix: 描述你的修改"
git push origin main
```

---

## 在两台电脑间切换

| 操作 | 电脑 A（收工） | 电脑 B（开工） |
|------|----------------|----------------|
| 内容项目 | `cd my-channel && git push` | `cd my-channel && git pull` |
| Skill 源码 | `cd cheat-on-content && git push` | `cd cheat-on-content && git pull` |

---

## 常见问题

### Q: `.gitignore` 忽略了 videos/ 怎么办？
**A**: 这是故意设计。`videos/` 只在 `my-channel` 仓库中被跟踪，不在 `cheat-on-content` 中被跟踪。

### Q: 文件太大（视频 MP4）无法 push？
**A**: 如果 GitHub 拒绝大文件（>100MB），安装 Git LFS：
```powershell
cd "$env:USERPROFILE\Documents\GitHub\my-channel"
git lfs install
git lfs track "*.mp4"
git add .gitattributes
git commit -m "chore: enable Git LFS for video files"
git push origin main
```

### Q: push 时提示认证失败？
**A**: 使用 GitHub Personal Access Token：
```powershell
git remote set-url origin https://你的用户名:你的Token@github.com/Kevin-hr/my-channel.git
```

---

*最后更新: 2026-06-28*
