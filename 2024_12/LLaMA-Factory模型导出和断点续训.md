# LLaMA-Factory 模型导出和断点续训

> 比较猥琐的一点是,Qwen 在去年的基础上做了一些变动,模型推理的显存占用比去年更多了一些.<br>
> 目前 lla-factory 的 web_demo.py 运行非量化模型占用需要 17G 以上显存.哪怕只是接近 16G,也会让我运行不稳定,超过了 16G 反而让我直接死心.<br>
> 不过我在用 self-llm 提供的 fast-api 方法来调用模型时候,依然表现正常.<br>

目前我用的是 master 分支,这是我当前的分支.<br>

![alt text](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/11/202412252256079.png)

因为如果使用那个人当时的模型,应该就不会碰到显存问题,所以模型版本也同样重要.<br>

## 1. 模型导出

训练的时候,我使用的是`int-4`的量化模型,而它有两个缺点:<br>

- llama-factory,以及底层库都不支持直接导出量化模型,挺好奇预训练模型是咋来的.
- 量化模型的推理速度非常慢,基本上是一个字一个字蹦出来的.

这个是我目前碰到的,而似乎它还有不易训练的问题,因为本身就是把 float32 转成 int4 进行训练的.所以梯度上估计也有点困难.<br>

但是它有一个绝对的优势,那就是显存占用很小,有多小,从模型大小来看,非量化版本有 15.5G,而量化版本只有 5.5G,但是它们的模型架构和参数量是一样的.<br>

这也是我们后续要利用的 bug.<br>

因为量化模型本身就相当于把模型参数 dtype 设置成了`int4`,所以我们反其道而行,只要再把 dtype 设置回`bfloat16`就可以导出非量化模型.<br>

而 llama-factory 支持我们卡的这个 Bug.<br>

