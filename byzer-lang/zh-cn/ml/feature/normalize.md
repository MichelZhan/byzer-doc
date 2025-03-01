# NormalizeInPlace

特征归一化本质上是为了统一量纲，让一个向量里的元素变得可以比较。
它应用于任何依赖于距离的算法，比如 KMeans, nearest neighbors methods, RBF kernels 等等。

### 数据准备

```sql
-- create test data
SET jsonStr='''
{"a":1,    "b":100, "label":0.0},
{"a":100,  "b":100, "label":1.0}
{"a":1000, "b":100, "label":0.0}
{"a":10,   "b":100, "label":0.0}
{"a":1,    "b":100, "label":1.0}
''';
LOAD jsonStr.`jsonStr` AS data;
```

### 训练
对第一列数据a,b两列数据按照行的方式进行归一化。

```sql
TRAIN data AS NormalizeInPlace.`/tmp/model`
WHERE inputCols="a,b"
and scaleMethod="standard"
and removeOutlierValue="false"
;

LOAD parquet.`/tmp/model/data` 
as output;
```

结果如下：

```
a                   b   label
-0.5069956180959223	0	0
-0.2802902604107538	0	1
1.7806675367271416	0	0
-0.48638604012454334	0	0
-0.5069956180959223	0	1
```

`removeOutlierValue` 设置为 true，会自动用中位数填充异常值。


>如果 `inputCols` 只有一列，那么该列可以为  


### API 预测

> API 预测的相关原理及示例，详见 [部署算法 API 服务](/byzer-lang/zh-cn/ml/api_service/README.md)

```sql
REGISTER NormalizeInPlace.`/tmp/model` as convert;
```

通过上面的命令，NormalizeInPlace 就会把训练阶段学习到的东西应用起来。
现在，任意给定两个数字，都可以使用 `convert` 函数将内容转化为向量。

```sql
SELECT convert(array(7,8)) as features as output;
```

输出结果为：

```
features
[-0.4932558994483363,0]
```

