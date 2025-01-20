---
layout: post
title: "了解 Git 命令背后的原理"
subtitle: ""
date: 2025-01-14 12:00:00
author: "Peter Dong"
header-img: "img/bg-post_the-principles-behind-git-commands.jpg"
catalog: true
tags:
  - Git
  - Principle
---

Git 不仅是一个版本控制工具，更是一个内容管理系统。它通过独特的对象模型和内容寻址机制，实现了高效的版本管理。虽然我们每天都在使用 `git add`、`git commit` 这样的命令，但你是否思考过这些命令背后究竟发生了什么？

本文将通过具体实例，分析 Git 的原理和内部实现。理解这些原理不仅能帮助你更准确地使用 Git 命令，还能在遇到问题时快速定位和解决。

## 一、从创建仓库开始

在开始之前，我们需要理解一个核心概念：Git 本质上是一个内容寻址（Content-addressable）的文件系统。它将你的所有文件内容、目录结构、提交信息等都转换为对象，并用 SHA-1 哈希值来标识。这就像是给每个文件和文件夹都贴上了一个唯一的标签。

### 1.1 创建 Git 仓库

让我们从创建一个仓库开始：

```bash
mkdir git-demo
cd git-demo
git init
```

当执行 `git init` 时，Git 做了什么？让我们看看：

```bash
tree .git
# 输出：
.git
├── HEAD
├── config 
├── description
├── hooks
│   ├── applypatch-msg.sample
│   ├── commit-msg.sample
│   ├── fsmonitor-watchman.sample
│   ├── post-update.sample
│   ├── pre-applypatch.sample
│   ├── pre-commit.sample
│   ├── pre-merge-commit.sample
│   ├── pre-push.sample
│   ├── pre-rebase.sample
│   ├── pre-receive.sample
│   ├── prepare-commit-msg.sample
│   ├── push-to-checkout.sample
│   └── update.sample
├── info
│   └── exclude
├── objects
│   ├── info
│   └── pack
└── refs
    ├── heads
    └── tags
```

git init 命令执行完成后，就会在项目根目录下创建一个 `.git` 子目录，用来保存版本信息。

我来用表格形式简单解释下每个文件的作用：

| 文件/目录                 | 作用解释                                                |
| ------------------------- | ------------------------------------------------------- |
| HEAD                      | 指向当前所在的分支，存储当前工作分支的引用              |
| config                    | 存储项目的配置信息，包括远程仓库地址、用户信息等        |
| description               | 仓库的描述信息                                          |
| hooks/                    | 存放钩子脚本目录，可以设置在特定 Git 事件发生时自动执行 |
| ├── applypatch-msg.sample | 应用补丁消息钩子示例                                    |
| ├── commit-msg.sample     | 提交消息钩子示例，用于验证提交信息格式                  |
| ├── pre-commit.sample     | 提交前钩子示例，可以在提交前进行代码检查                |
| ├── pre-push.sample       | 推送前钩子示例，可以在推送前进行验证                    |
| └── 其他 *.sample 文件    | 其他各类钩子的示例文件，可以根据需要启用                |
| info/                     | 包含仓库的附加信息                                      |
| └── exclude               | 配置仅在本地生效的忽略文件规则                          |
| objects/                  | 存储所有数据内容的目录                                  |
| ├── info/                 | 包含对象库的附加信息                                    |
| └── pack/                 | 存储包文件，用于优化存储和传输效率                      |
| refs/                     | 存储引用的目录                                          |
| ├── heads/                | 存储所有本地分支的引用                                  |
| └── tags/                 | 存储所有标签的引用                                      |

一些重要补充说明：
1. objects 目录是 Git 的核心，存储了所有的提交、树和数据对象。
2. 实际使用时，objects 目录下会有很多以哈希值命名的文件和目录。
   

### 1.2 创建第一个文件

```bash
echo "Hello, Git!" > hello.txt
```

此时，这个文件只存在于工作目录中，Git 还不知道它的存在：
```bash
git status
# 输出：
# On branch main

# No commits yet

# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#         hello.txt

# nothing added to commit but untracked files present (use "git add" to track)
```

## 二、理解 git add

### 2.1 添加文件到暂存区

