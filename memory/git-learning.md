# Git 深入学习笔记

**学习日期**: 2026-03-11
**Git 版本**: 2.47.3

---

## 🔬 Git 内部原理

### 1. 四大对象 (The Four Objects)

Git 是一个内容寻址文件系统，由四种对象组成：

```
┌─────────────────────────────────────────────────────────┐
│                    Git 对象模型                          │
├─────────────────────────────────────────────────────────┤
│                                                         │
│   ┌─────────┐    ┌─────────┐    ┌─────────┐            │
│   │  Blob   │    │  Tree   │    │ Commit  │            │
│   │ (文件)  │◄───│ (目录)  │◄───│ (提交)  │            │
│   └─────────┘    └─────────┘    └────┬────┘            │
│                                       │                 │
│                                       ▼                 │
│                                  ┌─────────┐            │
│                                  │   Tag   │            │
│                                  │ (标签)  │            │
│                                  └─────────┘            │
└─────────────────────────────────────────────────────────┘
```

#### Blob (Binary Large Object)
- 存储文件内容
- 只存储内容，不存储文件名
- SHA-1 哈希作为唯一标识

```bash
# 查看 blob 内容
git cat-file -p <blob-sha>
git cat-file -s <blob-sha>  # 查看大小
```

#### Tree (目录树)
- 存储文件名和目录结构
- 包含 blob 引用和其他 tree 引用
- 类似 Unix 文件系统

```bash
# 查看 tree 内容
git cat-file -p HEAD^{tree}
# 输出类似:
# 100644 blob 9f4d96d5b00d98959ea9960f069585ce42b1349a	test.txt
# 040000 tree 27b65ab8cc727e0a7e037d93b120514739a3f8d3	src
```

#### Commit (提交)
- 包含 tree 引用
- 包含父提交引用（第一个提交没有父提交）
- 包含作者和提交者信息
- 包含提交消息

```bash
# 查看 commit 内容
git cat-file -p <commit-sha>
# 输出类似:
# tree 43bd1cff5fe2dcc90c3c0b4c66c9a5c19175e617
# parent a7fc1ce...
# author shenchaojue <email> <timestamp> +0800
# committer ...
#
# Initial commit
```

#### Tag (标签)
- 指向 commit 的引用
- 可以包含标签消息和签名

### 2. SHA-1 哈希

- 40 位十六进制字符串
- 前 2 位作为目录名，后 38 位作为文件名
- 内容相同 → 哈希相同（去重）

```bash
# 计算文件哈希
echo -n "content" | git hash-object --stdin

# 计算文件哈希并存储
echo -n "content" | git hash-object -w --stdin
```

### 3. .git 目录结构

```
.git/
├── HEAD              # 当前指向的引用
├── config            # 仓库配置
├── description       # 仓库描述 (GitWeb)
├── hooks/            # 客户端/服务端钩子
├── info/            # 包含 exclude 文件
├── objects/          # 所有数据内容
│   ├── 12/          # 对象存储 (sha1 前两位)
│   ├── 43/
│   ├── info/
│   └── pack/        # 打包对象
└── refs/            # 提交对象的指针
    ├── heads/       # 本地分支
    └── tags/       # 标签
```

---

## 🚀 进阶命令

### 1. Reflog (引用日志)
记录所有 HEAD 和分支引用的变化

```bash
git reflog                    # 查看所有引用变化
git reflog show master        # 查看特定分支
git checkout HEAD@{2}         # 恢复到某个状态
git reset --hard HEAD@{1}     # 撤销操作
```

### 2. Rebase (变基)
将一系列提交"移动"到另一个基点上

```bash
# 基本变基
git checkout feature
git rebase master

# 交互式变基
git rebase -i HEAD~3          # 修改最近3个提交
# 选项: pick, reword, edit, squash, fixup, drop
```

**Rebase 原理**:
1. 找到 feature 和 master 的共同祖先
2. 依次保存 feature 上的每个提交
3. 将 master 移动到 feature 的位置
4. 重新应用每个提交（产生新的 SHA-1）

### 3. Cherry-pick (挑拣)
将某个提交的更改应用到当前分支

```bash
# 挑拣单个提交
git cherry-pick <commit-sha>

# 挑拣多个提交
git cherry-pick commit1 commit2

# 挑拣范围
git cherry-pick commit1..commit3
git cherry-pick commit1^..commit3  # 包含 commit1
```

