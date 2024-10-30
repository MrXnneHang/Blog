# vscode copilot中无法正常补全Markdown解决:

## 问题描述:

在VsCode中，Python,CPP等常见语言的代码片段可以正常补全，但是在MarkDown里面无法正常工作。<br>

OS: deepinV23<br>
VsCode:  软件商店下的.<br>
ps: 无额外配置.<br>

## Solve:

[https://docs.github.com/zh/copilot/troubleshooting-github-copilot/troubleshooting-common-issues-with-github-copilot](https://docs.github.com/zh/copilot/troubleshooting-github-copilot/troubleshooting-common-issues-with-github-copilot)<br>

里面有`GitHub Copilot 在某些文件中不起作用`这个问题的解决方案.<br>

其实很神奇，它就在你的右下角，有一个copilot的头像，如果它被禁用了(有一个反斜杠)，那就无法工作。<br>

点击一下头像，有一个 `Enable/Disable Completion for 'Markdown'`。<br>

点上就行了。<br>


还有一种奇怪的现象，是虽然出现了代码片段，但是无法tab补全.<br>

这个似乎是和插件`markdown all in one`有关，卸载这个插件就可以了.<br>