```bash
git add hello.txt
```
此时，hello.txt 文件被添加到暂存区，但还没有提交，我们来查看下 .git 目录下发生的变化：

```bash
➜ tree .git
.git
├── HEAD
├── config
├── description
├── hooks
│   ├── applypatch-msg.sample
│   ├── commit-msg.sample
│   ├── fsmonitor-watchman.sample
│   ├── post-update.sample
│   ├── pre-applypatch.sample
│   ├── pre-commit.sample
│   ├── pre-merge-commit.sample
│   ├── pre-push.sample
│   ├── pre-rebase.sample
│   ├── pre-receive.sample
│   ├── prepare-commit-msg.sample
│   ├── push-to-checkout.sample
│   └── update.sample
├── index
├── info
│   └── exclude
├── objects
│   ├── 7f
│   │   └── 118ff7695c4888a5ca943591eb013e40a63187
│   ├── info
│   └── pack
└── refs
    ├── heads
    └── tags
```

我们从上面的目录结构可以看到， `.git/objects` 下面多了一个子目录，目录名是哈希值的前 2 个字符，该子目录下面有一个文件，文件名是哈希值的后 38 个字符。

那么这个简单的命令背后，Git 做了什么？

1). 首先使用 `git hash-object` 命令计算文件内容的 SHA-1 哈希值，然后将文件内容写入到 `.git/objects` 目录 (创建 blob 对象)。此命令不会更新暂存区 (`.git/index` 文件）.

```bash
git hash-object -w hello.txt
# 输出：7f118ff7695c4888a5ca943591eb013e40a63187
```

2). 使用 `git update-index` 命令更新暂存区。

```bash
# 更新索引
git update-index --add --cacheinfo 100644 7f118ff7695c4888a5ca943591eb013e40a63187 hello.txt
```

所以完整模拟 git add 需要：

- `git hash-object -w` 存储内容
- `git update-index` 更新暂存区

3). 可以用 `git cat-file` 命令查看对象内容和类型：

```bash
# 查看对象内容
git cat-file -p 7f118ff7695c4888a5ca943591eb013e40a63187
# 输出：Hello, Git!

# 查看对象类型
git cat-file -t 7f118ff7695c4888a5ca943591eb013e40a63187
# 输出：blob
```

### 2.2 理解 blob 对象
blob（Binary Large Object）对象是 Git 中最基础的对象类型，它用于存储文件的内容。重要特点：
- 只存储文件内容，不包含文件名
- 相同内容的文件共享同一个 blob 对象
- 内容会被压缩存储

验证 blob 对象的去重机制：

```bash
# 创建两个不同名称但内容相同的文件
echo "Hello, Git!" > hello1.txt
echo "Hello, Git!" > hello2.txt
git add hello1.txt hello2.txt

# 查看暂存区
git ls-files --stage
# 输出：
# 100644 7f118ff7695c4888a5ca943591eb013e40a63187 0       hello1.txt
# 100644 7f118ff7695c4888a5ca943591eb013e40a63187 0       hello2.txt
```

注意两个文件指向同一个 blob 对象！这就是 Git 的去重机制。

根据这个特点，我们可以在项目中查找内容相同的文件：

```bash
# 找出仓库中内容相同的文件
git ls-files | while read file; do
  sha=$(git hash-object "$file")
  echo "$sha $file"
done | sort | uniq -w 40 -D
```

## 三、深入理解 git commit

### 3.1 创建第一个提交
```bash
git commit -m "First commit"
```

这个命令执行时，Git 做了什么？

1). 创建 tree 对象：目录结构的快照

tree 对象包含：
- 文件名
- 文件权限
- 对应的 blob/tree 对象的 SHA-1 值

```bash
# 查看当前提交的 tree 对象
git cat-file -p HEAD^{tree}
# 输出：
# 100644 blob 7f118ff7695c4888a5ca943591eb013e40a63187    hello1.txt
# 100644 blob 7f118ff7695c4888a5ca943591eb013e40a63187    hello2.txt
```

2). 创建 commit 对象：提交的快照
   
```bash
# 查看提交对象
git cat-file -p HEAD
# 输出：
# tree fd206836d0e3e53d0e17ab2a6d6109ba0c00089e
# author Peter Dong <dynckm@gmail.com> 1736923822 +0800
# committer Peter Dong <dynckm@gmail.com> 1736923822 +0800

# First commit
```

