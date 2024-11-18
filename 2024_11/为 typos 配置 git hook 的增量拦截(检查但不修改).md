# 为 typos 配置 git hook 的增量拦截(检查但不修改):

> [!Note]
>
> Chatgpt 严重依赖于训练集，对于新诞生的或者较为小众的工具可以说是胡言乱语。<br>
> 先前是在替代 pip 的`uv`包管理器，现在是 `typos`。<br>
> 这个时候，使用 AI 搜索引擎反而可以更快地接近想要的内容,而不是被误导。比如[devv.ai](http://devv.ai/).<br>
> 所以博客记录的必要性依然是存在的，虽然我可能写出来一些不准确的数据集来误导 gpt。<br>
> 我们将在这里尝试使用 [typos](https://github.com/crate-ci/typos) 逼迫常见 typo 类型就范。<br>

## 编写.pre-commit-config.yaml

```shell
(yutto) xnne@xnne-PC:~/code/yutto-uiya$ pre-commit run --all-file
black....................................................................Passed
ruff.....................................................................Passed
typos....................................................................Failed
- hook id: typos
- files were modified by this hook
(yutto) xnne@xnne-PC:~/code/yutto-uiya$ pre-commit run --all-file
black....................................................................Passed
ruff.....................................................................Passed
typos....................................................................Passed
```

```shell
- repo: https://github.com/crate-ci/typos
rev: v1.27.3
hooks:
    - id: typos
    files: \.(py|md)$ # 同时检查 .py 和 .md 文件
```

按照仓库提供的默认配置的情况下，`typos`会直接修改文件，但是我们希望它只检查和统计，因为直接改动到代码那是直接毁灭仓库的级别了。<br>

```yaml
- repo: https://github.com/crate-ci/typos
rev: v1.27.3
hooks:
    - id: typos
    files: \.(py|md)$
    pass_filenames: false
    verbose: true
    args: ['--format', 'brief'] # 这个是日志等级,brief 是最低的,json是最高的
    # 增量拦截我们使用brief,存量修复我们使用json
```

这样配置之后，`typos`就不会直接修改文件了，而是只会输出检查结果，我们可以根据这个结果来手动修复。<br>

## 我们哪些属于增量拦截的检查内容:

### 排除:

所有自动生成的文件，不需要检查:`build/`,`dist/`,`__pycache__/`,等等。<br>

### 包含：

我们先直接只排除不包含看看到底有哪些类型的文件。<br>

然后确定用户编写的文件，以及看看是否需要对不同类型的文件做一些不同操作。<br>

## typos 的原理：

它似乎是有一个词库，用来映射常见的拼写错误:`recieve -> receive`,`transfered -> transferred`等等。<br>

因为我尝试输入一些明显不存在的单词，它并没有报错，所以它应该主要是靠映射来检查的。<br>

```txt
recieve
transffered
transferd
transfered
transfere
transferee
yutto
uiya
heeel
```

```shell
typos....................................................................Failed
- hook id: typos
- duration: 0.01s
- exit code: 2

./README.md:51:1: `recieve` -> `receive`
./README.md:52:1: `transffered` -> `transferred`
./README.md:53:1: `transferd` -> `transferred`
./README.md:54:1: `transfered` -> `transferred`
./README.md:55:1: `transfere` -> `transferred`
```

我注意到 transferee 就没有被当作错误，这也验证了它是通过相近词库管理的，具体如何映射，我不懂。不过，它够快，而且准确，这就够了。<br>

## `_typos.toml` 配置词汇:

看了上面那个原理之后，实际上，我们大概可以了解到具体需要配置哪些词汇。<br>

- 专有词汇的缩写(`vec`,`grad`,`dim`)和常见变量缩写或者单词相似(`vex`)。如果对方配置了`专有词汇->常见变量名缩写,或者短词汇`,就会形成`误报`。虽然说这个库也是程序员维护的，但是，程序员之间的语言并不相通=-=，就像一个只学计算机视觉的，一个只学后端的，那这俩对话起来就是跨服聊天了。<br>
- 一切可能出现在对面词库 alias 里面的词汇，但是我们确实认为这个词汇是对的。<br>

配置我们自己的词库，`unpacket`为例:

```shell
./README.md:60:1: `unpacket`->`unpacked`
```

```toml
# _typos.toml
[default.extend-words]
unpacket = "unpacket"
```

这样配置之后，`typos`就不会再报那个错。<br>

如果依然报，检查你下你的文件命名是`_typos.toml`还是`_typo.toml`。有被被坑到一下`(>_<)`。<br>

## 接下来咱们去实战一下: 给 Paddle 仓库做 typos 增量拦截。

最开始，我按照上面的方式直接配置，完成了拦截，但是我忽略了一个问题:存量问题对日常开发的影响。<br>

### 最初发现的问题：存量问题对日常开发的影响。

即使 pre-commit 而 typos 工作时只会对暂存区（本次 commit 的内容）进行检查。不会检查出来以前已经存在的问题。<br>

但是存在的问题是:如果一个开发者在修改一个文件的时候，这个文件里面存在了上百处的 typo 问题，那么他就需要额外处理一下这个问题，把所有的问题都解决掉，然后再提交或者把每次报错的单词都填入`_typos.toml`。这样会阻塞开发者的工作。<br>

**解决方案**：把目前存在的 typo 问题先加入`_typos.toml`进行忽略，然后独立开设 typos 修复的任务。<br>

typos 支持 `[default.extend-words]`,本来是自定义词库用的，这里我们用来先忽略存量问题。<br>

### 问题二: pre-commit 版本过低，导致 typos 无法正常工作。

```shell
2024-11-17 01:37:17 [INFO] Initializing environment for https://github.com/crate-ci/typos.
2024-11-17 01:37:24 An error has occurred: ******:
2024-11-17 01:37:24 ==> File /root/.cache/pre-commit/repoib6tp4kx/.pre-commit-hooks.yaml
2024-11-17 01:37:24 ==> At Hook(id='typos')
2024-11-17 01:37:24 ==> At key: stages
2024-11-17 01:37:24 ==> At index 0
2024-11-17 01:37:24 =====> Expected one of commit, commit-msg, manual, merge-commit, post-checkout, post-commit, post-merge, post-rewrite, prepare-commit-msg, push but got: 'pre-commit'
```

似乎是 typos 的`.pre-commit-hooks.yaml`中对于 stages 有特殊需求，但是 pre-commit 版本过低，并不支持这个特性。导致了这个问题。<br>

**解决方案**：<br>

- 升级 pre-commit 版本。<br>
- 自定义 typos 的 hooks 文件。<br>

之类我们记录的是第二种解决方案。<br>

#### Refrence:

- [/psf/black#4065](https://github.com/psf/black/issues/4065)<br>
- [官方 typos-pre-commit-hooks.yaml](https://github.com/crate-ci/typos/blob/ba61bb6f5629c484099838f3ad64cd24c376f0d1/.pre-commit-hooks.yaml)<br>
- [自定义 typos-mirror-pre-commit-hooks.yaml](https://github.com/PFCCLab/typos-pre-commit-mirror/blob/main/.pre-commit-hooks.yaml)<br>

> [!Note]
>
> - 里面的`language`是 typos 的实现或者构建方式，而不是检查的文件类型。
> - 文件类型限制`[text]`,即使`.cxx、.h、.cpp`都是会进行检查的。如果放空的话，会连着图片一并检查，不宜这么做。<br>
> - 修改 mirror 仓库之后似乎需要新建 tag 才能生效，不然会一直使用缓存的旧版本。<br>

stages 具体应该怎么设置。这似乎存在争论，我看得还有点懵懵的，但总结来说，如果`pre-commit`版本>3.2.0,应该采取:<br>

- [#4067](https://github.com/psf/black/pull/4067#issuecomment-1822673095)
- [#3940](https://github.com/psf/black/pull/3940/files)

如果版本低于 3.2.0，那么应该采取:<br>

- [#4041](https://github.com/psf/black/pull/4041)

#### Further Studying:

我发现`.pre-commit-config.yaml`中存在`hooks`的参数，似乎可以进行配置，诸如`args,id`等等。似乎和我们的`pre-commit-hooks.yaml`很像。<br>

我尝试把 mirror 的`.pre-commit-hooks`直接照搬到`.pre-commit-config.yaml`中，然后进行尝试，发现问题依然存在。所以，配置 mirror 来自定义 hooks 的必要性依然是有的。而且这么做也能够避免 hooks 配置被官方默认配置 hooks 污染的情况,比如，官方的`hooks: args`中默认参数就是`--write-changs`,会默认直接把 typos 问题写入文件。<br>
