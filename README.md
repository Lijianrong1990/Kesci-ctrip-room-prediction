# Kesci-ctrip-room-prediction
2st Place Solution for kesci-ctrip room prediction

题目说明及数据地址：
https://www.kesci.com/apps/home/#!/lab/dataset/58d4e28c84a25f34b1d94906/document
# 1.	队伍介绍
&emsp;&emsp;队名“到极限了吗？”由3个小伙伴组成，之前也都没什么经验，我们是在进入复赛
前几天组队的，复赛开始后，我们迅速排到了第一名，并一直保持领先第2名超过2%，然后我们把队名改成了“到极限了吗”，由于保持领先优势太多再加上决赛结束前一段时间突然很忙，就没有花很多时间在上面，结果在比赛截止之前突然被现在的第一名反超，最后没有拿冠军，只拿到了第2名，有些遗憾，还是too young, too simple啊。
# 2.	解决方案概述
&emsp;&emsp;携程每天向超过2.5亿会员提供全方位的旅行服务，海量的网站访问产生了海量的数据，从中挖掘潜在的数据是具有重大的意义。合理利用这些数据使其能真正为用户带来更好的旅行体验。调研表明，大部分用户除了对于酒店有偏好外，也有对于酒店房型的偏好。不同的酒店房型会提供酒店不同服务和礼惠政策等，这使得提供更多服务的同时，带来了用户一定程度的挑选时间。如何根据在用户的历史信息，挖掘出用户对于某些房型偏好，也为了节省用户的挑选时间和提供更好的服务。针对需要解决的问题，我们主要从以下几个方面来进行处理：数据预处理，数据集划分与特征工程，模型设计与融合。
# 3.	数据预处理
&emsp;&emsp;数据预处理主要是处理了2部分的数据，第一个是时间信息，经过统计，所给出的训练集的时间在2013-04-14到2013-04-20之间，而测试集的时间在2013-04-21到2013-04-27之间，我们把训练集和测试集的时间全部转化成0到6之间。  
&emsp;&emsp;另一方面，通过观察，我们发现所给出的数据里面也存在一些异常数据，比如有些数据上次预定的时间(orderdate_lastord)比本次预定时间(orderdate)还大，故在训练数据中把这部分数据剔除，因为异常数据可能会影响模型的学习，在前期测试时结果如下，没有剔除这部分数据结果(线下auc: 0.95543, 线上：0.502123)，剔除这部分数据之后(线下auc: 0.9559, 线上：0.502843)，可以看出过滤掉这部分数据还是有一些提升。
&emsp;&emsp;此外数据中roomtag_6全部都为0，roomtag_6_lastord（也几乎全为0，极少部分为空），orderbehavior_4_ratio_1month, orderbehavior_5_ratio_1month和orderbehavior_3_ratio_1month这几列特征也全为空，我们也把这部分特征全部剔除掉，一方面也可以节省空间和时间，另一方面这一部分无用特征也可能会对模型学习有一些微弱的影响，这部分数据删除之后，结果也还是有很微弱的提升。
# 4.	数据集划分和特征工程
&emsp;&emsp;坊间戏言“特征没做好，参数调到老”，机器学习大牛 Andrew Ng 也说过“ Applied
machine learning  is basically feature engineering”，可见特征工程的重要性，我们在这部分投入了最多的时间和精力，另一部分我个人认为很重要的就是划分训练集，划分一个好的训练集，一方面要构造尽可能多的训练样本，另一方面测试数据也不能过少，否则会导致过拟合，划分训练集也要尽量保证线下和线上结果保持同步，否则线下提高，线上结果却降低了，这就让人更迷茫和困惑了。
* # 4.1 数据集划分    
&emsp;&emsp;首先是数据集划分，这部分个人感觉主要也是凭经验再加上多进行实验测试，我们分别测试了通过随机划分数据集和按uid划分数据集2种方式，其实从直观上感觉随机划分数据集可能会把那些属于同一个uid的那些特征分开，直接随机划分数据集可能没有按uid划分效果好，通过实验也验证了我们的想法，我们最后采用的方式是按uid划分数据集，其中4/5的数据用于训练，1/5的数据用于测试，这样一方面既能保证训练数据有足够的多，另一方面也能保证有一定的数据用于验证，使模型不至于过拟合。在这个题目中我们测试过这样划分训练集比随机划分训练集一半用于训练一半用于测试的线上结果要好差不多千分之五，这是一个很大的提升了，可以看出数据集划分对结果的影响是相当大的。

