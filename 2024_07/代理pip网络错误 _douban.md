## pip 开了网络代理之后无法安装包

[https://blog.csdn.net/qq_42951560/article/details/132339370](https://blog.csdn.net/qq_42951560/article/details/132339370)

like:

```cmd
$ pip install netsm
Looking in indexes: https://pypi.tuna.tsinghua.edu.cn/simple
WARNING: Retrying (Retry(total=4, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLEOFError(8, 'EOF occurred in violation of protocol (_ssl.c:1123)'))': /simple/netsm/
WARNING: Retrying (Retry(total=3, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLEOFError(8, 'EOF occurred in violation of protocol (_ssl.c:1123)'))': /simple/netsm/
WARNING: Retrying (Retry(total=2, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLEOFError(8, 'EOF occurred in violation of protocol (_ssl.c:1123)'))': /simple/netsm/
WARNING: Retrying (Retry(total=1, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLEOFError(8, 'EOF occurred in violation of protocol (_ssl.c:1123)'))': /simple/netsm/
WARNING: Retrying (Retry(total=0, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLEOFError(8, 'EOF occurred in violation of protocol (_ssl.c:1123)'))': /simple/netsm/
Could not fetch URL https://pypi.tuna.tsinghua.edu.cn/simple/netsm/: There was a problem confirming the ssl certificate: HTTPSConnectionPool(host='pypi.tuna.tsinghua.edu.cn', port=443): Max retries exceeded with url: /simple/netsm/ (Caused by SSLError(SSLEOFError(8, 'EOF occurred in violation of protocol (_ssl.c:1123)'))) - skipping
ERROR: Could not find a version that satisfies the requirement netsm (from versions: none)
ERROR: No matching distribution found for netsm

```

solve:

添加该条到代理服务器。

```cmd
pypi.tuna.tsinghua.edu.cn
```



![image-20240722205528137](https://image.baidu.com/search/down?url=https://img1.doubanio.com/view/photo/l/public/p2910936439.webp)