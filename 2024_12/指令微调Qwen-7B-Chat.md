# 指令微调 Qwen-7B-Chat

## 指令微调

通常是指用以下格式作为模型训练集:<br>

```json
{
  "instruction": "现在你要扮演皇帝身边的女人--甄嬛",
  "input": "你是谁？",
  "output": "家父是大理寺少卿甄远道。"
}
```

有一个很有意思的模型,它用甄嬛的对话来微调大模型.它似乎叫做 **Chat-嬛嬛**.<br>

但是我这次要做的相对来说就没那么有意思了,是一个医生,用来回答一些关于症状和医疗的问题问答.可以的话我还是更想调一个赛博女友呀.<br>

但我们本次还是先熟悉一遍流程,后面就可以随心所欲地调了.<br>

大模型把 tokenizer 什么的都封装好了,我们只需要把数据集放进去就可以了.<br>

那些 embedding 啊,mask 啊,attention 啊,都不用考虑,倒是让我这个懒人也能玩.<br>

我做的 QA 对数据集:<br>

```txt
链接: https://pan.baidu.com/s/1VX-ffLlXjShXJOUlkAZIwA?pwd=tv2y 提取码: tv2y
```

Top3 预览:<br>

```json
{
    "instruction": "头痛头昏十几年前。吹风和晒太阳。吵闹都会头疼没有检查出来结果应该怎样治疗",
    "input": "",
    "output": ":考虑是神经性头疼，是一种功能性的疾病，是由于紧张压力或是精神因素引起的神经功能失调造成的头痛，也有一定的遗传因素，一般可考虑吃些中草药川芎茶调散来治疗。同时结合针灸按摩，效果会更好的。"
},
{
    "instruction": "眼睛疲劳，老睁不开干涩",
    "input": "",
    "output": "一般是由于结膜炎症活干炎症引起的，可以到医院检查一下。"
},
{
    "instruction": "女人的左胸下是什么地？那里痛是怎么回事",
    "input": "",
    "output": "心脏在那个位置，心脏绞痛在那，也可能是胃痛导致的，胃痛位置比心脏偏下一点。"
}
```

有的问题会对应多个回答.<br>

