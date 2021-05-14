---
layout: post
category: NLP
title: 【深度学习和自然语言处理】Word2Vec 中文文本聚类分析
tagline: by Kanglei Zhou
author: Kanglei Zhou
tags: 
  - NLP
published: true
---

# 问题描述

>利用 `Word2Vec` 模型训练 `Word Embedding`，根据小说中人物、武功、派别或者其他你感兴趣的特征，基于 `Word Embedding`来进行聚类分析。

# 引言

Word2vec (W2V) 是一种自然语言处理算法，它接受文本语料库作为输入并输出每个单词的向量表示，如下图所示：

![Word2Vec](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-5-14/1620955977505-image.png)

word2vec 有两种类型，比如 CBOW 和 Skip-Gram。

- 给定一组句子 (也称为语料库)，模型的循环在每个句子的话说，尝试使用当前词 $w$ 为了预测其领域 (即它的上下文)，这种方法被称为 Skip-Gram。
- 或者使用这些上下文预测当前词 $w$，在这种情况下，这种方法被称为连续语言袋 (CBOW)。为了限制每个上下文中的字数并调优模型的性能，使用了一个称为 `window size` 的参数。

![Skip-Gram 模型](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-5-14/1620956195137-image.png)

我们用来表示单词的向量被称为神经单词嵌入，这种表示很奇怪。一件事描述另一件事，即使这两件事完全不同。正如埃尔维斯·科斯特洛(Elvis Costello)所说:“写音乐就像跳舞写建筑。”Word2vec对单词进行“向量化”，通过这样做，它使自然语言具有计算机可读性——我们可以开始对单词执行强大的数学运算来检测它们的相似性。

因此，一个神经单词嵌入表示一个具有连续数字的单词。这是一个简单却不太可能的翻译。Word2vec类似于一个自动编码器，将每个单词编码到一个向量中，但是Word2vec并不像受限的玻尔兹曼机器那样通过重构来训练输入单词，相反，Word2vec训练单词与输入语料库中与它们相邻的其他单词进行训练。

它使用两种方法来实现这一点，一种是使用上下文来预测目标词（CBOW），另一种是使用一个词来预测目标上下文，这种方法称为 Skip-Gram。

![CBOW & Skip-Gram](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-5-14/1620956567556-image.png)



# 实验与结果

## 数据库

