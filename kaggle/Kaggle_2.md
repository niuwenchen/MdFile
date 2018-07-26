## Kaggle_2  房价预测
### 1 数据探测
```python
train_df = pd.read_csv('../input/train.csv', index_col=0)
test_df = pd.read_csv('../input/test.csv', index_col=0)
train_df.head()

```
### 2 合并数据
因为需要对训练数据和测试数据均实施同样的处理过程，因此合并
```python
all_df = pd.concat((train_df, test_df), axis=0)
```

### 3 变量转化
类似特征工程，将不方便处理或者 不 unify的数据给统一了

**正确化变量属性**
(1) 对category数据的处理
一些类别数据并不会以word的形式来表达，而是1 2 3 4 这样，所以DF会认为是数字，需要我们转换为category
```python
all_df["MSSubclass"].dtypes  ==> dtype('int64')
all_df["MSSubclass"]=all_df["MSSubClass"].astype(str)

```

当我们用numerical来表达categorical的时候，要注意，数字本身有大小的含义，所以乱用数字会给之后的模型带来学习麻烦，所以可以用one-hot来表达category

一般都是找sklearn-learn的方法，这里有更简单的

**重点**
pandas自带的get_dummies方法，可以帮你一键做到One-Hot。
```python
pd.get_dummies(all_df['MSSubClass'], prefix='MSSubClass').head()

MSSubClass_120	MSSubClass_150	MSSubClass_160	MSSubClass_180	MSSubClass_190	MSSubClass_20	MSSubClass_30	MSSubClass_40	MSSubClass_45	MSSubClass_50	MSSubClass_60	MSSubClass_70	MSSubClass_75	MSSubClass_80	MSSubClass_85	MSSubClass_90
Id																
1	0	0	0	0	0	0	0	0	0	0	1	0	0	0	0	0
2	0	0	0	0	0	1	0	0	0	0	0	0	0	0	0	0
3	0	0	0	0	0	0	0	0	0	0	1	0	0	0	0	0
4	0	0	0	0	0	0	0	0	0	0	0	1	0	0	0	0
5	0	0	0	0	0	0	..
```
此时MSSubclass被我们分成了12个column，每一个代表一个category，是就是1，不是就是0

同理，将所有的category数据，都给one-hot了。

```python

all_dummy_df = pd.get_dummies(all_df) #默认one-hot的是category数据

```

**处理numerical数据**
numerical数据确实
```python
all_dummy_df.isnull().sum().sort_values(ascending=False).head(10)

LotFrontage     486
GarageYrBlt     159
MasVnrArea       23
BsmtHalfBath      2
BsmtFullBath      2
BsmtFinSF2        1
GarageCars        1
TotalBsmtSF       1
BsmtUnfSF         1
GarageArea        1

```
处理这些缺失的信息，得靠好好审题。一般来说，数据集的描述里会写的很清楚，这些缺失都代表着什么。当然，如果实在没有的话，也只能靠自己的『想当然』。。
在这里，我们用平均值来填满这些空缺。

```python
mean_cols = all_dummy_df.mean() # 默认是求的numerical的数据
all_dummy_df = all_dummy_df.fillna(mean_cols) #默认是按照列名去填充的，不会填充错误

all_dummy_df.isnull().sum().sum()
0

```

**标准化numerical数据**
这一步并不是必要，但是得看你想要用的分类器是什么。一般来说，regression的分类器都比较傲娇，最好是把源数据给放在一个标准分布内。不要让数据间的差距太大。
这里，我们当然不需要把One-Hot的那些0/1数据给标准化。我们的目标应该是那些本来就是numerical的数据：
先来看看 哪些是numerical的：
```python
numeric_cols = all_df.columns[all_df.dtypes != 'object']
numeric_cols

Index(['LotFrontage', 'LotArea', 'OverallQual', 'OverallCond', 'YearBuilt',
       'YearRemodAdd', 'MasVnrArea', 'BsmtFinSF1', 'BsmtFinSF2', 'BsmtUnfSF',
       'TotalBsmtSF', '1stFlrSF', '2ndFlrSF', 'LowQualFinSF', 'GrLivArea',
       'BsmtFullBath', 'BsmtHalfBath', 'FullBath', 'HalfBath', 'BedroomAbvGr',
       'KitchenAbvGr', 'TotRmsAbvGrd', 'Fireplaces', 'GarageYrBlt',
       'GarageCars', 'GarageArea', 'WoodDeckSF', 'OpenPorchSF',
       'EnclosedPorch', '3SsnPorch', 'ScreenPorch', 'PoolArea', 'MiscVal',
       'MoSold', 'YrSold'],
      dtype='object')
```