当执行 `git commit -m "First commit"` 时，Git 会：
1. 创建一个 commit 对象，包含：
   - 顶层 tree 对象的引用
   - 父 commit 的引用
   - 作者信息
   - 提交信息
2. 更新当前分支指向新的 commit

### 3.2 理解提交关系

让我们创建第二个提交：

```bash
echo "New line" >> hello1.txt
git add hello1.txt
git commit -m "Add new line"
```

现在查看提交历史：

```bash
git log --graph --oneline
# 输出：
# * 7cfb1b0 (HEAD -> main) Add new line
# * 2b77d0b First commit
```

每个提交都指向其父提交，形成了一个链表：
```bash
git cat-file -p HEAD
# 输出：
# tree 4579573af8a8eea89868724744f4e909d59470be
# parent 2b77d0beccb2e342f179a4a5f6fbc6b32fc34b67
# author Peter Dong <dynckm@gmail.com> 1736924188 +0800
# committer Peter Dong <dynckm@gmail.com> 1736924188 +0800

# Add new line
```


## 四、分支的秘密

### 4.1 创建分支

```bash
git branch feature
```

这个命令实际上只是创建了一个文件，也就是会在 `.git/refs/heads/` 下创建 feature 文件，内容是当前 commit 的 SHA-1。

```bash
cat .git/refs/heads/feature
# 7cfb1b0... SHA-1）
```

### 4.2 切换分支

```bash
git checkout feature
```

这个命令做了两件事：

1). 更新 `HEAD` 指向
   
```bash
cat .git/HEAD
# 输出：ref: refs/heads/feature
```

2). 更新工作目录
- Git 读取新分支指向的提交
- 根据提交中的 tree 对象更新工作目录

下面是当前最新的 `.git` 目录的结构：

```bash
tree .git
.git
├── COMMIT_EDITMSG
├── HEAD
├── config
├── description
├── hooks
│   ├── applypatch-msg.sample
│   ├── commit-msg.sample
│   ├── fsmonitor-watchman.sample
│   ├── post-update.sample
│   ├── pre-applypatch.sample
│   ├── pre-commit.sample
│   ├── pre-merge-commit.sample
│   ├── pre-push.sample
│   ├── pre-rebase.sample
│   ├── pre-receive.sample
│   ├── prepare-commit-msg.sample
│   ├── push-to-checkout.sample
│   └── update.sample
├── index
├── info
│   └── exclude
├── logs
│   ├── HEAD
│   └── refs
│       └── heads
│           ├── feature
│           └── main
├── objects
│   ├── 0b
│   │   └── 2c260b4d3b617d3c17ed693b6f8920c58009da
│   ├── 2b
│   │   └── 77d0beccb2e342f179a4a5f6fbc6b32fc34b67
│   ├── 45
│   │   └── 79573af8a8eea89868724744f4e909d59470be
│   ├── 7c
│   │   └── fb1b0e1ad97bec83e75030bb135dd0aa507a32
│   ├── 7f
│   │   └── 118ff7695c4888a5ca943591eb013e40a63187
│   ├── fd
│   │   └── 206836d0e3e53d0e17ab2a6d6109ba0c00089e
│   ├── info
│   └── pack
└── refs
    ├── heads
    │   ├── feature
    │   └── main
    └── tags
```

## 总结

通过这些示例，我们可以看到：
1. Git 是一个内容寻址的文件系统，通过 SHA-1 哈希值标识和存储文件内容
2. 所有的 Git 操作都是围绕三种主要对象进行：
   - blob 对象：存储文件内容
   - tree 对象：管理目录结构和文件权限
   - commit 对象：记录提交信息、作者和时间戳
3. 分支只是指向提交的轻量级指针，本质上是一个包含 commit SHA-1 值的文件
4. 每个 Git 命令都可以分解为对这些基本对象的一系列底层操作

通过理解这些原理可以帮助我们：
1. 更好地理解 Git 命令的行为，知道每个命令背后实际发生了什么
2. 在出现问题时能够准确定位原因，并使用底层命令进行修复
3. 更高效地使用 Git 功能，设计更合理的工作流程
4. 在特殊情况下可以使用底层命令解决复杂问题