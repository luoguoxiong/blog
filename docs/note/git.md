# Git revert 与 reset

## Git revert 
Git revert 用于撤回某次提交的内容，同时再产生一个新的提交(commit)。原理就是在一个新的提交中，对之前提交的内容相反的操作。

下面通过例子具体解释一下：

现有一个git项目，已经有3次提交，每次添加一个文件，具体提交步骤如下：

```shell
# 第一次提交
$ echo "first commit" > test.txt
$ git add test.txt
$ git commit -m "1 commit" 

# 第二次提交
$ echo "second commit" > test2.txt
$ git add test2.txt
$ git commit -m "2 commit"

# 第三次提交
$ echo "third commit" > test3.txt
$ git add test3.txt
$ git commit -m "3 commit"

$ ls
test.txt  test2.txt test3.txt
```

查看提交记录，使用命令 `git log --pretty=oneline --abbrev-commit`:

```shell
$ git log --pretty=oneline --abbrev-commit
* 224a71b - (HEAD -> master) 3 commit
* 54b25d8 - 2 commit
* b2b9b24 - 1 commit
```

可以看到有三次提交，每一次提交的内容都是添加一个文件。

现在我们做实验，使用git revert撤回第二次提交的内容，即54b25d8 - 2 commit这次提交的内容，使用git revert 54b25d8（第二次提交的hash值）

```shell
$ git revert 54b25d8
Removing test2.txt
[master e9fe99e] Revert "2 commit"
 1 file changed, 1 deletion(-)
 delete mode 100644 test2.txt
```

test2.txt文件已经不见了，说明第二次提交的内容已经被撤回了

我们再查看一下git提交记录：

test2.txt文件已经不见了，说明第二次提交的内容已经被撤回了

我们再查看一下git提交记录：

```shell
$ git log --pretty=oneline --abbrev-commit

* e9fe99e - (HEAD -> master) Revert "2 commit"
* 224a71b - 3 commit
* 54b25d8 - 2 commit
* b2b9b24 - 1 commit
```
发现多了1次提交记录，即：e9fe99e - (HEAD -> master) Revert "2 commit"， 而这次提交的所作的事情就是撤销2 commit所提交的内容。

这就是git reset的作用：撤回某次提交的内容，再产生一个新的提交(commit)

注意这句话的重点：

1. 撤回提交的内容，不是撤回提交(commit)，之前的提交的历史不会发生任何改变。
2. 会产生一个新的提交。它是通过在新的提交中，将之前提交的内容反向操作了一遍，比如：之前提交内容是 添加 test2.txt，而新提交中的内容就变成了 删除 test2.txt

### 使用场景

在工作中，如果发现之前某次提交的内容是多余不需要的，需要撤销掉，这时就可以使用git revert 将这次提交的内容撤回掉。

其实git revert在使用过程中有很多局限性，比如在对相同的一个文件执行多次commit以后，再使用git revert撤回其中某次提交的内容将会出现冲突，这时就需要手动解决，这回带来一些麻烦。

## Git reset

git reset用于版本回退，能将当前提交(commit)回退到特定的历史版本(commit)。这是一个相当实用的功能，基本在提交内容中有错误需要更正时都会用上这个功能。

在使用git reset有三个最常用的三个参数：

--soft: git reset --soft v1：版本回退到v1，将v1版本之后修改的内容 存入暂存区
--mixed: git reset --mixed v1 ：版本回退到v1，将v1版本之后修改的内容 存入工作区
--hard: git reset --hard v1：版本回退到v1，v1版本之后修改的内容 全部清除

### 使用场景

在实际使用中，一般--hard 和--mixed使用较多，--soft用的较少。

之所以--soft用的比较少是因为它的功能和--mixed功能重合了，我完全可以先使用--mixed将之后修改的内容存入工作区，然后检查一遍是否有错误，再使用git add提交到暂存区然后提交。

## git reset 和git revert的区别
git revert和 reset都能撤回特定提交v1的内容，但git revert是通过在新的提交里对提交v1的内容进行反向操作来实现撤销，所以git revert不会改变提交历史；git reset 通过版本回退到特定提交v1-1(v1的前一次提交)，来直接撤销掉v1这次提交，从而撤回提交v1的内容，所以git reset会改变提交历史。

## 在工作中如何判断使用git reset 还是 git revert？
问题的重点在于你是否想保留之前的提交历史

如果觉得git提交历史就应该真实，不可更改，那就通过git revert命令，使用一个新的提交来中撤回错误提交的内容
如果觉得错误内容不应该保留在git提交历史中，就通过git reset撤回错误提交，然后更正后重新提交，从而覆盖掉之前的提交
