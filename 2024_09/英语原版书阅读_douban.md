# 前言：为什么要开发它

在所有之前，我早上去测了一下我的英语词汇量:[Test Your Vocabulary](https://preply.com/en/learn/english/test-your-vocab)

![Snipaste_2024-09-07_08-19-40](https://image.baidu.com/search/down?url=https://img1.doubanio.com/view/photo/l/public/p2912730169.webp)

学了这么多年英语真的是学到狗身上了，但我发现我却能无障碍的看懂海明威的小说。我记得他说过自己词汇量不多。我也搞不懂，能用很少词汇量说清楚的为什么别人要搞那么复杂呢。

因为我本身也是小说爱好者，所以决定走上看原版小说学英语的路。当然也是受了一个佬的启发:[Learn English Again](https://thiscute.world/posts/learn-english-again/#1-%E8%AF%8D%E6%B1%87%E9%87%8F)

但，我发现，国内的英文原版阅读app的书品真不是一般的差。

随便吐槽一下：

这个10本里面，有七本是同一个人的书，而且标题里带着浓浓的营销的味道。我原本就反感这种把标题起的花里胡哨【不得不说，这其实还要感谢一些译者，为了迎合市场有的会刻意扭曲书名。比如[戒掉恋爱脑](https://book.douban.com/subject/36461204/),它原本的书名是:Vaincre La Dependance Affective【克服情感依赖】，它讲得是独立和情感依赖，精神疾病临床手记。我就想不明白译者是怎么写出这书名的。像这样的译者，这样的市场推出的这样书名的书，我不能够说服自己信信任它。】

![8fc6813a246ddd4eb1166c692a2e2dd1](https://image.baidu.com/search/down?url=https://img1.doubanio.com/view/photo/l/public/p2912730170.webp)

And 科普这一版我更是没绷住。

![4741ce6f805fc49b0f3779aa85a5acf2](https://image.baidu.com/search/down?url=https://img2.doubanio.com/view/photo/l/public/p2912730171.webp)

大英百科是一套书，但就这么被拆成了一百多本独立的。每个栏都开出来充数。

但凡，你在科普里面放上一些泛泛之谈:Computer Science,Artificial Intelligence等等各方面和领域都涉及一点，我都会对你刮目相看。但你这书品真就纯渣。



于是这边，我打算自己做一个主要是网页端的原版书阅读器。【主要还是本地开发太麻烦了，我只会python的QT，打包又大又难写。网页端，也是可以运行在本地的。】

单词查询我打算C有道翻译。不得不说，python真能扩展思维。

![Snipaste_2024-09-07_10-14-56](https://image.baidu.com/search/down?url=https://img3.doubanio.com/view/photo/l/public/p2912730172.webp)

整句的翻译估计得gpt。但那样就需要各自备好token，前期不作考虑。

对于单词的管理和后续的一个复习我这边暂时打算做成yaml和SQLite两种。

前期用yaml，然后学一段时间MYSQL，最后转到SQLite。

结构大概如下:

![Snipaste_2024-09-07_09-51-30](https://image.baidu.com/search/down?url=https://img9.doubanio.com/view/photo/l/public/p2912730174.webp)

![Snipaste_2024-09-07_09-51-54](https://image.baidu.com/search/down?url=https://img3.doubanio.com/view/photo/l/public/p2912730173.webp)

这么做的一个好书，就是我可以随时为word添加新的词性，词条。

甚至后面可以再开一个表添加例句。

所以目前我创建yaml也应该是main.yaml,pos.yaml,以及后面可能会有的{sample/sentence.yaml}。

用于关联word的是唯一标识符ID，后面可以直接移到SQL里面。

前端方面，目前打算同样使用vue。之前没有机会学一下，现在可以补上。后端可能是用python。不太想引入太高复杂度，毕竟C有道等等都要经过python，如果再引入nodejs语言方面就太杂了，有冗余。

但是，我不禁感到有意思，我的初衷是来学英语的，结果变成了软件开发，那么学英语什么时候才能安排上呢？