---
title: pefile classification 1
date: 2016-08-26 09:56:51
tags:
  - python
  - security
---
The code is from ![here](http://clicksecurity.github.io/data_hacking/)
## 导入模块（import the modules）
```
import os
import sklearn.feature_extraction
sklearn.__version__
```
```
import pandas as pd
pd.__version__
```
```
import numpy as np
np.__version__
```
## 作图默认设置（plot default）
```
import matplotlib.pyplot as plt
plt.rcParams['font.size'] = 18.0
plt.rcParams['figure.figsize'] = 16.0, 5.0

%matplotlib inline
```
## 作图函数
```
def plot_cm(cm, labels):
    # Compute percentanges
    percent = (cm*100.0)/np.array(np.matrix(cm.sum(axis=1)).T)  # Derp, I'm sure there's a better way   
    print 'Confusion Matrix Stats'
    for i, label_i in enumerate(labels):
        for j, label_j in enumerate(labels):
            print "%s/%s: %.2f%% (%d/%d)" % (label_i, label_j, (percent[i][j]), cm[i][j], cm[i].sum())

    # Show confusion matrix
    # Thanks kermit666 from stackoverflow :)
    fig = plt.figure()
    ax = fig.add_subplot(111)
    ax.grid(b=False)
    cax = ax.matshow(percent, cmap='coolwarm',vmin=0,vmax=100)
    plt.title('Confusion matrix of the classifier')
    fig.colorbar(cax)
    ax.set_xticklabels([''] + labels)
    ax.set_yticklabels([''] + labels)
    plt.xlabel('Predicted')
    plt.ylabel('True')
    plt.show()
```
>numpy.matrix.sum(axis=None,dtype=None,out=None)
>numpy.sum(a,axis=None,dtype=None,out=None,keepdims=False)
>sklearn.metrics.confusion_matrix(y_true, ypred, labels=None)
```
>>> from sklearn.metrics import confusion_matrix
>>> y_true = [2, 0, 2, 2, 0, 1]
>>> y_pred = [0, 0, 2, 2, 0, 2]
>>> confusion_matrix(y_true, y_pred)
array([[2, 0, 0],
       [0, 0, 1],
       [1, 0, 2]])
```
```
>>> y_true = ["cat", "ant", "cat", "cat", "ant", "bird"]
>>> y_pred = ["ant", "ant", "cat", "cat", "ant", "cat"]
>>> confusion_matrix(y_true, y_pred, labels=["ant", "bird", "cat"])
array([[2, 0, 0],
       [0, 0, 1],
       [1, 0, 2]])
```

### axis
```
>>> a = np.arange(12).reshape((3,2,2))
>>> a
array([[[ 0,  1],
        [ 2,  3]],

       [[ 4,  5],
        [ 6,  7]],

       [[ 8,  9],
        [10, 11]]])
>>> [x.sum() for x in a] # sum along axis 0
[6, 22, 38]
>>> a.sum(axis=0)
array([[12, 15],
       [18, 21]])
>>> a.sum(axis=1)
array([[ 2,  4],
       [10, 12],
       [18, 20]])
>>> a.sum(axis=2)
array([[ 1,  5],
       [ 9, 13],
       [17, 21]])
```

参考

http://scikit-learn.org/stable/modules/generated/sklearn.metrics.confusion_matrix.html
