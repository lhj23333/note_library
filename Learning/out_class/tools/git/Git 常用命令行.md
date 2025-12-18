---
share_link: https://share.note.sx/ukg6tnms#c80oZyzMF/ecXIfHnCyZjMBFPVAsN1vi5o0eOXZTLdE
share_updated: 2025-03-08T11:45:09+08:00
---
### 1. 创建 Git 仓库：

- 该操作主要内容为，在所需 Git 文件夹下创建 `.git` 文件目录，以配置该文件仓库的 git 配置
<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250105171838.png" width = "300 px"/>
</div>
<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250105171850.png" width = "300 px"/>
<br>
<span style="font-size:10px; color:gray;">图 1 . git 配置内容</span>
</div>
命令行：

（1）创建⼀个 Git 本地仓库对应的命令为  `git init`，注意命令要在⽂件⽬录下执⾏

```shell
git init
```




### 2. Git 配置：

- 在安装 Git 后⾸先要做的事情是设置你的用户名称和 e-mail 地址

命令行：

（1）配置用户名和 e-mail 地址命令：

``` shell
git config [--global] user.name "Your Name"

git config [--global] user.email "email@example.com" 
```

> [!Tip] 
> 其中  -- global  是⼀个可选项。如果使⽤了该选项，表⽰这台机器上所有的 Git 仓库都会使⽤这个配置。如果你希望在不同仓库中使⽤不同的  name  或  e-mail ，可以不要  -- global  选项，但要注意的是，执⾏命令时必须要在仓库⾥。

（2）查看 Git 配置命令：

```shell
git config -l
```

（3）删除对应 Git 配置命令：

```shell
git config [--global] --unset user.name

git config [--global] --unset user.email
```



### 3. Git 版本上传：

> [!NOTE] Git 工作流程：
> 1. 在工作区修改文件。
> 2. 将你想要下次提交的更改选择性地暂存 
> 3. 提交更新，找到暂存区的文件，将快照永久性存储到 Git 目录

命令行：

（1）添加文件到暂存区：

```shell
# git add xxx
git add file_x   	# 添加一个或多个文件到暂存区
git add [dir]	 	# 添加指定目录到暂存区
git add .			# 添加当前目录下的所有文件改动到暂存区
```

（2）提交暂存区文件到版本库：
``` shell
# git commit

git commit -m "message"				# 提交暂存区全部内容到版本仓库中
git commit file_1... -m "message" 	# 提交暂存区的指定⽂件到版本库
```




### 4. Git 版本回退：

（1）回退版本：

> [!Note] 版本回退的本质
> “回退”本质是要将版本库中的内容进⾏回退，⼯作区或暂存区是否回退由命令参数决定

- `git reset`：恢复整个工作区

```shell
# git reset

git reset --soft
git reset --mixed
git reset --hard
```
- `--mixed` 为默认选项，使⽤时可以不⽤带该参数。该参数将暂存区的内容退回为指定提交版本内容，⼯作区⽂件保持不变。
- `--soft`  参数对于⼯作区和暂存区的内容都不变，只是将版本库回退到某个指定版本。 
- `--hard` 参数将暂存区与⼯作区都退回到指定版本。切记⼯作区有未提交的代码时不要⽤这个命令，因为⼯作区会回滚，你没有提交的代码就再也找不回了，所以使⽤该参数前⼀定要慎重

#### 4.1 仅撤销最新提交，但保留修改

- 如果你提交（`git commit`）了代码但还没推送，想撤销提交但保留修改：

```shell
git reset --soft HEAD~1
```
- `--soft`：仅撤销 `commit`，不影响工作区和暂存区，代码仍可修改和重新提交

#### 4.2 彻底回退到某个历史版本

- 如果你想让当前分支回退到某个具体的提交（但会完全抹掉之后的提交），可以使用：

```shell
git reset --hard <commit-hash>
```
- `--hard`：会直接重置提交记录、暂存区和工作区，所有更改都会丢失
- 注意：
	- 如果当前 `commit` 已经被 `push` ，在回退后进行强制推送, 否则远程仓库仍然会保留原来的提交

