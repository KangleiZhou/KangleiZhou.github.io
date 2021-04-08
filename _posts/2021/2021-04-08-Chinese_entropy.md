---
layout: post
category: NLP
title: 【深度学习和自然语言处理】中文平均信息熵
tagline: by Kanglei Zhou
author: Kanglei Zhou
tags: 
  - NLP
published: true

---

# 问题描述

> 首先阅读[文章](https://docs.qq.com/pdf/DUUR2Z1FrYUVqU0ts "【腾讯文档】Entropy_of_English_PeterBrown")，参考文章来计算所提供[数据库](https://share.weiyun.com/5zGPyJX "金庸小说数据库")中中文的平均信息熵。

# 引言

[熵](https://zh.wikipedia.org/wiki/%E7%86%B5_(%E4%BF%A1%E6%81%AF%E8%AE%BA) "维基百科-熵")在信息论中是接收的每条消息中包含的信息的平均量，又被称为信息熵、信源熵、平均自信息量。

依据 Boltzmann's H-theorem，香农把随机变量 $X$ 的熵值 $Η$ 定义如下，其值域为 ${x_1, ..., x_n}$：
$$
\begin{equation}
{\displaystyle \mathrm {H} (X)=\mathrm {E} [\mathrm {I} (X)]=\mathrm {E} [-\ln(\mathrm {P} (X))]}

\end{equation}
$$
其中，$P$ 为 $X$ 的概率质量函数（probability mass function），$E$ 为期望函数，而$\mathrm {I} (X)$ 是 $X$ 的信息量（又称为自信息）。$\mathrm {I} (X)$ 本身是个随机变数。

当取自有限的样本时，熵的公式可以表示为：
$$
\begin{equation}
 \mathrm {H} (X)=\sum _{i}{\mathrm {P} (x_{i})\,\mathrm {I} (x_{i})}=-\sum _{i}{\mathrm {P} (x_{i})\log _{b}\mathrm {P} (x_{i})},

\end{equation}
$$
在这里$b$是对数所使用的底，通常是 2, 自然常数 e，或是 10。当 $b = 2$，熵的单位是 `bit`；当 $b=\mathrm{e}$，熵的单位是 `nat`；而当 $b = 10$，熵的单位是 `Hart`。

$pi = 0$ 时，对于一些 $i$ 值，对应的被加数 $0\times \log_b 0$ 的值将会是 0，这与极限一致。
$$
\begin{equation}
\lim _{p\to 0+}p\log p=0
\end{equation}
$$
还可以定义事件 $X$ 与 $Y$ 分别取 $x_i$ 和 $y_j$ 时的条件熵为
$$
\begin{equation}
 \mathrm {H} (X|Y)=-\sum _{i,j}p(x_{i},y_{j})\log {\frac {p(x_{i},y_{j})}{p(y_{j})}}
\end{equation}
$$
其中$p(x_i, y_j)$为 $X = x_i$ 且 $Y = y_j$ 时的概率。这个量应当理解为你知道$Y$的值前提下随机变量 $X$ 的随机性的量。

# 实验过程

## 多线程数据处理

为了加速数据处理速度，使用 `multiprocessing` 进行实验。实验中，需要逐个读取小说文件，进行句读短句，然后将每一句话作为一行生成的到一个新的文件 `novel_sentence` 中。

```python
    # get every single corpus
    p = multiprocessing.Pool(64)
    args = [i for i in range(len_data_txt)]
    pbar = tqdm(range(len_data_txt))
    for i in p.imap(get_idx, args):
        with open('./corpus/novel_sentence_%s.txt' % filenames[i][:-4], 'w', encoding='utf-8') as f:
            text = data_txt[i]
            pbar.update()
            for x in text:
                if x in ['\n', '。', '？', '！', '，', '；', '：'] and line != '\n':  # 以部分中文符号为分割换行
                    if line.strip() != '':
                        f.write(line.strip() + '\n')  # 按行存入语料文件
                        line = ''
                elif x not in punctuations:
                    line += x

            pbar.set_description("选取中文金庸小说篇数: %d - %s" % ((i + 1), filenames[i][:-4]))
        f.close()

    p.close()
    p.join()
    pbar.close()
```


## `jieba` 分词
"结巴"中文分词：做最好的Python中文分词组件 "Jieba" ，
- 支持三种分词模式：

  - 精确模式，试图将句子最精确地切开，适合文本分析；

  - 全模式，把句子中所有的可以成词的词语都扫描出来, 速度非常快，但是不能解决歧义；

  -   搜索引擎模式，在精确模式的基础上，对长词再次切分，提高召回率，适合用于搜索引擎分词。

- 支持繁体分词

- 支持自定义词典

例如，在获取语料库 `corpus` 后，利用 `jieba.cut` 进行分词。

```python
    for line in corpus:
        for x in jieba.cut(line):
            split_words.append(x)
            words_len += 1
```

## 计算信息熵

首先进行词频统计，方便计算信息熵。

```python
# 词频统计，方便计算信息熵
def get_tf(words):
    tf_dic = {}
    for w in words:
        tf_dic[w] = tf_dic.get(w, 0) + 1
    return tf_dic.items()

```
计算信息熵

```python
  entropy = [-(uni_word[1]/words_len)*math.log(uni_word[1]/words_len, 2) for uni_word in words_tf]
  entropy = round(sum(entropy), 4)
  logging.info("基于词的一元模型的中文信息熵为（%s）: %.4f 比特/词" % (file_path, entropy))
```

打印输出结果，输出格式为 `Markdown` 的 `table`。

```python
    head_name = "#"
    rows_name = [str(i + 1) for i in range(len(filenames))]
    cols_name = ['小说名称', '语料字数', '分词个数', '平均词长', '信息熵（unigram）', '运行时长']
    table = print_markdown_table(head_name, rows_name, cols_name, data)

    entropy = [item[4] for item in data]
    logging.info('Average entropy: %.4f' % (sum(entropy) / len(entropy)))
```


# 实验结果

完整的实验程序见附录，下表详细列举出各个数据文件中的语料库字数、分词字数、平均词长、中文信息熵以及运行时间等。

|    # |     小说名称 | 语料字数 | 分词个数 | 平均词长 | 信息熵（unigram） | 运行时长（s） |
| ---: | -----------: | -------: | -------: | -------: | ----------------: | ------------: |
|    1 | 三十三剑客图 |    61130 |    39288 |   1.5559 |           10.0952 |        0.5773 |
|    2 |   书剑恩仇录 |   497378 |   315547 |   1.5762 |           10.1402 |        3.4593 |
|    3 |       侠客行 |   351180 |   225058 |   1.5604 |            9.8088 |        3.9399 |
|    4 |   倚天屠龙记 |   927849 |   586149 |    1.583 |           10.2504 |        6.3542 |
|    5 |     天龙八部 |  1161260 |   745407 |   1.5579 |           10.2147 |        9.7953 |
|    6 |   射雕英雄传 |   878650 |   565813 |   1.5529 |           10.2301 |       13.0147 |
|    7 |   白马啸西风 |    66480 |    44735 |   1.4861 |            8.9349 |        1.9289 |
|    8 |       碧血剑 |   473019 |   300046 |   1.5765 |           10.2266 |        5.3463 |
|    9 |     神雕侠侣 |   937948 |   609971 |   1.5377 |            10.171 |        8.0408 |
|   10 |     笑傲江湖 |   937966 |   596226 |   1.5732 |             9.937 |       13.3171 |
|   11 |       越女剑 |    15848 |    10381 |   1.5266 |            8.7242 |        0.2463 |
|   12 |       连城诀 |   221523 |   144593 |    1.532 |            9.6638 |        3.2869 |
|   13 |     雪山飞狐 |   128761 |    82802 |    1.555 |            9.7298 |        1.8246 |
|   14 |     飞狐外传 |   426022 |   272067 |   1.5659 |           10.0681 |        6.2722 |
|   15 |       鸳鸯刀 |    34396 |    22574 |   1.5237 |            8.9903 |         0.463 |
|   16 |       鹿鼎记 |  1172641 |   755617 |   1.5519 |            9.9011 |       18.1554 |

通过计算，得到所给 16 个小说中中文平均信息熵为 `9.8179 比特/词`。

# 总结
英语里，很多单词拼写中的字母组合是经常一起出现的，比如 
`ing`, `tion` 等等。 这些组合中，你即使丢掉了一个字母，也不妨碍我们阅读。而中文的话，字与字之间的关联就小多了，一句话丢掉很多字的话，这句话的意思就很难还原了。而关联小，也就是字与字之间出现的频率差距不大，我们不容易猜到下一个字，这时，每个字提供的信息量就大。因此，中文信息熵要比英文高。

# 附录
- 预处理程序

```python
import os
import multiprocessing

import logging
logging.basicConfig(format='%(asctime)s - %(levelname)s %(message)s', level=logging.INFO)

from opencc import OpenCC
opencc = OpenCC('t2s')

from tqdm import tqdm

def stop_punctuation(path):  # 中文字符表
    with open(path, 'r', encoding='UTF-8') as f:
        items = f.read()
        return [l.strip() for l in items]

def read_data(data_dir='./data'):
    data_txt = []
    for root, dirs, files in os.walk(data_dir):
        for file in files:
            file_fullname = os.path.join(root, file)
            logging.info('Read file: %s' % file_fullname)

            with open(file_fullname, 'r', encoding='ANSI') as f:
                data = f.read()
                ad = '本书来自www.cr173.com免费txt小说下载站\n更多更新免费电子书请关注www.cr173.com'
                data = data.replace(ad, '')

                data_txt.append(data)

            f.close()

    return data_txt, files

def get_idx(args):
    return args

def preprocess_sentence():
    line = ''
    data_txt, filenames = read_data(data_dir='./data')
    len_data_txt = len(data_txt)
    punctuations = stop_punctuation('./stop_punctuation.txt')

    # get entire corpus
    with open('./novel_sentence.txt', 'w', encoding='utf-8') as f:
        p = multiprocessing.Pool(64)
        args = [i for i in range(len_data_txt)]
        pbar = tqdm(range(len_data_txt))
        for i in p.imap(get_idx, args):
            pbar.update()
            text = data_txt[i]
            for x in text:
                if x in ['\n', '。', '？', '！', '，', '；', '：'] and line != '\n':  # 以部分中文符号为分割换行
                    if line.strip() != '':
                        f.write(line.strip() + '\n')  # 按行存入语料文件
                        line = ''
                elif x not in punctuations:
                    line += x

            pbar.set_description("选取中文金庸小说篇数: %d - %s" % ((i + 1), filenames[i][:-4]))

        p.close()
        p.join()
        f.close()
        pbar.close()

    # get every single corpus
    p = multiprocessing.Pool(64)
    args = [i for i in range(len_data_txt)]
    pbar = tqdm(range(len_data_txt))
    for i in p.imap(get_idx, args):
        with open('./corpus/novel_sentence_%s.txt' % filenames[i][:-4], 'w', encoding='utf-8') as f:
            text = data_txt[i]
            pbar.update()
            for x in text:
                if x in ['\n', '。', '？', '！', '，', '；', '：'] and line != '\n':  # 以部分中文符号为分割换行
                    if line.strip() != '':
                        f.write(line.strip() + '\n')  # 按行存入语料文件
                        line = ''
                elif x not in punctuations:
                    line += x

            pbar.set_description("选取中文金庸小说篇数: %d - %s" % ((i + 1), filenames[i][:-4]))
        f.close()

    p.close()
    p.join()
    pbar.close()

    return filenames


def print_markdown_table(head_name, rows_name, cols_name, data):
    """
    Params:
        head_name: {str} 表头名， 如"count比例"
        rows_name, cols_name: {list[str]} 项目名， 如 1,2,3
        data: {ndarray(H, W)}

    Returns:
        table: {str}
    """
    ELEMENT = " {} |"

    H, W = len(data), len(data[0])
    LINE = "|" + ELEMENT * W

    lines = []

    lines += ["| {} | {} |".format(head_name, ' | '.join(cols_name))]

    ## 分割线
    SPLIT = "{}:"
    line = "| {} |".format(SPLIT.format('-' * len(head_name)))
    for i in range(W):
        line = "{} {} |".format(line, SPLIT.format('-' * len(cols_name[i])))
    lines += [line]

    ## 数据部分
    for i in range(H):
        d = list(map(str, list(data[i])))
        lines += ["| {} | {} |".format(rows_name[i], ' | '.join(d))]

    table = '\n'.join(lines)
    print(table)
    return table
```

- 信息熵计算程序
```python
import jieba
import math
import time

from util.tools import *

# 词频统计，方便计算信息熵
def get_tf(words):
    tf_dic = {}
    for w in words:
        tf_dic[w] = tf_dic.get(w, 0) + 1
    return tf_dic.items()

def calculate_entropy(file_path='./novel_sentence.txt'):
    before = time.time()
    with open(file_path, 'r', encoding='utf-8') as f:

        corpus = f.read()  # 读取文件得到语料库文本
        split_words = [x for x in jieba.cut(corpus)]  # 利用jieba分词
        words_len = len(split_words)

        logging.info("语料库字数: %d" % len(corpus))
        logging.info("分词个数: %d" % words_len)
        logging.info("平均词长: %.4f" % round(len(corpus)/words_len, 4))

        words_tf = get_tf(split_words)  # 得到词频表
        # logging.info("词频表: " + tf_words)

        entropy = [-(uni_word[1]/words_len)*math.log(uni_word[1]/words_len, 2) for uni_word in words_tf]
        entropy = round(sum(entropy), 4)
        logging.info("基于词的一元模型的中文信息熵为（%s）: %.4f 比特/词" % (file_path, entropy))

    after = time.time()
    runtime = round(after - before, 4)
    logging.info("运行时间: %.4f s" % runtime)

    return [file_path[24:-4], len(corpus), words_len, round(len(corpus)/words_len, 4),  entropy, runtime]


if __name__ == '__main__':

    filenames = preprocess_sentence()

    # calculate all the data
    item = calculate_entropy(file_path='./novel_sentence.txt')

    # calculate every single data
    data = []
    runtime_single = []
    for i in range(len(filenames)):
        item = calculate_entropy(file_path='./corpus/novel_sentence_%s.txt' % filenames[i][:-4])
        data.append(item)

    head_name = "#"
    rows_name = [str(i + 1) for i in range(len(filenames))]
    cols_name = ['小说名称', '语料字数', '分词个数', '平均词长', '信息熵（unigram）', '运行时长']
    table = print_markdown_table(head_name, rows_name, cols_name, data)

    entropy = [item[4] for item in data]
    logging.info('Average entropy: %.4f' % (sum(entropy) / len(entropy)))

```