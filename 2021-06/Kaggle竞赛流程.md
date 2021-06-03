## 流程

![图片](https://raw.githubusercontent.com/louisyanglu/TyporaImages/master/20210603111708)

------

### 问题理解，分析，规划

-   数据收集方式——是否引入脏数据——需要设计一些特定的后处理或者预处理来对数据进行处理
-   特殊数据——对于缺失严重但又非常重要的特征，尝试对这些数据进行还原
-   标签来源/打标方式——是否合理（如在之前kaggle的IEEE问题中，主办方是对**某个用户**进行打标，如果该用户存在欺诈逾期之类的，那么该用户的**所有记录**都会被打标为1，也就是说可能该用户之前即使是没有欺诈的，但只要后面进行了一次欺诈或者预期，就会导致该用户的所有记录全部被标记为1）
-   评估指标——是否有优化空间（为了更好的排名）
    -   **可以直接优化我们的评估指标，那么就直接用对应的优化函数对其进行优化**
    -   不能直接对我们的损失函数进行优化，那么就考虑是否可以**使用近似函数对其进行优化**
    -   最后我们再**考虑能否使用一些后处理技巧对预测结果进行纠正从而得到更好的结果**

------

### 数据探索分析

-   数据的整体情况
-   每个字段的含义
-   数据字段中空值、异常值
-   标签是否分布平衡
-   特征字段与标签的关系
-   训练集与测试集是否差异大（iid）

![图片](https://raw.githubusercontent.com/louisyanglu/TyporaImages/master/20210603125932)

#### 全局数据分析

通过数据全局分析，我们可以了解到数据的整体情况，包括数据类型、大小、质量等等；

```python
df.head()  # 整体
df.dtypes  # 类型
df.info()  # 数据很大的话，是否需要租一个服务器？数据很小，波动非常大
df.isnull().sum(axis=0)  # 每个字段的缺失值统计

import missingno as msno  # 数据集缺失
msno.matrix(df)
msno.bar(df)

df.nunique()  # 每个字段的nunique分布，为1，直接删除
```

#### 单变量数据分析

在这一模块，我们会单独的对每个变量进行观测，包括类别变量，连续变量，文本变量等等；

```python
# 数据按照类型拆分成数值型、类别型、时间类型、字符串(Object型)、图像

'''1、数值型'''
# 重点观察是否存在奇异值、整体分布情况

# 根据分位数查看是否存在 奇异值
df['Fare'].describe(percentiles = np.array(list(range(10))) * 0.1 )

# 可视化
sns.boxplot(data = df['Fare'], ax=ax)  # 画箱线图
sns.distplot(df['Fare'])  # 画分布图
```

```python
'''2、类别型'''
# 重点观察类别变量的Nunique情况、在数据集的占比分布（出现次数&占比）

df['Pclass'].nunique()  # Nunique情况
df['Pclass'].value_counts()  # 类别数量
df['Pclass'].value_counts(normalize=True)  # 类别占比情况

# 可视化
sns.countplot(x='Pclass', data=df)  # 使用seaborn
df['Pclass'].value_counts(normalize=True).plot(kind='bar')
```

```python
'''时间类型'''
# 拆分为月/天/日/小时，然后当做类别变量进行观测
# 抽取周期性信息，例如：工作日七天，然后观察周中和周末的分布信息
```

```python
'''字符串类型'''
# 当前的字符串数据是不是类型误标记了，它本质可能是其它类型的变量，例如时间类型的变量
# 如果是简单的字符串，例如国家等信息，我们就可以将其作为无相对大小的类别变量进行分析

# 生成词云
from wordcloud import WordCloud, STOPWORDS, ImageColorGenerator

wordcloud = WordCloud().generate(text)  
plt.imshow(wordcloud, interpolation='bilinear')
plt.axis("off")
plt.show() 

# 文本可视化
import scattertext as st
df = st.SampleCorpora.ConventionData2012.get_data().assign(
    parse=lambda df: df.text.apply(st.whitespace_nlp_with_sentences)
)

corpus = st.CorpusFromParsedDocuments(
    df, category_col='party', parsed_col='parse'
).build().get_unigram_corpus().compact(st.AssociationCompactor(2000))

html = st.produce_scattertext_explorer(
    corpus,
    category='democrat', category_name='Democratic', not_category_name='Republican',
    minimum_term_frequency=0, pmi_threshold_coefficient=0,
    width_in_pixels=1000, metadata=corpus.get_df()['speaker'],
    transform=st.Scalers.dense_rank
)
open('./demo_compact.html', 'w').write(html)
```

```python
# 图像数据
from PIL import Image
import numpy as np 

img_path = './pic/picture.jpg'
I = np.array(Image.open(path) )
plt.imshow(I) 
```

#### 交叉特征分析

在交叉特征分析模块，我们又将其分为特征与标签的交叉分析以及特征与特征之间的交叉等等；

-   特征&标签的关系分析：这块我们重点探讨特征与标签的关系，特征与标签是否强相关等等；

    ```python
    '类别变量 与 数值标签'
    # 如果不同类别之间的标签分布相差较大，则说明该类别信息是非常有有价值的
    df.groupby(类别变量)[数值标签].mean().plot(kind='bar')  # 画柱状图
    sns.boxplot(x=类别变量, y=数值标签, data=df)  # 画箱线图
    ```

    ```python
    '数值变量 与 数值标签'
    # 数值特征与数值标签的pearson相关系数.如果该数值的绝对值越大，往往说明该特征能为模型带来非常大的帮助
    df[[数值变量,数值标签]].corr('pearson')  # 相关性系数
    plt.scatter(x=数值变量, y=数值标签, data=df)  # 散点图，可视化
    ```

    ```python
    '时间变量 与 数值标签'
    # 随着时间变化，数据是否表现出某些特殊的模式
    plt.plot(df.index, df[数值标签].values, color='black', linestyle='--', linewidth='1', label=label)
    
    # 拆解时间变量信息
    from statsmodels.tsa.seasonal import seasonal_decompose
    
    decomposition = seasonal_decompose(self.ts, freq=freq, two_sided=False)
    self.trend = decomposition.trend
    self.seasonal = decomposition.seasonal
    self.residual = decomposition.resid
    decomposition.plot()
    plt.show()
    ```

    ```python
    '类别变量 与 二元标签'
    # 不同类之间的分布差不大，那说明该类别变量意义不大
    df.groupby(类别变量)[二元标签].mean()
    sns.barplot(x=类别变量, y=二元标签, data=df)  # 画柱状图
    ```

    ```python
    '数值变量 与 二元标签'
    # 对数值变量进行 分桶 ，然后基于类别变量与二元标签的关系进行分析
    # 使用boxplot函数，观测在不同标签下，数值特征的分布差异
    sns.boxplot(df[二元标签], df[数值变量])  # 不是箱线图
    ```

    ```python
    '类别变量 与 多元标签'
    # 将多分类转化为多个二分类然后进行观测
    # 多分类转化为二分类（others）
    sns.countplot(x=类别变量, hue=多元标签, data=df)
    ```

    ```python
    '数值变量 与 多元标签'
    # 使用boxplot即可，观测在每个类处数值变量的分布情况。如果所有类处的数值变量分布都类似，那可能该数值变量带来的影响会相对较小，反之影响较大
    sns.boxplot(x=多元标签, y=数值变量, data=df)  # 不是箱线图
    ```

-   特征&特征的关系分析：这块我们重点观察特征之间的冗余关系，是否是衍生关系等等；

-   可视化的技巧：特征之间的分析是做不完的，很多情况下我们一般也就只会看到三阶左右的特征关系，但是当数据特征字段上百的时候，即使是二阶交叉分析也需要进行好几天的分析，我们一般选取感兴趣的进行观测探索。

#### 训练集、测试集分布分析

关于训练集和测试集的分布探索，在目前数据竞赛中最为核心的模块之一，训练集和测试集的分布不一致也是导致线上和线下不一致的重要原因之一。

-   缺失值

    有些字段在训练集中都很正常，但是在测试集合中缺失极为严重。最简单的策略就是将训练集中的字段直接删除

-   数值特征分布

    直接绘制对应字段在训练集合和测试集合中的分布，查看训练集和测试集的交集`sns.distplot()`

-   类别特征分布

    ```python
    # 在测试集合中的类别是否在训练集合中都存在
    # 在测试集中出现,而未在训练集中出现的类别数和比例是多少
    len(set(te['f1'] ) - set(tr['f1'] ) 
    len(set(te['f1'] ) - set(tr['f1'] ) / len(set(tr['f1'])
    len(set(te['f1'] ) - set(tr['f1'] ) / len(set(te['f1'] ) + set(tr['f1'] )                                        
    # KL散度
    # KL散度经常被用于衡量两个概率分布的匹配程度的指标，两个分布差异越大，KL散度越大
    ```

    ![image-20210603154346967](https://raw.githubusercontent.com/louisyanglu/TyporaImages/master/20210603154347.png)，其中$P(x)$和$Q(x)$分别是训练集和测试集的分布，即计算每个类别的百分比

    ```python
    '多变量——衍生变量探索'
    # 先对特征进行演化
    tr.groupby(feature1)[feature2].agg(stas) 
    te.groupby(feature1)[feature2].agg(stas)
    # 使用单变量观测技巧，对演化之后的特征再进行细致的研究观察
    
    '多变量——基于模型的对抗验证'
    # 1、将训练集的数据全部打标签为1，将测试集的数据全部打标签为0；
    # 2、将训练集和测试集的数据合并，然后进行N折交叉验证；
    # 3、如果交叉验证的AUC接近0.5，那么说明训练集和测试集的分布是类似的；如果AUC非常大，例如大于0.9，那么我们就认为训练集和测试集的分布是存在较大差异的。
    
    '基于模型寻找分布差异最大的特征'
    # 1、训练好模型
    # 2、输出模型的feature importance

------

### 样本筛选、样本组织

-   对样本进行精心的组织从而得到我们的训练集、验证集以及测试集
-   需要对标签集合和特征集合进行细致的设计，尽可能多的使用到更多有价值的信息来提升我们模型的效果

#### 样本筛选

```python
'样本筛选'
1、明显异常的或者错误的数据——直接删除
2、针对我们的预测任务进行特定数据的筛选——如删掉大促时间段的数据来预测平时的商品的销量
3、可能收集错误的或者误标记的样本——
    3.1 基于模型的策略
        1）先使用训练数据进行模型的训练(例如典型的树模型等)，得到我们的模型
        2）使用模型对训练集进行预测
        3）按一定的比例，删除绝对误差大的样本
    3.2 基于假设的策略
        1）经常会假设过早的数据的分布与当前差异较大，将该部分数据剔除
4、数据量与计算的平衡
    4.1 随机降采样
        1）保留所有的正样本
        2）从负样本中随机抽取一定比例的样本
    4.2 时间序列采样
        1）推荐问题，电商搜索，销量预测，流量预测等问题，只需保留最近N天的数据
```

#### 样本组织

```python
'样本组织'
涉及到样本组织的问题最为常见的就是与时间序列建模相关的问题，例如各大电商或者视频网站的搜索与推荐等问题，商店的销量预测，股价的预测等
```

1.  基于最小单位的滑动建模

    如果我们的预测问题是预测未来N天商店每一天的销量，那么这边最小的单位就是**1天的销量**

    ![image-20210603161337484](https://raw.githubusercontent.com/louisyanglu/TyporaImages/master/20210603161337.png)

2.  基于测试组的滑动建模

    如果我们的预测问题是预测未来N天商店每一天的销量，那么这边测试组就是**N天商店每一天的销量**

    ![image-20210603161427516](https://raw.githubusercontent.com/louisyanglu/TyporaImages/master/20210603161427.png)

3.  基于周期形式的建模

    上面两种形式的建模很多时候可以拿到不错的效果，但也存在一些问题，它们都忽略了周期性，比如我们预测的是本周三到下周三每天商店的销量，而我们在使用上面的方式进行样本组织的时候则往往会很随意的将某个周四到下个周四的数据当做标签去训练

    ![image-20210603161630222](https://raw.githubusercontent.com/louisyanglu/TyporaImages/master/20210603161630.png)

------

### 验证策略设计

-   采用简单的五折交叉验证

-   还是进行分组进行交叉验证

-   按照时间顺序进行训练集和验证集合的划分

    <img src="https://raw.githubusercontent.com/louisyanglu/TyporaImages/master/20210603161711" alt="图片" style="zoom: 50%;" />

#### 分割验证

1.1 随机划分

```python
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=验证集的比例)
```

1.2 分层划分

```python
train_test_split(X, y, test_size=验证集的比例, stratify=y)
```

#### k折交叉验证

2.1 随机K折交叉验证

```python
对数据集进行shuffle打乱顺序；
将数据分成K折。K=5或10适用于大多数情况；
保留一折用于验证，使用剩下的其它折数据进行模型的训练；
在训练集合上训练模型，在验证集上评估模型，并记录该Fold的结果；
现在对所有其它的Fold重复该过程；
通过K折验证我们可以得到K个训练好的模型，采用K个模型分别对测试集进行预测；
取均值或者中位数等作为最终预测结果。

from sklearn.model_selection import KFold

kf = KFold(n_splits=K, shuffle=True) 
for train_index, test_index in kf.split(X, y):
    X_tr, X_val,y_tr,y_val = X[train_index],X[test_index], y[train_index],y[test_index]
```

2.2 分层K折交叉验证

```python
from sklearn.model_selection import StratifiedKFold

skf = StratifiedKFold(n_splits=5, shuffle=True)
for train_index, test_index in skf.split(X, y):
    X_train, X_test = X[train_index], X[test_index]
    y_train, y_test = y[train_index], y[test_index]
```

2.3 分组K折交叉验证

```python
分组的K折交叉验证常常被用于，判断基于某个特定组的数据训练得到的模型是否具有很好的泛化性，能够在未见过的组上取得很好的效果
确保验证时每一折的所有样本都来是训练折中未出现的组(id)

from sklearn.model_selection import GroupKFold

group_kfold = GroupKFold(n_splits=5)
for train_index, test_index in group_kfold.split(X, y, groups):
    X_train, X_test = X[train_index], X[test_index]
    y_train, y_test = y[train_index], y[test_index]
```

2.4 分层分组K折交叉验证

```python
import random
import numpy as np
import pandas as pd
from collections import Counter, defaultdict
def stratified_group_k_fold(X, y, groups, k, seed=None):
    labels_num = np.max(y) + 1
    y_counts_per_group = defaultdict(lambda: np.zeros(labels_num))
    y_distr = Counter()
    for label, g in zip(y, groups):
        y_counts_per_group[g][label] += 1
        y_distr[label] += 1

    y_counts_per_fold = defaultdict(lambda: np.zeros(labels_num))
    groups_per_fold = defaultdict(set)

    def eval_y_counts_per_fold(y_counts, fold):
        y_counts_per_fold[fold] += y_counts
        std_per_label = []
        for label in range(labels_num):
            label_std = np.std([y_counts_per_fold[i][label] / y_distr[label] for i in range(k)])
            std_per_label.append(label_std)
        y_counts_per_fold[fold] -= y_counts
        return np.mean(std_per_label)
    
    groups_and_y_counts = list(y_counts_per_group.items())
    random.Random(seed).shuffle(groups_and_y_counts)

    for g, y_counts in sorted(groups_and_y_counts, key=lambda x: -np.std(x[1])):
        best_fold = None
        min_eval = None
        for i in range(k):
            fold_eval = eval_y_counts_per_fold(y_counts, i)
            if min_eval is None or fold_eval < min_eval:
                min_eval = fold_eval
                best_fold = i
        y_counts_per_fold[best_fold] += y_counts
        groups_per_fold[best_fold].add(g)

    all_groups = set(groups)
    for i in range(k):
        train_groups = all_groups - groups_per_fold[i]
        test_groups = groups_per_fold[i]

        train_indices = [i for i, g in enumerate(groups) if g in train_groups]
        test_indices = [i for i, g in enumerate(groups) if g in test_groups]

        yield train_indices, test_indices
```

2.5 Repeated K折交叉验证

```python
相同的特征，相同的K折验证，不同的随机种子，两次的验证结果分数相差大
多做几次验证
步骤：
    1）设置重复的验证的次数M
    2）对于每一次验证，我们选用不同的随机种子进行K折验证
    3）将M次的验证结果取均值作为最终的验证结果

from sklearn.model_selection import RepeatedKFold
    
rkf = RepeatedKFold(*, n_splits=5, n_repeats=10
for train_index, test_index in rkf.split(X):
    print("TRAIN:", train_index, "TEST:", test_index)
    X_train, X_test = X[train_index], X[test_index]
    y_train, y_test = y[train_index], y[test_index]
```

2.6 Nested K折交叉验证

1.  基于特定的问题，我们将数据集进行特定的K折划分(随机/分层/分组...)，$D_1,D_2,...,D_K$;

2.  在第L轮中，我们选用$D_L$为验证集，其它折的数据集进行拼接得到我们新的训练集合；

3.  基于新的训练集合，我们采用$K_{inner}$折交叉进行超参数的调优，我们基于最优的参数重新训练得到我们的模型；

4.  使用重新训练得到的模型对我们的验证集进行预测，然后进行评估；

5.  **内部**的交叉验证用来进行**模型选择**以及参数调优，**外部**的交叉验证用来进行**效果评估**。在整个流程中只在最后一次对其进行预测，所以得到的验证结果会更加靠谱。

    ```python
    from sklearn.model_selection import KFold
    from sklearn.model_selection import GridSearchCV
    from sklearn.ensemble import RandomForestClassifier
    from sklearn.metrics import accuracy_score
    
    # 1.外层K折交叉验证
    cv_outer = KFold(n_splits=10, shuffle=True, random_state=1)
    
    outer_results = list()  # 结果评估
    for train_ix, test_ix in cv_outer.split(X):
        # 第一层分割
        X_train, X_test = X[train_ix, :], X[test_ix, :]
        y_train, y_test = y[train_ix], y[test_ix]
        # 2. 内部交叉验证
        cv_inner = KFold(n_splits=3, shuffle=True, random_state=1) 
        model = RandomForestClassifier(random_state=1) 
        space = dict()
        space['n_estimators'] = [10, 100, 500]
        space['max_features'] = [2, 4, 6] 
        search = GridSearchCV(model, space, scoring='accuracy', cv=cv_inner, refit=True)
        result = search.fit(X_train, y_train)
        # 获取最好的模型
        best_model = result.best_estimator_ 
        yhat = best_model.predict(X_test)
        # 模型
        acc = accuracy_score(y_test, yhat)
        # store the result
        outer_results.append(acc)
        # report progress
        print('>acc=%.3f, est=%.3f, cfg=%s' % (acc, result.best_score_, result.best_params_))
    # summarize the estimated performance of the model
    print('Accuracy: %.3f (%.3f)' % (mean(outer_results), std(outer_results)))
    ```

#### 时间序列验证

3.1 单折时间划分

```python
1、按照时间信息对我们的数据集进行排序；
2、选取某个相对时间/绝对时间作为划分点，之前的作为训练集，之后的数据作为验证集

df = df.sort_values('time')
# 按时间节点划分
val_time = ''
df_tr = df.loc[df['time'] <= val_time] 
df_val= df.loc[df['time'] > val_time]

# 按比例划分
tr_ratio = 0.7
tr_size = df.shape[0] * tr_ratio 
df_tr = df.iloc[:tr_size] 
df_val= df.iloc[tr_size:]
```

3.2 基于时间的N折验证

```python
1、选用T+1天的数据作为验证集，1,2,...,T的数据进行模型的训练；
2、选用T天的数据作为验证集，1,2,...,T-1的数据进行模型的训练；
3、选用T-1天的数据作为验证集，1,2,...,T-2的数据进行模型的训练；

from sklearn.model_selection import TimeSeriesSplit

tscv = TimeSeriesSplit(n_splits=5)
for train_index, test_index in tscv.split(X):
    X_train, X_test = X[train_index], X[test_index]
    y_train, y_test = y[train_index], y[test_index]
```

3.3 时间序列的Nested CV

![img](https://raw.githubusercontent.com/louisyanglu/TyporaImages/master/20210603220559)

------

### 模型理解和选择

-   神经网络模型往往需要对数值类的特征进行归一化操作，如果数值特征中存在奇异值，在很多时候会对模型带来灾难性的影响，导致模型无法拿到理想的结果
-   梯度提升树相关的模型，例如XGBoost，LightGBM，CatBoost等在建模的时候则往往不需要对特征进行归一化，对于特征中出现的极大极小值也有较好的鲁棒性。在基于表格形的数据建模中，基于梯度提升树的建模往往可以取得更好的成绩（90%的获奖方案）

------

### 特征工程

-   特征预处理
-   组合特征的构建
-   特征的筛选

------

### 模型训练、验证、测试

-   在模型的训练过程中，我们会比较如何得到好的模型参数，使用调参技巧，例如暴利式的调参，贪心的贝叶斯调参等

------

### 模型预测结果分析

-   在多分类中，需要观察哪几个类会相互分错，能不能基于这些分错的类进行某些处理来达到更好的预测效果
-   在回归问题中，需要观察我们预测最大的误差在哪里？这些预测最大的误差能否通过某种方式来缓解

------

### 后处理

-   存在一些特殊的问题，它们的评估指标是难以直接优化的，这个时候我们就需要考虑对最终的预测结果进行后处理等操作来提升我们的指标预测效果
-   需要对模型的预测结果进行某些加权或者分组加权的操作来修正我们模型的预测结果，最典型的就是F1等指标的优化

------

### 模型融合

-   auc问题我们一般可以采用对预测结果先进行rank，然后对rank进行加权融合
-   回归类的模型则可以采用MSE的优化和MAE的优化得到的结果进行融合

### 复盘总结

------

## 评估指标

### 回归

1.  **RMSE(Root Mean Square Error)**

![image-20210603120404962](https://raw.githubusercontent.com/louisyanglu/TyporaImages/master/20210603120405.png)

2.  **MSE(Mean Square Error)**

    ![image-20210603120454352](https://raw.githubusercontent.com/louisyanglu/TyporaImages/master/20210603120454.png)

3.  **MAE(Mean Absolute Error)**

    ![image-20210603120531900](https://raw.githubusercontent.com/louisyanglu/TyporaImages/master/20210603120531.png)

4.  **RMSLE(Root Mean Squared Logarithmic Error)**

    ![image-20210603120612412](https://raw.githubusercontent.com/louisyanglu/TyporaImages/master/20210603120612.png)

5.  **MAPE(Mean Absolute Percentage Error)**

    ![image-20210603120645472](https://raw.githubusercontent.com/louisyanglu/TyporaImages/master/20210603120645.png)

6.  **SMAPE(Symmetric Mean Absolute Percentage Error)**

    ![img](https://raw.githubusercontent.com/louisyanglu/TyporaImages/master/20210603121041.jpg)

### 二分类

1.  **Accuracy**

    ![image-20210603121409488](https://raw.githubusercontent.com/louisyanglu/TyporaImages/master/20210603121409.png)

    目前常采用的损失函数为Binary Logloss/Binary Cross Entropy：![image-20210603121439796](https://raw.githubusercontent.com/louisyanglu/TyporaImages/master/20210603121439.png)，其中$p_i$为模型对第$i$个样本的预测概率

2.  **F1**

    ![image-20210603121549031](https://raw.githubusercontent.com/louisyanglu/TyporaImages/master/20210603121549.png)

    常出现在一些类别不平衡的问题中，如交易欺诈、银行贷款逾期预估等

    使用binary cross entropy进行优化，得到最终的预测概率之后，需要通过一些策略寻找最优阈值。或者对损失函数进行加权优化，例如标签为1的样本的权重就设置大一些

3.  **Logloss**

    ![image-20210603121439796](https://raw.githubusercontent.com/louisyanglu/TyporaImages/master/20210603121905.png)

4.  **ROC**、**AUC**

    一般我们以TPR为y轴，以FPR为x轴，就可以得到ROC曲线

    -   x轴：![image-20210603122102020](https://raw.githubusercontent.com/louisyanglu/TyporaImages/master/20210603122102.png)
    -   y轴：![image-20210603122126683](https://raw.githubusercontent.com/louisyanglu/TyporaImages/master/20210603122126.png)

    AUC（Area Under Curve）被定义为ROC曲线下与坐标轴围成的面积，AUC越接近1.0，检测方法真实性越高;等于0.5时，一般就无太多应用价值了

5.  **Normalized Gini Coefficient**

    ![image-20210603122352957](https://raw.githubusercontent.com/louisyanglu/TyporaImages/master/20210603122353.png)

    使用Logloss进行优化

### 多分类

1.  **categorization accuracy**

    ![image-20210603122531965](https://raw.githubusercontent.com/louisyanglu/TyporaImages/master/20210603122532.png)

    使用Logloss函数优化：![image-20210603124732110](https://raw.githubusercontent.com/louisyanglu/TyporaImages/master/20210603124732.png)，$y_{i,k}$表示第$i$个样本标签$k$为的情况，如果标签为$k$则是1，反之为0。$p_{i,k}$则是模型预测样本$i$属于第$k$类的概率

2.  **MultiLogloss**

    ![image-20210603124732110](https://raw.githubusercontent.com/louisyanglu/TyporaImages/master/20210603125137.png)

3.  **MAP(Mean Average Precision)**

    ![image-20210603125311224](https://raw.githubusercontent.com/louisyanglu/TyporaImages/master/20210603125311.png)

    ![image-20210603125323467](https://raw.githubusercontent.com/louisyanglu/TyporaImages/master/20210603125323.png)

    使用sigmoid_cross_entropy函数进行优化

4.  **Mean F1**

    ![image-20210603125411847](https://raw.githubusercontent.com/louisyanglu/TyporaImages/master/20210603125411.png)

5.  **Average Jaccard Index**

    ![image-20210603125507146](https://raw.githubusercontent.com/louisyanglu/TyporaImages/master/20210603125507.png)

    使用基于sigmoid的损失函数进行优化