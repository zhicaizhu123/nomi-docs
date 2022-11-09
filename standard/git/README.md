# Git规范
## 分支说明

| 分支名称 | 描述       | 备注                           |
| -------- | ---------- | ------------------------------ |
| dev      | 开发分支   | 只能同步feat_xxx 代码          |
| test     | 测试分支   | 只能同步feat_xxx 代码          |
| release  | 预发布分支 | 只能同步feat_xxx/ fix_xxx 代码 |
| master   | 生产分支   | 只能同步feat_xxx / fix_xxx代码 |
| fix_xxx  | 热修复分支 | 只能同步master代码             |
| feat_xxx | 功能分支   | 只能同步master代码             |



## 提交流程

### 流程图

![](https://tcs-devops.aliyuncs.com/storage/112n7daab8b9950c01a27315e36034dd0c4b?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9hcHBJZCI6IjVlNzQ4MmQ2MjE1MjJiZDVjN2Y5YjMzNSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTY2ODQxMzUyNSwiaWF0IjoxNjY3ODA4NzI1LCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzExMm43ZGFhYjhiOTk1MGMwMWEyNzMxNWUzNjAzNGRkMGM0YiJ9.JtgG9ZPn4HBW-CmVCna8unKkqQfhsYRwO5_jVJ4FvHg&download=image.png "")

### 操作步骤

#### 功能分支操作

- 1. 提交本地修改

```bash
# 提交到暂存区
git add .

# 提交到代码库
git commit -m 'feat(xxx): xxx'
```

- 2. 同步拉取远端代码

```bash
git pull origin feat_xxx -r

# 如果有冲突，则解决冲突后执行以下操作
git add .
git rebase --continue

# 如果有冲突，且如果需要取消同步远程代码，则执行以下操作
git rebase --abort
```

- 3. 提交远端代码库

```text
# 无冲突或处理完冲突后提交代码
git push origin feat_xxx
```

#### 联调dev/提测test/预发布release，提测为例

1. 拉取dev最新代码

```bash
# 切换到dev
git checkout dev

# 拉取最新代码
git pull origin dev
```

1. 合并feat_xxx分支代码到dev

```bash
# 合并功能分支代码
git merge feat_xxx

# 有冲突则解决冲突，解决完后
git add .
git commit -m 'feat(合并代码): xxxx'

# 提交代码
git push origin dev
```

#### 上线操作（管理员操作）

1. 更新远端master代码到feat_xxx

```bash
# 切换到功能分支
git checkout feat_xxx

# 拉取同步master最新代码
git pull origin master -r

# 如果有冲突，则解决冲突后执行以下操作
git add .
git rebase --continue

# 如果有冲突，且如果需要取消同步远程代码，则执行以下操作
git rebase --abort

# 无冲突或处理完冲突后提交代码
git push origin feat_xxx
```

1. 合并feat_xxx到master

```bash
# 切换到master 
git checkout master

# 拉去远端最新代码
git pull origin master

# 合并feat_xxx代码
git merge feat_xxx

# 提交代码
git push origin master

```

1. 打tag

```bash
# 打tag
git tag v1.0 -m 'xxxx'

# 提交tag
git push origin v1.0
```

1. 删除功能分支

```bash
# 删除分支
git branch -d feat_xxx

# 删除远程分支
git push origin --delete feat_xxx
git branch -dr origint/feat_xxx
```

### 注意事项

- feat_xxx/fix_xxx只能从master拉出，并且在合并到master的时候，需要先rebase master 远端最新代码。

- 提交feat_xxx分支代码时，需要先rebase feat_xxx远端最新代码。

- 不定期删除dev/test/release分支, 并从master新建，确保dev/test/release代码及时与master同步， 避免分支图杂乱，这一步需要与管理员确认并操作。

- feat_xxx/fix_xxx合并master上线后，需要及时删除，避免分支过多。

- 合并代码发生冲突在不确定代码合并的准确性情况下，找相关的开发者确认合并。

- 只要有代码合并到master后必须新建tag，便于后面基于tag进行代码回滚。

- 不定期删除tag，一般只保留最近的10个tag。

- tag不需要跟着需要的版本号走，例如需求版本号是v1.2，实际代码的最新tag是v1.4，新建tag时只需要进行递增1，如v1.5



## Commit规范

### 格式

```bash
<type>(<scope>): <subject>
```

### type(必须)

> 用于说明git commit的类别，只允许使用下面的标识。

- feat：新功能（feature）。

- fix：修复bug，可以是QA发现的BUG，也可以是研发自己发现的BUG。

- docs：文档（documentation）。

- style：格式（不影响代码运行的变动）。

- refactor：重构（即不是新增功能，也不是修改bug的代码变动）。

- perf：优化相关，比如提升性能、体验。

- test：增加测试。

- chore：构建过程或辅助工具的变动。

- revert：回滚到上一个版本。

- merge：代码合并。

- sync：同步主线或分支的Bug。

### scope(可选)

> scope用于说明 commit 影响的范围，比如直播管理，员工管理等。

### **subject(必须)**
> subject是commit目的的简短描述，不超过50个字符。
> 
> 建议使用中文（感觉中国人用中文描述问题能更清楚一些）。

- 结尾不加句号或其他标点符号。

- 根据以上规范git commit message将是如下的格式：

### 示例

```bash
git commit -m 'feat(直播管理): 新建/编辑直播新增结束时间字段'

git commit -m 'docs(README): 项目文档新增条目说明'

git commit -m 'style(落地页编辑)：修改输入框组件样式'

git commit -m 'fix(直播管理): 修复新建直播接口报错问题'

#...
```



## Git常用命令

### 仓库

```bash
# 在当前目录新建一个Git代码库
git init

# 新建一个目录，将其初始化为Git代码库
git init [project-name]

# 下载一个项目和它的整个代码历史
git clone [url]



```

### 配置

```bash
# 显示当前的Git配置
git config --list

# 编辑Git配置文件
git config -e [--global]

# 设置提交代码时的用户信息
git config [--global] user.name "[name]"
git config [--global] user.email "[email address]"



```

### 增加/删除文件

```bash
# 添加指定文件到暂存区
git add [file1] [file2] ...

# 添加指定目录到暂存区，包括子目录
git add [dir]

# 添加当前目录的所有文件到暂存区
git add .

# 添加每个变化前，都会要求确认
# 对于同一个文件的多处变化，可以实现分次提交
git add -p

# 删除工作区文件，并且将这次删除放入暂存区
git rm [file1] [file2] ...

# 停止追踪指定文件，但该文件会保留在工作区
git rm --cached [file]

# 改名文件，并且将这个改名放入暂存区
git mv [file-original] [file-renamed]



```

### 代码提交

```bash
# 提交暂存区到仓库区
git commit -m [message]

# 提交暂存区的指定文件到仓库区
git commit [file1] [file2] ... -m [message]

# 提交工作区自上次commit之后的变化，直接到仓库区
git commit -a

# 提交时显示所有diff信息
git commit -v

# 使用一次新的commit，替代上一次提交
# 如果代码没有任何新变化，则用来改写上一次commit的提交信息
git commit --amend -m [message]

# 重做上一次commit，并包括指定文件的新变化
git commit --amend [file1] [file2] ...

```

### 分支

```bash
# 列出所有本地分支
git branch

# 列出所有远程分支
git branch -r

# 列出所有本地分支和远程分支
git branch -a

# 新建一个分支，但依然停留在当前分支
git branch [branch-name]

# 新建一个分支，并切换到该分支
git checkout -b [branch]

# 新建一个分支，指向指定commit
git branch [branch] [commit]

# 新建一个分支，与指定的远程分支建立追踪关系
git branch --track [branch] [remote-branch]

# 切换到指定分支，并更新工作区
git checkout [branch-name]

# 切换到上一个分支
git checkout -

# 建立追踪关系，在现有分支与指定的远程分支之间
git branch --set-upstream [branch] [remote-branch]

# 合并指定分支到当前分支
git merge [branch]

# 选择一个commit，合并进当前分支
git cherry-pick [commit]

# 删除分支
git branch -d [branch-name]

# 删除远程分支
git push origin --delete [branch-name]
git branch -dr [remote/branch]

```

### 标签

```bash
# 列出所有tag
git tag

# 新建一个tag在当前commit
git tag [tag]

# 新建一个tag在指定commit
git tag [tag] [commit]

# 删除本地tag
git tag -d [tag]

# 删除远程tag
git push origin :refs/tags/[tagName]

# 查看tag信息
git show [tag]

# 提交指定tag
git push [remote] [tag]

# 提交所有tag
git push [remote] --tags

# 新建一个分支，指向某个tag
git checkout -b [branch] [tag]

```

### 查看信息

```bash
# 显示有变更的文件
git status

# 显示当前分支的版本历史
git log

# 显示commit历史，以及每次commit发生变更的文件
git log --stat

# 搜索提交历史，根据关键词
git log -S [keyword]

# 显示某个commit之后的所有变动，每个commit占据一行
git log [tag] HEAD --pretty=format:%s

# 显示某个commit之后的所有变动，其"提交说明"必须符合搜索条件
git log [tag] HEAD --grep feature

# 显示某个文件的版本历史，包括文件改名
git log --follow [file]
git whatchanged [file]

# 显示指定文件相关的每一次diff
git log -p [file]

# 显示过去5次提交
git log -5 --pretty --oneline

# 显示所有提交过的用户，按提交次数排序
git shortlog -sn

# 显示指定文件是什么人在什么时间修改过
git blame [file]

# 显示暂存区和工作区的差异
git diff

# 显示暂存区和上一个commit的差异
git diff --cached [file]

# 显示工作区与当前分支最新commit之间的差异
git diff HEAD

# 显示两次提交之间的差异
git diff [first-branch]...[second-branch]

# 显示今天你写了多少行代码
git diff --shortstat "@{0 day ago}"

# 显示某次提交的元数据和内容变化
git show [commit]

# 显示某次提交发生变化的文件
git show --name-only [commit]

# 显示某次提交时，某个文件的内容
git show [commit]:[filename]

# 显示当前分支的最近几次提交
git reflog

```

### 远程同步

```bash
# 下载远程仓库的所有变动
git fetch [remote]

# 显示所有远程仓库
git remote -v

# 显示某个远程仓库的信息
git remote show [remote]

# 增加一个新的远程仓库，并命名
git remote add [shortname] [url]

# 取回远程仓库的变化，并与本地分支合并
git pull [remote] [branch]

# 上传本地指定分支到远程仓库
git push [remote] [branch]

# 强行推送当前分支到远程仓库，即使有冲突
git push [remote] --force

# 推送所有分支到远程仓库
git push [remote] --all

```

### 撤销

```bash
# 恢复暂存区的指定文件到工作区
git checkout [file]

# 恢复某个commit的指定文件到暂存区和工作区
git checkout [commit] [file]

# 恢复暂存区的所有文件到工作区
git checkout .

# 重置暂存区的指定文件，与上一次commit保持一致，但工作区不变
git reset [file]

# 重置暂存区与工作区，与上一次commit保持一致
git reset --hard

# 重置当前分支的指针为指定commit，同时重置暂存区，但工作区不变
git reset [commit]

# 重置当前分支的HEAD为指定commit，同时重置暂存区和工作区，与指定commit一致
git reset --hard [commit]

# 重置当前HEAD为指定commit，但保持暂存区和工作区不变
git reset --keep [commit]

# 新建一个commit，用来撤销指定commit
# 后者的所有变化都将被前者抵消，并且应用到当前分支
git revert [commit]

暂时将未提交的变化移除，稍后再移入
git stash
git stash pop

```

