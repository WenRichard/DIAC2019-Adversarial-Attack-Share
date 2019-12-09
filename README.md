# DIAC2019-Adversarial-Attack-Share
 
### 目录
- 比赛背景
- 数据
- 评价指标
- 资料区
- Trick区
- 完整思路区
- 疑惑区
- 前排解决方案

----------------------------------------------------------------
**如何准确的判别用户的输入是否为给定问题的语义等价问法** 是比较有意思的任务，也具有很大的挑战性，感谢主办方提供这一赛道！比赛前期也有参与，个人主要将一些比较传统的模型进行实验对比，但是成绩并不理想（不过80），再加上计算资源紧缺，也就放弃了。相比于传统ESIM模型，裸BERT-Base就可以到86分左右。因此，目前看来，BERT算是一个起跑线了，哭瞎，贫穷的孩子不配玩比赛啊 TT 。引用张俊林老师一句话，快0202年了，放弃幻想，全面拥抱BERT吧。  
  
**声明：**  
本项目主要搬运大佬们的前排解决方案，真的非常感谢 **沐鑫**， **hanyan**， **大白**，**pengbo**，**观** 等大佬们的无私分享！非常感谢！非常感谢！

## [DIAC2019基于Adversarial Attack的问题等价性判别比赛](https://www.biendata.com/competition/2019diac/)

--------------------------------------------------------------
  
# 比赛背景  
虽然近年来智能对话系统取得了长足的进展，但是针对专业性较强的问答系统（如法律、政务等），如何准确的判别用户的输入是否为给定问题的语义等价问法仍然是智能问答系统的关键。举例而言，**“市政府管辖哪些部门？”** 和 **“哪些部门受到市政府的管辖？”** 可以认为是语义上等价的问题，而 **“市政府管辖哪些部门？”** 和 **“市长管辖哪些部门？”** 则为不等价的问题。 
  
针对问题等价性判别而言，除去系统的准确性外，系统的鲁棒性也是很重要、但常常被忽略的一点需求。举例而言，虽然深度神经网络模型在给定的训练集和测试集上常常可以达到满意的准确度，但是对测试集合的稍微改变（Adversarial Attack）就可能导致整体准确度的大幅度下降。
  
如以下样例：  

|origin example|-adversarial example|
|-|-|
|**检察机关**提起公益诉讼是什么意思|**监察机关**提起公益诉讼是什么意思|
|检察机关**提起**公益诉讼是什么意思|检察机关**发起**公益诉讼是什么意思|
|**寻衅滋事**一般会怎么处理|**寻衅兹事**一般会怎么处理|
|什么是公益诉讼|**请问**什么是公益诉讼|
  
右列数据是左列数据出现一些错别字或者添加一些无意义的干扰词产生的，并不影响原句的意思，但是这样的微小改动可能会导致完全不同结果。从用户的角度而言，这意味着稍有不同的输入就可能得到完全不一样的结果，从而严重降低用户的产品使用体验。  
  
--------------------------------------------------------------
# 数据
## 1. 数据详情
本次大赛提供的是一个 **法律领域** 的问句等价性数据集，该数据集为我们在实际项目中开发系统所使用的数据集。除去该数据集外，参赛选手可以使用数据增强后的数据集或从其他渠道获得的无等价性标注的数据集。 **参赛选手不得使用人工进行标注的数据。**  

训练集根据在实际项目中的数据情况，以问题组的形式提供，每组问句又分为等价部分和不等价部分，等价问句之间互相组合可以生成正样本，等价问句和不等价问句之间互相组合可以生成负样本。我们提供 **6000组问句** 的训练集， **每组平均有三个等价问句和3个不等价问句** 。验证集和测试集则以问句对的格式提供，其中验证集有5000条数据。测试集中除了人工标注的样本外，还会有大量adversarial example。  

