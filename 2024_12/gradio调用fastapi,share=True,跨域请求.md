# gradio 调用 fastapi 超时 | localhost is not accessible | share=True | 跨域请求.md

## 1.When localhost is not accessible #4046

[When localhost is not accessible #4046 ](https://github.com/gradio-app/gradio/issues/4046)<br>

在开着 VPN 的时候，gradio 无法访问 localhost，但是在 share 的时候可以访问。但是访问的到底是 vpn 的 localhost，还是 gradio 远程服务器的 localhost 呢?这是个问题.<br>

在 share=True 的时候,在我尝试用 port=7860 的 gradio 去调用 port=6006 的 fastapi 的时候,发现跨域请求的问题,表现的形式是一直超时，并且 fastapi 没有收到任何请求.<br>

我一直调试,但是没有发现问题所在,最后我取消了 share=True,发现开着 VPN 无法启动 gradio 应用,也许也是因为这个原因 gradio 无法访问 localhost.<br>

## 尝试一: 关闭代理后再次尝试 share=True

似乎正常工作,也就是说大 boss 是我的 VPN,而不是什么防火墙端口没开.<br>

对于 gradio 的 7860 我是开了.<br>

## 尝试二: 取消 fastapi 的跨域请求允许<br>

测试通过,说明 gradio 和 fastapi 跨端口访问是不需要设置 CORS 的.<br>

也就是说,我折腾了很久的 gradio webui 无法访问 localhost 的问题,以及本地允许模型也无法访问仅仅只是因为我开了 VPN,在我取消 share=True 后才发现的,归根到底大 boss 是它:<br>

```txt
When localhost is not accessible #4046
```

以及一个教训,应该先测试本地可以允许然后再设置 share=True.<br>
