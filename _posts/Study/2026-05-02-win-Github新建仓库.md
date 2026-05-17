---
layout:     post
title:      "win-Github新建仓库"
subtitle:   " \"Git\""
date:       2026-05-02 00:00:00
author:     "Zemiss"
header-img: "img/bg-weeklyplan.jpg"
catalog: true
tags:
  - Study
---

# Windows 下将现有项目关联到 GitHub 仓库

这份文档记录如何在 Windows / PowerShell 中，把一个已经存在的本地项目关联到 GitHub 远程仓库，并完成第一次推送。

适用场景：

- 你已经有一个本地项目文件夹；
- GitHub 上已经创建了一个空仓库；
- Windows 已经配置好 GitHub SSH key；
- `ssh -T git@github.com` 已经测试成功；
- 想把本地项目上传到 GitHub。

---

## 1. 确认 SSH 已配置成功

在 PowerShell 中执行：

```powershell
ssh -T git@github.com
```

如果看到类似：

```text
Hi Zemiss! You've successfully authenticated, but GitHub does not provide shell access.
```

说明 SSH 配置成功。

以后推荐使用 SSH 地址操作 GitHub 仓库，例如：

```text
git@github.com:Zemiss/beta.git
```

而不是：

```text
https://github.com/Zemiss/beta.git
```

---

## 2. 进入已有项目目录

不要在 `C:\Users\12445` 这种用户主目录里直接执行 Git 操作。

应该先进入你的项目目录，例如：

```powershell
cd C:\Users\12445\Desktop\MyProject
```

如果项目在其他位置，就换成对应路径：

```powershell
cd D:\Code\MyProject
```

进入后检查当前目录：

```powershell
pwd
```

查看文件：

```powershell
ls
```

确认这里确实是你的项目目录。

---

## 3. 检查当前目录是不是 Git 仓库

执行：

```powershell
git status
```

如果正常显示分支和文件状态，说明当前项目已经是 Git 仓库。

如果出现：

```text
fatal: not a git repository (or any of the parent directories): .git
```

说明这个项目还没有初始化 Git，需要执行：

```powershell
git init
git branch -M main
```

---

## 4. 检查是否已经有关联的远程仓库

执行：

```powershell
git remote -v
```

### 情况 A：没有任何输出

说明还没有绑定远程仓库。

添加远程仓库：

```powershell
git remote add origin git@github.com:Zemiss/beta.git
```

把 `beta.git` 换成你自己的 GitHub 仓库名。

例如仓库叫 `MyProject`：

```powershell
git remote add origin git@github.com:Zemiss/MyProject.git
```

### 情况 B：已经有 `origin`

如果你看到类似：

```text
origin  https://github.com/xxx/xxx.git (fetch)
origin  https://github.com/xxx/xxx.git (push)
```

或者已经有一个错误的远程地址，就修改它：

```powershell
git remote set-url origin git@github.com:Zemiss/beta.git
```

再次检查：

```powershell
git remote -v
```

理想输出类似：

```text
origin  git@github.com:Zemiss/beta.git (fetch)
origin  git@github.com:Zemiss/beta.git (push)
```

---

## 5. 添加并提交本地项目文件

查看当前文件状态：

```powershell
git status
```

添加文件：

```powershell
git add .
```

提交：

```powershell
git commit -m "initial commit"
```

如果出现：

```text
nothing to commit, working tree clean
```

说明当前没有新的文件改动需要提交。

可能原因：

- 文件已经提交过；
- 当前目录不是你真正修改的项目目录；
- 文件被 `.gitignore` 忽略了。

---

## 6. 第一次推送到 GitHub

第一次推送执行：

```powershell
git push -u origin main
```

成功后会看到类似：

```text
branch 'main' set up to track 'origin/main'.
```

以后更新时就可以直接：

```powershell
git push
```

---

## 7. 以后日常更新流程

每次修改项目后，进入项目目录：

```powershell
cd C:\Users\12445\Desktop\MyProject
```

然后执行：

