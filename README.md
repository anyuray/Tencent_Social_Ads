# Tencent_Social_Ads
第一届腾讯社交广告高校算法大赛(决赛14名)

队名：竟然有这种操作 <br>
最终成绩：0.101595 排名第14<br>
特征维数：64<br>
训练数据为：28，29两天<br>
训练机器内存：16G<br>
算法模型：lightGBM<br>

文件说明:
---
`doFeats_1.py`：根据原始文件生成中间文件<br><br>
`doFeats_2.py`：根据中间文件生成特征<br><br>
`train_cv.py`：训练及预测(LGB相当的快,一组CV预测大概40分钟)<br><br>

主要特征：
---
* Trick特征：<br>
通过观察原始数据是不难发现的,有很多只有clickTime和label不一样的重复数据，按时间排序发现重复数据如果转化，label一般标在头或尾，少部分在中间，在训练集上出现的情况在测试集上也会出现，所以标记这些位置后onehot，让模型去学习，再就是时间差特征，关于trick我比赛分享的[这篇文章](https://www.qcloud.com/community/article/401437)有较详细的说明.

* 统计特征：<br>
原始特征主要三大类：广告特征、用户特征、位置特征，通过交叉组合算统计构造特征，由于机器限制，统计特征主要使用了转化率，丢掉了点击次数和转化次数。初赛利用了7天滑窗构造，决赛采用了周冠军分享的clickTime之前所有天算统计。三组合特征也来自周冠军分享的下载行为和网络条件限制，以及用户属性对app需求挖掘出。贝叶斯平滑user相关的特征特别废时间，初赛做过根据点击次数阈值来操作转化率，效果和平滑差不多但是阈值选择不太准。

* 活跃数特征：<br>
特征构造灵感来自[这里](https://github.com/plantsgo/Rental-Listing-Inquiries/blob/master/xgb.py),比如某个广告位的app种数。

* 均值特征：<br>
比如点击某广告的用户平均年龄

* 平均回流时间特征：<br>
利用回流时间方式不对的话很容易造成leackage，这里参考了官方群里的分享，计算了每个appID的平均回流时间，没有回流的app用其所在类的平均回流时间代替

* 用户流水和历史特征：<br>
利用installed文件关联user和app获得历史统计特征，利用actions进行7天滑动窗口获得用户和app流水特征。

特征筛选：
---
&emsp; 在特征筛选上没太多经验，基本是参考lgb输出的特征的weight重要性和gain重要性来增删特征，weight是特征分裂次数，gain是分裂特征获得的信息增益总量，不过重要性只能做参考，一个比较好的特征选择方法是，每次构造的特征计算其在训练集和测试集上的均值和方差，保证分布相同，差不多则放入模型训练，如果结果比之前好那这个特征就基本可用。

模型与训练：
---
&emsp; 由于队友比较忙，主要工作是自己来，整个特征构造与筛选的过程是利用lgb重要性，所以最后构造出的特征在其他模型上表现不好，模型融合阶段就处于很大的劣势,预测主要采用交叉预测，实际上是stacking第一层做的操作，然后利用不同折数加参数，特征，样本（随机数种子）扰动，再加权平均得到最终成绩.

不足与思考：
---
&emsp; 由于缺乏比赛经验，在整个比赛的过程中有很多欠缺的地方，比如分工不明确，导致最后只有一个模型能达到比较好的效果，继而导致模型融合效果不理想，最后一天被挤出前十；又比如数据处理经验不足，对异常数据(19，20上午以及30号的label)基本没做什么处理；然后一个小小的客观原因，决赛的数据量比较大，其中一个文件有5.7亿条记录，而16G内存的机器也一定程度上影响了成绩。<br>
&emsp; 不过总体来说结果还是不错的，也非常感谢分享经验的各位大神，比赛过程中通过他们的分享，网上的博客，自己的努力等等，让我一个接触机器学习才1个月的小白感受到了ML的魅力，也让我一个连pandas都没用过的新手见识到了一系列神奇的操作，也很感谢腾讯能提供这个平台让我们学到更多。