* # 4.2	特征工程
&emsp;&emsp;根据个人的理解和相关经验，构造特征应该与实际生活经验相结合，如果不能想到一个特征对最后实际决策有任何影响，那就不要用，另一方面要多构造一些自己觉得对实际决策影响最大的相关特征。

&emsp;&emsp;最开始可能随便加很多特征结果都会有提高，但是越往后面发现再加入一些特征结果开始很难提高，甚至还可能下降(其实这个时候还远没有做到最好的程度)，因为前期加的太多不相关特征会影响模型的学习，因此对于这一点我个人的理解和建议是一方面不要随便乱加特征，另一方面就是每做到一定程度，就多测试一些，删除一些特征，事实上，个人认为剔除过拟合和不相关的特征是数据挖掘中最难的一个步骤，感觉也只能主要靠经验和多测试，虽然也有很多方法计算特征之间的相关性和相似度，但是这些方法实际运用时效果一般并不好。

&emsp;&emsp;另一方面，一般来讲，用户最终预定哪一个房型，个人感觉和房间的价格，优惠价格，优惠率，还有一般用户订购酒店的价格一般也和以前订购酒店的价格相差不大，还有一些重要的服务标签等，因此，我们构造了大量的这些特征，主要是这些房间的价格，优惠价格，优惠率，还有这些特征的排序特征，本次与上次之间的差等等。

&emsp;&emsp;我们主要构造了以下特征： 相关特征(用户相关特征和物理房型相关特征)，特征之间做一些差，比率，排序，等等，想了解更详细的，可以参考我们的代码目录和详细代码。

# 5.	模型设计和融合
&emsp;&emsp;由于这题题目的数据量比较大，我们最开始是用xgboost训练的，但是感觉xgboost还是太慢了，后来了解到lightGBM精度和xgboost相当，但是速度快很多，故采用lightGBM来训练模型，lightGBM由微软亚洲研究院DMTK团队开发，与xgboost类似，也是boosted tree的一种实现形式，精度和效率都很高，与xgboost不同的是，lightGBM采用leaf-wise的叶子生长模式(有点类似深度优先的贪心算法)，并且在结点分裂时采用Histogram算法，基本的思想是把浮点特征值离散成k个整数，同时构造一个宽度为k的直方图。在遍历数据的时候，根据离散化后的值作为索引在直方图中累积的统计量，这样可以大大的加快模型训练的速度，同时最后结果与xgboost也相当。

&emsp;&emsp;根据经验树的深度一般设置为5到6左右就够了，学习率一般越小结果会稍微好一些，但是当学习率很小之后就不一定了，但是学习率设小之后模型收敛一般会变慢一些，所以最开始可以把学习率设大一些，这样训练较快，比赛最后几天再把学习率设小一点(同时把迭代次数和early_stopping次数都变大).
我们一共用了xgboost和lightGBM来训练，树的深度都是6，lightGBM线上最好的结果是0.505632，xgboost线上最好是0.502789，由于数据量很大，就没有采用Random Forest和GBDT这些算法，这些算法一般精度会差些并且很慢，另外，逻辑回归效果差很多故没有采用，另外由于这个题目的数据量很大，模型的variance比较小，模型融合主要是减小variance，故在这个题目中模型融合应该影响比较小，事实上最后的实验也证实了我们的想法，我们最后采用了2种模型融合方式：第一种是线性融合。P*xgb_score+q*lgb_score,  其中xgb_score和lgb_score分别是xgb和lgb预测的用户选购房型的概率(概率在0到1之间)，由于lgb线上结果比xgb要好，故q设置的会比p大一些，最后p=0.2, q=0.8时线上效果最好，结果是0.506046.

&emsp;&emsp;另一种是采用预测结果相乘的融合，具体是 （xgb_score^p）*(lgb_score^q), p设置的q大一些，最终结果也差不多。

&emsp;&emsp;由于时间仓卒，可能还有很多细节不能详尽的描述，最后建议大家多学习优秀的开源代码，多分析，多思考，多尝试。