```powershell
git status
git add .
git commit -m "说明这次修改了什么"
git push
```

例如：

```powershell
git add .
git commit -m "update README"
git push
```

---

## 8. 常见错误解释

### 8.1 `fatal: not a git repository`

错误示例：

```text
fatal: not a git repository (or any of the parent directories): .git
```

意思是：当前目录不是 Git 仓库。

解决方法：

先进入正确的项目目录：

```powershell
cd C:\Users\12445\Desktop\MyProject
```

如果项目还没有初始化 Git：

```powershell
git init
git branch -M main
```

---

### 8.2 `remote origin already exists`

错误示例：

```text
error: remote origin already exists.
```

意思是：当前项目已经有一个名为 `origin` 的远程仓库。

不要重复执行 `git remote add origin ...`，应该改地址：

```powershell
git remote set-url origin git@github.com:Zemiss/beta.git
```

---

### 8.3 HTTPS 要求输入密码

如果执行 `git push` 时出现：

```text
Username for 'https://github.com':
Password for 'https://...':
```

说明当前 remote 是 HTTPS 地址。

GitHub 不支持用账号密码直接 push。

推荐改成 SSH：

```powershell
git remote set-url origin git@github.com:Zemiss/beta.git
git push
```

---

### 8.4 `Permission denied (publickey)`

错误示例：

```text
git@github.com: Permission denied (publickey).
```

意思是：SSH key 没配置好，或者 GitHub 没添加当前 Windows 机器的公钥。

检查：

```powershell
ls ~/.ssh
```

应该有：

```text
id_ed25519
id_ed25519.pub
```

查看公钥：

```powershell
cat ~/.ssh/id_ed25519.pub
```

把输出的整行添加到 GitHub：

```text
GitHub -> Settings -> SSH and GPG keys -> New SSH key
```

然后测试：

```powershell
ssh -T git@github.com
```

---

## 9. 不要误把用户主目录当仓库

如果你在这里执行 Git：

```powershell
C:\Users\12445>
```

通常是不对的。

应该进入具体项目目录，例如：

```powershell
cd C:\Users\12445\Desktop\MyProject
```

或者进入 clone 下来的仓库目录：

```powershell
cd C:\Users\12445\beta
```

判断当前目录是不是仓库：

```powershell
git status
```

---

## 10. 已 clone 空仓库后怎么办

如果你执行过：

```powershell
git clone git@github.com:Zemiss/beta.git
```

并看到：

```text
warning: You appear to have cloned an empty repository.
```

说明 GitHub 上的仓库是空的，并且本地生成了一个文件夹：

```text
C:\Users\12445\beta
```

如果你想直接在这个仓库里开始项目：

```powershell
cd C:\Users\12445\beta
echo "# beta" > README.md
git add README.md
git commit -m "initial commit"
git push -u origin main
```

如果你想把另一个已有项目关联到这个 GitHub 仓库，就进入那个已有项目目录，而不是进入 `C:\Users\12445\beta`：

```powershell
cd C:\Users\12445\Desktop\MyProject
git init
git branch -M main
git remote add origin git@github.com:Zemiss/beta.git
git add .
git commit -m "initial commit"
git push -u origin main
```

---

## 11. 最小命令模板

把已有项目关联到 GitHub 空仓库：

```powershell
cd C:\Users\12445\Desktop\你的项目文件夹

git init
git branch -M main

git remote add origin git@github.com:Zemiss/仓库名.git

git add .
git commit -m "initial commit"

git push -u origin main
```

如果已经有 `origin`，使用：

```powershell
git remote set-url origin git@github.com:Zemiss/仓库名.git
git push -u origin main
```

---

## 12. 安全提醒

不要提交这些内容到 GitHub：

- SSH 私钥，例如 `id_ed25519`；
- `.env` 文件；
- API key、token、密码；
- 数据集；
- 模型权重；
- 临时输出文件；
- 个人隐私文件。

如果不确定哪些文件会被提交，先执行：

```powershell
git status
```

确认无误后再：

```powershell
git add .
git commit -m "message"
```
