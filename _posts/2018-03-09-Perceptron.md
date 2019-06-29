---
layout:     post
title:      "利用Python实现一个感知机学习算法"
subtitle:   "感知机"
date:       2018-03-09 22:44:52
author:     "TyurinTsien"
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
    - BigData
---

# 利用Python实现一个感知机学习算法

+ 本文主要内容包括利用Python实现一个感知机模型并利用这个感知机模型完成一个分类任务。
+ Warren和McCullock于1943年首次提出MCP neuron神经元模型[1]，之后，Frank Rosenblatt在MCP neuron model的基础之上提出了感知机Perceptron模型[2]。

+ 采用面向对象的方法编写一个感知机接口，这样就可以初始化一个新的感知机对象，这个对象可以通过fit()方法从数据中学习参数，通过predict()方法做预测。下面通过代码来讲解实现过程：

```
# -*- coding:utf-8 -*-
# 2018-03-09 22:36:52
__author__ = "TyurinTsien"

import sys
reload(sys)
sys.setdefaultencoding("utf-8")

import numpy as np
 
class Perceptron(object):
    """Perceptron classifier.
    Parameters
    ------------
    eta:float,Learning rate (between 0.0 and 1.0)
    n_iter:int,Passes over the training dataset.
     
    Attributes
    -------------
    w_: 1d-array,Weights after fitting.
    errors_: list,Numebr of misclassifications in every epoch.
    """
    def __init__(self,eta=0.01,n_iter=10):
        self.eta = eta
        self.n_iter = n_iter
    def fit(self,X,y):
        """Fit training data.先对权重参数初始化，然后对训练集中每一个样本循环，根据感知机算法学习规则对权重进行更新
        Parameters
        ------------
        X: {array-like}, shape=[n_samples, n_features]
            Training vectors, where n_samples is the number of samples and n_featuers is the number of features.
        y: array-like, shape=[n_smaples]
            Target values.
        Returns
        ----------
        self: object
        """
        self.w_ = np.zeros(1 + X.shape[1]) # add w_0　　　　　#初始化权重。数据集特征维数+1。
        self.errors_ = []#用于记录每一轮中误分类的样本数
         
        for _ in range(self.n_iter):
            errors = 0
            for xi, target in zip(X,y):
                update = self.eta * (target - self.predict(xi))#调用了predict()函数
                self.w_[1:] += update * xi
                self.w_[0] += update
                errors += int(update != 0.0)
            self.errors_.append(errors)
        return self
     
    def net_input(self,X):
        """calculate net input"""
        return np.dot(X,self.w_[1:]) + self.w_[0]#计算向量点乘
     
    def predict(self,X):#预测类别标记
        """return class label after unit step"""
        return np.where(self.net_input(X) >= 0.0,1,-1)
```

+ 接下来使用鸢尾花Iris数据集来训练感知机模型。加载两类花：Setosa和Versicolor。属性选定为：sepal length和petal length。当然，不局限于两个属性。我们可通过One-vs-All(OvA)或One-vs-Rest(OvR)技术讲二分类扩展到多分类的情形。

```
# -*- coding:utf-8 -*-
# 2018年03月09日22:36:37
__author__ = "TyurinTsien"

import sys
reload(sys)
sys.setdefaultencoding("utf-8")

import pandas as pd#用pandas读取数据
import matplotlib.pyplot as plt
import numpy as np
from matplotlib.colors import ListedColormap
from Perceptron import Perceptron
 
df = pd.read_csv('https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data',header=None)#读取数据还可以用request这个包
print(df.tail())#输出最后五行数据，看一下Iris数据集格式
 
"""抽取出前100条样本，这正好是Setosa和Versicolor对应的样本，我们将Versicolor
对应的数据作为类别1，Setosa对应的作为-1。对于特征，我们抽取出sepal length和petal
length两维度特征，然后用散点图对数据进行可视化"""  
 
y = df.iloc[0:100,4].values
y = np.where(y == 'Iris-setosa',-1,1)
X = df.iloc[0:100,[0,2]].values
plt.scatter(X[:50,0],X[:50,1],color = 'red',marker='o',label='setosa')
plt.scatter(X[50:100,0],X[50:100,1],color='blue',marker='x',label='versicolor')
plt.xlabel('petal length')
plt.ylabel('sepal lenght')
plt.legend(loc='upper left')
plt.show()
 
#train our perceptron model now
#为了更好地了解感知机训练过程，我们将每一轮的误分类
#数目可视化出来，检查算法是否收敛和找到分界线
ppn=Perceptron(eta=0.1,n_iter=10)
ppn.fit(X,y)
plt.plot(range(1,len(ppn.errors_)+1),ppn.errors_,marker='o')
plt.xlabel('Epoches')
plt.ylabel('Number of misclassifications')
plt.show()
 
#画分界线超平面
def plot_decision_region(X,y,classifier,resolution=0.02):
    #setup marker generator and color map
    markers=('s','x','o','^','v')
    colors=('red','blue','lightgreen','gray','cyan')
    cmap=ListedColormap(colors[:len(np.unique(y))])
     
    #plot the desicion surface
    x1_min,x1_max=X[:,0].min()-1,X[:,0].max()+1
    x2_min,x2_max=X[:,1].min()-1,X[:,1].max()+1              
     
    xx1,xx2=np.meshgrid(np.arange(x1_min,x1_max,resolution),
                        np.arange(x2_min,x2_max,resolution))
    Z=classifier.predict(np.array([xx1.ravel(),xx2.ravel()]).T)
    Z=Z.reshape(xx1.shape)
     
    plt.contour(xx1,xx2,Z,alpha=0.4,cmap=cmap)
    plt.xlim(xx1.min(),xx1.max())
    plt.ylim(xx2.min(),xx2.max())
     
    #plot class samples
    for idx,cl in enumerate(np.unique(y)):
        plt.scatter(x=X[y==cl,0],y=X[y==cl,1],alpha=0.8,c=cmap(idx), marker=markers[idx],label=cl)
 
plot_decision_region(X,y,classifier=ppn)
plt.xlabel('sepal length [cm]')
plt.ylabel('petal length [cm]')
plt.legend(loc='upperleft')
plt.show()
```

# 结果如下图：
![pic1](http://www.qianqiulin.com/img/posts/pic1.png)
![pic2](http://www.qianqiulin.com/img/posts/pic2.png)
![pic3](http://www.qianqiulin.com/img/posts/pic3.png)

+ 若两类模式是线性可分的，即存在一个线性超平面能将它们分开，则感知机的学习过程一定会收敛converge而求得适当的权向量；否则感知机学习过程就会发生振荡fluctuation，权重向量难以稳定下来，不能求得合适解，具体的证明过程见文章[3]

+ References：
+ > [1] W. S. McCulloch and W. Pitts. A Logical Calculus of the Ideas Immanent in Nervous Activity. The bulletin of mathematical biophysics, 5(4):115–133, 1943 
+ > [2] F. Rosenblatt, The Perceptron, a Perceiving and Recognizing Automaton. Cornell Aeronautical Laboratory, 1957
+ > [3] Minsky, M. and S. Papert. (1969). Perceptrons. MIT Press, Cambridge, MA.