数据库：[金庸小说文本]( https://share.weiyun.com/5zGPyJX)

## 小说预处理

### 读取数据

读取数据集以及自定义词库，[金庸小说人物名称、武功类型、门派](https://blog.csdn.net/weixin_50891266/article/details/116750204)。

```python
def get_data(data_path='./data/*.txt', custom_path='./custom/*.txt'):
    data = []
    novels = glob.glob(data_path)
    for novel in novels:
        with open(novel, 'r', encoding='ANSI') as f:
            data += f.readlines()

    custom_words = []
    custom_files = glob.glob(custom_path)
    for custom_file in custom_files:
        with open(custom_file, 'r', encoding='ANSI') as f:
            for line in f.readlines():
                custom_words.append(line.strip())

    return data, custom_words
```

## 去停词与分词

多线程处理语料库、分词、去停词处理。

```python
def normalize(corpus, custom_words):
    normalized_corpus = []
    stopwords = stop_words()

    for words in custom_words:
        for word in words:
            jieba.add_word(word)

    p = multiprocessing.Pool(64)
    args = [i for i in range(len(corpus))]
    pbar = tqdm(range(len(corpus)))
    long_string = ''
    for i in p.imap(get_idx, args):
        pbar.set_description('Process corpus sentence %5d' % i)

        text = corpus[i]
        # 去尾部
        text = text.strip()

        # 分词并去掉标点符号
        result = jieba.tokenize(text)
        tokens = []
        for res in result:
            tokens.append(res[0])

        # 去掉停用词
        filtered_tokens = [token for token in tokens if token not in stopwords]

        # 重新组成字符串
        normalized_corpus.append(filtered_tokens)
        long_string += ' '.join(filtered_tokens)

        pbar.update()

    pbar.close()
    p.close()
    p.join()

    plot_word_cloud(long_string)

    return normalized_corpus
```

## 词云

操作后语料库词云如下：

![语料库词云](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-5-14/1620995387783-cloud_img.png)

## 训练模型

通过 `gensim.models.Word2Vec` 构建 Word2Vec 模型，训练模型并保存。

```python
def train():
    norm_corpus = normalize(corpus=corpus, custom_words=custom_words)

    import gensim
    model = gensim.models.Word2Vec(vector_size=200, window=5,
                                   min_count=5, sg=1)
    model.build_vocab(norm_corpus)
    model.train(norm_corpus, total_examples=model.corpus_count, epochs=20)

    model.save(saved_model)
```

## 实验结果

### 查看词嵌入

以 `令狐冲` 为例查看其嵌入的向量：

```python
model.wv['令狐冲']
Out[8]: 
array([ 0.16383415,  0.26941463,  0.03263314, -0.35859594,  0.30975577,
       -0.245559  ,  0.09852615,  0.06929309,  0.2507578 , -0.28308553,
       -0.30238128, -0.41232523,  0.16410291,  0.4930056 ,  0.06439818,
       -0.1863949 , -0.04899335,  0.6459807 , -0.19528963, -0.3140584 ,
        0.2116151 , -0.08661968, -0.36736426, -0.2641598 ,  0.25120872,
        0.05615507,  0.16276185,  0.51612407,  0.04362673,  0.11912314,
       -0.36782798,  0.39724383, -0.81228423,  0.39434844,  0.29756242,
        0.18215784,  0.5232929 , -0.23581408,  0.1351785 , -0.39415947,
        0.34234893, -0.23584719,  0.23376088, -0.53005457,  0.44552347,
        0.16316983, -0.40741006,  0.0989117 , -0.1866204 , -0.04757728,
       -0.14931923, -0.07125449, -0.41635424, -0.3593136 ,  0.30076352,
        0.22683367,  0.53120863,  0.12497195, -0.6726652 ,  0.14259414,
        0.32427472, -0.08628706,  0.5682082 ,  0.3798373 , -0.0671015 ,
        0.2241241 , -0.4151039 ,  0.2837045 , -0.14595926,  0.18722235,
        0.0024842 ,  0.3334524 , -0.04202671,  0.02137416, -0.48620147,
       -0.01953511,  0.5335392 , -0.19765994, -0.01947588, -0.6493784 ,
        0.23959564, -0.2712484 , -0.07869682,  0.18532208, -0.00336053,
       -0.09525409, -0.13534947,  0.5062487 , -0.10846655, -0.02310495,
       -0.22710837,  0.13785207, -0.05622861,  0.5779048 , -0.24833012,
       -0.31272402, -0.12381389, -0.27721483,  0.02720468,  0.0744441 ,
       -0.43680334, -0.22813736, -0.2627481 ,  0.0032804 , -0.14142025,
       -0.38555196,  0.4055573 ,  0.11679924,  0.63316095, -0.13905331,
        0.2627934 , -0.39064264, -0.3122677 ,  0.13288297, -0.15779033,
        0.12704913, -0.10645746, -0.18252599, -0.11423763,  0.16313401,
       -0.62354773,  0.08368128, -0.08954749,  0.11983813,  0.2225815 ,
       -0.01778151,  0.0687526 , -0.06429934, -0.17900665,  0.2183366 ,
        0.12210406,  0.1561541 , -0.31141546, -0.7763977 ,  0.00760035,
       -0.17516743, -0.4079042 , -0.6684582 ,  0.33667275, -0.23411557,
        0.12042478, -0.15075366, -0.01846235,  0.25211698,  0.05110613,
       -0.49119702, -0.06471252,  0.37811613, -0.05658907, -0.26165402,
       -0.18720679,  0.22312479,  0.25461906, -1.0482081 , -0.21202442,
        0.11204756,  0.00746619, -0.0632154 , -0.51421845,  0.45785317,
        0.12428394, -0.11102039, -0.21326531,  0.52435845,  0.16726999,
       -0.12969409,  0.602377  ,  0.16192631, -0.11834691, -0.25618786,
       -0.14465308,  0.21423307,  0.15210441, -0.1453783 , -0.5315343 ,
        0.11270276, -0.08390492, -0.05077346,  0.68695533,  0.10158756,
        0.06952148,  0.16219075, -0.6428004 ,  0.02823924,  0.5273208 ,
        0.1683068 , -0.07116082, -0.51681507,  0.0880248 ,  0.05936658,
        0.30453426, -0.32419795, -0.40934226, -0.5230098 ,  0.14864339,
        0.28540105,  0.3296504 , -0.2303007 , -0.28696567,  0.4124479 ],
      dtype=float32)
```

### 武功

查看 `降龙十八掌` 最相似的武功：

```python
model.wv.most_similar('降龙十八掌')
Out[11]: 
[('打狗棒法', 0.8700282573699951),
 ('伏虎掌', 0.8582083582878113),
 ('空明拳', 0.8556089997291565),
 ('亢龙有悔', 0.8552833199501038),
 ('六合拳', 0.8515743613243103),
 ('逍遥游', 0.8510607481002808),
 ('这路', 0.8503561615943909),
 ('折梅手', 0.8479607701301575),
 ('高山流水', 0.8375046849250793),
 ('神剑掌', 0.8352670073509216)]
```

### 门派

查看 `少林寺` 最相似的词语：

```python
model.wv.most_similar('少林寺')
Out[12]: 
[('寺', 0.7871038913726807),
 ('方丈', 0.7419167757034302),
 ('少林', 0.7213168740272522),
 ('清凉寺', 0.7164002060890198),
 ('高僧', 0.694434642791748),
 ('僧众', 0.6689623594284058),
 ('本寺', 0.6639842987060547),
 ('妙谛', 0.6500354409217834),
 ('大师', 0.6495388150215149),
 ('僧', 0.6415991187095642)]
```

### 人物

查看 `韦小宝` 最相似的词语：

```python
model.wv.most_similar('韦小宝')
Out[13]: 
[('多隆', 0.6073522567749023),
 ('康熙', 0.6028948426246643),
 ('索额图', 0.588067352771759),
 ('吴应熊', 0.5510739088058472),
 ('小桂子', 0.5335976481437683),
 ('奴才', 0.5261167883872986),
 ('施琅', 0.5197171568870544),
 ('苏菲亚', 0.5179336667060852),
 ('费要多罗', 0.5138397216796875),
 ('康亲王', 0.5124404430389404)]
```

这个小说应该是博主比较熟悉的，从结果上来看比较合理。

# 总结

其实在真正应用的时候，只需要调用 Gensim （一个 Python 第三方库）的接口就可以。但对理论的探究仍然有必要，你能更好地知道参数的意义、模型结果受哪些因素影响，以及举一反三地应用到其他问题当中，甚至更改源码以实现自己定制化的需求。

# 参考资料

- [[NLP] 秒懂词向量Word2vec的本质](https://zhuanlan.zhihu.com/p/26306795)

# 附录

## 预处理

```python
import glob
from wordcloud import WordCloud
import multiprocessing
from tqdm import tqdm
import jieba

def plot_word_cloud(long_string):
    # Create a WordCloud object
    wordcloud = WordCloud(
        font_path='simkai.ttf',
        mode='RGBA',
        width=800,
        height=450,
        background_color=None,
        max_words=1000)

    # Generate a word cloud
    wordcloud.generate(long_string)

    # Visualize the word cloud
    wordcloud.to_file('cloud_img.png')

def stop_words(path='./stop_words.txt'):  # 中文字符表
    with open(path, 'r', encoding='UTF-8') as f:
        items = f.read()
        return [l.strip() for l in items]

def get_data(data_path='./data/*.txt', custom_path='./custom/*.txt'):
    data = []
    novels = glob.glob(data_path)
    for novel in novels:
        with open(novel, 'r', encoding='ANSI') as f:
            data += f.readlines()

    custom_words = []
    custom_files = glob.glob(custom_path)
    for custom_file in custom_files:
        with open(custom_file, 'r', encoding='ANSI') as f:
            for line in f.readlines():
                custom_words.append(line.strip())

    return data, custom_words

def get_idx(args):
    return args


def normalize(corpus, custom_words):
    normalized_corpus = []
    stopwords = stop_words()

    for words in custom_words:
        for word in words:
            jieba.add_word(word)

    p = multiprocessing.Pool(64)
    args = [i for i in range(len(corpus))]
    pbar = tqdm(range(len(corpus)))
    long_string = ''
    for i in p.imap(get_idx, args):
        pbar.set_description('Process corpus sentence %5d' % i)

        text = corpus[i]
        # 去尾部
        text = text.strip()

        # 分词并去掉标点符号
        result = jieba.tokenize(text)
        tokens = []
        for res in result:
            tokens.append(res[0])

        # 去掉停用词
        filtered_tokens = [token for token in tokens if token not in stopwords]

        # 重新组成字符串
        normalized_corpus.append(filtered_tokens)
        long_string += ' '.join(filtered_tokens)

        pbar.update()

    pbar.close()
    p.close()
    p.join()

    plot_word_cloud(long_string)

    return normalized_corpus
```

## 主程序

```python
import warnings
warnings.filterwarnings('ignore')

import os

from data_normalize import *

saved_model = './model/novel_CBOW.model'

def train():
    norm_corpus = normalize(corpus=corpus, custom_words=custom_words)

    import gensim
    model = gensim.models.Word2Vec(vector_size=200, window=5,
                                   min_count=5, sg=1)
    model.build_vocab(norm_corpus)
    model.train(norm_corpus, total_examples=model.corpus_count, epochs=20)

    model.save(saved_model)

    print('-' * 20)

    return model

if __name__ == "__main__":
    corpus, custom_words = get_data(data_path='./data/*.txt', custom_path='./custom/*.txt')

    model = train()

    import gensim
    model = model.load(saved_model)

    for word in custom_words[1]:
        # print('Word2Vec: ', model[word])
        print(model[word])
        print('Most similar: ', model.most_similar(word))

    print('-' * 20)

```