具体怎么卡可以看:[4060Ti16G 显卡图形化微调训练通义千问 Qwen 模型（适合新手朋友）](https://www.bilibili.com/video/BV1wu4y1p7Yr/)<br>

我也去导出一下.<br>

跟着视频在直接加载模型进行 chat 的时候出了点小 bug ,`web demo has no object system promot`.<br>

不过好在导出是正常的.<br>

wc<br>

```shell
torch.cuda.OutOfMemoryError: CUDA out of memory. Tried to allocate 32.00 MiB. GPU 0 has a total capacty of 16.00 GiB of which 1.92 GiB is free. Including non-PyTorch memory, this process has 17179869184.00 GiB memory in use. Of the allocated memory 12.57 GiB is allocated by PyTorch, and 928.50 KiB is reserved by PyTorch but unallocated. If reserved but unallocated memory is large try setting max_split_size_mb to avoid fragmentation.  See documentation for Memory Management and PYTORCH_CUDA_ALLOC_CONF
```

感觉到一点受伤害,超了我 1G.<br>

看起来并不能直接用 LLaMA-Factory 做演示了.<br>

**导出的注意事项,打出的最大模型 part 限制最好是整数,我用了一个 1.88 似乎报错了.** <br>

值得高兴的是,我用之前最初用的`self-llm`的 fastapi 版本的调用是可以使用我的模型并且正常回复的,速度也还行.<br>

```python
from fastapi import FastAPI, Request
from transformers import AutoTokenizer, AutoModelForCausalLM, GenerationConfig
import uvicorn
import json
import datetime
import torch

# 设置设备参数
DEVICE = "cuda"  # 使用CUDA
DEVICE_ID = "0"  # CUDA设备ID，如果未设置则为空
CUDA_DEVICE = f"{DEVICE}:{DEVICE_ID}" if DEVICE_ID else DEVICE  # 组合CUDA设备信息

# 清理GPU内存函数
def torch_gc():
    if torch.cuda.is_available():  # 检查是否可用CUDA
        with torch.cuda.device(CUDA_DEVICE):  # 指定CUDA设备
            torch.cuda.empty_cache()  # 清空CUDA缓存
            torch.cuda.ipc_collect()  # 收集CUDA内存碎片

# 创建FastAPI应用
app = FastAPI()

# 处理POST请求的端点
@app.post("/")
async def create_item(request: Request):
    global model, tokenizer  # 声明全局变量以便在函数内部使用模型和分词器
    json_post_raw = await request.json()  # 获取POST请求的JSON数据
    json_post = json.dumps(json_post_raw)  # 将JSON数据转换为字符串
    json_post_list = json.loads(json_post)  # 将字符串转换为Python对象
    prompt = json_post_list.get('prompt')  # 获取请求中的提示
    history = json_post_list.get('history')  # 获取请求中的历史记录
    max_length = json_post_list.get('max_length')  # 获取请求中的最大长度
    top_p = json_post_list.get('top_p')  # 获取请求中的top_p参数
    temperature = json_post_list.get('temperature')  # 获取请求中的温度参数
    # 调用模型进行对话生成
    response, history = model.chat(
        tokenizer,
        prompt,
        history=history,
        max_length=max_length if max_length else 2048,  # 如果未提供最大长度，默认使用2048
        top_p=top_p if top_p else 0.7,  # 如果未提供top_p参数，默认使用0.7
        temperature=temperature if temperature else 0.95  # 如果未提供温度参数，默认使用0.95
    )
    now = datetime.datetime.now()  # 获取当前时间
    time = now.strftime("%Y-%m-%d %H:%M:%S")  # 格式化时间为字符串
    # 构建响应JSON
    answer = {
        "response": response,
        "history": history,
        "status": 200,
        "time": time
    }
    # 构建日志信息
    log = "[" + time + "] " + '", prompt:"' + prompt + '", response:"' + repr(response) + '"'
    print(log)  # 打印日志
    torch_gc()  # 执行GPU内存清理
    return answer  # 返回响应

# 主函数入口
if __name__ == '__main__':
    # 加载预训练的分词器和模型
    tokenizer = AutoTokenizer.from_pretrained("./model_top100", trust_remote_code=True)
    model = AutoModelForCausalLM.from_pretrained("./model_top100", device_map="auto", trust_remote_code=True).eval()
    model.generation_config = GenerationConfig.from_pretrained("./model_top100", trust_remote_code=True) # 可指定
    model.eval()  # 设置模型为评估模式
    # 启动FastAPI应用
    # 用6006端口可以将autodl的端口映射到本地，从而在本地使用api
    uvicorn.run(app, host='0.0.0.0', port=6006, workers=1)  # 在指定端口和主机上启动应用

```

因为需要在学校用,所以我需要把它改成 gradio-webui 的版本,然后再用`share=True`搞到公网上,最后再调用 gradio-webui 的 API 进行调用.<br>

主要是以前写过一个 UI 了,正好可以用上,当时花了好长的时间做 QT 的对话气泡和即时回复,处理了不少多进程的问题.<br>

如果你是本地使用的话,直接用上面那个 fastapi 的版本即可.<br>

这么调用:<br>

```python
import requests
import json

def get_completion(prompt):
    headers = {'Content-Type': 'application/json'}
    data = {"prompt": prompt, "history": []}
    response = requests.post(url='http://127.0.0.1:6006', headers=headers, data=json.dumps(data))
    return response.json()['response']

if __name__ == '__main__':
    print(get_completion("摸下可以吗？"))
```

这些代码都是`self-llm`提供的.<br>

gradio 的版本,看看 copilot 能不能直接帮我写出来:<br>

```python
import gradio as gr
from transformers import AutoTokenizer, AutoModelForCausalLM, GenerationConfig
import torch

# 设置设备参数
DEVICE = "cuda"  # 使用CUDA
DEVICE_ID = "0"  # CUDA设备ID，如果未设置则为空
CUDA_DEVICE = f"{DEVICE}:{DEVICE_ID}" if DEVICE_ID else DEVICE  # 组合CUDA设备信息

# 清理GPU内存函数
def torch_gc():
    if torch.cuda.is_available():  # 检查是否可用CUDA
        with torch.cuda.device(CUDA_DEVICE):  # 指定CUDA设备
            torch.cuda.empty_cache()  # 清空CUDA缓存
            torch.cuda.ipc_collect()  # 收集CUDA内存碎片

# 加载预训练的分词器和模型
tokenizer = AutoTokenizer.from_pretrained("./model_top100", trust_remote_code=True)
model = AutoModelForCausalLM.from_pretrained("./model_top100", device_map="auto", trust_remote_code=True).eval()
model.generation_config = GenerationConfig.from_pretrained("./model_top100", trust_remote_code=True) # 可指定
model.eval()  # 设置模型为评估模式

def chat(prompt):
    response, history = model.chat(
        tokenizer,
        prompt,
        history=[],
        max_length=2048,  # 如果未提供最大长度，默认使用2048
        top_p=0.7,  # 如果未提供top_p参数，默认使用0.7
        temperature=0.95  # 如果未提供温度参数，默认使用0.95
    )
    torch_gc()  # 执行GPU内存清理
    return response

iface = gr.Interface(fn=chat, inputs="text", outputs="text")
iface.launch(share=True)
```

## 2. 断点续训

按照 gpt 的说法,似乎不需要额外操作,只需要选中之前的断点,然后再次训练即可,不过似乎比较重要的一点是,要尽量保证训练超参数一致.<br>

我去尝试一下.<br>

![alt text](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/11/202412252256933.png)

训练还算顺畅,但是,呃我忘记断点续训这个茬了.我直接写了一个新的断点.<br>

[请问网页版能否断点续训](https://github.com/hiyouga/LLaMA-Factory/issues/3660)<br>

根据里面一个老哥的回答,似乎用旧的断点可以直接继续训练.<br>

明天可以做一下测试.<br>

载入新的断点,然后在原来的 top100 上继续训练一轮,大概就半分种左右,那样如果模型依然保持原样,就说明是支持断点续训的.<br>

因为 loss 比较难降,它主要和说话方式有关.我上次那个 top100 的 LOSS 降到了 1.0 左右,而这次大概只降到了 2.0,还是略有差距的.<br>

所以如果可以,我还是希望从 TOP100 的模型上面续训练.<br>

## 关于续训:

有个好消息,直接在 checkpoint name 那边写上之前的 checkpoint,就可以续训了.<br>

我稍微练了两分钟:<br>

![alt text](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/11/202412260907821.png)

效果和原来一致,是可以继承的,但是也有个坏消息.<br>

就是我发现它终止与否是按照 max_step 来的.<br>

比如我似乎练了 180 step?<br>

奇怪,我的 batch_size 是 8,样本是 100,60 轮应该有个六七百 step 才对.<br>

以及,我尝试输入 50 epoch,step 似乎才 120.<br>

这一点需要注意,后续转到大样本集上,需要一点点推出需要多少 epoch,或者说 step.<br>
