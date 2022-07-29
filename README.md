# Git-Notes

常用git命令的分类整理，用于日常查阅和复习

- [Git-Notes](#git-notes)
  - [创建仓库与配置](#创建仓库与配置)
    - [init](#init)
    - [clone](#clone)
    - [config](#config)
  - [提交和基本修改](#提交和基本修改)
    - [add](#add)
    - [commit](#commit)
    - [rm](#rm)
    - [mv](#mv)
    - [tag](#tag)
  - [恢复相关](#恢复相关)
    - [reset](#reset)
    - [revert](#revert)
    - [checkout](#checkout)
    - [restore](#restore)
    - [clean](#clean)
    - [stash](#stash)
    - [reflog](#reflog)
    - [fsck](#fsck)
    - [gc](#gc)
  - [分支管理](#分支管理)
    - [branch](#branch)
    - [checkout](#checkout-1)
    - [switch](#switch)
    - [merge](#merge)
    - [rebase](#rebase)
    - [cherry-pick](#cherry-pick)
  - [查看状态和日志](#查看状态和日志)
    - [status](#status)
    - [diff](#diff)
    - [log](#log)
    - [show](#show)
    - [bisect](#bisect)
    - [grep](#grep)
    - [blame](#blame)
  - [远程操作](#远程操作)
    - [remote](#remote)
    - [fetch](#fetch)
    - [pull](#pull)
    - [push](#push)
  - [参考资料](#参考资料)

## 创建仓库与配置

### init

```
// 当前目录下创建git仓库
git init 
// 指定路径下创建git仓库
git init <path> 
// 生成裸仓库（通常用于远程仓库），支持clone和push，但不能提交变更
git init --bare
```



### clone

```
// 克隆到当前目录下仓库同名目录
git clone <url>
// 克隆到指定目录
git clone <url> <dir_name>
```



### config

```
配置有三个级别local，global和system，对应项目级别，用户级别和机器级别（默认是local）

// 显示配置信息
git config --list
// 编辑配置文件
git config -e (--local/global/system)
// 配置用户信息
git config --global user.email "abc123@gmail.com"
git config --global user.name "Alice"
```



## 提交和基本修改

### add

```
// 提交当前目录下的所有变化
git add .
// 提交所有变化
git add -A 
// 交互式选择代码片段提交
git add -p
// 提交被修改(modified)和被删除(deleted)文件，不包括新文件(new)
git add -u
```



### commit

```
// 提交一个描述为 message 的 commit
git commit -m "message"
// 相当于 git add -a 和 git commit -m "message" 两条命令
git commit -am "message"
// 在不增加一个新 commit 的情况下将新修改的代码追加到前一次的 commit 中，会弹出一个编辑器界面重新编辑 message 信息
git commit --amend
// 在不增加一个新 commit 的情况下将新修改的代码追加到前一次的 commit 中，不需要再修改 message 信息
git commit --amend --no-edit
// 提交一次没有任何改动的空提交，常用于触发远程 ci
git commit --allow-empty -m "message"
// 修改 commit 时间
git commit -m "message" --date=" Wed May 27 00:35:36 2020 +0800"
```



### rm

```
// 用于删除工作区文件，并将此次删除放入到暂存区。（注：要删除的文件没有修改过，就是说和当前版本库文件的内容相同。）
git rm <file>

// 用于删除工作区和暂存区文件，并将此次删除放入暂存区。（注：要删除的文件已经修改过，就是说和当前版本库文件的内容不同）
git rm -f <file>

// 用于删除暂存区文件，并将此次删除放入暂存区，但会保留工作区的文件。可以理解成解除 git 对这些文件的追踪，将他们转入 untracked 状态。
git rm --cached <file>
```



### mv

```
// 用于移动或重命名一个文件、目录或软连接。(改变工作区,并提交这次修改给暂存区)
git mv [file] [newfile]
// 新文件名已经存在，若想强制覆盖则可以使用 -f 参数：
git mv -f <old_file> <new_file>
```



### tag

```
// 查看所有 tag
git tag
// 基于本地当前分支最后一个 commit 创建 tag (-a 带附注)
git tag -a <tagName> 
// 基于指定 commit 创建 tag
git tag -a <tagName> <commitId>
// 基于指定 commit 创建 tag 并指定 message
git tag -a <tagName> -m "message"
// 删除本地指定 tag
git tag -d <tagName>
```





## 恢复相关

### reset

[参考博客](https://www.jianshu.com/p/c2ec5f06cf1a)

```
// 本质是移动HEAD指针。有三个常用参数，分别是--hard，--soft，--mixed，默认是--mixed.
git reset [--soft/mixed/hard] <commitId>(/<branch>)

commitId可以是HEAD,HEAD^,HEAD^2这样的形式

区别：
--hard：重置位置的同时，直接将 working Tree工作目录、 index 暂存区及 repository 都重置成目标Reset节点的內容,所以效果看起来等同于清空暂存区和工作区。

--soft：重置位置的同时，保留working Tree工作目录和index暂存区的内容，只让repository中的内容和 reset 目标节点保持一致，因此原节点和reset节点之间的【差异变更集】会放入index暂存区中(Staged files)。所以效果看起来就是工作目录的内容不变，暂存区原有的内容也不变，只是原节点和Reset节点之间的所有差异都会放到暂存区中。

--mixed（默认）：重置位置的同时，只保留Working Tree工作目录的內容，但会将 Index暂存区 和 Repository 中的內容更改和reset目标节点一致，因此原节点和Reset节点之间的【差异变更集】会放入Working Tree工作目录中。所以效果看起来就是原节点和Reset节点之间的所有差异都会放到工作目录中。

使用场景:
--hard：(1) 要放弃目前本地的所有改变時，即去掉所有add到暂存区的文件和工作区的文件，可以执行 git reset -hard HEAD 来强制恢复git管理的文件夹的內容及状态；(2) 真的想抛弃目标节点后的所有commit（可能觉得目标节点到原节点之间的commit提交都是错了，之前所有的commit有问题）。

--soft：原节点和reset节点之间的【差异变更集】会放入index暂存区中(Staged files)，所以假如我们之前工作目录没有改过任何文件，也没add到暂存区，那么使用reset --soft后，我们可以直接执行 git commit 將 index暂存区中的內容提交至 repository 中。为什么要这样呢？这样做的使用场景是：假如我们想合并「当前节点」与「reset目标节点」之间不具太大意义的 commit 记录(可能是阶段性地频繁提交,就是开发一个功能的时候，改或者增加一个文件的时候就commit，这样做导致一个完整的功能可能会好多个commit点，这时假如你需要把这些commit整合成一个commit的时候)時，可以考虑使用reset --soft来让 commit 演进线图较为清晰。总而言之，可以使用--soft合并commit节点。

--mixed（默认）：(1)使用完reset --mixed后，我們可以直接执行 git add 将這些改变果的文件內容加入 index 暂存区中，再执行 git commit 将 Index暂存区 中的內容提交至Repository中，这样一样可以达到合并commit节点的效果（与上面--soft合并commit节点差不多，只是多了git add添加到暂存区的操作）；(2)移除所有Index暂存区中准备要提交的文件(Staged files)，我们可以执行 git reset HEAD 来 Unstage 所有已列入 Index暂存区 的待提交的文件。(有时候发现add错文件到暂存区，就可以使用命令)。(3)commit提交某些错误代码，或者没有必要的文件也被commit上去，不想再修改错误再commit（因为会留下一个错误commit点），可以回退到正确的commit点上，然后所有原节点和reset节点之间差异会返回工作目录，假如有个没必要的文件的话就可以直接删除了，再commit上去就OK了。

```



### revert

```
撤销某次操作，此次操作之前和之后的commit和history都会保留，并且把这次撤销作为一次最新的提交
和reset区别:
1. git revert是用一次新的commit来回滚之前的commit，git reset是直接删除指定的commit。 
2. 在回滚这一操作上看，效果差不多。但是在日后继续merge以前的老版本时有区别。因为git revert是用一次逆向的commit“中和”之前的提交，因此日后合并老的branch时，导致这部分改变不会再次出现，但是git reset是之间把某些commit在某个branch上删除，因而和老的branch再次merge时，这些被回滚的commit应该还会被引入。 
3. git reset 是把HEAD向后移动了一下，而git revert是HEAD继续前进，只是新的commit的内容和要revert的内容正好相反，能够抵消要被revert的内容。

// 可以撤销指定的提交
git revert <commitId>
// 可以撤销不包括 commit1，但包括 commit2 的一串提交
git revert <commit1>...<commit2>
// revert 命令会对每个撤销的 commit 进行一次提交，--no-commit 后可以最后一起手动提交
git revert -n(--no-commit) <commit1>...<commit2>
// 退出 revert 过程，常在处理冲突出错时使用
git revert --abort
// 继续 revert 过程，常在处理完冲突时使用
git revert --continue
```



### checkout

```
checkout可以进行分支管理和还原工作区,这里只介绍还原工作区相关功能
原理是把工作区更新为HEAD指针
这里的"--"主要是区分文件名和分支名

// 切换到指定 commit
git checkout <commitId>
// 放弃工作区单个文件的变更，默认从暂存区检出该文件，如果暂存区为空，则该文件会回滚到最近一次的提交状态
git checkout -- <filepath>
// 放弃工作区单个文件的变更，从版本库复制到暂存区再到工作区 
// 不加具体文件路径就和git reset --hard一样
git checkout HEAD -- <filepath>
// 放弃工作区所有文件的变更（不包含未跟踪的)
git checkout .

```



### restore

```
分担了checkout更新工作区的功能的新命令

// 默认从暂存区恢复工作区, 如果暂存区为空, 则该文件会回滚到最近一次的提交状态
// 和git checkout -- <filepath>一样
git restore <filepath>

// --staged 从HEAD恢复暂存区
git restore --staged <filepath>

// --staged + --worktree 同时恢复暂存区和工作区, --source 可以指明源头
// 和git checkout HEAD -- <filepath>一样
git restore --source=HEAD --staged --worktree <filepath>

```



### clean

```
用来从工作目录中删除所有没有 tracked 过的文件，git clean 经常和 git reset --hard 一起结合使用。记住 reset 只影响被 track 过的文件，所以需要 clean 来删除没有 track 过的文件。
// 展示哪些文件会被删除，但不真正删除文件
git clean -n
// 删除指定目录（默认当前目录）下所有没有 track 过的文件，但不会删除 .gitignore 文件里面指定的文件夹和文件
git clean -f (<path>)
// 删除指定目录（默认当前目录）下所有没有 track 过的文件，包括 .gitignore 文件里面指定的文件夹和文件
git clean -fx (<path>)
// 删除指定目录（默认当前目录）下所有没有被 track 过的文件和文件夹，但不会删除 .gitignore 文件里面指定的文件夹和文件
git clean -df (<path>)
// 删除指定目录（默认当前目录）下所有没有被 track 过的文件和文件夹，包括 .gitignore 文件里面指定的文件夹和文件
git clean -dfx (<path>)s
```



### stash

```
// 保存当前工作进度（工作区和暂存区），将工作区和暂存区恢复到修改之前。默认 message 为当前分支最后一次 commit 的 message
git stash
// 保留已在暂存区的进度（不放到栈中）并保存工作区的进度
git stash --keep-index
// -u：保存未被git追踪的文件
git stash -u
// 作用同上，message 为此次进度保存的说明
git stash save message
// 显示此次工作进度做了哪些文件行数的变化，此命令的 stash@{num} 是可选项，不带此项则默认显示最近一次的工作进度相当于 git stash show stash@{0}
git stash show stash@{num}
// 显示此次工作进度做了哪些具体的代码改动，此命令的 stash@{num} 是可选项，不带此项则默认显示最近一次的工作进度相当于 git stash show stash@{0} -p
git stash show stash@{num} -p
// 显示保存的工作进度列表，编号越小代表保存进度的时间越近
git stash list
// 恢复工作进度到工作区，此命令的 stash@{num} 是可选项，在多个工作进度中可以选择恢复，不带此项则默认恢复最近的一次进度相当于 git stash pop stash@{0}
git stash pop stash@{num}
// 恢复工作进度到工作区且该工作进度可重复恢复（没有出栈），此命令的 stash@{num} 是可选项，在多个工作进度中可以选择恢复，不带此项则默认恢复最近的一次进度相当于 git stash apply stash@{0}
git stash apply stash@{num}
// 删除一条保存的工作进度，此命令的 stash@{num} 是可选项，在多个工作进度中可以选择删除，不带此项则默认删除最近的一次进度相当于 git stash drop stash@{0}
git stash drop stash@{num}
// 删除所有保存的工作进度
git stash clear
```



### reflog

```
// 查看git本地日志，跟踪更新HEAD的更改，常用来恢复本地错误操作;允许恢复任何提交，即使它没有被任何分支或标签引用。
git reflog
```



### fsck

```
常用来恢复本地错误操作。只要是对 HEAD 进行操作的错误，一般情况下 reflog 都能够恢复，然而有些错误并不会对 HEAD 进行操作，例如将部分代码 stash 之后又不小心 drop 或 clear 掉了，那么此时 HEAD 并没有发生变化，reflog 对此类错误是无能为力的，这个时候 fsck 就可以派上用场了，该命令会更加底层，即直接检查 git 中的 blob，tree 和 commit 对象并找到悬空的对象，找到后即可以通过 git show <commitId> 来查看该 commit 是否是被误操作删掉的 commit，如果是的话知道这个 commitId 无论如何都可以恢复了，不论用 merge 还是 cherry-pick 还是 checkout 等

git fsck
```



### gc

```
对git进行垃圾回收（如reset和rebase造成的孤立引用，但detached提交不会删除）和对象压缩，会在pull,merge,rebase,commit等常用命令调用时自动运行。大部分时候可以不用手动运行，特别是想恢复孤立引用时。
git gc
```





## 分支管理

### branch

```
// 查看本地分支
git branch
// 可以查看本地分支对应的远程分支
git branch -vv 
// 查看远程版本库分支列表
git branch -r
// 查看所有分支列表，包括本地和远程
git branch -a
// 在当前位置新建 name 分支
git branch <name>
// 在指定 commit 上新建 name 分支
git branch <name> <commitId>
// 强制创建 name 分支，原来的分支会被新建分支覆盖，从而迁移 name 分支指针
git branch -f <name>
// 删除 dev 分支，如果在分支中有一些未 merge 的提交则会失败
git branch -d dev
// 强制删除 dev 分支
git branch -D dev
// 重命名分支
git branch -m <oldName> <newName>
```



###  checkout

```
checkout可以进行分支管理和还原工作区,这里只介绍分支管理相关功能
// 切换到指定分支
git checkout <branch>
// 在当前位置新建分支并切换
git checkout -b <branch>
// 在指定 commit 上新建分支并切换
git checkout -b <branch> <commitId>
// 切换到指定 v1.2 的标签
git checkout tags/v1.2
// 切换到指定 commit
git checkout <commitId>
// 基于当前所在分支新建一个赤裸分支，该分支没有任何的提交历史但包含当前分支的所有内容，相当于 git checkout -b <new_branch> 和 git reset --soft <firstCommitId> 两条命令 
git checkout --orphan <new_branch>
```



### switch

```
分担了checkout分支管理的功能的新命令
// 切换到指定分支
git switch <branch>
// 在当前位置新建分支并切换
git switch -c <branch>
```



### merge

```
// 合并某个分支
git merge <branch>
// 退出 merge 过程，常在处理冲突出错时使用
git merge --abort
// 继续 merge 过程，常在处理完冲突时使用
git merge --continue
// 合并分支上的多个 commit 合并成一个 commit 放在当前分支上,之后手动提交。相比 rebase 的 squash，其主要区别是该 commit 的 author 是当前分支的，而不是被合并的分支的。
git merge --squash <branch>
// fast-forward 合并时再提交一个 commit (fast-forward本身只移动指针，不生成commit) 
git merge --no-ff <branch>
```



### rebase

[参考博客](https://www.jianshu.com/p/6960811ac89c)



```
git merge 操作合并分支会让两个分支的每一次提交都按照提交时间（并不是push时间）排序，并且会将两个分支的最新一次commit点进行合并成一个新的commit，最终的分支树呈现非整条线性直线的形式

git rebase操作实际上是将当前执行rebase分支的所有基于原分支提交点之后的commit打散成一个一个的patch，并重新生成一个新的commit hash值，再次基于原分支目前最新的commit点上进行提交，并不根据两个分支上实际的每次提交的时间点排序，rebase完成后，切到基分支进行合并另一个分支时也不会生成一个新的commit点，可以保持整个分支树的完美线性

// 变基某个分支
git rebase <branch>
// 退出 rebase 过程，常在处理冲突出错时使用
git rebase --abort
// 继续 rebase 过程，常在处理完冲突时使用
git rebase --continue
```



### cherry-pick

```
// 将指定的提交应用于当前分支
git cherry-pick <commitId>
// 将指定分支的最近一次提交应用于当前分支
git cherry-pick <branch>
// 将指定的两个提交先后应用于当前分支
git cherry-pick <commitId1> <commitId2>
// 将指定范围内的多个提交(不包含 commitId1，包含 commitId2)先后应用于当前分支
git cherry-pick <commitId1>..<commitId2>
// 将指定范围内的多个提交(包含 commitId1，包含 commitId2)先后应用于当前分支
git cherry-pick <commitId1>^..<commitId2>
// 退出 cherry-pick 过程，常在处理冲突出错时使用
git cherry-pick --abort
// 继续 cherry-pick 过程，常在处理完冲突时使用
git cherry-pick --continue
```





## 查看状态和日志

### status

```
// 显示当前工作树、暂存区和版本库HEAD之间的差异
git status
// -s：简短格式
git status -s
// 展示中包含被忽略的文件
git status --ignored
```



### diff

```
// 查看工作区与暂存区所有文件的变更
git diff
// 查看暂存区与最后一次 commit 之间所有文件的变更
git diff --cached
// 查看工作区与最后一次 commit 之间所有文件的变更
git diff HEAD
// 查看两次 commit 之间的变动
git diff <commitId>...<commitId>
// 查看两个分支上最后一次 commit 的内容差别
git diff <branch>...<branch>
```



### log

```
// 默认显示：commit 哈希id，提交的Author信息，提交的日期和时间，commit info信息
git log

--oneline: 只显示提交的 SHA1 值前几位和提交信息
--graph：绘制树形图
--stat：输出文件增删改的统计数据
-p：可以看到每个文件更为详细的修改内容，以diff的形式给出
--grep <key>：搜索commit信息中带关键字的commit
-S <key>: 从更改内容中查找包含指定字符串的 commit ，比如想查找哪些 commit 对代码中名为 <key> 的函数进行了更改
--author：限定作者
--after和--before：限定日期
--decoreate：显示branch和tag
--pretty：定制格式
```



### show

```
// 用来显示某个对象的具体信息和元信息
git show
git show <commitId>
git show <branch>
git show <tag>
```



### bisect

```
用于在 git 历史里面二分查找有 bug 的 commit。这是找 bug 时一个非常有用的命令，可以快速定位出引入 bug 的 commit。

// 开始查找，切换到中间那次提交
git bisect start [endCommitId] [startCommitId]
// 标记这次提交正确
git bisect good
// 标记这次提交错误
git bisect bad
// 退出查找，回到最近一次提交
git bisect reset
```



### grep

```
// 检索所有包含指定关键字的文件
git grep text
// 检索关键字出现在哪一行
git grep -n text
// 统计每一个文件中检索到指定关键字的行数
git grep -c text
```



### blame

```
// 用于查看某个文件的每一行内容由谁所写，类似于 IDEA 的 annotate 功能
git blame <file>
```



## 远程操作

### remote

```
// 显示远程仓库信息(-v 详细信息)
git remote -v
// 添加远程仓库关联
git remote add <name> <url>
// 删除远程仓库关联
git remote remove <name>
// 更名远程仓库关联
git remote rename <old_name> <new_name>
// 显示某个远程仓库的信息
git remote show <name>
// 更新远程仓库 url
git remote set-url <name> <new_url>
```



### fetch

```
将远程仓库的分支更新到本地的远程仓库副本中，本地仓库的分支不受fetch操作的影响。

// 取回指定远程仓库的所有分支
git fetch <remote> 
// 取回指定远程仓库的指定分支
git fetch <remote>  <remote_branch>:<local_branch>
// 取回所有远程仓库的所有分支
git fetch --all
```



### pull

 ```
// 相当于 fetch + merge, 这样可能会产生冲突，需要手动解决
git pull <remote> <remote_branch>:<local_branch>
// 相当于 fetch + rebase
git pull --rebase <remote> <remote_branch>:<local_branch>
 ```



### push

```
// 向远程推送指定分支，如果本地和远程分支名相同，则可以省略冒号
git push <remote> <branch>:<remote_branch>
// 向远程推送指定分支并绑定
git push -u <remote> <branch>
// 删除远程分支
git push <remote> :<branch>
// 向远程推送 tag
git push <remote> <tagName>
// 删除远程 tag
git push <remote> :<tagName>
// 向远程推送本地所有 tag
git push --tags <remote>
```



## 参考资料

官方文档

[Pro Git](https://git-scm.com/book/en/v2)

[33 条常用 git 命令详解](https://github.com/OneSizeFitsQuorum/git-tips)

各类blog