# **机器学习实验一**
### 1. **封面**
   - 题目：贝叶斯分类器
   - 作者姓名：蔡俊贤
   - 学号：202202300407
   - 专业：数字媒体技术
   - 日期：2024/10/28

### 2. **实验目的**
   - 探讨贝叶斯分类器在鸢尾花数据集上的表现。
   - 验证贝叶斯分类器的分类效果。

### 3. **引言**
   - 实验的背景和动机：主要是自己动手实操实现贝叶斯分类器使得课堂掌握的知识更加深刻。
   - 问题描述：探讨贝叶斯分类器在鸢尾花数据集上的表现以及验证贝叶斯分类器的分类效果。

### 4. **实验方法**
   - **数据集**：数据集来源`http://download.tensorflow.org/data/iris_training.csv`，内含一百五十个数据以及四种特征值。
   - **模型选择**：模型选择为贝叶斯分类器，其核心思想是：根据已知的输入特征，计算各类别的后验概率，然后选择具有最高后验概率的类别作为预测结果。
   - **评估指标**：准确率、精度、召回率、F1值。

### 5. **实验过程**
   - **预先准备的库以及导入**：
	
```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import matplotlib as mpl
from sklearn.naive_bayes import GaussianNB
from sklearn.metrics import confusion_matrix
from sklearn.model_selection import train_test_split, learning_curve
from sklearn.metrics import accuracy_score, precision_score,recall_score, f1_score
```

   - **读取并处理数据**：
```python
# 读取并处理数据
data = pd.read_csv('iris.csv', header=None)
x = data.drop([4], axis=1).drop([0], axis=0)
x = np.array(x, dtype=float)
y = pd.Categorical(data[4]).codes[1:151]
```

   - **划分训练集和测试集（按照70%和30%划分，以及创建和训练分类器）**：

```python
# 划分训练集和测试集
x_train, x_test, y_train, y_test = train_test_split(x, y, train_size=0.7, random_state=14)
print('训练数据集样本数目：%d，测试数据集样本数目：%d' % (x_train.shape[0], x_test.shape[0]))

# 创建和训练高斯朴素贝叶斯分类器
clf = GaussianNB()
ir = clf.fit(x_train, y_train)
```

   - **根据得到的参数进行计算评估**：
```python
# 评估模型性能
y_pred_test = ir.predict(x_test)
acc_test = accuracy_score(y_test, y_pred_test)
precision_test = precision_score(y_test, y_pred_test, average='weighted')
recall_test = recall_score(y_test, y_pred_test, average='weighted')
f1_test = f1_score(y_test, y_pred_test, average='weighted')

print('测试集准确率：%.3f' % acc_test)
print('测试集精确率：%.3f' % precision_test)
print('测试集召回率：%.3f' % recall_test)
print('测试集 F1 分数：%.3f' % f1_test)
```
   - **进行可视化的绘制**：
部分代码如下：
```python
# 设置图形参数和绘图
mpl.rcParams['font.sans-serif'] = [u'SimHei']
mpl.rcParams['axes.unicode_minus'] = False
cm_light = mpl.colors.ListedColormap(['#77E0A0', '#FF8080', '#A0A0FF'])
cm_dark = mpl.colors.ListedColormap(['g', 'r', 'b'])

plt.figure(figsize=(10, 6))
plt.pcolormesh(p1, p2, y_hat.reshape(p1.shape), shading='auto', cmap=cm_light, alpha=0.5)
scatter = plt.scatter(p_test[:, 0], p_test[:, 1], c=y_test_p, edgecolors='k', s=80, cmap=cm_dark, label=data[4].unique())
plt.xlabel(u'花萼长度', fontsize=14)
plt.ylabel(u'花萼宽度', fontsize=14)
plt.title(u'GaussianNB对鸢尾花数据的分类结果', fontsize=16)
plt.grid(True)
plt.xlim(p1_min, p1_max)
plt.ylim(p2_min, p2_max)

# 绘制学习曲线
plt.figure(figsize=(10, 6))
plt.title("对于GaussianNB的学习曲线")
plt.xlabel("Training Examples")
plt.ylabel("Score")
plt.ylim(0, 1.1)
plt.grid()
```


### 6. **实验结果**

   - **结果展示**：

![image-20241028212455787](https://s2.loli.net/2024/10/28/WiGr7Kp8ZXR9tvc.png)

![image-20241028212520933](https://s2.loli.net/2024/10/28/cADmJjQIiXPpvUt.png)

![image-20241028212802727](https://s2.loli.net/2024/10/28/PgToVQ6fE2nJ7St.png)

### 7. **结论**

分析如下：

   - **准确率**：根据准确率达到了95.6%说明 大部分样本都被正确分类了。
   - **精确率**：根据精准率达到了96.1%，显示出在正类预测中，有较高比例是正确的。
   - **召回率**：根据召回率达到了95.6%，表明模型能成功识别出大多数真实的正类样本。
   - **混淆矩阵分析**：对于类别一和类别三都正确判断成功了，类别二正确分类了15个样本，担忧两个被误判为了类别三。总体上区分精确度较为优秀。

综上所述，高斯朴素贝叶斯分类器在鸢尾花数据集上的分类性能较优异。
