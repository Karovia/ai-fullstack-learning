---
title: Git 与 GitHub 入门
chapter: 00
section: 02
duration: 4-6 小时
tags: [Git, GitHub, 版本控制, SSH, PR, 协作]
---

# Git 与 GitHub 入门

> 不会 Git 的程序员就像不会保存文件的作家——所有努力都可能瞬间归零。

## 一、什么是版本控制？📼

想象三个场景：

1. **打游戏存档：** 你怕 BOSS 太难，先存档，打死了再继续；打不过就读档重来。
2. **Word 撤销：** Ctrl+Z 一下回到上一秒。
3. **多人改 PPT：** 张三改一版，李四改一版，最后谁的版本是对的？

Git 解决的就是这三件事：**存档、回滚、合并多人改动**。

- **Git** = 工具本身（在你电脑上跑）
- **GitHub** = 一个网站，把你的"存档"放到云端，方便分享和协作

⚠️ Git ≠ GitHub。Gitee、GitLab 都是类似 GitHub 的网站，底层用的都是 Git。

---

## 二、Git 安装与初始配置 ⚙️

### 安装

```bash
# macOS（如果上一节装了 Xcode 工具就已经有）
brew install git

# WSL / Ubuntu
sudo apt install git
```

### 初始配置（只需做一次）

```bash
git config --global user.name "你的名字"
git config --global user.email "你的邮箱@example.com"
git config --global init.defaultBranch main
git config --global core.editor "code --wait"   # 装了 VSCode 后再设
```

> 这个邮箱要和后面 GitHub 注册邮箱一致，否则贡献统计不显示头像。

### 验证

```bash
git config --list
```

---

## 三、四大区域模型（必须看懂）🗺️

```
┌─────────────┐  git add   ┌──────────┐  git commit  ┌──────────┐  git push   ┌──────────┐
│  工作区     │ ─────────> │ 暂存区    │ ───────────> │ 本地仓库  │ ──────────> │ 远程仓库  │
│ (你编辑文件) │            │ (Staging) │              │ (.git 目录)│            │ (GitHub)  │
└─────────────┘ <───────── └──────────┘ <─────────── └──────────┘ <────────── └──────────┘
                git restore --staged   git reset       git pull / git fetch
```

| 区域 | 含义 | 类比 |
|------|------|------|
| 工作区 | 你正在改的文件 | 草稿纸 |
| 暂存区 | 准备提交的内容 | 装订夹 |
| 本地仓库 | 你机器上的存档 | 自己的存档槽 |
| 远程仓库 | GitHub 上的存档 | 云端备份 |

---

## 四、核心 10 命令场景手册 📚

### 1. `git init` —— 创建仓库

> 场景：你写了一个新项目，想用 Git 管理。

```bash
mkdir my-project && cd my-project
git init
```

会生成隐藏的 `.git` 目录（这就是仓库本体，别删）。

### 2. `git clone` —— 克隆别人的仓库

> 场景：你想要别人在 GitHub 上的项目代码。

```bash
git clone https://github.com/torvalds/linux.git
```

### 3. `git add` —— 加入暂存区

> 场景：改完文件，告诉 Git "这次提交要带上它们"。

```bash
git add README.md           # 加单个文件
git add .                   # 加全部改动
```

### 4. `git commit` —— 存档

> 场景：到了一个里程碑，存一下。

```bash
git commit -m "feat: 添加登录页面"
```

### 5. `git push` —— 推到远程

> 场景：把本地存档传到 GitHub。

```bash
git push origin main
```

### 6. `git pull` —— 拉取远程更新

> 场景：同事更新了代码，你要同步下来。

```bash
git pull origin main
```

### 7. `git branch` —— 分支管理

> 场景：你想在不影响主线的情况下试个新功能。

```bash
git branch                  # 列出所有分支
git branch feature/login    # 新建分支
git branch -d old-branch    # 删除分支
```

### 8. `git checkout` / `git switch` —— 切换分支

```bash
git switch feature/login        # 切到分支
git switch -c feature/signup    # 新建并切换
```

### 9. `git merge` —— 合并分支

> 场景：feature 分支测好了，合到 main。

```bash
git switch main
git merge feature/login
```

### 10. `git log` —— 看历史

```bash
git log --oneline --graph --all    # 漂亮的图形化历史
```

### 自我检查 ✅

