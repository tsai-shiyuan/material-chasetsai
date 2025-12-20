# Git｜分布式版本控制

主要的学习途径: [Git教程by廖雪峰](https://www.liaoxuefeng.com/wiki/896043488029600)

[Git Pro](https://www.progit.cn)

[掘金｜多年Git使用心得&常见问题整理](https://juejin.cn/post/6844904191203213326)

[可视化理解Git(基础)](https://learngitbranching.js.org/)

[可视化理解Git(进阶)](https://dev.to/lydiahallie/cs-visualized-useful-git-commands-37p1)

> 持续施工中……👷

## 安装与配置

查看Git的配置信息

```bash
git config --list
```

返回

```bash
credential.helper=osxkeychain
init.defaultbranch=main
user.name=tsai-shiyuan
user.email=Chase_Tsai@163.com
```

## 版本库

版本库又名仓库（Repository），可以简单理解成一个目录，这个目录里面的所有文件都可以被Git管理起来，每个文件的修改、删除，Git都能跟踪

如何跟踪？

⌘⇧. 查看隐藏文件夹，发现有 `.git`

![截屏2024-09-09 13.47.22](./0img/%E6%88%AA%E5%B1%8F2024-09-09%2013.47.22.png)

### 创建版本库

Step 1. 首先创建一个文件夹并进入

Step 2. 通过 `git init` 命令把这个目录变成Git可以管理的仓库

```bash
git init
```

返回

```bash
Initialized empty Git repository in /Users/chasetsai/Desktop/Java/oopre_self/.git/
```

`.git` 目录是Git来跟踪管理版本库的，不要手动修改这个目录里面的文件

### 把文件添加到版本库

版本控制系统其实只能跟踪文本文件的改动

- 在 `oopre_self` 文件夹内写 `readme.txt` 文件

- `git add` 把文件添加到仓库(开始track追踪)：

```bash
git add readme.txt
```

- `git commit` 把文件提交到仓库(版本就会留在历史记录)：

```bash
git commit -m "wrote a readme file"
```

`-m` 后面输入的是本次提交的说明

返回：

```bash
[main (root-commit) 2178f6d] wrote a readme file
 1 file changed, 2 insertions(+)	#readme.txt有两行内容
 create mode 100644 readme.txt
```

## 版本控制

### 修改文件内容

修改 `readme.txt` 的内容

```bash
Git is a distributed version control system.
Git is free software.
```

运行 `git status` 命令:

```bash
$ git status
On branch main
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	modified:   readme.txt

no changes added to commit (use "git add" and/or "git commit -a")
```

`git diff` 查看修改(但尚未暂存)内容:

```bash
$ git diff readme.txt 
diff --git a/readme.txt b/readme.txt
index d8036c1..013b5bc 100644
--- a/readme.txt
+++ b/readme.txt
@@ -1,2 +1,2 @@
-Git is a version control system.
+Git is a distributed version control system.
 Git is free software.
\ No newline at end of file
```

`git diff --cached` 或 `git diff --staged` 查看已经暂存起来的变化

### 查看提交记录

再改一次并提交:

```bash
Git is a distributed version control system.
Git is free software distributed under the GPL.
```

查看以上三次历史记录:

```bash
$ git log
commit d1fc845fd70d0136e5776f3d841c2e5a57096378 (HEAD -> main)
Author: tsai-shiyuan <Chase_Tsai@163.com>
Date:   Thu Sep 5 17:02:59 2024 +0800

    append GPL

commit d830c503a915a9f6de1b9c93b6988a1637bda218
Author: tsai-shiyuan <Chase_Tsai@163.com>
Date:   Thu Sep 5 17:00:44 2024 +0800

    add distributed

commit 2178f6d545dfa36d9fcde4507b974dff72be0212
Author: tsai-shiyuan <Chase_Tsai@163.com>
Date:   Thu Sep 5 16:51:15 2024 +0800

    wrote a readme file
```

显示从最近到最远的提交日志

添加 `--pretty=oneline` 参数可以不显示作者和日期等信息

进而查看内容:

```bash
$ git checkout [commit_id or branch_name]
```

看完后需要回到当前分支，再进行其他修改:

```bash
$ git checkout main
```

### 版本回退

用 `HEAD` 表示当前版本，也就是最新的提交 `d1fc8...`，上一个版本就是 `HEAD^`，上上一个版本就是 `HEAD^^`，当然往上100个版本写100个 `^` 比较容易数不过来，所以写成 `HEAD~100`

现在，我们要把当前版本 `append GPL` 回退到上一个版本 `add distributed` ，就可以使用 `git reset` 命令：

```bash
$ git reset --hard HEAD^
HEAD is now at d830c50 add distributed
```

- `--hard` 会回退到上个版本的已提交状态
- `--soft` 会回退到上个版本的未提交状态
- `--mixed` 会回退到上个版本已添加但未提交的状态

回退其实是改变指向:

![截屏2024-09-05 17.18.20](./0img/%E6%88%AA%E5%B1%8F2024-09-05%2017.18.20.png)

此时 `git log`:

```bash
commit d830c503a915a9f6de1b9c93b6988a1637bda218 (HEAD -> main)
Author: tsai-shiyuan <Chase_Tsai@163.com>
Date:   Thu Sep 5 17:00:44 2024 +0800

    add distributed

commit 2178f6d545dfa36d9fcde4507b974dff72be0212
Author: tsai-shiyuan <Chase_Tsai@163.com>
Date:   Thu Sep 5 16:51:15 2024 +0800

    wrote a readme file
```

发现没有第三个版本了

### 版本穿梭

指定回到未来的某个版本:

```bash
$ git reset --hard d1fc8	#未来的那个"地址"，不用写全
HEAD is now at d1fc845 append GPL
```

可以用 `git reflog` 查看输过的命令，就能得到地址

### 工作区和暂存区

电脑里能看到的目录，比如 `oopre_self` 文件夹就是一个**工作区 (Working Directory)**

工作区有一个隐藏目录 `.git`，这个不算工作区，而是Git的**版本库 (Repository)**

![截屏2024-09-05 17.24.48](./0img/%E6%88%AA%E5%B1%8F2024-09-05%2017.24.48.png)

### 撤销修改

不小心加了一句但没 `git add`：

```bash
Git is a distributed version control system.
Git is free software distributed under the GPL.
Git has a mutable index called stage.
Git tracks changes.
My stupid boss still prefers SVN.
```

用以下命令把在工作区的修改全部撤销:

```bash
$ git checkout -- readme.txt
```

如果已经 `git add`:

```bash
$ git reset HEAD <file>		# 但我的电脑显示 use "git restore --staged <file>" to unstage
```

可以把暂存区的修改撤销掉 (工作区不会变)，`HEAD` 表示最新的版本

### 删除文件

删除也是一个修改操作，必须要从已跟踪文件清单中移除(移出暂存区)

假设 `test.txt` 已经通过了 `git commit -m "add..."` ，现使用命令 `rm test.txt` 删除了工作区的文件，工作区和版本库就不一致

- 从版本库中删除文件：`git rm test.txt` > `git commit -m "remove test.txt"`

很多时候，输入 `git status` ，`git` 会给出提示

从来没有被添加到版本库就被删除的文件，是无法恢复的！

停止跟踪文件 `git rm --cached README.md`

## 远程仓库

GitHub：提供Git仓库托管服务

本地Git仓库和GitHub仓库之间的传输是通过SSH加密

在用户主目录已找到 `.ssh` 目录，内含 `id_rsa` 和 `id_rsa.pub` 两个文件

- `id_rsa` 是私钥，不能泄露出去，
- `id_rsa.pub `是公钥

SSH key的作用：GitHub需要识别出我们推送的提交确实是自己推送的，而Git支持SSH协议，所以，GitHub只要知道了你的公钥，就可以确认只有我们自己才能推送

GitHub允许添加多个Key，把每台电脑的Key都添加到GitHub，就可以在每台电脑上往GitHub推送

### 添加远程库

Now: 已经在本地创建了一个Git仓库后 (进行了init)，又想在GitHub创建一个Git仓库，并且让这两个仓库进行远程同步

`create a new repo`

![截屏2024-09-09 09.42.58](./0img/%E6%88%AA%E5%B1%8F2024-09-09%2009.42.58.png)

```bash
$ git remote add origin git@github.com:tsai-shiyuan/oopre_self.git
```

从 `origin` 仓库的 `main` 分支下载文件

```bash
$ git pull origin main
```

远程库的简称就是 `origin` (默认)

```bash
$ git push -u origin main
```

把本地库的内容推送到远程，教程上写的是 `master` 分支

`-u` 参数会把本地的 `main` 分支和远程的 `main` 分支关联起来，今后再使用就只需 `git push origin main` 把本地的最新修改推送至GitHub

![截屏2024-09-09 09.49.39](./0img/%E6%88%AA%E5%B1%8F2024-09-09%2009.49.39.png)

### 删除远程库

先查看远程库信息:

```bash
$ git remote -v
origin	git@github.com:tsai-shiyuan/oopre_self.git (fetch)
origin	git@github.com:tsai-shiyuan/oopre_self.git (push)
```

再根据名字删除，比如 `origin` :

```bash
$ git remote rm origin
```

此处的"删除"是解除了本地和远程的绑定关系，并不是物理上删除了远程库。远程库本身并没有任何改动。

### 从远程库克隆

登陆GitHub，创建一个 repo

用 `git clone <...>` 克隆一个本地库:

```bash
$ git clone git@github.com:tsai-shiyuan/gitskills.git
```

`oopre_self` 里就多了 `gitskills` 文件夹，在url后添加名字参数，可以重命名本地的仓库

## 分支管理

### 创建与合并分支

每次提交，Git都把它们串成一条时间线，这条时间线就是一个分支

`HEAD` 指向 `master` ，`master` 指向提交

一开始，`master` 分支是一条线，Git用 `master` 指向最新的提交，再用 `HEAD` ( `HEAD` 是当前的工作目录) 指向 `master`，就能确定当前分支，以及当前分支的提交点

![截屏2024-09-09 16.20.34](./0img/%E6%88%AA%E5%B1%8F2024-09-09%2016.20.34.png)

当我们创建新的分支，例如 `dev` 时，Git新建了一个指针叫 `dev`，指向 `master` 相同的提交，再把 `HEAD` 指向 `dev`，就表示当前分支在 `dev` 上

![截屏2024-09-09 16.24.30](./0img/%E6%88%AA%E5%B1%8F2024-09-09%2016.24.30.png)

从现在开始，对工作区的修改和提交就是针对 `dev` 分支了，比如新提交一次后，`dev` 指针往前移动一步，而 `master` 指针不变

![截屏2024-09-09 16.25.29](./0img/%E6%88%AA%E5%B1%8F2024-09-09%2016.25.29.png)

假如我们在 `dev` 上的工作完成了，就可以把 `dev` 合并到 `master` 上，方法：直接把 `master` 指向 `dev` 的当前提交

![截屏2024-09-09 16.26.34](./0img/%E6%88%AA%E5%B1%8F2024-09-09%2016.26.34.png)

命令:

创建 `dev` 分支并切换到 `dev`:

```bash
$ git checkout -b dev
Switched to a new branch 'dev'
```

`-b` 参数表示创建并切换，相当于 `git branch dev` > `git checkout dev`

查看当前分支:

```bash
$ git branch
* dev
  master
```

`git branch -v` 可以查看分支的最后一次提交

提交(推荐切换分支前先 commit):

```bash
$ git add readme.txt 
$ git commit -m "branch test"
[dev 7edc459] branch test
 1 file changed, 2 insertions(+), 1 deletion(-)
```

切换回 `master` 分支:

```bash
git checkout master
```

切换回 `master` 分支后，再查看 `readme.txt`文件，不会显示刚才添加的内容。因为那个提交是在 `dev` 分支上，而 `master` 分支此刻的提交点并没有变

![截屏2024-09-09 16.37.36](./0img/%E6%88%AA%E5%B1%8F2024-09-09%2016.37.36.png)

现在，把 `dev` 分支的工作成果合并到 `master` 分支上:

```bash
$ git merge dev
Updating 90c965a..7edc459
Fast-forward
 readme.txt | 3 ++-
 1 file changed, 2 insertions(+), 1 deletion(-)
```

合并完成后，可以删除 `dev`:

```bash
$ git branch -d dev
Deleted branch dev (was 7edc459).
```

用途: 使用分支完成某个任务，合并后再删掉分支

switch切换分支 (更加科学) :

```bash
$ git switch -c dev		# 创建并切换到新的分支
Switched to a new branch 'dev'
$ git switch main		# 切换回已有分支
```

### 解决冲突

移动到新的分支:

```bash
$ git switch -c feature1
```

修改 `readme.txt` 的最后一行:

```bash
Creating a new branch is quick AND simple.
```

在 `feature1` 上提交分支:

```bash
$ git add readme.txt 
$ git commit -m "AND simple"
[feature1 ea9b4dc] AND simple
 1 file changed, 1 insertion(+), 1 deletion(-)
```

切换回 `main` 分支:

```bash
$ git switch main
Switched to branch 'main'
```

修改 `readme.txt` 的最后一行:

```bash
Creating a new branch is quick & simple.
```

在 `main` 上提交分支:

```bash
$ git add readme.txt 
$ git commit -m "& simple"
[main f0fd8a8] & simple
 1 file changed, 1 insertion(+), 1 deletion(-)
```

现在，`master` 分支和 `feature1` 分支各自都分别有新的提交:

![截屏2024-09-10 14.08.14](./0img/%E6%88%AA%E5%B1%8F2024-09-10%2014.08.14.png)

这种情况下，Git无法执行“快速合并”，只能试图把各自的修改合并起来，但这种合并就可能会有冲突:

```bash
$ git merge feature1
Auto-merging readme.txt
CONFLICT (content): Merge conflict in readme.txt
Automatic merge failed; fix conflicts and then commit the result.
```

查看 `readme.txt`:

```bash
$ cat readme.txt          

balabala

<<<<<<< HEAD
Creating a new branch is quick & simple.
=======
Creating a new branch is quick AND simple.
>>>>>>> feature1
```

我们删掉 `& simple` 有关句子，再保存并提交:

```bash
$ git add readme.txt 
$ git commit -m "conflict fixed"
[main 5bcb92b] conflict fixed
```

![截屏2024-09-10 14.14.04](./0img/%E6%88%AA%E5%B1%8F2024-09-10%2014.14.04.png)

也可以用 `git log` 查看分支情况:

```bash
$ git log --graph --pretty=oneline --abbrev-commit		# --graph 参数用于看图
*   5bcb92b (HEAD -> main) conflict fixed
|\  
| * ea9b4dc (feature1) AND simple
* | f0fd8a8 & simple
|/  
* 7edc459 (dev) branch test
* 90c965a remove test.txt
* 047c9bb add test.txt
* edabc19 git tracks changes
* 69401a9 understand how stage works
* d1fc845 append GPL
* d830c50 add distributed
* 2178f6d wrote a readme file
```

删除 `feature1` 分支:

```bash
$ git branch -d feature1
Deleted branch feature1 (was ea9b4dc).
```

### 分支管理策略

切换到 `dev` 分支，修改 `readme.txt` 并提交

切换回 `main`

准备合并 `dev` 分支，`--no-ff` 参数表示禁用 `Fast forward`

```bash
```



## 使用GitHub

GitHub作为免费的远程仓库

### 参与开源项目

[bootstrap项目](https://github.com/twbs/bootstrap)

点击 `Fork` 就在自己的账号下克隆了一个bootstrap仓库，然后，从自己的账号下clone，这样就能推送修改

![截屏2024-09-10 14.21.36](./0img/%E6%88%AA%E5%B1%8F2024-09-10%2014.21.36.png)

- Bootstrap的官方仓库 `twbs/bootstrap`
- 在GitHub上克隆的仓库 `my/bootstrap`
- 自己克隆到本地电脑的仓库

想让bootstrap的官方库接受我的修改，就可以在GitHub上发起一个pull request

## 使用Gitee

国内的Git托管服务——[Gitee](https://gitee.com/?utm_source=blog_lxf)
