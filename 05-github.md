# 第 5 章：GitHub 入门

> 本地 Git 玩熟之后，把仓库放到 GitHub 上：备份、分享、协作。本章覆盖"本地已有项目推上去"和"把别人项目拉下来"两条路线，以及 SSH 免密配置。

## 5.1 注册账号与概念回顾

1. 打开 [github.com](https://github.com) → Sign up
2. 用户名建议用真名或常用英文 ID（会出现在你的项目 URL 里：`github.com/用户名/仓库名`）
3. 邮箱用第 1 章配置 `git config user.email` 时填的那个——**必须一致**，提交记录才能关联到你的账号

```
本地仓库（你的电脑）          远程仓库（GitHub 云端）
┌──────────────┐   push →   ┌──────────────┐
│ 工作区/历史   │ ─────────▶ │ 历史 + 网页   │
│              │  ← pull    │ 界面 + 协作   │
└──────────────┘            └──────────────┘
```

## 5.2 路线一：本地已有项目 → 推送到 GitHub

### 第 1 步：在 GitHub 上创建空仓库

GitHub 右上角 **+** → New repository：

- Repository name：`git-practice`
- Public（公开，免费）/ Private（私有，仅自己可见）
- ⚠️ **不要勾选** "Add a README file"、"Add .gitignore"、"Choose a license"——因为你本地已有代码，勾了会产生历史冲突

创建后会看到关键信息：

```
…or push an existing repository from the command line
git remote add origin https://github.com/你的用户名/git-practice.git
git branch -M main
git push -u origin main
```

### 第 2 步：本地关联并推送

```bash
# 在本地仓库目录里执行（注意把 URL 换成你自己的）

git remote add origin https://github.com/你的用户名/git-practice.git
# remote add：告诉本地仓库"远程仓库在哪里"，起个名字叫 origin（惯例）

git branch -M main
# 确保本地分支名叫 main（与 GitHub 默认一致；老仓库可能叫 master）

git push -u origin main
# push：把本地 main 分支的提交推送到远程
# -u：把 origin/main 设为上游，之后只需敲 git push / git pull
```

成功后刷新 GitHub 页面，就能看到你的代码了 🎉

### remote 相关命令

```bash
git remote -v          # 查看已关联的远程仓库
# origin  https://github.com/用户名/git-practice.git (fetch)
# origin  https://github.com/用户名/git-practice.git (push)

git remote add 名字 URL    # 添加远程（名字惯例叫 origin；fork 工作流会加 upstream，见第 6 章）
git remote remove 名字     # 解除关联
```

## 5.3 路线二：把 GitHub 上的项目拉下来（git clone）

```bash
# 在 GitHub 仓库页面点绿色 "Code" 按钮，复制 HTTPS 地址
git clone https://github.com/用户名/仓库名.git

# 克隆到指定目录名
git clone https://github.com/用户名/仓库名.git 我的目录名

# 克隆指定分支
git clone -b 分支名 https://github.com/用户名/仓库名.git
```

`clone` 做了三件事：

1. 把**全部历史**下载到本地（不是只下载最新文件！）
2. 自动设置 `origin` 远程关联
3. 自动切到默认分支（main）

> 💡 找练手项目：`git clone https://github.com/octocat/Hello-World.git`（GitHub 官方演示仓库），或用你自己的仓库。

## 5.4 同步：push / pull / fetch

```bash
git push              # 把本地新提交推送到远程（有 -u 后简写）
git push origin main  # 完整写法：推送到 origin 的 main 分支
git push -u origin feature-login   # 新分支第一次推送要 -u

git pull              # 从远程拉取并合并到当前分支（= fetch + merge）
git fetch             # 只下载远程更新，不合并（安全，先看看再决定）
```

### ⭐ fetch 与 pull 的区别（重要概念）

```bash
git fetch origin              # 1. 只下载：远程更新存到 origin/main，不影响你的工作区
git log --oneline main..origin/main   # 2. 看看远程比我多了什么提交
git diff main origin/main     # 3. 看看具体差异
git merge origin/main         # 4. 确认无误后手动合并（等价于此时 git pull 的合并部分）

# git pull = git fetch + git merge origin/main 的快捷方式
```

| | fetch | pull |
|---|-------|------|
| 下载远程更新 | ✅ | ✅ |
| 自动合并到工作区 | ❌ 不碰你的代码 | ✅ 直接合并 |
| 适合场景 | 谨慎操作、审查后再合并 | 日常快速同步 |

## 5.5 推送到远程后，本地和 GitHub 的关系

```
本地分支 main ──── 对应 ────▶ 远程分支 origin/main
      │                             │
   git push：本地新提交 → 远程
   git pull：远程新提交 → 本地
```

```bash
# 常用检查命令
git status
# Your branch is ahead of 'origin/main' by 2 commits.  ← 本地领先 2 个提交，该 push 了
# Your branch is behind 'origin/main' by 1 commit.    ← 远程领先，该 pull 了
```

### ⚠️ push 被拒绝怎么办（远程有新提交时）

```bash
git push
# ! [rejected]  main -> main (fetch first)
# error: failed to push some refs
```

**原因**：别人推了新提交，你的本地历史落后于远程。**正确解法**（先合并再推送，不要强推）：

```bash
git pull                      # 拉取并合并远程改动（有冲突就解决冲突，见第 4 章）
git push                      # 再次推送，成功
```

> ⚠️ 绝对不要一开始就用 `git push --force`！强推会**覆盖远程历史**，抹掉别人的提交。强推只在"确认远程那些提交确实不要了"时才用（第 7 章讨论）。

## 5.6 HTTPS vs SSH：两种连接方式

| | HTTPS | SSH |
|---|-------|-----|
| 地址形式 | `https://github.com/user/repo.git` | `git@github.com:user/repo.git` |
| 认证方式 | 每次输密码（或用凭据管理器记住） | 密钥对，配一次永久免密 |
| 配置难度 | 零配置 | 需生成密钥（5 分钟） |
| 公司网络 | 兼容性好 | 22 端口可能被防火墙挡 |

### 配置 SSH（一次性，以后 push/pull 免输密码）

```bash
# 1. 生成密钥对（一路回车即可；邮箱填 GitHub 邮箱）
ssh-keygen -t ed25519 -C "你的邮箱@example.com"
# 生成两个文件：
#   ~/.ssh/id_ed25519      私钥（绝对保密，不发给任何人！）
#   ~/.ssh/id_ed25519.pub  公钥（可以公开，就是要给 GitHub 的）

# 2. 查看公钥内容并复制
cat ~/.ssh/id_ed25519.pub
# ssh-ed25519 AAAA...你的邮箱@example.com

# 3. GitHub → 右上角头像 → Settings → SSH and GPG keys → New SSH key
#    粘贴公钥，起个名字（如 "我的笔记本"）→ Add SSH key

# 4. 测试连接
ssh -T git@github.com
# Hi 你的用户名! You've successfully authenticated.

# 5. 改用 SSH 地址（已有仓库切换）
git remote set-url origin git@github.com:你的用户名/仓库名.git
```

以后 `git push` / `git pull` 不再要密码。

## 5.7 GitHub 页面导览

打开你的仓库页面，认识这些区域：

| 标签页 | 用途 |
|--------|------|
| **Code** | 代码浏览、文件目录、Clone 按钮 |
| **Issues** | 问题/任务跟踪（第 6 章） |
| **Pull requests** | 合并请求（第 6 章） |
| **Actions** | CI/CD 自动化流水线（进阶） |
| **Insights** | 贡献统计图、活跃度 |
| **Settings** | 仓库设置、协作者管理 |

其他实用功能：

- **README.md**：仓库首页展示的说明文档（Markdown 格式），任何人访问你的仓库第一眼看到它
- **贡献图（Contribution Graph）**：个人主页的绿色小方格，颜色越深当天提交越多（开源界"打卡记录"）
- **Star / Fork**：收藏、复制别人的项目
- **Release**：发布版本（配合第 7 章的 tag）

### 写一个合格的 README.md

```markdown
# 项目名

一句话介绍项目是干什么的。

## 功能特性
- 特性 1
- 特性 2

## 快速开始
```bash
git clone ...
cd 项目
```

## 使用说明
...

## 许可证
MIT
```

## 本章小结

| 操作 | 命令 |
|------|------|
| 本地项目上 GitHub | 建空仓库 → `remote add` → `push -u origin main` |
| 下载项目 | `git clone URL` |
| 推送 | `git push` |
| 拉取合并 | `git pull`（= fetch + merge） |
| 只看不合 | `git fetch` |
| 换 SSH 免密 | `ssh-keygen` + GitHub 添加公钥 + `set-url` |
| push 被拒 | 先 `git pull` 合并，再 `git push`（严禁乱用 --force） |
