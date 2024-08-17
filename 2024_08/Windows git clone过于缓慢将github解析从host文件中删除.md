## Issue:

git clone 速度只有10kb。



## 原因:

前阵子在搞github学生认证的时候，因为不能开代理，所以把github的很多ip直接加到hosts文件中。呃然后这几天在clone的时候相当奇怪。

速度峰值只有10kb。最初我以为是代理问题，但是我直接从网页端下载是正常的，包括使用代理时对github访问也是正常的。

但是我在使用代理进行git clone 的时候，就会出现速度很慢很慢的问题。  



## Solve：

修改hosts文件并刷新DNS解析。



## 做法：

hosts文件路径:

```cmd
C:\Windows\System32\drivers\etc\hosts
```

将和github相关的删除，然后cmd刷新DNS解析:

```cmd
ipconfig /flushdns
```

恢复正常:

```cmd
Zhouyuan@DESKTOP-3I1GRP0 MINGW64 /d/program
$ git clone git@github.com:MrXnneHang/Vtuber_Tutorial.git
Cloning into 'Vtuber_Tutorial'...
remote: Enumerating objects: 344, done.
remote: Counting objects: 100% (69/69), done.
remote: Compressing objects: 100% (47/47), done.
remote: Total 344 (delta 26), reused 37 (delta 21), pack-reused 275
Receiving objects:  95% (327/344), 17.27 MiB | 2.39 MiB/s
Receiving objects: 100% (344/344), 17.72 MiB | 2.06 MiB/s, done.
Resolving deltas: 100% (144/144), done.
```

