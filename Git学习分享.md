---
share: true  
---  
## 底层结构
#### 逻辑结构
- 工作区（working directory）当前版本所以文件，从版本库解压出来的文件，放入本地文件目录中。
- 暂存区（staging area）包含的 git版本库中，主要保存了下一次提交次的文件系统，又称为 index 索引。
- 版本库（git directory）git存储自己的元数据，以及文档数据库。
![[Pasted image 20230921151940.png|left|800 ]] 
#### 物理目录
- COMMIT_EDITMSG 最近一次提交时，commit变基信息
- FETCH_HEAD 当前所以分支 fetch 到的最新分支
- HEAD 指向当前的分支文件
- ORIG_HEAD 指向 merge、reset 等上次的提交
- REBASE_HEAD：变基信息，指向当前，被整合的commit的hashkey信息
- rebase-merge目录: 变基处理目录，完成之后删除
- config 用户当前仓库的配置
	- fetch = +refs/heads/*:refs/remotes/origin/* 表示同步refs/remotes/origin/* 下更新到 refs/heads/*
- description 用语 GitWeb 展示项目描述信息
- hooks 钩子
- index 暂存区文件，二进制文件
- info
- log 保存记录变更信息
	- HEAD 记录所有更改记录
	- logs/ref 下存储本地与远程分支的更改记录
- objects 对象
	- 2 位目录名/38 位文件名 ，提高效率
	- blob 代码文件
	- tree文件
	- commit文件
	- tag 重量级文件
	- info/pack 压缩目录
- refs 引用
	- heads
		- 分支名，指向 commit 的 hash 值
	- tags
		- 轻量级 tags，指向 commit
		- 重量级 tags，指向 tag object 的 hash
	- remote
		- 记录每个分支最近一次 push 到远程对应的 commit
	- stash 文件
		- 记录 stash 的提交
![[Pasted image 20230921151747.png|left|800]]
#### 对象
- blob：一次提交的文件一个 blob
	- git 追踪的是内容而不是文件
		- 目录不同，但是内容一样，只会存在一份 blob
		- git根据内容计算每一个文件的 SHA1 散列码
		- 版本变更是，git记录的是两个不同的文件内容，而不是内容之间的差异。
	- 打包文件
	- blob 内容 = header + content，header = 类型+空格+长度，计算 SHA-1
- tree： 一次提交，所以目录一个 tree，包含各个文件的 blob。
	- tree-object 是提交时，所以文件的版本。对应文件系统的一个目录，tree = blob + tree
- commit：一次提交一个 commit
	- 记录作者、提交者、日期、日志消息。
	- 指向父提交，附属到一个 tree-obejct
	- 提交会生成 40 位完成的 SHA-1 hash，也可以提供至少前 4 位，一般 7 位。
- tag
	- lightweight
	- annotated
		- 创建 对象
- 存储优化
	- 一般 loose格式
	- git gc 或 push packfile（看情况）打包，差异性文件打包。只对最新文件做了全量处理，最老的都做了增量叠加

![[Pasted image 20230921152650.png|left|1200]]
#### 引用
- 可以指向任何对象，一般是指向提交对象
- 分支名、标签名都是引用
- 引用存储在.git/refs目录中
- 特殊引用
	- HEAD：当前分支最近提交
	- ORIG_HEAD：merge 与 reset之前的 HEAD
	- FETCH_HEAD：其他分支 fetch 时，记录
	- CHERRY_PICK_HEAD
	- MERGE_HEAD：merge 时，其他分支的 head 记录在MERGE_HEAD
	- REBASE_HEAD：变基信息，指向当前，被整合的commit的hashkey信息

#### 状态机
- 被忽律的（Ignored）
	- gitignore
		- **/*.txt`
- 未追踪的（Untracked）
- 已修改（modified）
- 已暂存（staged）
- 已提交/未改变（committed/unmodified）
![[Pasted image 20230921152833.png|left|600]]

## 场景使用
#### 初始化
`git init` 初始化.git目录。此时 HEAD虽然指向refs/heads/master，但是 master 没有创建。
![[Pasted image 20230921152927.png|left|1000]]

`git add --all` 添加， `git add -i` 交互
![[Pasted image 20230921152948.png|left|1000]]

`git clone origin/xx.git` 克隆远程仓库

`git commit -m` '初始化'
![[Pasted image 20230921153030.png|left|1000]]

`.gitignore` 忽略当前目录下，某些文件

`git checkout -b dev` 创建 dev，并切换到 dev
`git switch -c dev`，创建并切换到 dev分支
![[Pasted image 20230921153125.png|left|1600]]

`git config` 三级目录
- `git config -a` .git/config目录
- `git config -e --global` ~/.gitconfig
- `git config -e --system` /etc/gitconfig

`git fetch` 拉取所以目录最新
![[Pasted image 20230921153208.png|left|1600]]

`git stash` 暂存
- `save`, `list`, `apply`
- `git stash save --include-untracked` 包含缓存区，untrached 的文件
- `git stash apply --index` 同时将工作区和暂存区中的内容进行恢复
![[Pasted image 20230921153316.png|left|1600]]

#### 查看
`git status` 检查缓存区哪些变更没有 add，哪些变更没有commit。

`git log` 目录操作日志
- `git log --stat` 统计信息
- `git log --pretty=oneline` 显示一行
- `git log --pretty=format:"%h - %an, %ar : %s"` 可以显示短hash、作者、多长时间以前、提交说明。
- `git log --oneline --abbrev-commit --graph` 看到结构树
- `git log --abbrev-commit master ..feature/001` 后者与前者落后多上个 commit
- `git log --abbrev-commit feature/001..master` 前者领先落后多少个 commit

`git reflog` 全部 commit，而不是当前分支的树
`git show` 查看各种对象，引用等
- git show master/HEAD/xxxx 查看 master指向那个 commit
- git show HEAD~2 倒置前几个提交

`git diff`（只能产看 tracked 的文件）
- `git diff`：工作区和暂存区
- `git diff --cached`：暂存区和仓库
- `git diff HEAD`：工作区和仓库
- `git diff feature/001 master`：两个分支之间的差异
- `git diff HEAD HEAD^`：最近两次提交之间的差异
![[Pasted image 20230921153451.png|left|400]]

`git branch` 查看分支
- `--list`
- `--all`
- `--remote`

`git bisect` 二分查找
- `git bisect start h e`：开始二分查找git
- `bisect bad`：标注当前这个commit是有bug的git
- `biset good v1.0.0`：用tag来标注说从上一次哪个上线为止，是没有bug的，就是那个tag对应的commit之前都是好的

#### 分支变更
`git reset` 变更分支指向 commit
![[Pasted image 20230921153748.png|left|1000]]

`git reset --hard HEAD^（xx）` master指向上一个commit，抹掉暂存区和工作区
![[Pasted image 20230921153807.png|left|1000]]

`git reset --mixed HEAD^（xx）` master指向上一个commit，抹掉暂存区
![[Pasted image 20230921153822.png|left|1000]]

`git reset --soft HEAD^（xx）` master指向上一个commit，但是暂存区和工作区内容不变
![[Pasted image 20230921153838.png|left|1000]]

- `git branch -u origin/serverfix` 让本地分支 track指定远程分支
	- `git branch -vv` 查看每个本地分支 track 的远程分支
	- `git branch -d dev` 删除 dev分支
- `git fetch origin` 拉取所以远程分支到最新
- `git pull --rebase` 变基式合并
- `git rebase -i HEAD~3` 编辑提交
	- `pick`：保留该commit（缩写:p）
	- `reword`：保留该commit，但我需要修改该 commit的注释（缩写:r）
	- `edit`：保留该commit, 但我要停下来修改该提交(不仅仅修改注释)（缩写:e）
	- `squash`：将该commit和前一个commit合并（缩写:s）
	- `fixup`：将该commit和前一个commit合并，但我不要保留该提交的注释信息（缩写:f）
	- `exec`：执行shell命令（缩写:x）
	- `drop`：我要丢弃该commit（缩写:d）
![[Pasted image 20230921154105.png|left|1600]]
#### 提交
- `git commit -m` 描述提交信息
- `git cherry-pick` 先后 pick m 2, b2~b3的提交
![[Pasted image 20230925101310.png|left|1600]]

#### 合并
`git merge dev` 当前分支合并 dev
有两种 merge。
`ff`：如果合并分支的最后 Commit，在被合并分支 Commit 的祖先节点上，那么，直接把分支指向从最新 Commit 即可。
![[Pasted image 20230921155141.png|left|1600]]
`3w`：如果合并分支的最后 Commit，不在被合并分支 Commit 的祖先节点上，则需要想三叉路口一样，进行合并
![[Pasted image 20230921155301.png|left|1600]]

#### 回退合并
- `git merge --abort` 合并过程中，不想合并
- `git reset --hard HEAD^` 合并之后，没有推送，不想合并
- `git revert -m 1 HEAD` 远程推送之后，恢复最后一次提交的代码
![[Pasted image 20230921154409.png|left|1600]]
#### 推送
- `git push -u origin` 分支名称
- `git push --set-upstream origin` 分支名称 推送并创建
- `git pull = git fetch + git merge origin/xx` (会自动 fetch)

## 底层命令
- `git hash-object --stdin` 计算 hash 值
- `git update-index -add new.txt` 将文件放入暂存区
- `git read-tree` 将存库的文件引用，写到暂存区
- `git write-tree` 将暂存区的文件，写 tree
- `git commit-tree xx` 提交文件
- `git ls-files -s` 查看版本库文件名
- `git ls-files --stage` 查看索引区文件
- `git cat-file -p` 查看版本库文件内容
- `git update-ref xx` xx 更新指针引用

## 方法论
#### 规范
分支名
- master 主分支，一般保护分支，不允许直接pull
- feature 用来开发一个分支
- hotfix 紧急线上修复
- bugfix 非紧急修复
使用/隔开，在.git底层目录中，也生成“feature/add_xx”这种结构。
#### 工作流
集中式工作流（2 人）。直接 master 分支开发，不涉及 code review 什么的

##### 功能分支工作流。
**git flow**（版本稳定的中小型项目）
- master（发布）
- relesase（qa测试）
- develop（继承测试）
- feature/xx（功能开发）
![[Pasted image 20230921154745.png|left|1000]]

**github flow**
github 推出的较为简单个工作流。可基于分支，也可基于 fork 库。侧重点在敏捷开发与CICD集成
![[Pasted image 20230921154803.png|left|1000]]

**gitlab flow**
gitlab 推出，结合 gitflow 与 github flow。有两种模式
基于环境分支的流程：
![[Pasted image 20230921154814.png|Pasted image 20230921154814.png]]
基于稳定分支的flow
![[Pasted image 20230921154822.png|left|1000]]

**轻舟模型**
轻舟的 flow 与前三者都不一样。
![[Pasted image 20230921154836.png|left|1000]]

## 引用文件
流程全景图：[Git相关](https://whimsical.com/git-SBaowM8vJZgQ4mzzgvuZET)