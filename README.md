# Adaboost
利用相似的分类器来提高分类性能
我们的想法出自哪里：我感觉是多数表决吧，但是怎么表决比较科学。我弄了一堆分类器，
我们假设一个专家比较厉害，
总是答对那么魔门我们都希望听他的，那自然他的alpha会高点，这个alpha可以根据答对的正确率来衡量，
假设对于而分类问题，其分类性能都会在50%左右，那么我怎么多数表决，我知道分类器性能可以根据训练的，
要是还是像传统的方法一样，那么其实加多少个都是一样的，那么我假设每个分类器的性能不一致，因为都是50%，那么误判点也不一样
如果我加大勿判点权值，自然可以更加针对误判点，这样多个和起来就可以。
想法：就像把每个领域专家集合起来，看看他们表决结果，这里面有个诺奖那么自然要比其他人多些话语权。
想法有了怎么去量化。好吧，所有算法都是有目标函数的吧，我调参数的过程就是训练的过程，因此有必要从目标函数入手。
毫无疑问肯定是分类器全部分对最好，（不考虑过拟合问题）怎么衡量是分对还是分错，还记得感知器里面的吗，
把标签设为正负一，那么函数间隔就是想要的，分错肯定小于零，分对肯定大于零。如果我是最大化这个和函数，那么我关注误分类点效果就没了
所以我关注错误点，即加个符号，那么自然是最小化，但是如果还是求和的化，有正有负，到最后必然是负的比较多，说不得弄出个负无穷，
就不会收敛了，所以加个exp吧指数函数都是大于零的。但是我如果不care正确分类点是不是会更好点。
因此提升方法的目标函数就有了。找算法训练，然后看其收敛性了。
这一段就要参考统计学习方法，要证明收敛性还是比较困难的。其实有上界就可以（这快要问问做收敛性分析的人）先挖个坑。
下面介绍下流程吧
假设用的弱分类器是单层决策树分类器。那么其对数据的一次分类应该能确定出来，所以我可以切分数据集来得到最佳分类，怎么是最佳毫无疑问就是
在训练集上分类错误最小的，so，那么我怎么做不同分类器，假设我把错留给下一个分类器，就意味着下一个分类器要继承我的权值然后继续下去，
知道误差在可接受范围。最后我在把所有结果加起来来判断是带上权值即话语权的和，那么这个正负和就觉得最终的正负和。多数表决的方式，
应该是后验概率最大化。
处理非均衡分类代价的三种方法，一个是调整分类器的阈值，这个和ROC曲线在一起，还有用代价敏感学习，最后一种是数据抽样方法。
下面介绍点题外的东西：以前都是以错误率来作为判断的标准，但是这种计算方式不包含分错的信息，即到底是正例错分，还是反例错分。
所以为了更好了解分类中的错误，从而更好提高分类性能。用了下面三种表示。
ROC曲线：
要想画出ROC曲线必须的知道一些术语，比如：TP，表示真正例，FN表示伪反例，FP表示伪正例，TN表示真反例。
那么正确率：就是TP/（TP+FP），有个问题要是我的FP比较小，那么值就比较大，但是有不合理的地方，
召回率：TP/（TP+FN）
引入真阳率，和假阳率。
真阳率：TP（TP+FN），假阳率：FP/（FP+TN）所以要提供一个可信程度排序，然后利用不同阈值来获得Roc曲线。
首先AUC值是一个概率值，当你随机挑选一个正样本以及负样本，当前的分类算法根据计算得到的Score值将这个正样本排在负样本前面的概率就是AUC值，
AUC值越大，当前分类算法越有可能将正样本排在负样本前面，从而能够更好地分类。
既然已经这么多标准，为什么还要使用ROC和AUC呢？因为ROC曲线有个很好的特性：
当测试集中的正负样本的分布变换的时候，ROC曲线能够保持不变。在实际的数据集中经常会出现样本类不平衡
，即正负样本比例差距较大，而且测试数据中的正负样本也可能随着时间变化。
总之就是roc不会因为数据集的变化产生较大的变化，所以更加有普遍性。
![image](https://github.com/chenglu66/Adaboost/blob/master/roc10.png)
把分类器的数量设定为50时
![image](https://github.com/chenglu66/Adaboost/blob/master/roc2.png)
仔细观察发现：
当分类器是10的AUC是0.86
当分类器是50时AUC是0.89
当然这只是训练的曲线那么测试集上的曲线他们表现怎么样？
![image](https://github.com/chenglu66/Adaboost/blob/master/testroc10.png)
AUC大小是0.7627
![image](https://github.com/chenglu66/Adaboost/blob/master/testroc50.png)
AUC大小是0.8025
说了折磨多还没说AUC怎么计算：
其实就是曲线的面积，因为随机的是0.5
下面是基于代价的分类器，
比如给数量少的类更多的权重这样，在数量小的类中就会出现更少的错误。常见的有adaboost算法中，调整错误权重向量，朴素贝叶斯里用最小期望代价而不是最大概率做为最后的结果，svm中不同类别选择不同的参数C
最后是处理非均衡问题的数据抽样方法：
过抽样和欠抽样，过抽样就是加点噪声复制样例，欠抽样就是删除样例，前面的SVM分类就是这么做的，比如删去离分类边界较远的样例
第二种就是对已有数据进行插值。但这中会过拟合。
总结：多个分类器组合可以缓解过拟合，如果分类器差别明显的话，分类器差别可以是算法本身，或者数据不同也可以。
所以对于非均衡问题，一方面是改变阈值，一方面是抽样上，还有就是直接把小类别错误代价提高。