```shell
git push --force
```

#### 4.3 回退到某个版本但保留修改

- 如果你希望回退到某个版本，但不丢失修改，可以使用：

```shell
git reset --mixed <commit-hash>
```
- `--mixed`：回退 `commit` 记录，同时保留工作区的修改（不影响本地文件）

#### 4.4 回退单个文件到历史版本

- 如果你只想让某个文件回退到某个历史版本，而不影响整个项目：

```shell
git restore --source=<commit-hash> -- <file>
```

#### 4.5 回退已经推送的提交

- 如果你已经 `git push` 了错误的提交，需要回退，步骤如下：

（1）**只修改最新一次提交**：

- 如果只是提交信息写错了，可以使用：

```shell
git commit --amend -m "正确提交信息"
git push --force
```

（2）**彻底回退远程提交**：

- 如果你已经 `git push` 了多个错误提交，但想回退，可以使用

```shell
git reset --hard <commit-hash>
git push --force
```

- 远程仓库将回退到指定提交，并删除之后的提交

#### 4.6 撤销回退

- 如果你误用了 `git reset --hard`，导致代码丢失，可以使用：

```shell
git reflog
```

- 它会列出 `Git` 的历史操作，找到 `reset` 前的 `HEAD` 位置，例如：

```pgsql
abc123 HEAD@{1}: reset: moving to abc123
def456 HEAD@{0}: commit: some recent commit
```

- 要恢复回退前的状态：

```shell
git reset --hard def456
```

- 这样就可以撤销 `git reset --hard` 的操作




### 5. Git 版本分支：

（1）`git branch` ：分支查询与新建分支

```shell
git branch [BRANCH_NAME]				
```
- 通过 `git branch` 我们可以查看到当前 `git` 所在分支

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250308111216986.png" width = "600 px"/>
</div>

- 通过 `git branch` 并按下 `Tab` 键，可以查看到版本库所有的 `git` 分支

<div style = "text-align: center;">
<img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250308111806895.png" width = "650 px"/>
</div>

- 通过 `git branch [BRANCH_NAME]` 可以创建新的版本分支，`BRANCH_NAME` 须是添加的分支名称

---
（2）`git checkout` 切换分支

```shell
git checkout [BRANCH_NAME]			## 切换分支
```

- 通过 `git checkout [BRANCH_NAME]` 命令，可以实现切换 `git` 分支

```shell
git checkout master

M       open_source/linux/linux-5.10.y/arch/arm64/Makefile
M       open_source/u-boot/Makefile
M       open_source/u-boot/u-boot-2022.07/Makefile
M       smp/a55_linux/source/bsp/Makefile
Switched to branch 'master'
Your branch is up to date with 'origin/master'.
```

（3）切换分支并新建分支：

```shell
git checkout -b [BRANCH_NAME]
```

（4）合并分支

```shell
git merge [BRANCH_NAME] 			# 将另外一个分支的代码，合并到当前分支
```

> [!Tip] 分支合并情形
> 1. 合并没有分叉的分支:
> 当前开发分支与 `master` 目标分支之间不存在任何分叉
> <div style = "text-clign: center;"><img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250105183544.png" width = "400 px" /> </div>
> 此时，git 会直接将目标分支 `master` 的指针向前挪动到指向开发分支的头部，这个指针移动操作叫做 `fast forward`
> <div style = "text-clign: center;"><img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250105183945.png" width = "400 px" /> </div>
> 2. 合并包含分叉的分支:
> 当前开发分支与 `master` 目标分支之间存在分叉，两个分支的头部提交节点都不在另一个分支的提交历史中，如下图所示
>  <div style = "text-clign: center;"><img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250105184139.png" width = "400 px" /> </div>
>  git 会帮助创建一个新的合并提交，以合并两个分支中的不同内容，如下图所示
>    <div style = "text-clign: center;"><img src = "https://obsidian-picgo-1331635585.cos.ap-guangzhou.myqcloud.com/PicGo_pictures/20250105184443.png" width = "400 px" /> </div>
>   这种情况下，Git 只能试图把各⾃的修改合并起来，但这种合并就可能会有冲突




