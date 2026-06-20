---
layout:     post
title:      "wwl-Github新建仓库"
subtitle:   " \"Git\""
date:       2026-05-02 00:10:00
author:     "Zemiss"
header-img: "img/bg-weeklyplan.jpg"
catalog: true
tags:
  - Study
---

# GitHub 新项目建仓库与推送流程

这份文档记录一个本地项目从零开始建立 Git 仓库、绑定 GitHub 远程仓库、并完成第一次推送的标准流程。

适用场景：

- 本地已经有一个项目文件夹；
- 想把项目上传到 GitHub；
- 机器已经配置好 GitHub SSH key；
- 希望以后可以直接使用 `git push` 推送代码。

---

## 1. 进入项目目录

先进入你要上传的项目目录：

```bash
cd /path/to/your/project
```

例如：

```bash
cd /home/x12345/MyCode/NewProject
```

---

## 2. 初始化 Git 仓库

如果这个项目还不是 Git 仓库，执行：

```bash
git init
```

然后把默认分支改成 `main`：

```bash
git branch -M main
```

检查当前状态：

```bash
git status
```

---

## 3. 配置 `.gitignore`

在正式提交前，建议先写好 `.gitignore`，避免把数据集、模型权重、缓存文件提交到 GitHub。

可以创建或编辑 `.gitignore`：

```bash
vim .gitignore
```

常见 Python / 机器学习项目可以写：

```gitignore
__pycache__/
*.pyc

.env
.venv/
venv/

.ipynb_checkpoints/

data/
datasets/
outputs/
submission/
saved_models/
checkpoints/

*.pkl
*.pt
*.pth
*.ckpt
*.npy
*.npz
*.zip
*.tar
*.tar.gz

.DS_Store
```

注意：如果是比赛项目，通常不建议把大数据集、模型权重、临时输出文件提交到 GitHub。

---

## 4. 第一次提交代码

查看当前文件状态：

```bash
git status
```

添加所有需要提交的文件：

```bash
git add .
```

提交：

```bash
git commit -m "initial commit"
```

如果出现：

```text
nothing to commit, working tree clean
```

说明当前没有新的改动需要提交。

---

## 5. 在 GitHub 上创建远程仓库

打开 GitHub，新建仓库：

```text
GitHub -> New repository
```

填写仓库名，例如：

```text
NewProject
```

如果本地已经有 README、`.gitignore` 或 LICENSE，GitHub 新建仓库时建议不要勾选：

```text
Add a README file
Add .gitignore
Choose a license
```

否则可能会和本地提交历史冲突。

---

## 6. 绑定远程仓库

使用 SSH 地址绑定远程仓库：

```bash
git remote add origin git@github.com:Zemiss/NewProject.git
```

把 `NewProject` 换成你自己的仓库名。

查看远程地址：

```bash
git remote -v
```

正常应该看到类似：

```text
origin  git@github.com:Zemiss/NewProject.git (fetch)
origin  git@github.com:Zemiss/NewProject.git (push)
```

如果地址写错了，可以修改：

```bash
git remote set-url origin git@github.com:Zemiss/NewProject.git
```

---

## 7. 第一次推送

第一次推送需要加 `-u`，让本地 `main` 分支追踪远程 `main` 分支：

```bash
git push -u origin main
```

成功后，之后就可以直接使用：

```bash
git push
```

---

## 8. 日常更新流程

以后每次修改代码后，标准流程是：

```bash
git status
git add .
git commit -m "说明这次改了什么"
git push
```

例如：

```bash
git status
git add .
git commit -m "fix submission formatting"
git push
```

---

## 9. 常见问题

### 9.1 `nothing to commit, working tree clean`

说明当前没有新的文件改动需要提交。

可以用：

```bash
git status
```

确认 Git 是否检测到了变化。

如果你明明改了文件但 Git 没看到，可能是：

- 文件被 `.gitignore` 忽略了；
- 修改发生在另一个目录；
- 文件已经提交过了；
- 当前不在正确的 Git 仓库目录。

---

### 9.2 `fatal: No configured push destination`

说明本地仓库还没有绑定远程仓库。

解决：

```bash
git remote add origin git@github.com:Zemiss/仓库名.git
git push -u origin main
```

---

### 9.3 `Permission denied (publickey)`

说明 SSH key 没有配置好，或者 GitHub 没有添加这台机器的公钥。

检查本地是否有 SSH key：

```bash
ls ~/.ssh
```

正常应该看到：

```text
id_ed25519
id_ed25519.pub
known_hosts
```

测试 GitHub SSH：

```bash
ssh -T git@github.com
```

成功时会显示类似：

```text
Hi Zemiss! You've successfully authenticated, but GitHub does not provide shell access.
```

---

### 9.4 HTTPS 方式提示密码错误

如果远程地址是：

```text
https://github.com/...
```

GitHub 不再支持用账号密码直接 push。

推荐改成 SSH：

```bash
git remote set-url origin git@github.com:Zemiss/仓库名.git
```

然后：

```bash
git push
```

---

## 10. 最小命令模板

新项目最常用的一整套命令：

```bash
cd /path/to/project

git init
git branch -M main

git add .
git commit -m "initial commit"

git remote add origin git@github.com:Zemiss/仓库名.git
git push -u origin main
```

以后更新：

```bash
git add .
git commit -m "update"
git push
```

---

## 11. 安全提醒

不要把下面这些内容提交到 GitHub：

- 私钥，例如 `id_ed25519`；
- `.env` 文件；
- API key、token、密码；
- 大型数据集；
- 模型权重；
- 临时输出文件；
- 个人隐私文件。

如果不小心把私钥或 token 提交了，应该立即删除远程记录并更换密钥或 token。
