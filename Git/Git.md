## 版本控制器

### 集中式版本控制器

存放在中央服务器,集中管理,需要联网,如**SVN**与**CVS**

### 分布式版本控制器

每个人电脑上都有一个完整的版本库,即本地仓库,多人协作时将各自的修改推送给对方,如**Git**

## Git工作流程

### 远程仓库--->本地仓库

- **fetch**

  从远程仓库抓取,不进行任何的合并动作

- **clone**

  将远程仓库中克隆代码到本地仓库

### 本地仓库--->工作区

- **checkout**

  从本地仓库检出一个仓库分支然后进行修订

### 远程仓库--->工作区

- **pull**

  从远程仓库拉取到本地库,自动进行合并,然后放到工作区

### 工作区--->暂存区

- **add**

  在提交前现将代码提交到暂存区

#### add

```shell
git add
```

将工作区中未暂存或未跟踪状态的文件变为已暂存状态,加入暂存区,即**将所有修改加入暂存区**

### 暂存区--->本地仓库

- **commit**

  提交到本地仓库,本地仓库中保存修改的各个历史版本

#### commit

```shell
git commit -m '注释内容'
```

将已暂存状态的文件提交到本地仓库的当前分支

### 本地仓库--->远程仓库

- **push**

  修改完成后,需要和团队共享代码时,将代码推送到远程仓库

## 基本配置

### 设置用户信息

```shell
git config --global user.name "itcast"
git config --global user.email "hello@itcast"
```

### 查看用户信息

```shell
git config --global user.name
git config --global user.email
```

### 配置别名

- 打开用户目录,创建.bashrc文件:

  ```shell
   touch ~/.bashrc
  ```

- 打开文件,配置信息:

  ```shell
  #用于输出git提交日志
  alias git-log='git log --pretty=oneline --all --graph --abbrev-commit'
  #用于输出当前目录所有文件及基本信息
  alias ll='ls -al'
  ```

### 乱码问题

- 禁止对非ASCII字符进行转义:

  ```shell
  git config --global core.quotepath false
  ```

- git源文件根目录/etc/bash.bashrc文件追加:

  ```shell
  export LANG="zh_CN.UTF-8"
  export LC_ALL="zh_CN.UTF-8"
  ```

### 创建本地仓库

- 目标文件位置执行,初始化目录为一个git仓库:

  ```shell
  git init
  ```

### 基础操作

#### status

```shell
git status
```

查看暂存区及工作区的当前状态

#### log

```shell
git log [option]
# --all :显示所有分支
# --pretty=oneline :将提交信息显示为一行
# --abbrev-commit :使得输出的commitId更简短
# --graph :以图的形式显示
```

## 版本回退

### reset

```shell
git reset --hard commitID
```

版本切换,commitID通过日志查看

### reflog

```shell
git reflog
```

查看已删除的记录

## .gitignore文件

用于忽略部分文件

```shell
# 忽略特定文件
config.properties
*.log
# 忽略特定目录
# 忽略所有目录中的log目录
log/
# 忽略根目录下的log目录
/log
# 只忽略当前目录下的log目录
log
# 额外规则,'但不忽略'
*.txt
!important.txt
# 忽略doc目录下的.txt文件,但不会递归性地忽略更深层次的文件
doc/*.txt
# 忽略doc目录下的所有.txt文件
doc/**/*.txt
```

**.gitignore本身需要被提交到版本控制中**

## 提交

是某个时间点的完整快照,是Git版本控制系统的核心原子操作

- **快照,非差异**: 记录文件的**完整状态**,而非与前一个版本的差异
- 不可变性: 一但创建无法修改
- 完整性: 有唯一的SHA-1哈希值标识
- 可追溯性: 通过父指针形成完整的历史链

### 提交对象结构

```text
[ Commit Object ]
├── 提交元数据 (作者、提交者、时间、消息)
├── 指向树对象 (Tree) 的指针
├── 指向父提交 (Parent) 的指针
└── 唯一的 SHA-1 哈希标识
```

### 存储方式

#### 初始提交

- Git为项目中的**每一个文件创建一个blob对象(存储文件内容)**
- 然后创建一个**tree对象(存储目录结构及对应的blob)**
- 最后创建一个**commit对象指向该tree**

#### 后续提交

- Git**为修改后的文件创建一个blob对象**
- 创建一个新的tree对象,**对于未修改的文件:该tree对象直接指向该blob对象;对于修改的文件:tree指向新创建的blob对象**

## 分支

分支本质上是指向某个提交的可变指针

### 相关指令