训练集用于模型的学习，比赛期间选手可以提交验证集的预测结果，但最终成绩由测试集的预测结果决定，由于 **测试集中会有大量adversarial example** ，因此最终在测试集上的成绩可能会与验证集上不同。（测试集将于比赛结束前24小时发布，具体可参考时间轴页面。）  
  
 **训练数据集：** [DIAC2019](https://pan.baidu.com/share/init?surl=KY_b7lfpIABb54Ov33A2OQ) ,       提取码：ayvx  
 **测试集** [抽样样例](https://pan.baidu.com/s/1WrGhfl6MaicUcuLMGyphcw)，  提取码：k4im  
 
   
## 2. 数据格式
**train_set.xml**   
训练集以XML文件提供，用于训练模型。XML文件中内容格式如下：  
![训练集]( https://github.com/WenRichard/DIAC2019-Adversarial-Attack-Share/raw/master/picture/数据格式.png)  
  
每一个Questions标签中为一组数据，其中EquivalenceQuestions标签内的问句之间互为等价关系，NotEquivalenceQuestions标签内的问句与EquivalenceQuestions为不等价关系。EquivalenceQuestions之间的问句互相组合可以生成正样本（label为1），EquivalenceQuestions和NotEquivalenceQuestions之间的问句互相组合可以生成负样本（label为0），具体需要生成多少正样本多少负样本由参赛选手自行决定。  

**dev.set.csv**  
验证集数据格式如下：  
![验证集]( https://github.com/WenRichard/DIAC2019-Adversarial-Attack-Share/raw/master/picture/验证集数据格式.png)  
  

--------------------------------------------------------------
# 评价指标
本次比赛的评测标准为 **Macro F1** 值。Macro F1等于每个类别的F1的均值。  
Macro F1 = (F1_正样本 + F1_负样本) / 2  
  
--------------------------------------------------------------
# 资料区
- **文本对抗样本生成-官方推荐论文：**  
[1. A Survey: Towards a Robust Deep Neural Network in Text Domain](https://arxiv.org/pdf/1902.07285.pdf)  
[2. Analysis Methods in Neural Language Processing: A Survey](https://www.mitpressjournals.org/doi/full/10.1162/tacl_a_00254)
  
- **文本对抗样本生成-群友推荐论文：**  
[1. FreeLB: Enhanced Adversarial Training for Language Understanding](https://arxiv.org/abs/1909.11764)  
[2019/12/5 下午一点FreeLB的作者朱晨有个直播分享](https://mp.weixin.qq.com/s/Sh4ZELkQoU4g1dMP0aoBUg)  

- **可借鉴博客：**  
[1. NLP中的对抗训练 + PyTorch实现](http://fyubang.com/2019/10/15/adversarial-train/)  
[2. 图解2018年领先的两大NLP模型：BERT和ELMo](https://new.qq.com/omn/20181214/20181214A0M9D6.html)  
[3. 非平衡数据集 focal loss 多类分类](https://medium.com/swlh/multi-class-classification-with-focal-loss-for-imbalanced-datasets-c478700e65f5)  
[4. 华为诺亚方舟开源哪吒、TinyBERT模型，可直接下载使用](https://mp.weixin.qq.com/s/M2ZNjB0tbnB6uYAh97YyIQ)

- **其他比赛可借鉴代码：**  
[1. 郭大-CCF-BDCI-Sentiment-Analysis-Baseline](https://github.com/guoday/CCF-BDCI-Sentiment-Analysis-Baseline)   

- **非比赛可借鉴代码：**  
[1. focal-loss-keras](https://github.com/mkocabas/focal-loss-keras)  
[2. focal-loss-pytorch](https://github.com/clcarwin/focal_loss_pytorch)  
[3. focal-loss-tensorflow](https://github.com/ailias/Focal-Loss-implement-on-Tensorflow)  
[4. ZEN: A BERT-based Chinese Text Encoder Enhanced by N-gram Representations](https://github.com/sinovation/ZEN) 
  
--------------------------------------------------------------
# Trick区  
1. 对抗训练 + fgm + epsilon 1e-6  
2. ZEN模型，score: 89.04  
3. test 数据输入 neo4j 就会发现神奇的图特征 leak  
4. 一种很差的leak，dev_set中，a-b b-c ，像这种左右（即b）都只出现一次的，a,b,c之间都是等价的，那其他和a、b、c在一起的都不是等价的  
5. 对付这种不平衡样本，用凯明大神的focal loss, focal loss直接把正样本召回率拉起来，6fold+focal loss，score为92+；单fold，score只能到91  
6. 去掉dropout也有提升  
7. focal loss的gamma和alpha取多少合适? (gamma2,alpha 1)效果较好，gamma1和4都不好，别的没试过, **alpha设为1保留疑问**    
8. 蒸馏学习，单模，score：91  
9. 关于**测试集抽样样例：** 用一个单模简单测了一下测试集抽样样例，20条正常样本（问题A + 问题B）准确率100%，20条对抗样本（问题A + 问题B'）错了7条，准确率65%，整体准确率82.5%。模型对对抗攻击样本保持标签不变的误判较多。10条保持label不变的对抗样本错4条，另10条label改变的样本错3条；另一个模型对抗样本错误分别是6条和3条。**建议关注如何增强模型对句子中词同义替换、错别字攻击后保持标签不变这种情况的判别鲁棒性。**  
  
--------------------------------------------------------------
# 完整思路区（赛前）
**思路1：** 对应trick 1，2  
模型：用chinese-wwm-ext-base  
郭大代码：https://github.com/guoday/CCF-BDCI-Sentiment-Analysis-Baseline  
加入对抗：代码比较简单加入5行就行，大家可以参考：http://fyubang.com/2019/10/15/adversarial-train/  
  
**思路2：** score： 89.04  
zen模型，中文n-gram:https://github.com/sinovation/ZEN ，中文n-gram  
只是用来学习，大佬可以试试，没有调到90，到时可以通知一下，相互学习  

**思路3：** score： 91  
[图解2018年领先的两大NLP模型：BERT和ELMo](https://new.qq.com/omn/20181214/20181214A0M9D6.html)这篇文章采用多层cls分类，验证过的单折91,这代码很像bert出来前一个很流行的框架compare-aggregate  
  
 **思路4：** score： 90.8  
[纯BERT+large模型 ](https://github.com/xmxoxo/SameQuestion) 有数据增强，例如pingyin，同义词替换，随机插入等部分。5-Fold 90.8

--------------------------------------------------------------------
# 疑惑区
1. 比赛数据中的leak到底是啥？期待比赛结束后大佬们的解释  
  
--------------------------------------------------------------------
# 前排解决方案
未完待续...  
  
------------------------------------------------------------------
**留言请在Issues或者email richardxie1205@gmail.com**