本身数据集来源是: [zhangsheng93/cMedQA2](https://github.com/zhangsheng93/cMedQA2)<br>

我只是做了一些简单的处理,把数据集转换成了 json 格式.<br>

另外我清洗掉了长度超过 100 还是 150 的回答数据,因为我显存不够,哪怕输入了也需要截断.<br>

而且这里有两种练法,一种是我在小数据集上面尝试的,top100 练了 60 轮尝试让模型记住答案,另外一种则是我加大数据集,然后只练三四轮.<br>

暂时没决定好.<br>

![alt text](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/11/202412241809733.png)

这个是我的一个训练的 loss 情况,如果只练两三轮三四轮,实际上模型只是浅浅地看了一下数据,对它原本的回答方式我觉得是改变不了多少的,虽然有朋友可能会说 LLM 的 loss 没啥用.<br>

但是我希望模型能够改掉原本那种回答按点一个个把可能都列出来的那种方式,更希望是像我提供的数据集那样只提出来一个可能,然后给出方案,我希望它能学习那种说话方式,那样我才算达到了我微调的目的.<br>

因为如果以后要练一个赛博女友,我不希望她这样跟我对话:<br>

```txt
你今天晚上要吃什么,觉得这些不错:

- 火锅, 在冬天吃火锅是最好的选择
- 寿司, 寿司是一种日本料理(好像混入了什么奇怪的东西吼,这个是copilot写的)
- 烤鸭, 烤鸭是北京的特色菜
- 烤鱼, 烤鱼是一种湖南特色菜
```

我希望她直接这样告诉我:<br>

```txt
今天晚上吃火锅吧,冬天吃火锅最好了.
```

兴奋!<br>

promot 本身可以改变一些模型的回答方式,但是似乎无法根本地改变前一种回答方式,而我希望得到第二种.所以我选择微调,所以,我选择多练.<br>

经过 Top 100 的半小时 60 轮微调,效果让我相当经验,在训练集上是这样的:<br>

![alt text](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/11/202412241810128.png)
![alt text](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/11/202412241810314.png)

> 虽然炖狗肉这件事情我不是很认同,让我有点疑惑.

![alt text](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/11/202412241810518.png)

似乎跟狗肉过不去了.但是多样性是有了,我记得也有说吃六位地黄丸的=-=. <br>

这是 QA 对:<br>

```json
"instruction": "每天起床前都要流汗能治吗？以前没有用药，但我有胃病。每天起床前都要流汗有这方面的偏方吗？",
"input": "",
"output": "容易出汗多考虑气虚，可以多用黄芪泡茶，多喝鸡汤。可也适当服用一些补中益气丸，参苓白术散和补中益气汤丸。"
```

我再找一个训练集外的试试.<br>

![alt text](https://fastly.jsdelivr.net/gh/MrXnneHang/blog_img/BlogHosting/img/24/11/202412241810090.png)

还真别说,这个我认同度很高.<br>

```
coplilot,你觉得夜跑比较好还是早晨跑比较好？

- 早晨跑步，早晨跑步有助于提高人体的新陈代谢，有助于减肥。
- 夜跑，夜跑有助于放松身心，有助于睡眠。
- 早晨跑步，早晨跑步有助于提高人体的新陈代谢，有助于减肥。
- 夜跑，夜跑有助于放松身心，有助于睡眠。
```

这个是 copilot 写的.<br>

你不知道晚上九点后跑步更睡不着吗.<br>

打住,我们接着往下做.<br>

不过我倒是想到一个训练方法,就是在先在小数据集上多轮,然后再在大数据集上训练少轮,前者学说话语气,提高模型的表达能力.<br>

但这个涉及到断点续训,不知道 llama-factory 是否需要额外的操作<br>

## 微调的工具:

[~~datawhalechina/self-llm~~](https://github.com/datawhalechina/self-llm)<br>

这个仓库把很多大模型的微调过程都用 jupyter notebook + markdown 写好了,用户基本上跑一遍就好了.<br>

可以说已经够懒了,但是实际上还是潜在很大问题的,就是随着时间的推移,python 库版本的更新,在 transformers>4.35.0 的版本中,就引入了一些破坏性的更改,让很多模型按照原来的方式运行不起来.<br>

即使尝试和作者同步一样的环境,也会碰到问题,比如碰到类似 win error 找不到文件的问题.<br>

这个时候,就需要 docker 了.<br>

在经过一天的折磨后,我果断转了 docker.<br>

[4060Ti16G 显卡图形化微调训练通义千问 Qwen 模型（适合新手朋友）](https://www.bilibili.com/video/BV1wu4y1p7Yr/)<br>

这里有个佬把 llama-factory 打包放进 docker 镜像,然后我只需要安装一下镜像,然后就可以直接跑了.<br>

唯一美中不足的一点是他后续追加包的时候没有指定版本:<br>

```txt
pip install einops transformers_stream_generator optimum auto-gptq -i https://pypi.tuna.tsinghua.edu.cn/simple/ --trusted-host pypi.tuna.tsinghua.edu.cn
```

这么下默认是下最新的,如果和视频间隔时间太长,估计会出新的问题,所以到时候我如果跑通了应该还得来吧这个版本指定一下.<br>

不过本身 docker 中的好处就在于,甚至不用担心在系统环境上的微小差异带来的结果.起点已经很高了.<br>

> 果然,这个东西有卡了我一个小时,不过好在,我显卡能够直接被调用.这解决了很多麻烦,我等我测试用的练完了记录一下 version.<br>

最后确定下来的版本是:<br>

```shell
>>> import einops
>>> einops.__version__
'0.8.0'
>>> import transformers
>>> transformers.__version__
'4.34.1'

(llama-factroy) root@docker-desktop:/LLaMA-Factory# pip show transformers_stream_generator
Name: transformers-stream-generator
Version: 0.0.5

>>> import datasets
>>> datasets.__version__
'2.14.6'

(llama-factroy) root@docker-desktop:/LLaMA-Factory# pip show optimum
Name: optimum
Version: 1.23.3

(llama-factroy) root@docker-desktop:/LLaMA-Factory# pip show auto-gptq
Name: auto-gptq
Version: 0.6.0
```

如果写成 requirements.txt:<br>

```txt
einops==0.8.0
transformers==4.34.1
transformers-stream-generator==0.0.5
datasets==2.14.6
optimum==1.23.3
auto-gptq==0.6.0
```

另外除了这些包的问题,你还需要解决 `--gpu all`的问题,如果是 windows 可以参考[Windows 让 Docker 支持 NVIDIA Container Toolkit](https://blog.csdn.net/weixin_43954400/article/details/139997874).<br>

做这些事情当然还是 linux 最方便,奈何我的电脑放在家里只有 windows 可以连.<br>

## 步骤:

### 1. 下载数据集,放到 `data` 文件夹下.

你需要一个`dataset_info.json`.参考我数据集的那个网盘了链接.<br>

注意点是里面那个 sha1 的 key 不能删,我记得这个应该是用来校验的,但是即使我用之前遗留的也是可以的.<br>

但是如果你删除掉了,在 llama-factory 中,就会一直报错 dataset_info.json cannot be found.人

### 2.下载 docker 镜像:

```shell
docker pull bucess/llama-factory:1
```

我用的并不是官网的镜像.<br>

它的下载可能有一点久,并且有时候有一些 layer 似乎还会卡住,但是不用担心,找一个相对稳定的网络环境,接上你的适配器,搭上你的梯子,然后等待即可.<br>

有镜像的话,每次启动就相当快了.甚至容器都配完后,下次直接`start -i` 就可以了.<br>

### 3. 启动容器:

```shell
docker run -it --name llama-factory --gpus all --network host --shm-size 4g -v D:\senmen\data:/LLaMA-Factory/data bucess/llama-factory:1 /bin/bash
```

你需要改得只有`D:\senmen\data`,改成你数据集的位置,当然如果你够自信的话,把模型也放进来也可以.当然不小心误删了就扑街了.<br>

最好的尝试是用`docker cp`进行模型的拷贝.<br>

### 4. 追加 Python 包,启动 llama-factory

```shell
pip install einops==0.8.0 transformers==4.34.1 transformers-stream-generator==0.0.5 datasets==2.14.6 optimum==1.23.3 auto-gptq==0.6.0 -i https://pypi.tuna.tsinghua.edu.cn/simple/ --trusted-host pypi.tuna.tsinghua.edu.cn
```

`-i https://pypi.tuna.tsinghua.edu.cn/simple/ --trusted-host pypi.tuna.tsinghua.edu.cn`<br>

这个参数还挺好使的,因为容器内似乎默认是没有代理的,用这个的话就不用特地再设置一下临时的`http_proxy`了.<br>

修改 `src/train_web.py`:(可选)<br>

在我启动 gradio 应用后,我无法通过 ip+port 的形式在本地访问到面板,所以我用了 gradio 的 share 参数.<br>

```shell
(llama-factroy) root@docker-desktop:/LLaMA-Factory# cat src/train_web.py
from llmtuner import create_ui


def main():
    demo = create_ui()
    demo.queue()
    demo.launch(share=True, inbrowser=True)


if __name__ == "__main__":
    main()
```

值得注意的是,你需要下载一个这个:<br>
[https://cdn-media.huggingface.co/frpc-gradio-0.2/frpc_linux_amd64](https://cdn-media.huggingface.co/frpc-gradio-0.2/frpc_linux_amd64)<br>

它总是被 windows 报毒然后自动删除.<br>

你需要把它放到合适的地方,并且改名,运行 share=True 但是没有这个文件的时候就会自动抛出,这里不追述了.你也可以参考:<br>

[https://github.com/gradio-app/gradio/issues/8186](https://github.com/gradio-app/gradio/issues/8186)<br>

这里有报错的细节.<br>

### 5. 启动 llama-factory

```shell
python src/train_web.py
```

如果执行了 4 那么这样它会生成一个临时的网址,你可以在任何计算机上调用它.<br>

类似这样:[https://d6cdc0f5cda64dd72b.gradio.live/](https://d6cdc0f5cda64dd72b.gradio.live/)<br>

因为我的电脑放在家里,而我需要在学校训练,调用和演示,这个就很方便了.<br>

### 6. 载入我们的 top100 数据集并训练

在这之后都是 webui 操作了,我一个写博客的就不写了.<br>

你需要把 dataset_info.json 中的`"name"`后面改成`"qa_top100.json"`.<br>

具体操作可以参考: [4060Ti16G 显卡图形化微调训练通义千问 Qwen 模型（适合新手朋友）](https://www.bilibili.com/video/BV1wu4y1p7Yr/)<br>

有意思的是他用到了一种做法,就是用 int4 量化模型进行训练,最后偷天换日导出成 float32 的,不仅降低的训练门槛,而且导出非量化模型后,推理速度变得非常快.<br>

直接解决我最大的问题,就是我先前尝试直接训练非量化模型,哪怕用了 lora,batch_size 设置为 1,依然还是爆显存,而且还不稳定,远程桌面黑屏,闪烁.<br>

而经过这么一番操作,我的 batch_size 设置成 8,显存占用在 13G 左右,占用率一直拉满,但是不影响我远程桌面,这就很好了.<br>

### 7.断点续训(还没做)

这个是我最后要做的,因为我想要在小数据集上多轮,然后在大数据集上少轮,这样模型就能够既学会说话方式,又能够在大数据集上学到更多的知识.<br>
