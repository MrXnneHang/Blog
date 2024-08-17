这个issue我没法复现，不太靠谱。而且也是莫名其妙地解决了。但记录一下。

## Issue:

```cmd
could not found module named 'pydub'
```

仅仅只在bat里面运行的时候会碰到。

实际环境具有pydub，vscode运行正常，但bat脚本无法找到pydub。

以及，在用 Pyinstaller 封装打包成exe的时候也会出现。

## Solve:

```cmd
cd .\env\Scripts
pip remove pydub
pip3 install pydub
```



## pip3和pip的区别:

copy from gpt:

```cmd
pip 通常用于为 Python 2.x 安装包。
pip3 通常用于为 Python 3.x 安装包。
```

我一直用pip来着。但我从来没有用过2.x

