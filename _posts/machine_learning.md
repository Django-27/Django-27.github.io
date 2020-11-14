# 1 机器学习
利用计算机从**历史数据**中找出规律，并把这些规律用到未来不确定场景的决策 （规律、决策）
## 应用
购物篮分析-关联规则（啤酒+纸尿布）
用户细分精准营销-聚类（神舟行-动感地带-全球通-大众卡）
垃圾邮件-朴素贝叶斯
信用卡欺诈-决策树
互联网广告-ctr预估（线性回归、点击率预估）
推荐系统-协同过滤
自然语言处理-情感分析、实体识别
图像识别-深度识别
语音识别、人脸识别、自动驾驶、实时翻译、情感分析、智慧机器人、手势识别、视频内容识别等等
## 算法分类
- 有监督学习，提前打标签，包括分类算法、回归算法
- 无监督学习，聚类，
- 半监督学习（强化学习）
也可以分成：分类与回归、聚类、标注
也可以分成：生成模型-可能百分之多少（A类20%、B类30%、C类50%）、判别模型-非一即二（有罪、无罪）

分类 | C4.5
聚类 | K-Means
统计学习 | SVM
关联分析 | Apriori  ->   FP-Growth
统计学习 | EM
链接挖掘 | PageRank  ->   逻辑回归
集装与推进 | AdaBoost  ->  RF、GBDT
分类 | kNN
分类 | Naive Bayes
分类 | CART
还有：推荐算法、LDA、Word2Vector、HMM、CRF、深度xuexi 

训练模型：定义模型、定义损失函数、优化算法

# 神经网络 Neural Networks VS PyTorch
- Variable
```
import torch
from torch.autograd import Variable

tensor = torch.FloatTensor([[1, 2], [3, 4]])
variable = Variable(tensor, requies_grad=True)  # True表示进行反向图纸的计算

t_out = torch.mean(tensor * tensor)  # x^2
v_out = torch.mean(variable * variable)

v_out.backard()  # 反向传递
```
- 激励函数 activation functions

