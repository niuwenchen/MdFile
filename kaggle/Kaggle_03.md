## Kaggle_03 CTR
基本上大数据集不用Pandas，而是使用LibLinear，libsvm格式的数据，省内存，对于成千上万维的数据都可以很省内存。

下采样之后可以载入到单机进行训练的形式

下采样：了解现在的数据

在线性模型中 数据特征对ctr的表现
```
# 分析每一种特征对CTR的click均值，就是点击率
data.groupby('device_type', {'CTR':gl.aggregate.MEAN('click')})

device_type	CTR
0	0.227499406317
5	0.0990566037736
4	0.0725075528701
1	0.175977623465

可以找出特征对于结果的贡献

# 看特征类型
zip(data.column_names(), data.column_types())
[('id', str),
 ('click', int),
 ('hour', int),
 ('C1', int),
 ('banner_pos', int),
 ('site_id', str),
 ('site_domain', str),
 ('site_category', str),
 ('app_id', str),
 ('app_domain', str),
 ('app_category', str),
 ('device_id', str),
 ('device_ip', str)]


对于时间的处理
一般映射为category的数据

#统计频次 
data['C16'].sketch_summary().frequent_items()
{20: 2, 36: 912, 50: 95620, 90: 18, 250: 3427, 480: 20}
将出现频次很低的类别不单独拿出来作为一列，而是合并在一起作为一个类别。

```

## 整理一份关于数据探索的过程
从数据导入，到数据清洗，到数据格式，到数据特征确定

**数据探索之后，对数据进行修整**
```
def clean_df(df, training=True):
    df = df.drop(['site_id', 'app_id', 'device_id', 'device_ip', 'site_domain',
                  'site_category', 'app_domain', 'app_category', 'device_model'], axis=1)
    if training:
        # id column is not required for training purposes
        df = df.drop(['id'], axis=1)

    return df

def load_df(filename, training=True, **csv_options):
    df = pd.read_csv(filename, header=0, **csv_options)
    df = clean_df(df, training=training)
    return df

```
**数据加载之后，定义模型评价指标**
```
# 结果衡量
def print_metrics(true_values, predicted_values):
    print ("Accuracy: ", metrics.accuracy_score(true_values, predicted_values))
    print ("AUC: ", metrics.roc_auc_score(true_values, predicted_values))
    print ("Confusion Matrix: ", + metrics.confusion_matrix(true_values, predicted_values))
    print (metrics.classification_report(true_values, predicted_values))

# 拟合分类器
def classify(classifier_class, train_input, train_targets):
    classifier_object = classifier_class()
    classifier_object.fit(train_input, train_targets)
    return classifier_object

# 模型存储
def save_model(clf):
    joblib.dump(clf, 'classifier.pkl')
```


## Spark 对数据的处理过程
```
Row Data---> StringIndxer ----> OneHotEncoder  --->LR


```
![Markdown](http://i2.bvimg.com/654958/f4f98ee0ae113806.png)

* Parse Data into SparkSql DataFrames
	* SprkSql Convert a RDD of row Objets into a DataFrame
	* Rows are constructed by passing key/value 
* Feature Transfomer using StringIndexer
* Feature Transformer using One hot encoding



## 比赛过程

https://github.com/owenzhang/kaggle-avazu
https://tech.meituan.com/deep_understanding_of_ffm_principles_and_practices.html

