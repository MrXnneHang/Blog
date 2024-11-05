# 让 git commit message 更加可爱。

![](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/11/202411051243421.jpeg)

## Refrence:

- [打「『』」等特殊符号的的方法 | 「自定义短语」| 吵架神器?](https://xnnehang.top/blog/89)<br>
- [Git 提交信息规范](https://nyakku.moe/posts/2018/09/16/git-commit-message-convention.html)<br>
- [📝 docs: initial docs](https://github.com/yutto-dev/yutto/pull/86)<br>

## 前言：

> 说在所有之前,如果你在参与团队开发，那么规范应该以团队要求为主，这里主要是日常自己的仓库维护。

之前我大概学了一下基本的 commit message,比如 refactor,fix,feat,feature,perf 等等。<br>

但是这些写起来让人感觉没什么动力，通常想不到写什么的时候我就直接 update,update 了。结果搞得仓库就很难看，git log 也很糟糕。

> Git Commit Message 虽然可以随意描述，但使用没有意义的描述对于后续 review 代码以及理解代码用途等方面都会造成巨大的影响。因此 Commit Message 具有意义是最基本的要求，此外，你还应该遵守一定的格式规范，这样能够让大家更快更清晰地了解该 Commit 的详情。这里我主要介绍下常规的 Git Commit 规范和 Gitmoji 规范，最后介绍下我常用的相关配置。

不管你先前有没有学过，都不妨试试用这个 commit message 规范来作为你仓库规范,至少它写起来要让人觉得更加可爱些。<br>

成品展示:[https://github.com/yutto-dev/bilili](https://github.com/yutto-dev/bilili)<br>

而我丑到爆炸的仓库是这样的:[https://github.com/MrXnneHang/Blog](https://github.com/MrXnneHang/Blog)<br>

## 怎么做：

### 修改上一次 commit 的信息:

```bash
git commit --amend
```

这会打开你上一次的 commit message,你可以修改它，我以前 commit 错了总是`reset --hard`然后重新 add,然后再次 commit,而且，当碰到文件修改的，还要备份一下文件，有点悲催哈，但我这么做了半年。<br>

像这样:<br>

```yaml
:memo: docs:copilot无法在vscode中工作 (这是commit -m 的内容,:memo:是Gitmoji,push后会自动渲染)
# 请为您的变更输入提交说明。以 '#' 开始的行将被忽略，而一个空的提交
# 说明将会终止提交。
# ...
```

你可以选择在这里修改你的 commit message,然后保存退出。当然你也可以直接在 commit -m 的时候一次性写完。<br>

另外 commit message 是分成 header,body,footer 三部分。具体写法:<br>

```yaml
:memo: docs:copilot无法在vscode中工作 (这是commit -m 的内容,:memo:是Gitmoji,push后会自动渲染)

这里是body的部分

这里是footer的部分
# 请为您的变更输入提交说明。以 '#' 开始的行将被忽略，而一个空的提交
# 说明将会终止提交。
# ...
```

三个部分一般以空行分开。最后在你 push 到 github 后，你可以点进 commit 去看你的 commit message 的详细信息。<br>

[https://github.com/MrXnneHang/Blog/commit/17343e378c7e00dd971b5fd5209cb31faedb35f1](https://github.com/MrXnneHang/Blog/commit/17343e378c7e00dd971b5fd5209cb31faedb35f1)<br>

所以平时写 header 应该尽量简短，详细信息可以放到 body 里。<br>

### 自定义短语:

这个大大减少了我记忆和输入 Gitmoji 的时间。<br>

比如我可以用`ref`来替代`:recycle: refactor:`<br>

一般操作系统默认输入法就支持自定义短语，你可以仔细找找。<br>

不过上次 deepin 的搜狗输入法就会乱码，所以非必要，还是把短语加到默认输入法里，然后导出备份一份。<br>

包括上面那个`git commit --amend`我也放到自定义短语里了。<br>

### Gitmoji 和 Git Message 对照表:

可能会有所出入。<br>

因为实际上使用了 Gitmoji 就可以不用使用 Git Message 了，但是防止别人看不懂，还是加上的好。<br>

常用的就这些:<br>

```markdown
🎉 init: 初始化
🚀 release: 发布新版本
🎨 style: 代码风格修改（不影响代码运行的变动）
✨ feat: 添加新功能
🐛 fix: 修复 bug
📝 docs: 对文档进行修改
♻️ refactor: 代码重构（既不是新增功能，也不是修改 bug 的代码变动）
⚡ perf: 提高性能的代码修改
🧑‍💻 dx: 优化开发体验
🔨 workflow: 工作流变动
🏷️ types: 类型声明修改
🚧 wip: 工作正在进行中
✅ test: 测试用例添加及修改
🔨 build: 影响构建系统或外部依赖关系的更改
👷 ci: 更改 CI 配置文件和脚本
❓ chore: 其它不涉及源码以及测试的修改
⬆️ deps: 依赖项修改
🔖 release: 发布新版本
```

你把这些全部发给 gpt,它就会把它的代码给你，比如这个我肯定常用的:💩 :poop: bad code: <br>

`💩` 和`:poop:`是等效的，只要你写在 commit 里后 push 到仓库里就会自动渲染。<br>

比较完整的可以查看:[https://nyakku.moe/posts/2018/09/16/git-commit-message-convention.html](https://nyakku.moe/posts/2018/09/16/git-commit-message-convention.html)

你一条一条地把这些加到你的自定义短语里，然后就可以愉快地写 commit message 了。<br>

希望你以后的仓库都很可爱。<br>