### 4. Worktree (工作树)
在同一个仓库中同时工作在多个分支

```bash
# 列出工作树
git worktree list

# 创建工作树
git worktree add <path> <branch>

# 删除工作树
git worktree remove <path>

# 清理过期工作树
git worktree prune
```

### 5. Stash (储藏)
临时保存未提交的工作

```bash
git stash                    # 储藏
git stash push -m "message"  # 带消息储藏
git stash list               # 查看储藏
git stash show               # 查看储藏内容
git stash pop                # 恢复并删除
git stash apply              # 恢复（保留）
git stash drop               # 删除储藏
git stash clear              # 清空所有
git stash --include-untracked  # 包含未跟踪文件
```

---

## 🎯 高级技巧

### 1. Bisect (二分查找)
快速定位引入 bug 的提交

```bash
git bisect start              # 开始二分查找
git bisect bad                # 标记当前版本有 bug
git bisect good <good-commit> # 标记正常版本

# Git 会自动 checkout 中间版本
# 测试后:
git bisect good  # 或 git bisect bad

# 找到后
git bisect reset              # 恢复
```

### 2. Submodule (子模块)
在一个仓库中包含另一个仓库

```bash
# 添加子模块
git submodule add <url> <path>

# 克隆包含子模块的仓库
git clone --recursive <url>

# 初始化子模块
git submodule init

# 更新子模块
git submodule update

# 简写
git submodule update --init --recursive
```

### 3. Filter-branch / Filter-repo
重写历史

```bash
# 删除所有历史中的某个文件
git filter-branch --tree-filter 'rm -f password.txt' HEAD

# 更推荐使用 git-filter-repo
git filter-repo --path password.txt --invert-paths
```

---

## ⚙️ 高效配置

### 常用别名
```bash
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.st status
git config --global alias.lg "log --oneline --graph --all"
git config --global alias.last "log -1 HEAD"
git config --global alias.unstage "reset HEAD --"
git config --global alias.df "diff --word-diff"
git config --global alias.ss "status -s"
```

### 有用的配置
```bash
# 默认分支名
git config --global init.defaultBranch main

# 拉取时 rebase 而不是 merge
git config --global pull.rebase true

# 合并时保留特征分支
git config --global merge.conflictstyle diff3

# 彩色输出
git config --global color.ui auto

# 签名
git config --global user.signingkey <key>
```

---

## 🔄 团队协作工作流

### 1. Feature Branch
```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   master    │────►│   develop   │────►│  feature/x  │
│  (生产代码)  │     │  (开发分支)  │     │  (功能分支)  │
└─────────────┘     └─────────────┘     └──────┬──────┘
                                               │
                                               ▼
                                         (开发完成后)
                                               │
                                               ▼
                                        ┌─────────────┐
                                        │  Pull Request│
                                        └─────────────┘
```

### 2. Git Flow
- **master**: 生产代码
- **develop**: 开发主分支
- **feature/**: 功能分支
- **release/**: 发布分支
- **hotfix/**: 热修复分支

### 3. Fork 工作流
1. Fork 官方仓库
2. Clone 自己的 Fork
3. 添加官方仓库为 upstream
4. 从 upstream 更新代码
5. Push 到自己的 Fork
6. 创建 Pull Request

---

## 🧪 实践记录

### 实验 1: 探索对象存储
```bash
cd /tmp/git-test
echo "Hello Git" > test.txt
git add . && git commit -m "Initial commit"

# 查看对象
git cat-file -t <commit-sha>  # commit
git cat-file -p <commit-sha>   # 查看 commit 内容
git cat-file -p HEAD^{tree}   # 查看 tree 内容
git cat-file -p <blob-sha>   # 查看 blob 内容
```

### 实验 2: Rebase vs Merge
```
# Rebase 前:
* 227bfce Master commit
| * d60c5e7 Feature commit
|/
* a7fc1ce Add src folder

# Rebase 后:
* ab31b54 Feature commit  (新 SHA-1!)
* 227bfce Master commit
* a7fc1ce Add src folder
```

---

## 📖 已完成学习

- [x] Git 内部原理 (对象模型)
- [x] Rebase 变基
- [x] Cherry-pick 挑拣
- [x] Worktree 工作树
- [x] Stash 储藏
- [x] Bisect 二分查找
- [x] 别名配置
- [ ] Submodule 子模块
- [ ] Git Hooks
- [ ] GitHub Actions CI/CD
