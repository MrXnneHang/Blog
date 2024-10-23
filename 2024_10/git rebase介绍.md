# git rebase(变基)介绍：

```bash
(nlp) xnne@xnne-PC:~/code/AI_By_Doing$ git pull origin main
remote: Enumerating objects: 3, done.
remote: Counting objects: 100% (3/3), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
展开对象中: 100% (3/3), 951 字节 | 951.00 KiB/s, 完成.
来自 github.com:MrXnneHang/AI-By-Doing
 * branch            main       -> FETCH_HEAD
 * [新分支]          main       -> origin/main
提示： 您有偏离的分支，需要指定如何调和它们。您可以在执行下一次
提示： pull 操作之前执行下面一条命令来抑制本消息：
提示：
提示：   git config pull.rebase false  # 合并
提示：   git config pull.rebase true   # 变基
提示：   git config pull.ff only       # 仅快进
提示：
提示： 您可以将 "git config" 替换为 "git config --global" 以便为所有仓库设置
提示： 缺省的配置项。您也可以在每次执行 pull 命令时添加 --rebase、--no-rebase，
提示： 或者 --ff-only 参数覆盖缺省设置。
```

## rebase实际上是生成了一条新的分支。

远程分支:A->B->C<br>
本地分支:A->B->D<br>

```css
A --- B --- C  (origin/main)
     \
      D      (feature)
```


假设C和D是我们在不同的设备上面提交的，但是实际上已知互不干扰，比如说是两篇不同的博客。<br>

那么实际上merge就把历史记录搞得很复杂。<br>

```css
A --- B --- C  (origin/main)
     \     /
      D --- E  (feature)

```

它会把C和D合并变成一个E，而且C和D记录仍然存在。<br>


而变基则是新建了一个分支，并且变成这样：<br>

```css
A --- B --- C --- D  (feature, origin/main)
```

它保证log的线性，对于大型仓库来说，如果能一直保持线性而不是和堂兄堂妹关系不清要好得多。<br>


变基之后有两种选择，-f覆盖仓库。<br>
```shell
git push --force-with-lease 
```

或者push到一个新的分支来容纳它。<br>


## 什么时候需要解决冲突/解决冲突的方式。<br>

如果在`git pull --rebase`的时候需要解决冲突，应该会输出类似这种:<br>

```shell
First, rewinding head to replay your work on top of it...
Applying: Added new feature
Using index info to reconstruct a base tree...
M       file1.txt
Falling back to patching base and 3-way merge...
Auto-merging file1.txt
CONFLICT (content): Merge conflict in file1.txt
error: could not apply <commit-sha>... Added new feature
hint: Resolve all conflicts manually, mark them as resolved with
hint: "git add <conflicted-files>" or "git rm <conflicted-files>"
hint: and then run "git rebase --continue".
hint: You can instead skip this commit: run "git rebase --skip".
hint: To abort and get back to the state before "git rebase", run "git rebase --abort".
```

并且它会生成类似file1.txt，看提示输出。<br>


然后用`git mergetool`来进行合并。（我觉得我比较适合用合并工具，我比较菜。）<br>

然后:<br>

```shell
(nlp) xnne@xnne-PC:~/code/AI_By_Doing$ git config --global merge.tool meld
(nlp) xnne@xnne-PC:~/code/AI_By_Doing$ git config --global diff.tool meld
(nlp) xnne@xnne-PC:~/code/AI_By_Doing$ git config --global --get merge.tool
meld
```

合并完成或者无需合并时是这样的:<br>

```shell
(nlp) xnne@xnne-PC:~/code/AI_By_Doing$ git mergetool
No files need merging
```

```shell
git add file1.txt
git rebase --continue
```

这个时候就和上面来说一致了。<br>

一半的工作都是自动化做的，rebase比merge好用太多，而且还干净。<br>