#### branch

```shell
# 查看本地分支
git branch
# 创建本地分支
git branch 分支名
# 删除分支本质上是删除指针,不会删除提交历史
# 删除分支
git branch -d b1
# 强制删除分支
git branch -D b1
```

#### checkout

```shell
# 切换分支
git checkout 分支名
# 创建并切换分支
git checkout -b 分支名
```

#### merge

```shell
# 合并分支
# 若两个分支都有提交,Git会创建一个新的合并提交,该提交有两个父提交
git merge 分支名
```

两个分支进行合并操作,若修改了同一个值,从而产生冲突,此时需要手动解决冲突,后**将该文件添加到暂存区,再进行提交**

### 分支使用原则及流程

#### master分支

即线上分支,主分支,线上运行的应用作为的分支

#### develop分支

从master创建的分支,一般作为开发部门的主要开发分支

#### feature/xxxx分支

从develop创建的分支,一般是同期并行开发,但不同期上线时创建的分支，分支上的研发任务完成后合并到develop分支

#### hotfix/xxxx分支

从master派生的分支,一般作为线上bug修复并使用,修复完成后需要合并到master,test,develop分支

### HEAD指针

#### 标准情况下

head指针永远指向工作分支的尖端:

- 初始状态下:`git checkout main`,此时**head指针指向main分支**,main分支指向该分支的尖端即末尾
- 切换分支:新分支feature被创建,并同main一样指向同一个提交,此时**head指针指向feature分支**,间接指向同一个提交
- 进行提交:feature指针指向新的提交,此时**head指针仍然指向feature指针**,间接指向新提交

#### 分离头指针

此时,head指针指向一个具体的提交而非一个分支

如何进入该状态:

```bash
git checkout commitID
```

- 初始状态下:**head指针指向main分支的尖端**
- 分离head:执行`git checkout HEAD~`,此时**head直接指向原指向提交的父提交**
- 进行提交:**新提交的父提交指向head指向的提交,此时没有分支指针可以移动,head指针指向新的提交**,

### 不同分支的合并

#### 快进合并

- 当要合并进的目标分支的顶端,**直接是**当前所在分支的祖先,即**从原分支拉出新分支后,原分支就再也没有任何新的提交**
- 此时Git不会创建任何新的提交
- 只是**将原分支的指针快速移动到新分支所指向的提交**

#### 三方合并&合并冲突

- 寻找共同祖先

- 差异计算,计算共同祖先分别与两方的差异

- 应用合并,即将这两组差异合并应用到共同祖先之上,若两个分支修改了同一文件的同一区域,此时发生合并冲突

- 合并冲突:合并过程会暂停,冲突文件被标记为`Unmerged`状态,冲突文件出现类似标记:

  ```bash
  <<<<<<< HEAD
  这是当前分支（main）的内容
  =======
  这是要合并的分支（feature）的内容
  >>>>>>> feature
  ```

  此时需要:

  - 编辑该文件,手动保留需要内容,并完全删除该标记
  - 加入暂存区
  - 提交完成本次合并,Git提供一个预填好的提交信息

## 远程仓库

- GitHub
- 码云(gitee)
- GitLab

### 远程连接

使用ed25519生成公钥:

```shell
ssh-keygen -t ed25519 -C "your_email@example.com"
```

- 提示1:输入保存路径

- 提示2:设置密码

- 提示3:输入密码

- 生成密钥对(一个公钥(.pub后缀)一个私钥(无后缀))

- **默认路径**通常是 `~/.ssh/id_rsa`（私钥）和 `~/.ssh/id_rsa.pub`（公钥）。

- 获取密钥至剪贴板:

  ```shell
  cat 路径 | pbcopy
  ```

- 连接测试:

  ```shell
  ssh -T git@github.com
  ```

### 远程操作

#### 添加远程仓库

```shell
git remote add origin git@github.com:caokitten/markdown-personal.git
```

- origin: 远端名称,默认为该字段
- git@github.com:caokitten/markdown-personal.git: 仓库路径

#### 查看远程仓库

```shell
git remote
```

#### 推送到远程仓库

```shell
git push [-f] [--set-upstream] [远端名称[本地分支名][:远端分支名]]
```

- 如果远程分支名和本地分支名相同,可以只写本地分支

  ```shell
  git push origin master
  ```

- [-f]: 强制覆盖

- [--set-upstream]: 推送到远端的同时并建立起和远端分支的关联关系

  ```shell
  git push --set-upstream origin master
  ```

- 若当前分支已经和远端分支关联,则可以省略分支名和远端名