### 6. 其余常用命令：

- `git status`：查看 `git` 仓库在上次提交之后对文件的修改

```shell
git status

On branch master
Your branch is up to date with 'origin/master'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   open_source/linux/linux-5.10.y/arch/arm64/Makefile
        modified:   open_source/u-boot/Makefile
        modified:   open_source/u-boot/u-boot-2022.07/Makefile
        modified:   smp/a55_linux/source/bsp/Makefile

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        open_source/busybox/busybox-1.34.1/editors/awk.c.rej
        open_source/busybox/busybox-1.34.1/libbb/xconnect.c.rej
        open_source/busybox/busybox-1.34.1/loginutils/getty.c.rej
        open_source/busybox/busybox-1.34.1/networking/nslookup.c.rej

no changes added to commit (use "git add" and/or "git commit -a")
```

- 通过 `git status` 我们可以看到工作区与代码仓库中的修改状态，帮助我们找到在开发过程中主要修改的代码文件。
- 有时候项目编译的过程中会产生特别多的中间文件，所以在 `git status` 之前，最好能进行一些 `clean` 的操作，不然你就会收获一堆 `Untracked files` 信息

---
- `git diff [file]`：显示暂存区和工作区文件之间的差异

``` shell
git diff open_source/u-boot/Makefile

diff --git a/open_source/u-boot/Makefile b/open_source/u-boot/Makefile
index 31e488e6c..628f26722 100644
--- a/open_source/u-boot/Makefile
+++ b/open_source/u-boot/Makefile
@@ -52,8 +52,6 @@ endif
        cd $(BUILD_DIR)/$(UBOOT_VER) && \
                $(MAKE) $(CROSS) distclean >/dev/null;

-
-
        $(MAKE) -C $(BUILD_DIR)/$(UBOOT_VER) LLVM=$(LLVM) $(CROSS) $(UBOOT_CONFIG)

        echo "CONFIG_VENDER_UBOOT_NAME=\"$(CHIP)\"" >> $(BUILD_DIR)/$(UBOOT_VER)/.config
```

- 从前面的 `git status` 看到对比上一版本代码有进行修改的文件，再通过 `git diff [file]` 我们就可以对指定文件查看所修改的内容

```shell
git diff > xxx.patch
git apply xxx.patch
```

- 同时我们可以通过 `git diff [file] > xxx.patch` 命令将指定文件（或整个工程）修改的内容输出到 `xxx.patch`（也就是我们常说的补丁） 中进行查看修改处内容信息，或也可将该 `.patch` 文件传给其他开发人员，该人员可通过 `git apply xxx.patch` 直接将补丁应用到当前 `Git` 工作目录

---
- `git log`：查看 `git` 提交日志

```shell
git log

commit cd7d9d39a0d557e5b4c4a27969abac577a40a607 (HEAD -> demo_base, origin/demo_base)
Author: linhaojia <linhaojia@witstech.cn>
Date:   Fri Mar 7 18:17:15 2025 +0800

    修改可定制化配置 Uboot、Kernel

commit 736b3b68311c561ec43d93524469b219f2bdbd20
Author: lijiaming <lijiaming@witstech.cn>
Date:   Mon Feb 10 17:08:50 2025 +0800

    提交编译中权限问题导致编辑失败文件
```

- 通过 `git log` 命令，可以查看到当前分支（`branch`）的 `commit` 提交日志
- 其中查看到当前代码仓库中
	- `Author：` 代码版本 `commit` 用户以及其对应邮箱
	- `Data`：代码版本 `commit` 时间
	- `commit xxx`：`commit-hash` （重要哦！！)

---
-  `git cherry-pick`：提炼提交

```shell
git cherry-pick <commit-hash>
```

- `git cherry-pick` 命令允许选择特定的提交并将其应用到当前分支
- 在前面 `git log` 我们可以查询到提交 `commit` 的 `commit-hash`，再通过 `git chery-pick <commit-hash>` 就可以实现从一个分支移植特定修改到另一个分支