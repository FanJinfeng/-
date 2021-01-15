# Task 4 用户输入->知识库的查询语句 

> 文章编写人：Chris Van<br/>

## 目录

- [目录](#目录)
- [一、引言](#一引言)
- [二、什么是问答系统？](#二什么是问答系统)
  - [2.1 问答系统简介](#21-问答系统简介)
  - [2.2 Query Understanding](#22-Query-Understanding)
    - [2.2.1  搜索理解](#221--搜索理解)
    - [2.2.2 意图识别](#222-意图识别)
    - [2.2.3 槽值填充](#223-槽值填充)
- [三、任务实践](#三任务实践)
- [四、命名实体识别](#五命名实体识别)
  - [4.1 整体思路](#41-整体思路)
  - [4.2 代码注解](#42-代码注解)
    - [4.2.1 构建 AC Tree](#421-构建-AC-Tree)
    - [4.2.2 使用AC Tree进行实体匹配](#422-使用AC-Tree进行实体匹配)
    - [4.2.3 使用相似度进行实体匹配](#423-使用相似度进行实体匹配)
- [五、意图识别](#五意图识别)
  - [5.1 整体思路](#51-整体思路)
  - [5.2 代码注解](#52-代码注解)
    - [6.2.1 特征构建](#621-特征构建)
    - [6.2.2 使用朴素贝叶斯进行文本分类](#622-使用朴素贝叶斯进行文本分类)
- [参考资料](#参考资料)

## 一、引言

本部分任务主要是**将用户输入问答系统的自然语言转化成知识库的查询语句**。

## 二、什么是问答系统？

### 2.1 问答系统简介

问答系统(Question Answering System，QA System)是用来回答人提出的自然语言问题的系统。

* 问答系统从知识领域划分：
  * 封闭领域：回答特定领域的问题，可以导入领域知识或将答案来源全部转换成结构性资料来有效提升系统的表现。
  * 开放领域：不设限问题的内容范围，其难度也相对较大。

* 问答系统从实现方式划分：
  * 基于流水线（pipeline）实现：工业界主流，由自然语言理解（NLU）、对话状态跟踪器（DST）、对话策略（DPL）和自然语言生成（NLG）依次串联构成的一条流水线，各模块可独立设计，模块间协作完成任务。
  * 基于端到端（end-to-end）实现：结合深度学习技术，通过海量数据训练，挖掘出从用户自然语言输入到系统自然语言输出的整体映射关系。

![](https://upload-images.jianshu.io/upload_images/10798244-16aa357b7be5a646.png?imageMogr2/auto-orient/strip|imageView2/2/w/816/format/webp)

* 问答系统从答案来源划分：
  * 知识库问答：是目前的研究热点，给定自然语言问题，通过对问题进行语义理解和解析，进而利用知识库进行查询、推理得出答案。
  * 常问问题问答
  * 新闻问答
  * 网际网路问答

![](https://upload-images.jianshu.io/upload_images/10798244-afb41aa23fee13c7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 2.2 Query understanding

#### 2.2.1  搜索理解

- 搜索理解(QU, Query Understanding)：简单来说就是从词法、句法、语义三个层面对 Query 进行结构化解析。

- 包含的模块主要有：
  - Query预处理
  - Query纠错
  - Query扩展
  - Query归一
  - 意图识别
  - 槽值填充
  - Term重要性分析；
  - ...

#### 2.2.2 意图识别
   
- 意图识别：用来检测用户当前输入的意图，通常被建模为将一段自然语言文本分类为预先设定的一个或多个意图的文本分类任务。
- 文本分类人物常用模型：
  - 基于词典模板的规则分类
  - 传统的机器学习模型（文本特征工程+分类器）
  - 深度学习模型（Fasttext、TextCNN、BiLSTM + Self-Attention、BERT等）

![](https://upload-images.jianshu.io/upload_images/10798244-467c5be884303091.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
  
#### 2.2.3 槽值填充

- 槽值填充：根据既定的一些结构化字段，将用户输入的信息中与其对应的部分提取出来，通常被建模为序列标注的任务。
- 举例：Query "北京飞成都的机票"，通过意图分类模型可以识别出 Query 的整体意图是订机票，在此基础上进一步语义解析出对应的出发地 Depart="北京"，到达地 Arrive="成都"，所以生成的形式化表达可以是 Ticket=Order(Depart,Arrive)，Depart={北京}，Arrive={成都}。

![](https://upload-images.jianshu.io/upload_images/10798244-25aa5d0560dfee1a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 序列标注任务常用模型：
  - 词典匹配
  - BiLSTM + CRF
  - IDCNN
  - BERT
  - ...

## 三、任务实践

![](https://upload-images.jianshu.io/upload_images/10798244-322785573485895d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 框架介绍
```s
import os
import ahocorasick
from sklearn.externals import joblib
import jieba
import numpy as np

class EntityExtractor:

    # 初始化
    def __init__(self):
        pass

    # 构造actree，加速过滤
    def build_actree(self, wordlist):
        pass
	
    # 模式匹配, 得到匹配的词和类型。如疾病，疾病别名，并发症，症状
    def entity_reg(self, question):
        pass

    # 当全匹配失败时，就采用相似度计算来找相似的词
    def find_sim_words(self, question):
        pass

    # 采用DP方法计算编辑距离
    def editDistanceDP(self, s1, s2):
        pass

    # 计算词语和字典中的词的相似度
    def simCal(self, word, entities, flag):
        pass

    # 基于特征词分类
    def check_words(self, wds, sent):
        pass

    # 提取问题的TF-IDF特征
    def tfidf_features(self, text, vectorizer):
        pass

    # 提取问题的关键词特征
    def other_features(self, text):
        pass

    # 预测意图
    def model_predict(self, x, model):
        pass

    # 实体抽取主函数
    def extractor(self, question):
        pass
```

## 四、命名实体识别

### 4.1 整体思路

- step 1：对于用户的输入，先使用预先构建的疾病、疾病别名、并发症和症状的AC Tree进行匹配；
- step 2：若全都无法匹配到相应实体，则使用结巴切词库对用户输入的文本进行切分；
- step 3：然后将每一个词都去与疾病词库、疾病别名词库、并发症词库和症状词库中的词计算相似度得分（overlap score、余弦相似度分数和编辑距离分数），如果相似度得分超过0.7，则认为该词是这一类实体；
- step 4：最后排序选取最相关的词作为实体。

> 注：项目涉及7类实体，但是命名实体识别仅使用了疾病、别名、并发症和症状四种实体。

### 4.2 代码注解

#### 4.2.1 构建 AC Tree

- 函数模块
```s
    def build_actree(self, wordlist):
        # 构造Aho–Corasick自动机，是多模式匹配的一个经典数据结构。
        # AC自动机，就是在tire树的基础上，增加一个fail指针，如果当前点匹配失败，则将指针转移到fail指针指向的地方，这样就不用回溯，而可以路匹配下去了。
        actree = ahocorasick.Automaton()
        # 向树中添加单词
        for index, word in enumerate(wordlist):
            actree.add_word(word, (index, word))
        actree.make_automaton()
        return actree
```

- 函数调用模块
```s
    def __init__(self):
        ...
        # 实体库路径
        self.disease_path = cur_dir + 'disease_vocab.txt'  # 疾病
        self.symptom_path = cur_dir + 'symptom_vocab.txt'  # 症状
        self.alias_path = cur_dir + 'alias_vocab.txt'  # 别名
        self.complication_path = cur_dir + 'complications_vocab.txt'  # 并发症

        # 读取实体库词条
        self.disease_entities = [w.strip() for w in open(self.disease_path, encoding='utf8') if w.strip()]
        self.symptom_entities = [w.strip() for w in open(self.symptom_path, encoding='utf8') if w.strip()]
        self.alias_entities = [w.strip() for w in open(self.alias_path, encoding='utf8') if w.strip()]
        self.complication_entities = [w.strip() for w in open(self.complication_path, encoding='utf8') if w.strip()]

        # 构造领域actree
        self.disease_tree = self.build_actree(list(set(self.disease_entities)))
        self.alias_tree = self.build_actree(list(set(self.alias_entities)))
        self.symptom_tree = self.build_actree(list(set(self.symptom_entities)))
        self.complication_tree = self.build_actree(list(set(self.complication_entities)))
        ...
```

#### 4.2.2 使用AC Tree进行实体匹配

首先使用AC Tree进行实体匹配。

- 函数模块
```s
    def entity_reg(self, question):  # 对question进行实体识别
    
        self.result = {}

        for i in self.disease_tree.iter(question):  # 基于disease_tree对question进行匹配，返回匹配的结果
            word = i[1][1]
            if "Disease" not in self.result:
                self.result["Disease"] = [word]
            else:
                self.result["Disease"].append(word)

        for i in self.alias_tree.iter(question):  # 基于alias_tree对question进行匹配，返回匹配的结果
            word = i[1][1]
            if "Alias" not in self.result:
                self.result["Alias"] = [word]
            else:
                self.result["Alias"].append(word)

        for i in self.symptom_tree.iter(question):  # 基于symptom_tree对question进行匹配，返回匹配的结果
            wd = i[1][1]
            if "Symptom" not in self.result:
                self.result["Symptom"] = [wd]
            else:
                self.result["Symptom"].append(wd)

        for i in self.complication_tree.iter(question):  # 基于complication_tree对question进行匹配，返回匹配的结果
            wd = i[1][1]
            if "Complication" not in self.result:
                self.result["Complication"] = [wd]
            else:
                self.result["Complication"] .append(wd)

        return self.result
```

- 函数调用模块
```s
    def extractor(self, question):
        self.entity_reg(question)
        ...
```

- 一个多模式匹配的例子
```s
   import ahocorasick
   
   def build_actree(wordlist):
       actree = ahocorasick.Automaton()
       for index, word in enumerate(wordlist):
           actree.add_word(word, (index, word))
        actree.make_automaton()
        return actree
    
    if __name__ == '__main__':
        wordlist = ['我草','尼玛','SB','狗东西']
	sent = '我草尼玛，你是SB吗？狗东西'
	
        actree = build_actree(wordlist=wordlist)
	for i in actree:
	    print(i)
	sent_cp = sent
	for i in actree.iter(sent):
	    print(i)
	    sent_cp = sent_cp.replace(i[1][1], "**")
	    print("屏蔽词:", i[1][1])
	print("屏蔽结果:", sent_cp)
```

#### 4.2.3 使用相似度进行实体匹配

当AC Tree的匹配都没有匹配到实体时，使用查找相似词的方式进行实体匹配。

```s
    def find_sim_words(self, question):
        import re
        import string
        from gensim.models import KeyedVectors
    
        # 使用结巴加载自定义词典
        jieba.load_userdict(self.vocab_path)
        # 加载词向量
        self.model = KeyedVectors.load_word2vec_format(self.word2vec_path, binary=False)
    
        # 数据预处理，正则去除特殊符号
        sentence = re.sub("[{}]", re.escape(string.punctuation), question)  # 把'[{哈哈}]'替换为'[***哈哈***]'
        sentence = re.sub("[，。‘’；：？、！【】]", " ", sentence)  # 标点符号替换为空格
        sentence = sentence.strip()
    
        # 使用结巴进行分词（要求不是停用词且长度大于1）
        words = [w.strip() for w in jieba.cut(sentence) if w.strip() not in self.stopwords and len(w.strip()) >= 2]


        alist = []
        for word in words:
	    
	    # 四个词库中的全部单词
            temp = [self.disease_entities, self.alias_entities, self.symptom_entities, self.complication_entities] 
            
	    for i in range(len(temp)):
                flag = ''
                if i == 0:
                    flag = "Disease"
                elif i == 1:
                    flag = "Alias"
                elif i == 2:
                    flag = "Symptom"
                else:
                    flag = "Complication"
		
		# 计算每个单词与某个实体库的相似性得分
                scores = self.simCal(word, temp[i], flag)
		
		# 存储存储各个单词与四个实体库的相似性得分
                alist.extend(scores)
	
	# 按照相似性得分倒序排列
        temp1 = sorted(alist, key=lambda k: k[1], reverse=True)
        
	if temp1:
            self.result[temp1[0][2]] = [temp1[0][0]]  # 返回相似性得分最高的实体类别 & 实体名称
```

```s
    # 计算词语和字典中的词的相似度
    def simCal(self, word, entities, flag):
        a = len(word)
        scores = []
        for entity in entities:  # 遍历实体库
            sim_num = 0
            b = len(entity)
            c = len(set(entity+word))
            temp = []
            for w in word:
                if w in entity:
                    sim_num += 1
            if sim_num != 0:
                score1 = sim_num / c  # 计算overlap score（本质上是单词的重复性得分）
                temp.append(score1)
            try:
                score2 = self.model.similarity(word, entity)  # 计算余弦相似度分数，直接调用.similarity()计算两个词的余弦距离
                temp.append(score2)
            except:
                pass
            score3 = 1 - self.editDistanceDP(word, entity) / (a + b)  # 编辑距离分数
            if score3:
                temp.append(score3)

            score = sum(temp) / len(temp)  # 将得分的均值作为最终得分
	    
            if score >= 0.7:
                scores.append((entity, score, flag))  # 得分大于7的，记录(实体名称、得分、实体类型)

        scores.sort(key=lambda k: k[1], reverse=True)  # 按照得分降序排列
        return scores
```

```s
    # 计算编辑距离：将word1转换成word2所使用的最少操作数
    def editDistanceDP(self, s1, s2):
        m = len(s1)
        n = len(s2)
	solution = [[0 for j in range(n + 1)] for i in range(m + 1)]  # (m+1) X (n+1)的二维数组
	for i in range(len(s2) + 1):
	    solution[0][i] = i
	for i in range(len(s1) + 1):
	    solution[i][0] = i  # 最左边和最上边记录了更换两个字符需要经过的步数

	for i in range(1, m + 1):
	    for j in range(1, n + 1):
		if s1[i - 1] == s2[j - 1]:
		    solution[i][j] = solution[i - 1][j - 1]  # 取对角线上的值
		else:
		    # 插入、删除、替换
		    solution[i][j] = 1 + min(solution[i][j - 1], min(solution[i - 1][j],
								     solution[i - 1][j - 1]))  # 取一个三角形上的最小值再加1
	return solution[m][n]
```

- 编辑距离
    - 定义：也叫莱文斯坦距离（Levenshtein），是针对两个字符串的差异程度的量化量测，量测的方式是看至少需要多少次的处理才能将一个字符串变成另一个字符串。
    - 例子：计算字符串a='love'，b='lolpe'的编辑距离，love->lolve(插入l)，lolve->lolpe(用v替换成p)，所以编辑距离为2。
    - 公式定义：i和j分别是字符串a和字符串b的下标，下标从1开始。
    ![image](https://www.zhihu.com/equation?tex=%5Coperatorname%7Blev%7D_%7Ba%2C+b%7D%28i%2C+j%29%3D%5Cleft%5C%7B%5Cbegin%7Barray%7D%7Bll%7D%7B%5Cmax+%28i%2C+j%29%7D+%26+%7B%5Ctext+%7B+if+%7D+%5Cmin+%28i%2C+j%29%3D0%7D+%5C%5C+%7B%5Cmin+%5Cleft%5C%7B%5Cbegin%7Barray%7D%7Bll%7D%7B%5Coperatorname%7Blev%7D_%7Ba%2C+b%7D%28i-1%2C+j%29%2B1%7D+%5C%5C+%7B%5Coperatorname%7Blev%7D_%7Ba%2C+b%7D%28i%2C+j-1%29%2B1%7D++%5C%5C%7B%5Coperatorname%7Blev%7D_%7Ba%2C+b%7D%28i-1%2C+j-1%29%2B1_%7B%5Cleft%28a_%7Bi%7D+%5Cneq+b_%7Bj%7D%5Cright%29%7D%7D+%5Cend%7Barray%7D%5Cright.%7D+%26+%7B%5Ctext+%7B+otherwise+%7D%7D%5Cend%7Barray%7D%5Cright.)
    - 具体运算过程
        - 1.建立一个矩阵，用来存储上一步计算好的距离；
	- 2.初始化第一行和第一列的所有距离，即公式中的第一行；
	- 3.然后开始循环计算所有的距离，直到最后一个字符。
	
## 五、意图识别

### 5.1 整体思路

- step 1：利用TF-IDF表征文本特征，同时构建一些人工特征（每一类意图常见词在句子中出现的个数）；
- step 2：训练朴素贝叶斯模型进行意图识别任务；
- step 3：使用实体信息进行意图的纠正和补充。

![](https://upload-images.jianshu.io/upload_images/10798244-6113c647b881e4aa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
> 图 7 意图识别整体举例介绍

该项目通过手工标记210条意图分类训练数据，并采用朴素贝叶斯算法训练得到意图分类模型。其最佳测试效果的F1值达到了96.68%。

### 5.2 代码注解

#### 5.2.1 特征构建

1. TF-IDF特征
```s
# 提取问题的TF-IDF特征
def tfidf_features(self, text, vectorizer):
    """
    提取问题的TF-IDF特征
    :param text:
    :param vectorizer:
    :return:
    """
    jieba.load_userdict(self.vocab_path)
    words = [w.strip() for w in jieba.cut(text) if w.strip() and w.strip() not in self.stopwords]
    sents = [' '.join(words)]

    tfidf = vectorizer.transform(sents).toarray()
    return tfidf
```
2. 人工特征
```s	    
self.symptom_qwds = ['什么症状', '哪些症状', '症状有哪些', '症状是什么', '什么表征', '哪些表征', '表征是什么',
                     '什么现象', '哪些现象', '现象有哪些', '症候', '什么表现', '哪些表现', '表现有哪些',
                     '什么行为', '哪些行为', '行为有哪些', '什么状况', '哪些状况', '状况有哪些', '现象是什么',
                     '表现是什么', '行为是什么']  # 询问症状
self.cureway_qwds = ['药', '药品', '用药', '胶囊', '口服液', '炎片', '吃什么药', '用什么药', '怎么办',
                     '买什么药', '怎么治疗', '如何医治', '怎么医治', '怎么治', '怎么医', '如何治',
                     '医治方式', '疗法', '咋治', '咋办', '咋治', '治疗方法']  # 询问治疗方法
self.lasttime_qwds = ['周期', '多久', '多长时间', '多少时间', '几天', '几年', '多少天', '多少小时',
                      '几个小时', '多少年', '多久能好', '痊愈', '康复']  # 询问治疗周期
self.cureprob_qwds = ['多大概率能治好', '多大几率能治好', '治好希望大么', '几率', '几成', '比例',
                      '可能性', '能治', '可治', '可以治', '可以医', '能治好吗', '可以治好吗', '会好吗',
                      '能好吗', '治愈吗']  # 询问治愈率
self.check_qwds = ['检查什么', '检查项目', '哪些检查', '什么检查', '检查哪些', '项目', '检测什么',
                   '哪些检测', '检测哪些', '化验什么', '哪些化验', '化验哪些', '哪些体检', '怎么查找',
                   '如何查找', '怎么检查', '如何检查', '怎么检测', '如何检测']  # 询问检查项目
self.belong_qwds = ['属于什么科', '什么科', '科室', '挂什么', '挂哪个', '哪个科', '哪些科']  # 询问科室
self.disase_qwds = ['什么病', '啥病', '得了什么', '得了哪种', '怎么回事', '咋回事', '回事',
                    '什么情况', '什么问题', '什么毛病', '啥毛病', '哪种病']  # 询问疾病
 
def other_features(self, text):
	"""
	提取问题的关键词特征
	:param text:
	:return:
	"""
	features = [0] * 7
	for d in self.disase_qwds:
	    if d in text:
	        features[0] += 1
	
	for s in self.symptom_qwds:
	    if s in text:
	        features[1] += 1
	
	for c in self.cureway_qwds:
	    if c in text:
	        features[2] += 1
	
	for c in self.check_qwds:
	    if c in text:
	        features[3] += 1
	for p in self.lasttime_qwds:
	    if p in text:
	        features[4] += 1
	
	for r in self.cureprob_qwds:
	    if r in text:
	        features[5] += 1
	
	for d in self.belong_qwds:
	    if d in text:
	        features[6] += 1
	
	m = max(features)
	n = min(features)
	normed_features = []
	if m == n:
	    normed_features = features
	else:
	    for i in features:
	        j = (i - n) / (m - n)
	        normed_features.append(j)
	
	return np.array(normed_features)

```

#### 5.2.2 使用朴素贝叶斯进行文本分类
- 项目没有给出训练过程，可参考下面sklearn的例子
```s
    # 项目没有给出训练过程，可参考下面sklearn的例子
    from sklearn.naive_bayes import MultinomialNB 

    mnb = MultinomialNB()   
    mnb.fit(X_train,y_train)   
    y_predict = mnb.predict(X_test)

    # 意图分类模型文件
    self.tfidf_path = os.path.join(cur_dir, 'model/tfidf_model.m')
    self.nb_path = os.path.join(cur_dir, 'model/intent_reg_model.m')  #朴素贝叶斯模型
    self.tfidf_model = joblib.load(self.tfidf_path)
    self.nb_model = joblib.load(self.nb_path)

    # 意图预测
    tfidf_feature = self.tfidf_features(question, self.tfidf_model)

    other_feature = self.other_features(question)
    m = other_feature.shape
    other_feature = np.reshape(other_feature, (1, m[0]))
    feature = np.concatenate((tfidf_feature, other_feature), axis=1)
    predicted = self.model_predict(feature, self.nb_model)
    intentions.append(predicted[0])
```

- 根据所识别的实体进行补充和纠正意图
```s
# 已知疾病，查询症状
if self.check_words(self.symptom_qwds, question) and ('Disease' in types or 'Alia' in types):
    intention = "query_symptom"
    if intention not in intentions:
        intentions.append(intention)
# 已知疾病或症状，查询治疗方法
if self.check_words(self.cureway_qwds, question) and \
        ('Disease' in types or 'Symptom' in types or 'Alias' in types or 'Complication' in types):
    intention = "query_cureway"
    if intention not in intentions:
        intentions.append(intention)
# 已知疾病或症状，查询治疗周期
if self.check_words(self.lasttime_qwds, question) and ('Disease' in types or 'Alia' in types):
    intention = "query_period"
    if intention not in intentions:
        intentions.append(intention)
# 已知疾病，查询治愈率
if self.check_words(self.cureprob_qwds, question) and ('Disease' in types or 'Alias' in types):
    intention = "query_rate"
    if intention not in intentions:
        intentions.append(intention)
# 已知疾病，查询检查项目
if self.check_words(self.check_qwds, question) and ('Disease' in types or 'Alias' in types):
    intention = "query_checklist"
    if intention not in intentions:
        intentions.append(intention)
# 查询科室
if self.check_words(self.belong_qwds, question) and \
        ('Disease' in types or 'Symptom' in types or 'Alias' in types or 'Complication' in types):
    intention = "query_department"
    if intention not in intentions:
        intentions.append(intention)
# 已知症状，查询疾病
if self.check_words(self.disase_qwds, question) and ("Symptom" in types or "Complication" in types):
    intention = "query_disease"
    if intention not in intentions:
        intentions.append(intention)

# 若没有检测到意图，且已知疾病，则返回疾病的描述
if not intentions and ('Disease' in types or 'Alias' in types):
    intention = "disease_describe"
    if intention not in intentions:
        intentions.append(intention)
# 若是疾病和症状同时出现，且出现了查询疾病的特征词，则意图为查询疾病
if self.check_words(self.disase_qwds, question) and ('Disease' in types or 'Alias' in types) \
        and ("Symptom" in types or "Complication" in types):
    intention = "query_disease"
    if intention not in intentions:
        intentions.append(intention)
# 若没有识别出实体或意图则调用其它方法
if not intentions or not types:
    intention = "QA_matching"
    if intention not in intentions:
        intentions.append(intention)

self.result["intentions"] = intentions

```

## 参考资料 

1. [ QASystemOnMedicalGraph](https://github.com/zhihao-chen/QASystemOnMedicalGraph)
