---
layout: post
category: NLP
title: 【深度学习和自然语言处理】Topic Model 中文文本分类
tagline: by Kanglei Zhou
author: Kanglei Zhou
tags: 
  - NLP
published: true

---





# 问题描述

>在给定的数据库上利用`Topic Model`做无监督学习，学习到主题的分布。可以在数据库中随机选定 $K$ 本小说，在每本小说中随机抽出 $M$ 个段落作为训练数据，并抽出 $N$ 个段落作为测试，利用 `Topic Model` 和其他的分类器对给定的段落属于哪一本小说进行分类。 其中 $K$ 至少为3。

# 引言

`Topic model` 是一种统计建模，用于发现出现在文档集合中的抽象主题。潜在 `Dirichlet` 分配（[**Latent Dirichlet Allocation**](http://blog.echen.me/2011/08/22/introduction-to-latent-dirichlet-allocation/), LDA）是主题模型的一个例子，用于将文档中的文本分类为特定的主题。它构建了每个文档的主题模型和每个主题的单词模型，建模为 `Dirichlet` 分布。

# 狄利克雷分布

`Dirichlet` 分布是 `beta` 分布在高维度上的推广，`beta` 分布是二项分布的共轭先验分布。给定 $\alpha> 0, \beta> 0$，取值范围为 $[0,1]$ 的随机变量 $x$ 的概率密度函数：


$$
f(x;\alpha,\beta)=\frac{1}{B(\alpha,\beta)}x^{\alpha-1}(1-x)^{\beta-1}
$$

其中：



$$
\begin{align}
\frac{1}{B(\alpha,\beta)}&=\frac{\Gamma(\alpha+\beta)}{\Gamma(\alpha)\Gamma(\beta)} \\
\Gamma(z)&=\int_{0}^{\infty}t^{z-1}e^{-t}dt
\end{align}
$$


**注：这便是所谓的 gamma 函数。**

`Dirichlet` 分布的的密度函数形式跟 `beta` 分布的密度函数如出一辙：


$$
f(x_1,x_2,...,x_k;\alpha_1,\alpha_2,...,\alpha_k)=\frac{1}{B(\alpha)}\prod_{i=1}^{k}x_i^{\alpha^i-1}
$$



其中



$$
B(\alpha)=\frac{\prod_{i=1}^{k}\Gamma(\alpha^i)}{\Gamma(\sum_{i=1}^{k}\alpha^i)},\sum_{}x_i=1
$$



至此，我们可以看到二项分布和多项分布很相似，`beta` 分布和 `Dirichlet` 分布很相似。

总之，**可以得到以下几点信息。**

- `beta` 分布是二项式分布的共轭先验概率分布：对于非负实数 $\alpha$ 和 $\beta$ ，我们有如下关系：

  

  $$
  \mathrm{Beta}(p|\alpha,\beta)+\mathrm{Count}(m_1,m_2)=\mathrm{Beta}(p|\alpha+m_1,\beta+m_2)
  $$

  

  其中 $(m_1,m_2)$ 对应的是二项分布 $B(m_1+m_2,p)$ 的记数。针对于这种观测到的数据符合二项分布，参数的先验分布和后验分布都是 `beta` 分布的情况，就是 `Beta-Binomial` 共轭。

- 狄利克雷分布（`Dirichlet` 分布）是多项式分布的共轭先验概率分布，一般表达式如下：

  
  $$
  \mathrm{Dir}(\vec{p}|\vec\alpha)+\mathrm{MultCount}(\vec{m})=\mathrm{Dir}(p|\vec{\alpha}+\vec{m})
  $$
  

  针对于这种观测到的数据符合多项分布，参数的先验分布和后验分布都是 `Dirichlet` 分布的情况，就是 `Dirichlet-Multinomial` 共轭。 

# LDA 模型

对比下本文开头所述的 LDA 模型中一篇文档生成的方式是怎样的：

- 按照先验概率 $P(d_i)$ 选择一篇文档 $d_i$。
- 从狄利克雷分布（即 `Dirichlet` 分布）$\alpha$ 中取样生成文档 $d_i$ 的主题分布 $\theta_i$，换言之，主题分布 $\theta_i$ 由超参数为 $\alpha$ 的 `Dirichlet` 分布生成。
- 从主题的多项式分布 $\theta_i$ 中取样生成文档 $d_i$ 第 $j$ 个词的主题 $z_{i,j}$。
- 从狄利克雷分布（即 `Dirichlet` 分布）$\beta$ 中取样生成主题 $z_{i,j}$ 对应的词语分布 $\phi_{z_{i,j}}$，换言之，词语分布 $\phi_{z_{i,j}}$ 由参数为 $\beta$ 的 `Dirichlet` 分布生成。
- 从词语的多项式分布 $\phi_{z_{i,j}}$ 中采样最终生成词语 $w_{i,j}$。





# 实验与结果

## 数据库

数据库：[金庸小说文本]( https://share.weiyun.com/5zGPyJX)

## 选取小说

随机选取 $K = 5$ 本小说，选取结果打印如下：

```
2021-04-29 17:31:48,262 - INFO Load selected file 书剑恩仇录.txt
2021-04-29 17:31:48,262 - INFO Load selected file 射雕英雄传.txt
2021-04-29 17:31:48,262 - INFO Load selected file 笑傲江湖.txt
2021-04-29 17:31:48,262 - INFO Load selected file 连城诀.txt
2021-04-29 17:31:48,263 - INFO Load selected file 雪山飞狐.txt
```

## 划分数据集

每一本小说随机选取 $N = 200$ 段，其中 $\delta_{\mathrm{test}} = 0.3$ 的作为测试集，其余作为训练集。

```python
    # 选择段落
    for paragraph_idx in selected_paragraph_idx:
        paragraph = data[paragraph_idx]

        if paragraph_idx > train_test_split * paragraph_sum_num:  # train
            paragraph_name = './train/%d-%d' % (file_idx, paragraph_idx)
            with open(paragraph_name, 'w', encoding='UTF-8') as f:
                f.write(paragraph)

        else:
            paragraph_name = './test/%d-%d' % (file_idx, paragraph_idx) # test
            with open(paragraph_name, 'w', encoding='UTF-8') as f:
                f.write(paragraph)
```

## `jieba` 分词

使用 `jieba` 进行分词：

```python
        # 分词并去掉标点符号
        result = jieba.tokenize(text)
        tokens = []
        for res in result:
            tokens.append(res[0])
            
        # 去掉停用词
        filtered_tokens = [token for token in tokens if token not in stopwords]
```

例如如下段落文本：

> 张召重神色沮丧，不敢再行倔强，道：“在下听袁大侠吩咐就是。”陈正德道：“你这武功，在武林中也算顶儿尖儿的了。请教阁下万儿。”张召重道：“在下姓张名召重。不敢请教三位。”陈正德道：“啊，原来是火手判官。袁大哥，他是马真道长的师弟。”袁士霄点头道：“嗯，他师兄不及他。咱们走吧。”一马当先，向前驰去。

分词结果如下：

```
张召重 | 神色 | 沮丧 | ， | 不敢 | 再行 | 倔强 | ， | 道 | ： | “ | 在 | 下 | 听 | 袁大侠 | 吩咐 | 就是 | 。 | ” | 陈正德 | 道 | ： | “ | 你 | 这 | 武功 | ， | 在 | 武林中 | 也 | 算 | 顶儿 | 尖儿 | 的 | 了 | 。 | 请教 | 阁下 | 万儿 | 。 | ” | 张召重 | 道 | ： | “ | 在 | 下 | 姓张 | 名召重 | 。 | 不敢 | 请教 | 三位 | 。 | ” | 陈正德 | 道 | ： | “ | 啊 | ， | 原来 | 是 | 火手 | 判官 | 。 | 袁大哥 | ， | 他 | 是 | 马真道 | 长 | 的 | 师弟 | 。 | ” | 袁士霄 | 点头 | 道 | ： | “ | 嗯 | ， | 他 | 师兄 | 不及 | 他 | 。 | 咱们 | 走 | 吧 | 。 | ” | 一马当先 | ， | 向前 | 驰去 | 。
```

## 去停词

使用[中文停词表](https://github.com/goto456/stopwords/blob/master/cn_stopwords.txt)进行去停词操作：

```python
def stop_words(path='./stop_words.txt'):  # 中文字符表
    with open(path, 'r', encoding='UTF-8') as f:
        items = f.read()
        return [l.strip() for l in items]
```

同样以上述选段为例，去停词结果如下：

```
张召重 | 神色 | 沮丧 | 不敢 | 再行 | 倔强 | 听 | 袁大侠 | 吩咐 | 就是 | 陈正德 | 武功 | 武林中 | 顶儿 | 尖儿 | 请教 | 阁下 | 万儿 | 张召重 | 姓张 | 名召重 | 不敢 | 请教 | 三位 | 陈正德 | 原来 | 火手 | 判官 | 袁大哥 | 马真道 | 长 | 师弟 | 袁士霄 | 点头 | 师兄 | 不及 | 咱们 | 走 | 一马当先 | 向前 | 驰去
```

## 词云

训练集操作后词云如下：

![训练集词云](https://cdn.jsdelivr.net/gh/ZhouKanglei/jidianxia/2021-4-30/1619745233817-cloud_img.png)

## 构建 LDA 模型

通过 `gensim` 构建 LDA 模型：

```python
    corpus_train = [np.array(i.split(' ')) for i in norm_train_corpus]
    corpus_test = [np.array(i.split(' ')) for i in norm_test_corpus]
    corpus = corpus_train + corpus_test

    # Create Dictionary
    id2word = corpora.Dictionary(corpus)

    # Create Corpus
    texts = corpus

    # Term Document Frequency
    corpus = [id2word.doc2bow(text) for text in texts]

    # number of topics
    num_topics = 5

    # Build LDA model
    lda_model = models.LdaMulticore(corpus=corpus,
                                    id2word=id2word,
                                    num_topics=num_topics)

    # Print the Keyword in the 10 topics
    pprint(lda_model.print_topics())
    doc_lda = lda_model[corpus]
```



将所有的集合构建为语料库，聚类出 $N_{\mathrm{topic}} = 5$ 个主题词，如下：

```
[(0,
  '0.006*"陈家洛" + 0.003*"说道" + 0.003*"自己" + 0.003*"听" + 0.003*"他们" + 0.002*"一个" + 0.002*"…" + 0.002*"」" + 0.002*"咱们" + 0.002*"我们"'),
 (1,
  '0.009*"…" + 0.004*"听" + 0.004*"令狐冲" + 0.004*"甚么" + 0.003*"陈家洛" + 0.003*"「" + 0.003*"自己" + 0.003*"」" + 0.003*"说道" + 0.002*"一个"'),
 (2,
  '0.007*"陈家洛" + 0.004*"张召重" + 0.004*"…" + 0.003*"令狐冲" + 0.003*"听" + 0.003*"甚么" + 0.003*"说道" + 0.003*"一个" + 0.003*"自己" + 0.002*"一声"'),
 (3,
  '0.004*"…" + 0.003*"听" + 0.003*"陈家洛" + 0.003*"说道" + 0.003*"令狐冲" + 0.002*"派" + 0.002*"一个" + 0.002*"「" + 0.002*"霍青桐" + 0.002*"两人"'),
 (4,
  '0.009*"」" + 0.008*"「" + 0.005*"说道" + 0.004*"听" + 0.004*"一个" + 0.003*"陈家洛" + 0.003*"』" + 0.003*"…" + 0.003*"『" + 0.003*"甚么"')]
```



## 主题可视化

通过 `pyLDA` 将上述语料库主题可视化：

```python
	# 可视化topic
    pyLDAvis.enable_notebook()

    LDAvis_data_filepath = os.path.join('./ldavis_prepared_' + str(num_topics))

    # this is a bit time consuming - make the if statement True
    # if you want to execute visualization prep yourself
    if True:
        LDAvis_prepared = pyLDAvis.gensim.prepare(lda_model, corpus, id2word)
        with open(LDAvis_data_filepath, 'wb') as f:
            pickle.dump(LDAvis_prepared, f)

    # load the pre-prepared pyLDAvis data from disk
    with open(LDAvis_data_filepath, 'rb') as f:
        LDAvis_prepared = pickle.load(f)

    pyLDAvis.save_html(LDAvis_prepared, '.ldavis_prepared_' + str(num_topics) + '.html')

    print(LDAvis_prepared)
```





# 词袋模型

### 基本步骤

- 特征提取：提取数据集中每幅图像的特征点，然后提取特征描述符，形成特征数据(如：SIFT或者SURF方法)；
- 学习词袋：把处理好的特征数据全部合并，利用聚类把特征词分为若干类，此若干类的数目由自己设定，每一类相当于一个视觉词汇；
- 利用视觉词袋量化图像特征：每一张图像由很多视觉词汇组成，我们利用统计的词频直方图，可以表示图像属于哪一类；



### 具体实现

使用 `sklearn` 建立词袋模型，`CountVectorizer` 会将文本中的词语转换为词频矩阵，它通过fit_transform函数计算各个词语出现的次数。

```python
from sklearn.feature_extraction.text import CountVectorizer

def bow_extractor(corpus, ngram_range=(1, 1)):
    vectorizer = CountVectorizer(min_df=1, ngram_range=ngram_range)
    features = vectorizer.fit_transform(corpus)
    return vectorizer, features
```

另外也可以通过 `TF-IDF`（term frequency–inverse document frequency）将词进行向量化，如下：

```python
from sklearn.feature_extraction.text import TfidfVectorizer

def tfidf_extractor(corpus, ngram_range=(1, 1)):
    vectorizer = TfidfVectorizer(min_df=1, norm='l2', smooth_idf=True, use_idf=True, ngram_range=ngram_range)
    features = vectorizer.fit_transform(corpus)
    return vectorizer, features
```



## 构建分类器

主要构建 `Logistic` 回归模型和支持向量机模型。

```python
def train_predict_evaluate_model(classifier,
                                 train_features, train_labels,
                                 test_features, test_labels):
    # 构建模型
    classifier.fit(train_features, train_labels)
    # 在测试集上预测结果
    predictions = classifier.predict(test_features)
    # 评价模型预测表现
    get_metrics(true_labels=test_labels,
                predicted_labels=predictions)

    return predictions
```



##　实验结果

对模型的评估或测试计算其在验证集或测试集上的预测精度（prediction）、准确率（accuracy）、召回率（recall）和 $F_1$ 值，如下表所示：

| 方法 | 精度 | 准确率 | 召回率 | $F_1 $ 值 |
|-----|-----|-------|-------|----------|
| 基于词袋模型特征的逻辑斯蒂回归模型 | 0.7185 | 0.7556 | 0.7185 | 0.7280 |
| 基于词袋模型的支持向量机模型 | 0.6954 | 0.6928 | 0.6954 | 0.6906 |
| 基于 `tfidf` 的逻辑斯蒂回归模型 | 0.8079 | 0.8218 | 0.8079 | 0.8075 |
| 基于 `tfidf` 的支持向量机模型 | 0.7881 | 0.7879 | 0.7881 | 0.7872 |

由上表可知，基于 `tfidf` 的逻辑斯蒂回归模型取得了最好的结果。



# 总结

主题模型（Topic Model）是以非监督学习的方式对文档的隐含语义结构（latent semantic structure）进行聚类（clustering）的统计模型。通过本次实验，加深了对自然语言处理领域的认识。

# 参考资料

- [第十九节、基于传统图像处理的目标检测与识别(词袋模型BOW+SVM附代码)](https://www.cnblogs.com/zyly/p/9796600.html)
- [我是这样一步步理解--主题模型(Topic Model)、LDA(案例代码)](https://www.jianshu.com/p/433392480c98)
- [Topic Modeling in Python: Latent Dirichlet Allocation (LDA)](https://github.com/kapadias/mediumposts/blob/master/natural_language_processing/topic_modeling/notebooks/Introduction%20to%20Topic%20Modeling.ipynb)

# 附录

## 预处理

```python
import os
import multiprocessing
import numpy as np
np.random.seed(1024)

import logging
logging.basicConfig(format='%(asctime)s - %(levelname)s %(message)s',
                    level=logging.INFO)


from tqdm import tqdm

K = 5 # K 本小说
N = 200 # 每本小说的 10 个段落
train_test_split = 0.3 # test rate

def stop_punctuation(path='./stop_punctuation.txt'):  # 中文字符表
    with open(path, 'r', encoding='UTF-8') as f:
        items = f.read()
        return [l.strip() for l in items]

def generate_random_num(num=5, start=0, end=10):
    random_num = []
    while len(random_num) < num:
        random_num.append(np.random.randint(low=start, high=end))
        random_num = list(set(random_num))

    return random_num

def save_paragraphs(data, file_idx):
    # 替换广告
    ad = '本书来自www.cr173.com免费txt小说下载站\n更多更新免费电子书请关注www.cr173.com'
    data = data.replace(ad, '')
    data = data.replace(' ', '')
    data = data.replace('\t', '')
    data = data.replace('\n\n', '\n')  # 防止分段为空

    data = data.split(sep='\n')

    paragraph_sum_num = len(data)
    selected_paragraph_idx = generate_random_num(num=N, start=0,
                                                 end=paragraph_sum_num)

    # 选择段落
    for paragraph_idx in selected_paragraph_idx:
        paragraph = data[paragraph_idx]

        if paragraph_idx > train_test_split * paragraph_sum_num:  # train
            paragraph_name = './train/%d-%d.txt' % (file_idx, paragraph_idx)
            with open(paragraph_name, 'w', encoding='UTF-8') as f:
                f.write(paragraph)

        else:
            paragraph_name = './test/%d-%d.txt' % (file_idx, paragraph_idx) # test
            with open(paragraph_name, 'w', encoding='UTF-8') as f:
                f.write(paragraph)

# 清空数据集文件
def del_file(path_data):
    for i in os.listdir(path_data) : # 里面是当前目录下面的所有东西的相对路径
        file_data = path_data + "\\" + i #当前文件夹的下面的所有东西的绝对路径
        if os.path.isfile(file_data) == True: #os.path.isfile判断是否为文件,如果是文件,就删除.如果是文件夹.递归给del_file.
            os.remove(file_data)
        else:
            del_file(file_data)


def read_save_data(data_dir='../../data'):

    del_file('./train/')
    del_file('./test/')

    for root, dirs, files in os.walk(data_dir):
        novel_sum_num = len(files)
        # 选择小说
        selected_novel_idx = generate_random_num(num=K, start=0,
                                                 end=novel_sum_num)
        for file_idx in selected_novel_idx:
            file = files[file_idx]
            logging.info('Load selected file %s' % file)

        pbar = tqdm(range(K))
        for file_idx in selected_novel_idx:
            file = files[file_idx]
            pbar.set_description('Process selected file %s' % file)
            file_fullname = os.path.join(root, file)

            with open(file_fullname, 'r', encoding='ANSI') as f:
                data = f.read()
                save_paragraphs(data, file_idx)
                f.close()

            pbar.update()
        pbar.close()


def get_idx(args):
    return args

def preprocess_sentence(data_txt):
    line = ''
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

if __name__ == '__main__':
    read_save_data()
```

## 加载数据

```python
import glob
import os
import jieba

def stop_words(path='./stop_words.txt'):  # 中文字符表
    with open(path, 'r', encoding='UTF-8') as f:
        items = f.read()
        return [l.strip() for l in items]

def get_data(path_train, path_test):
    test_data = []
    train_data = []

    test_label = []
    train_label = []

    files_train = glob.glob(path_train)
    files_test = glob.glob(path_test)

    for train_paras in files_train:
        with open(train_paras, 'r', encoding='utf-8') as train_f:
            train_data.append(train_f.readline())
            _, label = os.path.split(train_paras)
            train_label.append(int(label.split('-')[0]))

    for test_paras in files_test:
        with open(test_paras, 'r', encoding='utf-8') as test_f:
            test_data.append(test_f.readline())
            _, label = os.path.split(test_paras)
            test_label.append(int(label.split('-')[0]))


    return train_data, train_label, test_data, test_label

def normalize(corpus):
    normalized_corpus = []
    stopwords = stop_words()

    for text in corpus:
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
        filtered_text = ' '.join(filtered_tokens)
        normalized_corpus.append(filtered_text)

    return normalized_corpus

```



## 主程序

```python

from data_normalize import get_data, normalize
from feature_extractor import bow_extractor, tfidf_extractor
from train_predict_evaluate import train_predict_evaluate_model
from sklearn.linear_model import SGDClassifier
from sklearn.linear_model import LogisticRegression

import numpy as np
import os


from gensim import corpora, similarities, models
import pprint


import pyLDAvis.gensim
import pickle
import pyLDAvis

if __name__ == "__main__":
    train_corpus, train_labels, \
    test_corpus, test_labels = get_data('./train/*.txt', './test/*.txt')

    norm_train_corpus = normalize(train_corpus)
    norm_test_corpus = normalize(test_corpus)

    corpus_train = [np.array(i.split(' ')) for i in norm_train_corpus]
    corpus_test = [np.array(i.split(' ')) for i in norm_test_corpus]
    corpus = corpus_train + corpus_test

    # Create Dictionary
    id2word = corpora.Dictionary(corpus)

    # Create Corpus
    texts = corpus

    # Term Document Frequency
    corpus = [id2word.doc2bow(text) for text in texts]

    # number of topics
    num_topics = 5

    # Build LDA model
    lda_model = models.LdaMulticore(corpus=corpus,
                                    id2word=id2word,
                                    num_topics=num_topics)

    # Print the Keyword in the 10 topics
    pprint(lda_model.print_topics())
    doc_lda = lda_model[corpus]

    # 可视化topic
    pyLDAvis.enable_notebook()

    LDAvis_data_filepath = os.path.join('./ldavis_prepared_' + str(num_topics))

    # this is a bit time consuming - make the if statement True
    # if you want to execute visualization prep yourself
    if True:
        LDAvis_prepared = pyLDAvis.gensim.prepare(lda_model, corpus, id2word)
        with open(LDAvis_data_filepath, 'wb') as f:
            pickle.dump(LDAvis_prepared, f)

    # load the pre-prepared pyLDAvis data from disk
    with open(LDAvis_data_filepath, 'rb') as f:
        LDAvis_prepared = pickle.load(f)

    pyLDAvis.save_html(LDAvis_prepared, '.ldavis_prepared_' + str(num_topics) + '.html')

    print(LDAvis_prepared)


    # 词袋模型特征
    bow_vectorizer, bow_train_features = bow_extractor(norm_train_corpus)
    bow_test_features = bow_vectorizer.transform(norm_test_corpus)

    # tfidf 特征
    tfidf_vectorizer, tfidf_train_features = tfidf_extractor(norm_train_corpus)
    tfidf_test_features = tfidf_vectorizer.transform(norm_test_corpus)

    # 导入分类器
    svm = SGDClassifier(loss='hinge', max_iter=100)
    lr = LogisticRegression(solver='liblinear')

    # 基于词袋模型特征的逻辑斯蒂回归模型
    print("| 方法 | 精度 | 准确率 | 召回率 | $F_1$ 值 |")
    print("|-----|-----|-------|-------|----------|")
    print("| 基于词袋模型特征的逻辑斯蒂回归模型" , end=' | ')
    lr_bow_predictions = train_predict_evaluate_model(classifier=lr,
                                                      train_features=bow_train_features,
                                                      train_labels=train_labels,
                                                      test_features=bow_test_features,
                                                      test_labels=test_labels)

    # 基于词袋模型的支持向量机模型
    print("| 基于词袋模型的支持向量机模型", end=' | ')
    svm_bow_predictions = train_predict_evaluate_model(classifier=svm,
                                                       train_features=bow_train_features,
                                                       train_labels=train_labels,
                                                       test_features=bow_test_features,
                                                       test_labels=test_labels)

    # 基于tfidf的逻辑斯蒂回归模型
    print("| 基于tfidf的逻辑斯蒂回归模型", end=' | ')
    lr_tfidf_predictions = train_predict_evaluate_model(classifier=lr,
                                                        train_features=tfidf_train_features,
                                                        train_labels=train_labels,
                                                        test_features=tfidf_test_features,
                                                        test_labels=test_labels)

    # 基于tfidf的支持向量机模型
    print("| 基于tfidf的支持向量机模型", end=' | ')
    svm_tfidf_predictions = train_predict_evaluate_model(classifier=svm,
                                                         train_features=tfidf_train_features,
                                                         train_labels=train_labels,
                                                         test_features=tfidf_test_features,
                                                         test_labels=test_labels)
    
    
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.feature_extraction.text import TfidfVectorizer


def bow_extractor(corpus, ngram_range=(1, 1)):
    vectorizer = CountVectorizer(min_df=1, ngram_range=ngram_range)
    features = vectorizer.fit_transform(corpus)
    return vectorizer, features


def tfidf_extractor(corpus, ngram_range=(1, 1)):
    vectorizer = TfidfVectorizer(min_df=1, norm='l2', smooth_idf=True, use_idf=True, ngram_range=ngram_range)
    features = vectorizer.fit_transform(corpus)
    return vectorizer, features
```

