---
share_link: https://share.note.sx/bp91cqth#cfiiR9qCIdPaURgR+yXhr46+gmK/M4zbziHVNBRuhb4
share_updated: 2025-02-05T17:20:15+08:00
---
# Git

## 1.1 关于版本控制

***什么是“版本控制”，你为什么要关心？***

**版本控制**（英语：Version control）是维护工程[蓝图](https://zh.wikipedia.org/wiki/藍圖)的标准做法，能追踪工程蓝图从诞生一直到定案的过程。此外，版本控制也是一种[软件工程](https://zh.wikipedia.org/wiki/軟體工程)技巧，借此能在软件开发的过程中，确保由不同人所编辑的同一程序文件都得到同步。

如果你是一名平面或网页设计师，并且希望保留图像或布局的每个版本（这是你肯定想要的）

版本控制系统（VCS）是一个非常明智的选择。它允许你将选定的文件恢复到以前的状态，将整个项目恢复到以前的状态，比较随时间的变化，查看谁最后一次修改了可能导致问题的东西，谁引入了问题，何时引入，等等。使用VCS通常也意味着如果你搞砸了事情或丢失了文件，**你可以很容易地恢复**。此外，你可以用很少的开销获得所有这些

##  1.2 Git的历史

和许多伟大的事情一样，Git始于激烈的争议。

Linux内核是一个相当大范围的开源软件项目。在Linux内核维护的早期（1991-2002年），对软件的更改以补丁和存档文件的形式传递。2002年，Linux内核项目开始使用名为 BitKeeper 的这个非开源但是有条件免费的DVCS（*Distributed version control systems*）。

![](data:image/jpeg;base64,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)



Linus对 BitKeeper 的评价是 **the best tool for the job**  ——这是个最后在 free 和 open source 社区甚至之外都引起了广泛瞩目的举动。

有人一直反对Linus使用BitKeeper－－想想看，Linux是free/open source软件的旗舰产品，却在使用一个非free/open source软件

![](data:image/jpeg;base64,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)

GNU (GNU's Not Unix) 的传奇人物，那个看上去和耶稣有几分相似的大胡子 Richard Stallman 就是其中之一。

2005年，Linux内核社区和开发BitKeeper的商业公司之间的关系破裂，该工具的免费状态被撤销。这促使Linux开发社区（尤其是Linux的创建者 Linus Torvalds ）根据他们在使用BitKeeper时学到的一些经验教训开发了自己的工具。新系统的一些目标如下：

- 速度 (Speed)
- 设计简单 (Simple design)
- 对非线性开发的强大支持（并行分支）
- 完全分布 (distributed)
- 能够有效地处理大型项目，如Linux内核



自2005年诞生以来，Git已经发展成熟，易于使用，但仍保留了这些初始品质。它速度惊人，在大型项目中非常高效，是首选的用于非线性开发的分支系统。

## 1.3 Git基础

![](data:image/png;base64,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)

**Git** 是一个 **分布式版本控制系统** ，用于追踪源代码（或其他文件、文件夹）改动的工具。

通过一系列的**快照**将某个文件夹及其内容保存了起来，每个快照都包含了顶级目录中所有的文件或文件夹的完整状态。同时它还维护了快照创建者的信息以及每个快照的相关信息等等。

**快照**

在 Git 的术语里，**文件被称作 Blob对象**（Binary Large Object），也就是一组数据。**目录则被称之为“树”**，它将名字与 Blob 对象或树对象进行映射（使得目录中可以包含其他目录）。快照则是被追踪的最顶层的树。例如，一个树看起来可能是这样的：

```
<root> (tree)
|
+- foo (tree)
|  |
|  + bar.txt (blob, contents = "hello world")
|
+- baz.txt (blob, contents = "git is wonderful")
```

这个顶层的树包含了两个元素，一个名为 “foo” 的树（它本身包含了一个 blob 对象 “bar.txt”），以及一个 blob 对象 “baz.txt”。

	在Git术语中
	文件被称为 Blob 对象，即 Blob -> 文件
	目录被称为 tree 对象，即 tree -> 文件夹（目录）
	Blob / tree 在 Git 中都会有个实例来对应本地的文件以及文件夹
	同时也使得目录可以包含其他目录

### 1.3.1 历史记录建模：关联快照

版本控制系统和快照有什么关系呢？线性历史记录是一种最简单的模型，它包含了一组按照时间顺序线性排列的快照。不过出于种种原因，Git 并没有采用这样的模型。

	自己理想的Git管理仓库的方式是一个链式的结构，即：
	master（boot node）<- add_node_1 <- add_node_2 <- add_node_3
	在一个基础的节点之上， 每一次迭代文件，都会进行增删选择，将增删的内容节点链接在前
	向节点后，这样可以做到比较"明智"，同时最小存储空间的文件储存
	
	但 Git 却不是这样子做的，因为这样子并不安全，磁盘本身并不是一个绝对稳定的存储容器
	当出现一点剧烈的碰撞时，可能就会损失一定的字节，如果 Git 是一个链式结构的话，这个
	损失出现在链式的中间时，这个错误是致命的，将会导致后面所有节点全都失效
	
	所以 Git 采取的措施，是最笨的方法，就是每一次迭代 commit 都将当前本地仓库
	中的所有文件都打包进行上传作为一个历史版本，同时 Git 会比较前后两个版本之间的 
	diff 来做历史更改的显示


在 Git 中，历史记录是一个由快照组成的有向无环图。有向无环图，听上去似乎是什么高大上的数学名词。不过不要怕，您只需要知道这代表 Git 中的每个快照都有一系列的“父辈”，也就是其之前的一系列快照。注意，快照具有多个“父辈”而非一个，因为某个快照可能由多个父辈而来。例如，经过合并后的两条分支。

	每一个 commit 可以称为快照，其前面具有一个至多个 "父辈"
	但是这个 “父辈” 的关系的意义仅仅是表现在这部分 commit 仅仅是在当前这个 commit 之
	前的关系，但后续版本管理切换并不是涉及到这个父辈时间逻辑（特别注意!!）
 
在 Git 中，这些快照被称为“commit”。通过可视化的方式来表示这些历史提交记录时，看起来差不多是这样的：

```
o <-- o <-- o <-- o
            ^
             \
              --- o <-- o
```

	 这里展示的是一个 Git 历史版本的分支结构

`o` 表示一次提交（快照）。箭头指向了当前提交的父辈（这是一种“在…之前”的关系）。在第三次提交之后，历史记录分岔成了两条独立的分支。这可能因为此时需要同时开发两个不同的特性，它们之间是相互独立的。开发完成后，这些分支可能会被合并并创建一个新的提交，这个新的提交会同时包含这些特性。新的提交会创建一个新的历史记录，看上去像这样（最新的合并提交用粗体标记）：

```
o <-- o <-- o <-- o <----  o 
            ^            /
             \          v
              --- o <-- o
```

	这里展示的是一个 Git 版本分支结构进行合并的操作


### 1.3.2 数据模型及其伪代码表示

以伪代码的形式来学习 Git 的数据模型，可能更加清晰：

```
// 文件就是一组数据
type blob = array<byte>

// 一个包含文件和目录的目录
type tree = map<string, tree | blob>

// 每个提交都包含一个父辈，元数据和顶层树
type commit = struct {
    parents: array<commit>
    author: string
    message: string
    snapshot: tree
}
```

这是一种简洁的历史模型。

	这部分主要讲解的是一个 Git commit 的数据模型
	type blob = array<byte>
	blob 是文件内容进行二进制转换的的bite数组
	
	 type tree = map<string, tree | blob>
	 这里讲解的是在 git 中通过名字 string 与 tree/blob 做的一个映射，同时这个tree包含了子目录
	 以及文件内容
	 
	type commit = struct {
    parents: array<commit>
    author: string
    message: string
    snapshot: tree
    }
    
    这部分讲解的是 commit 上报的结构体内容，其中包括
    parents: array<commit>  -- 文件内容
    author: string          -- 作者信息
    message: string         -- commit 提示信息
    snapshot: tree          -- 版本快照

### 1.3.3 对象和内存寻址

Git 中的对象可以是 blob、树或提交：

```
type object = blob | tree | commit
```

	 这里讲解的就是，Git的一个处理对象的类型，可以是 blob（文件）、tree（文件夹/目录）、commit
	（提交是数据结构体）
	这三者是有个递进的关系（思考.jpg）
	这涉及到了后面的一个每个对象都有一个对应的（唯一）哈希值，这个哈希值是用来对操作对象进行寻
	址，每一个对象仅有一个唯一的哈希值与其对应，很重要!!!
	
	延伸内容：为什么这个哈希值对于每个对象来说它是唯一的?? 它是怎么生成的
	接下来讲解一下哈希值的生成过程：
	输入 -> 哈希值生成器 -> 输入对应的哈希值 
	（你可以将这个哈希值生成器当作一个加密仪器，会将你想要发送的信息进行编码转换）
	
	哈希生成器中间的编码过程：
	1. 首先引入哈希算子 P = 1003
	2. 然后对于输入信息内容，该信息内容首先会被转换为 ASCII码，例如转换出来为 1234
	3. 然后进行SHA-1哈希计算， 
	 num = [(1 * 2^3) % p + (2 * 2^2) % p + (3 * 2^1) % p + (4 * 2^0) % p] % p
	即对信息内容转换后的ASCII码先进行位权计算，然后取哈希算子P的模，最后相加起来得到的SUM再取一
	次P的模
	4.将 num 值进行十六进制转换得到每个信息内容的所对应的唯一的哈希真值
	
	特点：
	1. 当你的哈希算子P越来越大的时候，不同信息内容所计算得出的哈希真值相同的可能性越小，（哈希碰
	撞的可能性越低），且当P > 10^40时，哈希碰撞的可能性逼近于0%（唯一性）
	2. 这个过程是一个不可逆的过程，即仅能从信息内容得出所对应的哈希真值，不能从哈希真值逆推出信息
	内容（安全性）

Git 在储存数据时，所有的对象都会基于它们的 [SHA-1 哈希](https://en.wikipedia.org/wiki/SHA-1) 进行寻址。

```
objects = map<string, object>

def store(object):
    id = sha1(object)
    objects[id] = object

def load(id):
    return objects[id]
```

	在这部分讲解的是一个通过哈希真值来寻址的过程
	objects = map<string, object>
	这是一个map映射谱结构，即 objects是一个map表，其中保存了string <-> object (可通过string
	找到对应的object，也可通过object找到对应的string，一一对应，唯一性)
	
	def store(object):
        id = sha1(object)
        objects[id] = object
    这里就是首先将 object（对象）的信息内容经过SHA-1哈希运算活得该object对象的哈希真值id
    再将objects结构中的键值string以哈希真值id来赋值，来实现object对象保存在对应哈希真值id地址
    下的objects位置中
    
    def load(id):
	    return objects[id]
    这部分代码即通过哈希真值id地址在objects中寻找该哈希真值所对应的object对象


Blobs、树和提交都一样，它们都是对象。当它们引用其他对象时，它们并没有真正的在硬盘上保存这些对象，而是仅仅保存了它们的哈希值作为引用。


例如，例子中的树（可以通过 **`git cat-file -p 698281bc680d1995c5f4caaf3359721a5a58d48d`** 来进行可视化），看上去是这样的：

```
100644 blob 4448adbf7ecd394f42ae135bbeed9676e894af85    baz.txt
040000 tree c68d233a33c5c06e0340e4c224f0afca87c8ce87    foo
```

	在这里可以看到当通过
	git cat-file -p 698281bc680d1995c5f4caaf3359721a5a58d48d
	指令来打开哈希真值id（698281bc680d1995c5f4caaf3359721a5a58d48d）下的object的时候
	可以看得到，该object中包含了两个内容
	100644 blob 4448adbf7ecd394f42ae135bbeed9676e894af85    baz.txt
	040000 tree c68d233a33c5c06e0340e4c224f0afca87c8ce87    foo
	
	这两个的类型可以看得出一个是blob文件（baz.txt），一个是tree文件夹（foo）
	他们同样也是git下的object对象，所以他们也会有对应的哈希真值id]
	object                  哈希真值id        
	baz.txt -> 4448adbf7ecd394f42ae135bbeed9676e894af85
	foo     -> c68d233a33c5c06e0340e4c224f0afca87c8ce87

树本身会包含一些指向其他内容的指针，例如 `baz.txt` (blob) 和 `foo` (树)。如果我们用 `git cat-file -p 4448adbf7ecd394f42ae135bbeed9676e894af85`，即通过哈希值查看 baz.txt 的内容，会得到以下信息：

```
git is wonderful
```

	然后我们再通过 git cat-file -p 这个指令来查看我们 baz.txt 文件中的内容
	输入 git cat-file -p 4448adbf7ecd394f42ae135bbeed9676e894af85 
	输出 git is wonderful
	这个即为 baz.txt 中的文件内容

### 1.3.4  引用

现在，所有的快照都可以通过它们的 SHA-1 哈希值来标记了。但这也太不方便了，谁也记不住一串 40 位的十六进制字符。

针对这一问题，Git 的解决方法是给这些哈希值赋予人类可读的名字，也就是引用（references）。引用是指向提交的指针。与对象不同的是，它是可变的（引用可以被更新，指向新的提交）。例如，**`master` 引用通常会指向主分支的最新一次提交**。

```
references = map<string, string>

def update_reference(name, id):
    references[name] = id

def read_reference(name):
    return references[name]

def load_reference(name_or_id):
    if name_or_id in references:
        return load(references[name_or_id])
    else:
        return load(name_or_id)
```

这样，Git 就可以使用诸如 “master” 这样人类可读的名称来表示历史记录中某个特定的提交，而不需要在使用一长串十六进制字符了。

		在这里很容易有一个误区，在 Git 中，master 并不是分支本质的存在，而是一个引用名称。更确切地
	说，Git 中的所谓“分支”本质上是一个指向某个提交（commit）的可移动指针，称为引用
	（reference）。
	
		就是 master 其实不是一个分支的名称，它本身应该是一个引用名称，我的理解就是，其实git它的仓库
	本身就是当你调用 git branch，应该不能说是创建新的分支，而是创建新的引用，当你对不同的引用通
	过 git push -u origin master（引用名称）执行时，实际上做的是将该引用的指针指向当前最新提
	交的commit节点，就是，其实git本身就是你每一次 commit 的时候，都会将 commit 的信息作为一个
	节点上传上去（所有的版本信息都会保存），只是你通过不断的移动你的引用（master）进行形成了一条
	条的分支
	

有一个细节需要我们注意， 通常情况下，我们会想要知道“我们当前所在位置”，并将其标记下来。这样当我们创建新的快照的时候，我们就可以知道它的相对位置（如何设置它的“父辈”）。在 Git 中，我们当前的位置有一个特殊的索引，它就是 “HEAD”。

## 1.4 Git基本操作

### 1.4.1 创建 Git 本地仓库   

创建⼀个 Git 本地仓库对应的命令为  git init ，注意命令要在⽂件⽬录下执⾏，例如：  

```shell
root@Chrix:/home/test/gittest# git init
hint: Using 'master' as the name for the initial branch. This default branch name
hint: is subject to change. To configure the initial branch name to use in all
hint: of your new repositories, which will suppress this warning, call:
hint: 
hint: 	git config --global init.defaultBranch <name>
hint: 
hint: Names commonly chosen instead of 'master' are 'main', 'trunk' and
hint: 'development'. The just-created branch can be renamed via this command:
hint: 
hint: 	git branch -m <name>
Initialized empty Git repository in /home/test/gittest/.git/
root@Chrix:/home/test/gittest# ll -a
total 12
drwxr-xr-x 3 root root 4096 Nov 16 13:03 ./
drwxr-x--- 3 test test 4096 Nov 16 13:02 ../
drwxr-xr-x 7 root root 4096 Nov 16 13:03 .git/
```

我们发现，当前⽬录下多了⼀个 .git 的隐藏⽂件

.git ⽬录是 Git 来跟踪管理仓库的，不要⼿动修改这个⽬录⾥⾯的⽂件，不然改乱了，就把 Git 仓库给破坏了。 

其中包含 Git 仓库的诸多细节  

``` shell
root@Chrix:/home/test/gittest# tree .git/
.git/
├── branches
├── config
├── description
├── HEAD
├── hooks
│   ├── applypatch-msg.sample
│   ├── commit-msg.sample
│   ├── fsmonitor-watchman.sample
│   ├── post-update.sample
│   ├── pre-applypatch.sample
│   ├── pre-commit.sample
│   ├── pre-merge-commit.sample
│   ├── prepare-commit-msg.sample
│   ├── pre-push.sample
│   ├── pre-rebase.sample
│   ├── pre-receive.sample
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

### 1.4.2 配置 Git  

**当安装 Git 后⾸先要做的事情是设置你的 用户名称 和 e-mail 地址 ！！！ **

``` shell
git config [--global] user.name "Your Name"

git config [--global] user.email "email@example.com" 
```

其中  -- global  是⼀个可选项。如果使⽤了该选项，表⽰这台机器上所有的 Git 仓库都会使⽤这个配置。如果你希望在不同仓库中使⽤不同的  name  或  e-mail ，可以不要  -- global  选项，但要注意的是，执⾏命令时必须要在仓库⾥。

查看配置命令为：  

```shell
git config -l
```

删除对应的配置命令为：

``` shell 
git config [--global]  --unset user.name

git config  [--global] --unset user.email 
```

### 1.4.3 认识⼯作区、暂存区、版本库

- ⼯作区 ：是在电脑上你要写代码或⽂件的⽬录。
- 暂存区：英⽂叫 stage 或 index。⼀般存放在  .git  ⽬录下的 index ⽂件（.git/index）中，我们把暂存区有时也叫作索引（index）。 
- 版本库：⼜名仓库，英⽂名  repository 。⼯作区有⼀个隐藏⽬录  .git ，它不算⼯作区，⽽是 Git 的版本库。这个版本库⾥⾯的所有⽂件都可以被 Git 管理起来，每个⽂件的修改、删除，Git都能跟踪，以便任何时刻都可以追踪历史，或者在将来某个时刻可以“还原”。

下⾯这个图展⽰了⼯作区、暂存区和版本库之间的关系：

![](https://git-scm.com/book/zh/v2/images/areas.png)

基本的 Git 工作流程如下：

1. 在工作区中修改文件。
2. 将你想要下次提交的更改选择性地暂存，这样只会将更改的部分添加到暂存区。
3. 提交更新，找到暂存区的文件，将快照永久性存储到 Git 目录

#### 1.4.3.1 lab0: 添加⽂件  

在包含 .git 的⽬录下新建⼀个 ReadMe ⽂件

我们可以使⽤ git add  命令可以将⽂件添加到暂存区：

- 添加⼀个或多个⽂件到暂存区： git add [file1] [file2] ...
- 添加指定⽬录到暂存区，包括⼦⽬录： git add [dir]
- 添加当前⽬录下的所有⽂件改动到暂存区： git add .

再使⽤ git commit 命令将暂存区内容添加到本地仓库中：

- 提交暂存区全部内容到本地仓库中:  git commit -m "message"  
- 提交暂存区的指定⽂件到仓库区： git commit [file1] [file2] ... -m "message"

注意 git commit  后⾯的 -m 选项，要跟上描述本次提交的 message，由⾃⼰完成，这部分内容绝对不能省略，并要好好描述，是⽤来记录你的提交细节，是给我们⼈看的。

``` shell
root@Chrix:/home/test/gittest# vim ReadMe 
root@Chrix:/home/test/gittest# cat ReadMe 
Hello World
root@Chrix:/home/test/gittest# git add ReadMe 
root@Chrix:/home/test/gittest# git commit -m "Init ReadMe"
[master (root-commit) 92bf0df] Init ReadMe
 1 file changed, 1 insertion(+)
 create mode 100644 ReadMe
```

git commit 命令执⾏成功后会告诉我们，1个⽂件被改动（就是我们新添加的ReadMe⽂件），插⼊了一⾏内容（ReadMe有一⾏内容）。 



然后我们还可以多次 add 不同的⽂件，⽽只 commit ⼀次便可以提交所有⽂件，是因为需要提交的⽂件是通通被 add 到暂存区中，然后⼀次性 commit 暂存区的所有修改。如：

``` shell
root@Chrix:/home/test/gittest# touch file1 file2 file3
root@Chrix:/home/test/gittest# ls
file1  file2  file3  ReadMe
root@Chrix:/home/test/gittest# git add .
root@Chrix:/home/test/gittest# git commit -m "add 3 files"
[master ab149d3] add 3 files
 3 files changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 file1
 create mode 100644 file2
 create mode 100644 file3
```

截⾄⽬前为⽌，我们已经能够将代码直接提交⾄本地仓库了。我们可以使⽤ `git log` 命令，来查看下历史提交记录：

```shell
root@Chrix:/home/test/gittest# git log 
commit ab149d32a2593635591fc48098b9365270542600 (HEAD -> master)
Author: Chris <2293611667@qqq.com>
Date:   Sat Nov 16 13:32:24 2024 +0800

    add 3 files

commit 92bf0df7a894dabd8ac5d83cd1d11d38c0f09207
Author: Chris <2293611667@qqq.com>
Date:   Sat Nov 16 13:30:12 2024 +0800

    Init ReadMe

```

该命令显⽰从最近到最远的提交⽇志，并且可以看到我们 commit 时的⽇志消息。 如果嫌输出信息太多，看得眼花缭乱的，可以试试加上 --pretty=oneline 参数：  

```shell
root@Chrix:/home/test/gittest# git log --pretty=oneline 
ab149d32a2593635591fc48098b9365270542600 (HEAD -> master) add 3 files
92bf0df7a894dabd8ac5d83cd1d11d38c0f09207 Init ReadMe
```

#### 1.4.3.2 查看 .git ⽂件  

```
root@Chrix:/home/test/gittest# tree .git/
.git/
├── branches
├── COMMIT_EDITMSG
├── config
├── description
├── HEAD
├── hooks
│   ├── applypatch-msg.sample
│   ├── commit-msg.sample
│   ├── fsmonitor-watchman.sample
│   ├── post-update.sample
│   ├── pre-applypatch.sample
│   ├── pre-commit.sample
│   ├── pre-merge-commit.sample
│   ├── prepare-commit-msg.sample
│   ├── pre-push.sample
│   ├── pre-rebase.sample
│   ├── pre-receive.sample
│   ├── push-to-checkout.sample
│   └── update.sample
├── index
├── info
│   └── exclude
├── logs
│   ├── HEAD
│   └── refs
│       └── heads
│           └── master
├── objects
│   ├── 55
│   │   └── 7db03de997c86a4a028e1ebd3a1ceb225be238
│   ├── 6f
│   │   └── 1bcc26900b167821d715d9e9a4dfb1496bac52
│   ├── 92
│   │   └── bf0df7a894dabd8ac5d83cd1d11d38c0f09207
│   ├── ab
│   │   └── 149d32a2593635591fc48098b9365270542600
│   ├── af
│   │   └── d91fd5e54ee72dd7a673da7a3642d060a1385b
│   ├── e6
│   │   └── 9de29bb2d1d6434b8b29ae775ad8c2e48c5391
│   ├── info
│   └── pack
└── refs
    ├── heads
    │   └── master
    └── tags

```

`index` 文件保存了你当前暂存的所有修改，Git 会将这个文件用作将来提交的参考。

`HEAD` 是 Git 的一个重要文件，指示当前分支的引用。它包含了当前工作目录指向的最新提交对象。通常，`HEAD` 会包含类似以下内容：

```
root@Chrix:/home/test/gittest/.git# cat HEAD 
ref: refs/heads/master
```

⽽默认的 master 分⽀，其实就是: 

```
root@Chrix:/home/test/gittest/.git# cat ./refs/heads/master 
ab149d32a2593635591fc48098b9365270542600
```

打印的就是当前最新的 commit id 。  

objects  为 Git 的对象库，⾥⾯包含了创建的各种版本库对象及内容。当执⾏  git add  命令时，暂存区的⽬录树被更新，同时⼯作区修改（或新增）的⽂件内容被写⼊到对象库中的⼀个新的对象中，就位于 ".git/objects" ⽬录下，让我们来看看这些对象有何⽤处

查找 object 时要将  commit id  分成2部分，其前2位是⽂件夹名称，后38位是⽂件名称。 

该类⽂件是经过  SHA-1 加密过的⽂件，这时候我们可以使⽤  git cat-file  命令来查看版本库对象的内容：

```
root@Chrix:/home/test/gittest# git cat-file -p ab149d32a2593635591fc48098b9365270542600
tree afd91fd5e54ee72dd7a673da7a3642d060a1385b
parent 92bf0df7a894dabd8ac5d83cd1d11d38c0f09207
author Chris <2293611667@qqq.com> 1731735144 +0800
committer Chris <2293611667@qqq.com> 1731735144 +0800

add 3 files
```

其中，还有⼀⾏tree ,我们使⽤同样的⽅法，看看结果：

```
root@Chrix:/home/test/gittest# git cat-file -p afd91fd5e54ee72dd7a673da7a3642d060a1385b
100644 blob 557db03de997c86a4a028e1ebd3a1ceb225be238	ReadMe
100644 blob e69de29bb2d1d6434b8b29ae775ad8c2e48c5391	file1
100644 blob e69de29bb2d1d6434b8b29ae775ad8c2e48c5391	file2
100644 blob e69de29bb2d1d6434b8b29ae775ad8c2e48c5391	file3
root@Chrix:/home/test/gittest# git cat-file -p 557db03de997c86a4a028e1ebd3a1ceb225be238
Hello World
```

#### 1.4.3.3 lab1: 修改⽂件  

我们将 ReadMe ⽂件进⾏⼀次修改：  

```shell
root@Chrix:/home/test/gittest# echo "Hello World" >> ReadMe 
root@Chrix:/home/test/gittest# cat ReadMe 
Hello World
Hello World
```

此时，仓库中的 ReadMe 和我们⼯作区的 ReadMe 是不同的，如何查看当前仓库的状态呢？ git status  

命令⽤于查看在你上次提交之后是否有对⽂件进⾏再次修改。

```shell
root@Chrix:/home/test/gittest# git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	modified:   ReadMe

no changes added to commit (use "git add" and/or "git commit -a")
```

上⾯的结果告诉我们，ReadMe 被修改过了，但还没有完成添加与提交。 

⽬前，我们只知道⽂件被修改了，如果能知道具体哪些地⽅被修改了，就更好了。有同学会说，我刚改的我知道呀！可是，你还记得你三天前写了什么代码吗？或者没写？  

```
root@Chrix:/home/test/gittest# git diff ReadMe
diff --git a/ReadMe b/ReadMe
index 557db03..8254f87 100644
--- a/ReadMe
+++ b/ReadMe
@@ -1 +1,2 @@
 Hello World
+Hello World
```

git diff [file]  命令⽤来显⽰暂存区和⼯作区⽂件的差异，显⽰的格式正是Unix通⽤的diff格式。也可以使⽤ git diff HEAD -- [file]  命令来查看版本库和⼯作区⽂件的区别  

知道了对 ReadMe 做了什么修改后，再把它提交到本地仓库就放⼼多了  

```
root@Chrix:/home/test/gittest# git add ReadMe 
root@Chrix:/home/test/gittest# git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
	modified:   ReadMe
```

git add 之后，就没有看到上⾯ 

`no changes added to commit (use "git add"and/or "git commit -a")` 

的消息了。接下来让我们继续  git commit  即可：

```
root@Chrix:/home/test/gittest# git commit -m "Update ReadMe file"
[master 897a471] Update ReadMe file
 1 file changed, 1 insertion(+)
```

#### 1.4.3.4 lab2: 版本回退

之前我们也提到过，Git 能够管理⽂件的历史版本，这也是版本控制器重要的能⼒。如果有⼀天你发现之前前的⼯作做的出现了很⼤的问题，需要在某个特定的历史版本重新开始，这个时候，就需要版本回退的功能了。执⾏ 

`git reset`  命令⽤于回退版本，可以指定退回某⼀次提交的版本。要解释⼀下“回退”本质是要将版本库中的内容进⾏回退，⼯作区或暂存区是否回退由命令参数决定

git reset  命令语法格式为： git reset [--soft | --mixed | --hard] [HEAD]  

- --mixed  为默认选项，使⽤时可以不⽤带该参数。该参数将暂存区的内容退回为指定提交版本内容，⼯作区⽂件保持不变。
- --soft  参数对于⼯作区和暂存区的内容都不变，只是将版本库回退到某个指定版本。 
- --hard  参数将暂存区与⼯作区都退回到指定版本。切记⼯作区有未提交的代码时不要⽤这个命令，因为⼯作区会回滚，你没有提交的代码就再也找不回了，所以使⽤该参数前⼀定要慎重

HEAD 说明：

可直接写成 commit id，表示指定退回的版本

- HEAD 表⽰当前版本 
- HEAD^ 上⼀个版本 
- HEAD^^ 上上⼀个版本 

以此类推... 

可以使⽤ ~ 表⽰： 

- HEAD~0 表⽰当前版本 
- HEAD~1 上⼀个版本 
- HEAD~2 上上⼀个版本 

以此类推... 

为了便于表述，⽅便测试回退功能，我们先做⼀些准备⼯作：更新3个版本的 ReadMe，并分别进⾏3次提交，如下表示

```
root@Chrix:/home/test/gittest# echo "version1" >> ReadMe 
root@Chrix:/home/test/gittest# git add .
root@Chrix:/home/test/gittest# git commit -m "version1"
[master bddaa9e] version1
 1 file changed, 1 insertion(+)
root@Chrix:/home/test/gittest# echo "version2" >> ReadMe 
root@Chrix:/home/test/gittest# git add .
root@Chrix:/home/test/gittest# git commit -m "version2"
[master 81848e6] version2
 1 file changed, 1 insertion(+)
root@Chrix:/home/test/gittest# echo "version3" >> ReadMe 
root@Chrix:/home/test/gittest# git add .
root@Chrix:/home/test/gittest# git commit -m "version3"
[master 8f1a4d5] version3
 1 file changed, 1 insertion(+)
 
root@Chrix:/home/test/gittest# git log --pretty=oneline 
8f1a4d5f3e3f048345715e73f425367e4c0e6e96 (HEAD -> master) version3
81848e6e9da06566a960dc7a0a2b44b26c80796b version2
bddaa9e4c83e7224b3d10df407fe89b7dab9c0dd version1
...
```

现在，如果我们在提交完 version3 后， 发现 version 3 编写错误，想回退到 version2，重新基于  version 2 开始编写。由于我们在这⾥希望的是将⼯作区的内容也回退到  version 2  版本，所以需要⽤到  --hard  参数，⽰例如下：  

```
root@Chrix:/home/test/gittest# git reset --hard 81848e6e9da06566a960dc7a0a2b44b26c80796b
HEAD is now at 81848e6 version2
root@Chrix:/home/test/gittest# cat ReadMe 
Hello World
Hello World
version1
version2
```

我们惊奇的发现，此时 ReadMe ⽂件的内容，已经回退到 version2 了！，当前，我们再次⽤ git log 

查看⼀下提交⽇志，发现 HEAD 指向了version2，如下所⽰： 

```
root@Chrix:/home/test/gittest# git log --pretty=oneline 
81848e6e9da06566a960dc7a0a2b44b26c80796b (HEAD -> master) version2
bddaa9e4c83e7224b3d10df407fe89b7dab9c0dd version1
897a471ee9338e7e433c4354fd6ac34666035961 Update ReadMe file
ab149d32a2593635591fc48098b9365270542600 add 3 files
92bf0df7a894dabd8ac5d83cd1d11d38c0f09207 Init ReadMe
```

到这⾥⼀般回退功能就演⽰完了，但现在如果我后悔了，想再回到 version 3 怎么办？我们可以继续使⽤  git reset  命令，回退到  version 3  版本，但我们必须要拿到  version 3  的  commit id  去指定回退的版本。 

但我们看到了  git log 并不能打印出  version 3 的 commit id 

Git 还提供了⼀个 git reflog 命令能补救⼀下，该命令⽤来记录本地的每⼀次命令。 

```
root@Chrix:/home/test/gittest# git reflog 
81848e6 (HEAD -> master) HEAD@{0}: reset: moving to 81848e6e9da06566a960dc7a0a2b44b26c80796b
8f1a4d5 HEAD@{1}: commit: version3
81848e6 (HEAD -> master) HEAD@{2}: commit: version2
bddaa9e HEAD@{3}: commit: version1
897a471 HEAD@{4}: commit: Update ReadMe file
ab149d3 HEAD@{5}: commit: add 3 files
92bf0df HEAD@{6}: commit (initial): Init ReadMe
```

这样，你就可以很⽅便的找到你的所有操作记录了，但 8f1a4d5 这个是啥东西？这个是  version3  

的 commit id 的部分。没错，Git 版本回退的时候，也可以使⽤部分 commit id 来代表⽬标版

本。示例如下：

```
root@Chrix:/home/test/gittest# git reset --hard 8f1a4d5
HEAD is now at 8f1a4d5 version3
root@Chrix:/home/test/gittest# cat ReadMe 
Hello World
Hello World
version1
version2
version3
root@Chrix:/home/test/gittest# git log --pretty=oneline 
8f1a4d5f3e3f048345715e73f425367e4c0e6e96 (HEAD -> master) version3
81848e6e9da06566a960dc7a0a2b44b26c80796b version2
bddaa9e4c83e7224b3d10df407fe89b7dab9c0dd version1
897a471ee9338e7e433c4354fd6ac34666035961 Update ReadMe file
ab149d32a2593635591fc48098b9365270542600 add 3 files
92bf0df7a894dabd8ac5d83cd1d11d38c0f09207 Init ReadMe
```


## 1.5 Git分支

几乎所有的版本控制系统都以某种形式支持分支。 使用分支意味着你可以把你的工作从开发主线上分离开来，以免影响开发主线。 在很多版本控制系统中，这是一个略微低效的过程——常常需要完全创建一个源代码目录的副本。对于大项目来说，这样的过程会耗费很多时间。

有人把 Git 的分支模型称为它的“必杀技特性”，也正因为这一特性，使得 Git 从众多版本控制系统中脱颖而出。 为何 Git 的分支模型如此出众呢？ Git 处理分支的方式可谓是难以置信的轻量，创建新分支这一操作几乎能在瞬间完成，并且在不同分支之间的切换操作也是一样便捷。 与许多其它版本控制系统不同，Git 鼓励在工作流程中频繁地使用分支与合并，哪怕一天之内进行许多次。 理解和精通这一特性，你便会意识到 Git 是如此的强大而又独特，并且从此真正改变你的开发方式。

### 1.5.1 新建分支

```shell
git checkout -b [BRANCH_NAME] # 在当前版本切换并新建分支
git branch [BRANCH_NAME] # 直接新建分支
```


```shell
root@Chrix:/home/test/gittest# git branch  #查看当前本地所有分⽀
* master
root@Chrix:/home/test/gittest# git branch dev
root@Chrix:/home/test/gittest# git branch 
  dev
* master

```
当我们创建新的分⽀后，Git 新建了⼀个指针叫 dev， * 表⽰当前 HEAD 指向的分⽀是 master 分⽀。另外，可以通过⽬录结构发现，新的 dev 分⽀：  

``` shell
root@Chrix:/home/test/gittest/.git# ls refs/heads/
dev  master
root@Chrix:/home/test/gittest/.git# cat refs/heads/*
8f1a4d5f3e3f048345715e73f425367e4c0e6e96
8f1a4d5f3e3f048345715e73f425367e4c0e6e96
```

发现⽬前 dev 和 master 指向同⼀个修改。  

### 1.5.2 切换分⽀

```
root@Chrix:/home/test/gittest# git checkout dev 
Switched to branch 'dev'
root@Chrix:/home/test/gittest# git branch 
* dev
  master
```

我们发现 HEAD 已经指向了 dev，就表⽰我们已经成功的切换到了 dev 上！接下来，在 dev 分⽀下修改 ReadMe ⽂件，新增⼀⾏内容，并进⾏⼀次提交操作：

``` shell
root@Chrix:/home/test/gittest# echo "This is dev branch" >> ReadMe 
root@Chrix:/home/test/gittest# git add .
root@Chrix:/home/test/gittest# git commit -m "update ReadMe"
[dev dda1e8e] update ReadMe
 1 file changed, 1 insertion(+)
root@Chrix:/home/test/gittest# cat ReadMe 
Hello World
Hello World
version1
version2
version3
This is dev branch
```

现在，dev 分⽀的⼯作完成，我们就可以切换回 master 分⽀：  

``` shell
root@Chrix:/home/test/gittest# git checkout master 
Switched to branch 'master'
root@Chrix:/home/test/gittest# cat ReadMe 
Hello World
Hello World
version1
version2
version3
```

切换回 master 分⽀后，发现ReadMe⽂件中新增的内容不⻅了！！！赶紧再切回 dev 看看:  

``` shell
root@Chrix:/home/test/gittest# git checkout dev 
Switched to branch 'dev'
root@Chrix:/home/test/gittest# cat ReadMe 
Hello World
Hello World
version1
version2
version3
This is dev branch
```



在 dev 分⽀上，内容还在。为什么会出现这个现象呢？我们来看看 dev 分⽀和 master 分⽀指向，发现两者的提交并不一样了

``` shell
root@Chrix:/home/test/gittest# cat .git/refs/heads/*
dda1e8e36bc3342c99ab7bdd11ae7a6f009c2ae0
8f1a4d5f3e3f048345715e73f425367e4c0e6e96
```



### 1.5.3 合并分支

#### 1.5.3.1 合并没有分叉的分支

``` text
git merge [BRANCH_NAME] # 将另外一个分支的代码，合并到当前分支
```

如果我们的开发分支与合入到的目标分支之间没有任何分叉（例如在下面的示意图中， 我们检出了 master 分支，并且想将 hotfix 分支合并到 master 分支上，hotfix 分支所指向的提交 C4 是我们所在的分支 master 所指向的提交 C2 的直接后继），那么这时候 git 会直接将目标分支的指针向前挪动到指向开发分支的头部，这个指针移动操作被叫做 fast forward

![img](https://picx.zhimg.com/v2-459e04b562eda4823a36252e7117f7c3_1440w.jpg)

![img](https://picx.zhimg.com/v2-a5ef795608afb131de010a3a9bdd24d9_1440w.jpg)



``` shell
root@Chrix:/home/test/gittest# git checkout master 
Switched to branch 'master'
root@Chrix:/home/test/gittest# git b
bisect   blame    branch   bundle   
root@Chrix:/home/test/gittest# git branch 
  dev
* master
root@Chrix:/home/test/gittest# git merge dev 
Updating 8f1a4d5..dda1e8e
Fast-forward
 ReadMe | 1 +
 1 file changed, 1 insertion(+)
root@Chrix:/home/test/gittest# cat ReadMe 
Hello World
Hello World
version1
version2
version3
This is dev branch
root@Chrix:/home/test/gittest# cat .git/refs/heads/*
dda1e8e36bc3342c99ab7bdd11ae7a6f009c2ae0
dda1e8e36bc3342c99ab7bdd11ae7a6f009c2ae0
```

#### 1.5.3.2 合并包含分叉的分支

![img](https://pic2.zhimg.com/v2-ab668145c45c7af2b180979f34d945d9_1440w.jpg)

当我们想把 experiment 分支上的修改通过 merge 的方式合并到 master 分支上时，两个分支的头部提交节点都不在另一个分支的提交历史中（如上图中 c3，c4 节点的情况），git 会帮助我们创建一个新的合并提交，合并之后的提交记录长这样：

![img](https://pic2.zhimg.com/v2-ef31da7508f47dd913245cc59f932767_1440w.jpg)

``` shell
root@Chrix:/home/test/gittest# git checkout -b feature
Switched to a new branch 'feature'
root@Chrix:/home/test/gittest# vim ReadMe 
root@Chrix:/home/test/gittest# git add .
root@Chrix:/home/test/gittest# git commit -m "feature"
[feature 51fe453] feature
 1 file changed, 1 insertion(+), 1 deletion(-)
root@Chrix:/home/test/gittest# cat ReadMe 
Hello World
Hello World
version1
version2
version3
This is feature branch
```

我们发现，切回来之后，⽂件内容由变成了⽼的版本。此时在 master 分⽀上，我们对 ReadMe ⽂件再进⾏⼀次修改  

``` shell
root@Chrix:/home/test/gittest# git checkout master 
Switched to branch 'master'
root@Chrix:/home/test/gittest# vim ReadMe 
root@Chrix:/home/test/gittest# git add .
root@Chrix:/home/test/gittest# git commit -m "master:update ReadMe"
[master 4708ee5] master:update ReadMe
 1 file changed, 1 insertion(+), 1 deletion(-)
root@Chrix:/home/test/gittest# cat ReadMe 
Hello World
Hello World
version1
version2
version3
This is master branch
```

这种情况下，Git 只能试图把各⾃的修改合并起来，但这种合并就可能会有冲突，如下所⽰：  

```shell
root@Chrix:/home/test/gittest# git merge feature 
Auto-merging ReadMe
CONFLICT (content): Merge conflict in ReadMe
Automatic merge failed; fix conflicts and then commit the result.
```

git 在监测到冲突后生成的冲突标记，将两部分冲突的代码放在两个代码块中，用=======号间隔开。

```shell
root@Chrix:/home/test/gittest# cat ReadMe 
Hello World
Hello World
version1
version2
version3
<<<<<<< HEAD
This is master branch
=======
This is feature branch
>>>>>>> feature
```

此时我们必须要⼿动调整冲突代码，并需要再次提交修正后的结果！！（再次提交很重要，切勿忘记）

``` shell
root@Chrix:/home/test/gittest# vim ReadMe 
root@Chrix:/home/test/gittest# git add .
root@Chrix:/home/test/gittest# git commit -m "Merge ReadMe"
[master fa41735] Merge ReadMe
root@Chrix:/home/test/gittest# cat ReadMe 
Hello World
Hello World
version1
version2
version3
This is master branch
```

## 1.6 远程仓库

![](https://pic2.zhimg.com/v2-468760f57d75b51ad696d1a2cd42ec57_1440w.jpg)



使⽤ SSH ⽅式克隆仓库，由于我们没有添加公钥到远端库中，服务器会拒绝了我们的 clone 链接。需要我们设置⼀下：

第⼀步：创建SSH Key。在⽤⼾主⽬录下，看看有没有.ssh⽬录，如果有，再看看这个⽬录下有没有

id_rsa 和 id_rsa.pub 这两个⽂件，如果已经有了，可直接跳到下⼀步。如果没有，需要创建SSH Key：  

```
test@Chrix:~$ ssh-keygen -t rsa -C "293611667@qq.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/home/test/.ssh/id_rsa): 
Created directory '/home/test/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/test/.ssh/id_rsa
Your public key has been saved in /home/test/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:STNhhBbeNrXC0fRB1fRvIhTqkgpf3XAxFtaax7BLbhg 293611667@qq.com
The key's randomart image is:
+---[RSA 3072]----+
|      .+=oo.X+.o.|
|     .o+ +.*.=. o|
|     .. X + +*  .|
|       o XE== o .|
|    .   S o=oo. o|
|     o o .. +. o |
|      o    .     |
|                 |
|                 |
+----[SHA256]-----+
```

顺利的话，可以在⽤⼾主⽬录⾥找到 .ssh ⽬录，⾥⾯有 id_rsa 和 id_rsa.pub 两个⽂件，这两个就是SSH Key的秘钥对， id_rsa 是私钥，不能泄露出去， id_rsa.pub 是公钥，可以放⼼地告诉任何⼈。

```
test@Chrix:~$ la -a .ssh/
.  ..  id_rsa  id_rsa.pub

```

**推送本地代码到远程仓库**

推送代码是为了跟别人一起合作，命令行非常简单

```text
git push [REMOTE] [BRANCH]
```

remote 默认为 origin，如果不填的话就推送到它上面，branch 默认为当前分支，其实可以不加，加了就把指定的分支推送到远程了。 如果要将推送本地功能分支，建议 push 后面加上 —set-upstream 参数。这个操作让你在未来使用 `git push` 和 `git pull` 时不必显式指定远程分支，因为 Git 知道本地分支与远程分支的关联。