```bash
mkdir git-test && cd git-test
git init
echo "hello" > a.txt
git add a.txt
git commit -m "first commit"
git log
```

看到一条提交记录就成功了。

---

## 五、GitHub 注册 + SSH key 配置 🔐

### 1. 注册 GitHub

去 [github.com](https://github.com) 注册，邮箱用之前 git config 设的那个。

### 2. 生成 SSH key

**什么是 SSH key？** 一对密钥（公钥 + 私钥）。把公钥告诉 GitHub，以后传代码就不用每次输密码了。

```bash
ssh-keygen -t ed25519 -C "你的邮箱@example.com"
```

一路回车（密码留空就行）。

### 3. 上传公钥到 GitHub

```bash
cat ~/.ssh/id_ed25519.pub
```

复制输出（以 `ssh-ed25519` 开头）。

打开 GitHub → Settings → SSH and GPG keys → New SSH key → 粘贴 → Save。

### 4. 测试

```bash
ssh -T git@github.com
```

看到 `Hi xxx! You've successfully authenticated.` 就成功了。

### 5. 装 gh CLI（GitHub 官方命令行）

```bash
# macOS
brew install gh

# WSL
sudo apt install gh

# 登录
gh auth login
```

之后可以 `gh repo create`、`gh pr create` 等。

---

## 六、第一个 PR 实战 🎯

PR = Pull Request = "请把我的改动合并到你的项目里"。开源协作的标准动作。

### 流程

```
你 fork → 你 clone → 改代码 → 你 push → 在 GitHub 上发起 PR → 项目作者审核合并
```

### 步骤

1. 在 GitHub 找一个项目，点右上角 **Fork**（复制一份到你账号下）
2. clone 你 fork 后的仓库：
   ```bash
   git clone git@github.com:你的用户名/项目名.git
   cd 项目名
   ```
3. 新建分支：
   ```bash
   git switch -c fix/typo
   ```
4. 改文件，比如修一个错别字
5. 提交：
   ```bash
   git add .
   git commit -m "docs: 修正 README 错别字"
   git push origin fix/typo
   ```
6. 用 gh 一键开 PR：
   ```bash
   gh pr create --fill
   ```

🎉 你已经迈出了开源贡献第一步！

---

## 七、约定式提交规范 ✍️

让你的提交信息一眼能看懂：

```
<类型>(<范围>): <简短描述>

[可选的详细说明]
```

| 类型 | 含义 | 例子 |
|------|------|------|
| feat | 新功能 | `feat(auth): 添加微信登录` |
| fix | 修 bug | `fix(api): 修正空指针` |
| docs | 文档 | `docs: 更新 README` |
| style | 格式（不影响逻辑） | `style: 统一缩进` |
| refactor | 重构 | `refactor: 抽取工具函数` |
| test | 测试 | `test: 添加登录用例` |
| chore | 杂项 | `chore: 升级依赖` |

---

## 八、常见报错解决方案 🚨

### merge conflict（合并冲突）

```
CONFLICT (content): Merge conflict in xxx.txt
```

打开冲突文件，会看到：

```
<<<<<<< HEAD
你的版本
=======
对方的版本
>>>>>>> branch-name
```

手动选择保留哪部分，删掉 `<<<<<<<` `=======` `>>>>>>>` 这些标记，然后：

```bash
git add xxx.txt
git commit
```

### detached HEAD（游离状态）

意思是你切到了某个具体的 commit，不在任何分支上。修了东西也保不下来。

**解决：**

```bash
git switch -c temp-branch    # 创建分支保住改动
```

### `! [rejected] main -> main (fetch first)`

**原因：** 远程有你本地没有的提交。

**解决：**

```bash
git pull --rebase origin main
git push origin main
```

### 不小心 commit 了密钥/密码

⚠️ 立刻去服务商**重置那个密钥**！Git 历史里清掉很麻烦，公开仓库等于已泄露。

---

## 我学完了吗？✅ checklist

- [ ] 能讲清楚 工作区/暂存区/本地仓库/远程仓库 四个区域
- [ ] 会用 init / add / commit / push / pull
- [ ] 会建分支并合并
- [ ] 配好了 SSH key，能免密 push
- [ ] 装了 gh CLI 并 `gh auth login`
- [ ] 完成了人生第一个 PR
- [ ] 提交信息会用约定式规范
- [ ] 遇到 merge conflict 不慌

全部打勾 → 进入下一节《命令行终端基础》🚀
