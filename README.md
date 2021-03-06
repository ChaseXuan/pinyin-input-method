# 拼音输入法实验报告

## 实验概况
本次拼音输入法大作业，使用带有OpenList的动态规划算法，实现了基于字的二元模型，并尽可能减少搜索时间。

## 基本思路
本次算法以动态规划为核心，并应用OpenList思想。

假设输入一个长度为l的拼音串pinyin[0],…,pinyin[l-1]，我们想要计算在这个输入下的最可能输出。我们用p_point(column, row)表示计算到第column个拼音，最后一个词选择的是该拼音对应的汉字中第row个的概率。计算P(w<sub>i</sub>|w<sub>i-1</sub>)的时候：

```
P(Wi|Wi-1) = f(W Wi-1)/f[Wi-1]
```

其中P(w<sub>i</sub>|w<sub>i-1</sub>) 表示确定上一个词是w<sub>i</sub>的情况下，这个词后跟着w<sub>i-1</sub>这个词的概率。

## 实现过程
下面的示例在Python 3中运行。在具体实现过程中，主要有以下一些步骤：

#### 数据预处理
* 在函数`data_processing`中对原始数据进行了如下操作：

1. 将一二级汉字表的每一个字转换为索引值的形式，存储到`char_list`
2. 从提供的新浪语料库中选取数据：筛选出语料库中的汉字部分，通过`char_list`将其转换成每个字索引值的列表，存储到`file_list`，因数据量较大大，只选取了2016年11月和2016年10月的数据

* 在函数`dict_gen`中生成字典

1. 根据拼音汉字表，生成每一个拼音的list
2. 通过`char_list`生成每一个拼音键对应的汉字字索引值的值，存储到`pinyin_dict`字典结构中

#### 单字字频及关联字频统计
在函数`p_cal`中对预处理的数据进行了如下操作：

1. 将每个字的单字概率统计存储到`p_char`向量
2. 将相邻两个字的共现概率存储到`p_matrix`矩阵

#### 动态规划过程
在函数`dp`中对输入的拼音字符串通过动态规划的思想得出输出的句子，具体操作如下：

1. `p_point`矩阵中存储当前节点在先验条件下的出现概率，通过实验概况中的公式进行计算
2. `relation`矩阵中存储与当前节点相关的节点，即上一个字的拼音所对应的汉字序列中的位置
3. `open_dict`词典中存储在概率评估条件下的open列表中的节点，key为位置二元组，value为节点的概率值
4. `prior_node`在`open_dict`中搜索概率最大的节点，进行扩展，并对扩展结果重新进行评估，更新`open_dict`
5. 当`open_dict`中元素为空时退出动态规划过程，从最后一个字开始依据`relation`矩阵反推，输出结果

## 实验结果
通过基于字的二元关系模型算法，拼音输入法结果的准确率可以达到75%左右，并且从命令行中的结果可以看到，加入OpenList后的动态规划过程比按层评估的动态规划过程（计算每一个字的最优结果，向后扩展）步骤数要少很多，在实际过程中可以大大减少拼音输入的响应时间。

最终，运行```accuracy_test.py```可以得到正确率：
>字符数 = 7471<br/>
>正确数 = 5574<br/>
>正确率 = 74.6%<br/>

## 总结
在本次算法实现的过程中，利用字的二元关系和动态规划的思想实现了输入法，从多种途径获得数据，从数据中训练程序，具有了初步的人工智能思想。不过从输出结果的不足中可以看到，本次程序还有很多弊端，还有不少可能可以改进的方法。具体可能有以下手段：

* 将字频替换为词频：思路是可以先用正则表达式选出语料库中的中文部分。再对取出的每个句子用jieba分词工具进行分词，将分出的词加入词频统计数据，随后的步骤与本文档相似
* 改善语料库质量，找到更好更多的语料库来提升训练质量
* 将算法拓展到三元甚至更多的模型，以此来提高正确率
* 改进语料库存储方案，如今采用的存储方式过于浪费空间，目前的关联信息需要几百M的存储空间，可以进一步节约空间的使用

## 程序说明
#### 文件说明
```
pinyin-input-method
│
├── README.md 				说明文档
│
├── src
│	├── pinyin_bigram_char.py 	基于字的二元关系程序
│	└── accuracy_test.py 		准确率测试
│
├── resource
│	├── 拼音汉字表.txt
│	├── 一二级汉字表.txt
│	├── test-set.txt 		测试集
│	└── sina_news_gbk
│		├── 2016-11.txt
│		├── 2016-10.txt
│		└── 2016-09.txt
│
└── data
	├── input.txt 			简单数据输入
	├── output.txt 			简单数据输出
	├── input-evaluation.txt 	由测试集得出的输入
	├── output-evaluation.txt 	由测试集得出的输出
	├── correct-answer.txt 		测试集中正确的汉字结果
	├── p_char.pkl 			单字出现概率矩阵文件
	└── p_matrix.pkl 		两字共现概率矩阵文件
```
#### 运行方式说明
* 进入源代码文件夹```cd pinyin-input-method/src```，有三种效果查看方式：
	* ```python3 pinyin_bigram_char.py```可以从键盘读入数据
	* ```python3 pinyin_bigram_char.py ../data/input.txt```可以从```input.txt```读入数据输出到```output.txt```	
	* ```python3 pinyin_bigram_char.py ../data/input-evaluation.txt ../data/output-evaluation.txt```可以从```input-evaluation.txt```读入数据输出到```output.txt-evaluation```
* ```python3 accuracy_test.py```可以得出算法在测试集下的正确率