计算标准分布:
```python
numeric_col_means = all_dummy_df.loc[:, numeric_cols].mean()
numeric_col_std = all_dummy_df.loc[:, numeric_cols].std()
all_dummy_df.loc[:, numeric_cols] = (all_dummy_df.loc[:, numeric_cols] - numeric_col_means) / numeric_col_std

#loc方法拿出列的数据

```

### 建立模型
```python
dummy_train_df = all_dummy_df.loc[train_df.index]
dummy_test_df = all_dummy_df.loc[test_df.index]

```

**Ridge Regression**
```python
from sklearn.linear_model import Ridge
from sklearn.model_selection import cross_val_score

X_train = dummy_train_df.values
X_test = dummy_test_df.values

alphas = np.logspace(-3, 2, 50)
test_scores = []
for alpha in alphas:
    clf = Ridge(alpha)
    test_score = np.sqrt(-cross_val_score(clf, X_train, y_train, cv=10, scoring='neg_mean_squared_error'))
    test_scores.append(np.mean(test_score))

#存储所有cv值，看看哪一个cv值是最好的
import matplotlib.pyplot as plt
%matplotlib inline
plt.plot(alphas, test_scores)
plt.title("Alpha vs CV Error");

```
![Markdown](http://i1.bvimg.com/654958/39aaf0c5329886ab.png)

**Random forest**
```python
from sklearn.ensemble import RandomForestRegressor
max_features = [.1, .3, .5, .7, .9, .99]
test_scores = []
for max_feat in max_features:
    clf = RandomForestRegressor(n_estimators=200, max_features=max_feat)
    test_score = np.sqrt(-cross_val_score(clf, X_train, y_train, cv=5, scoring='neg_mean_squared_error'))
    test_scores.append(np.mean(test_score))

plt.plot(max_features, test_scores)
plt.title("Max Features vs CV Error");
```
![Markdown](http://i1.bvimg.com/654958/8cccaf9fe371bae6.png)


**采用混合模型**
**Ensemble**
```python
ridge = Ridge(alpha=15)
rf = RandomForestRegressor(n_estimators=500, max_features=.3)
ridge.fit(X_train, y_train)
rf.fit(X_train, y_train)

y_ridge = np.expm1(ridge.predict(X_test))
y_rf = np.expm1(rf.predict(X_test))

y_final = (y_ridge + y_rf) / 2
submission_df = pd.DataFrame(data= {'Id' : test_df.index, 'SalePrice': y_final})

```

**bagging**
```python
from sklearn.ensemble import BaggingRegressor
from sklearn.model_selection import cross_val_score
params = [1,10,15,20,25,30,40]
test_scores=[]
for param in params:
    clf = BaggingRegressor(n_estimators=param,base_estimator=ridge)
    test_score = np.sqrt(-cross_val_score(clf, X_train, y_train, cv=10, scoring='neg_mean_squared_error'))
    test_scores.append(np.mean(test_score))

import matplotlib.pyplot as plt
%matplotlib inline
plt.plot(params, test_scores)
plt.title("n_estimator vs CV Error");
```
![Markdown](http://i1.bvimg.com/654958/dcc3399bef8bb83a.png)

**Boosting**
Boosting比Bagging理论上更高级点，它也是揽来一把的分类器。但是把他们线性排列。下一个分类器把上一个分类器分类得不好的地方加上更高的权重，这样下一个分类器就能在这个部分学得更加“深刻”。

```python
from sklearn.ensemble import AdaBoostRegressor
params = [10, 15, 20, 25, 30, 35, 40, 45, 50]
test_scores = []
for param in params:
    clf = BaggingRegressor(n_estimators=param, base_estimator=ridge)
    test_score = np.sqrt(-cross_val_score(clf, X_train, y_train, cv=10, scoring='neg_mean_squared_error'))
    test_scores.append(np.mean(test_score))

plt.plot(params, test_scores)
plt.title("n_estimator vs CV Error");


```
![Markdown](http://i1.bvimg.com/654958/06277bf79de738e3.png)

**XgBoost**
最后，我们来看看巨牛逼的XGBoost，外号：Kaggle神器
这依旧是一款Boosting框架的模型，但是却做了很多的改进。

```python
from xgboost import XGBRegressor
params = [1,2,3,4,5,6]
test_scores = []
for param in params:
    clf = XGBRegressor(max_depth=param)
    test_score = np.sqrt(-cross_val_score(clf, X_train, y_train, cv=10, scoring='neg_mean_squared_error'))
    test_scores.append(np.mean(test_score))

import matplotlib.pyplot as plt
%matplotlib inline
plt.plot(params, test_scores)
plt.title("max_depth vs CV Error");

```
![Markdown](http://i1.bvimg.com/654958/98078d4bd6668c71.png)
惊了，深度为5的时候，错误率缩小到0.127
这就是为什么，浮躁的竞赛圈，人人都在用XGBoost :)