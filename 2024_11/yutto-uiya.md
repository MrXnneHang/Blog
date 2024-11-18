# 这里是新仓库呀。 - yutto-uiya

仓库地址：[yutto-uiya](https://github.com/MrXnneHang/yutto-uiya)<br>

它是一个 bilibili 视频下载器。<br>

我目前正要为[yutto](https://github.com/yutto-dev/yutto)开发 gradio 的 webui.<br>

说实话我真的不敢相信，yutto 作者开发了这么多年，竟然连一个 webui 都没有。<br>

## 给 yutto-uiya 配置代码风格规范:

我这里使用了 black 和 ruff 来作为 git hook。<br>

```yaml
- repo: https://github.com/psf/black-pre-commit-mirror
rev: 24.8.0
hooks:
    - id: black
- repo: https://github.com/astral-sh/ruff-pre-commit
rev: v0.6.1
hooks:
    - id: ruff
    args: [--fix, --exit-non-zero-on-fix, --no-cache]
```

整体我直接 clone 了这个仓库[python-lib-starter](https://github.com/ShigureLab/python-lib-starter)。<br>

虽然有点不晓得怎么用，但是稍微把`.vscode`里的`settings.json`配了下，然后装好`extensions`，咱们就上阵了。<br>

### 尝试配置 typos:

因为一个任务，我在这里尝试配置了一下[`typos`](https://github.com/crate-ci/typos),一个自动检查英文拼写的工具，很酷，我记得我给 Paddle 交的第一个 PR 就是修改了三个词的 Typos,`recieve -> receive`,这就像一个回旋镖。🪃<br>

在我试图配置`.pre-commit-config.yaml`的时候，我又碰到了上次那个问题：

```shell
(yutto-uiya) xnne@xnne-PC:~/code/yutto-uiya$ pre-commit run --all-file
[INFO] Initializing environment for https://github.com/rafi/typos-pre-commit.
Username for 'https://github.com':
```

当时我用的 repo 地址是这个:<br>

```yaml
- repo: https://github.com/rafi/typos-pre-commit
  rev: v0.6.0 # 使用 typos 的最新版本
  hooks:
    - id: typos
```

这个是 gpt 给我写的，并不是原仓库。我尝试 google 了一下`https://github.com/rafi/typos-pre-commit`，结果发现`404 not found`，显示不存在这个仓库（好你个 gpt）。我先前就这么被坑了一次（这个是闭环，自从 github 推行 2FA 之后，不允许直接在终端里输入用户名和密码登陆，也就根本不能拉到仓库），终于知道原因了。<br>

然后我在原仓库找到了[pre-commit.md](https://github.com/crate-ci/typos/blob/master/docs/pre-commit.md)：<br>

```yaml
- repo: https://github.com/crate-ci/typos
  rev: v1.27.3
  hooks:
    - id: typos
```

```shell
(yutto-uiya) xnne@xnne-PC:~/code/yutto-uiya$ pre-commit run --all-file
[INFO] Initializing environment for https://github.com/crate-ci/typos.
[INFO] Installing environment for https://github.com/crate-ci/typos.
[INFO] Once installed this environment will be reused.
[INFO] This may take a few minutes...
black....................................................................Passed
ruff.....................................................................Passed
typos....................................................................Passed
```

舒服了。<br>

下次开个任务独立分析那个`typos`的任务，那个要复杂一些，这里就不展开了。<br>

## 我们准备使用的文件目录：

为了在不同的平台部署时不需要考虑产生部署步骤不同，以及能够傻瓜式地使用。这里 yutto 我们直接使用`uv pip install yutto`来安装。<br>

```css
yutto-uiya/
├── api/ # 会在这里复用yutto下面的库以及一点点爬虫实现功能扩展
│   ├── anime/ # 番剧
│   │   └── __init__.py
│   ├── user/ # 用户发布的单p和多p的视频
│   │   └── __init__.py
│   ├── favlist/ # 收藏夹
│   │   └── __init__.py
│   ├── collection/ # 合集
│   │   └── __init__.py
│   └── course/ # 课程
│       └── __init__.py
├── yutto/ # 把yutto shell指令使用python调用，形成最小模块
│   └── __init__.py
│
├── utils/ # 这里是我们的工具包,只依赖于python标准库，以及一些第三方库，不依赖我们自己写的代码
│
├── configs/ # ffmpeg 等等配置文件我们会尝试放在这里.
│
└── webui.py # 这个是我们的 webui 入口文件
```

这次，俺也要维护一个从零开始可爱的仓库。<br>
