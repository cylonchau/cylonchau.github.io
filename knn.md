# KNN


## Overview

K近邻值算法 **KNN (K — Nearest Neighbors)** 是一种机器学习中的分类算法；K-NN是一种**非参数**的**惰性学习算法**。非参数意味着没有对基础数据分布的假设，即模型结构是从数据集确定的。

它被称为**惰性算法**的原因是，因为它**不需要任何训练数据点来生成模型。**所有训练数据都用于测试阶段，这使得训练更快，测试阶段更慢且成本更高。

## 如何工作

KNN 算法是通过计算新对象与训练数据集中所有对象之间的距离，对新实例进行分类或回归预测。然后选择训练数据集中距离最小的 K 个示例，并通过平均结果进行预测。

如图所示：一个未分类的数据（红色）和所有其他已分类的数据（黄色和紫色），每个数据都属于一个类别。因此，计算未分类数据与所有其他数据的距离，以了解哪些距离最小，因此当K= 3 （或K= 6 ）最接近的数据并检查出现最多的类，如下图所示，与新数据最接近的数据是在第一个圆圈内（圆圈内）的数据，在这个圆圈内还有 3 个其他数据（已经用黄色分类），我们将检查其中的主要类别，会被归类为紫色，因为有2个紫色球，1个黄色球。


![image-20221021234452066](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221021234452066.png)


### KNN算法要执行的步骤

- 将数据分为训练数据和测试数据
- 选择一个值 K
- 确定要使用的距离算法
- 从需要分类的测试数据中选择一个样本，计算到它的 n 个训练样本的距离。
- 对获得的距离进行排序并取 k最近的数据样本。
- 根据 k 个邻居的多数票将测试类分配给该类。

### 影响KNN算法性能的因素

- 用于确定最近邻居的**距离**的算法

- 用于从 K 近邻派生分类的决策规则

- 用于对新示例进行分类的邻居**数**


## 如何计算距离

测量距离是KNN算法的核心，总结了问题域中两个对象之间的相对差异。比较常见的是，这两个对象是描述主题（例如人、汽车或房屋）或事件（例如购买、索赔或诊断）的数据行。

### 汉明距离

汉明距离（`Hamming Distance`）计算两个二进制向量之间的距离，也简称为二进制串 `binary strings` 或位串 `bitstrings `；换句话说，汉明距离是将一个字符串更改为另一个字符串所需的最小替换次数，或将一个字符串转换为另一个字符串的最小错误数。

示例：如一列具有类别 “红色”、“绿色” 和 “蓝色”，您可以将每个示例独热编码为一个位串，每列一个位。

> 注：独热编码 one-hot encoding：将分类数据，转换成二进制向量表示，这个二进制向量用来表示一种特殊的bit（二进制位）组合，该字节里，仅容许单一bit为1，其他bit都必须为0
>
> 如：
>
> | apple | banana | pineapple |
> | ----- | ------ | --------- |
> | 1     | 0      | 0         |
> | 0     | 1      | 0         |
> | 0     | 0      | 1         |
>
>  100 表示苹果，100就是苹果的二进制向量
>  010 表示香蕉，010就是香蕉的二进制向量

```
red = [1, 0, 0]
green = [0, 1, 0]
blue = [0, 0, 1]
```

而red和green之间的距离就是**两个等长bitstrings之间bit差**（对应符号不同的位置）的总和或平均数，这就是汉明距离

- $Hamming Distance d(a, b)\ =\ sum(xi\ !=\ yi\ for\ xi,\ yi\ in\ zip(x, y))$

上述的实现为：

```python
def hammingDistance(a, b):
    if len(a) != len(b):
        raise ValueError("Undefined for sequences of unequal length.")
    return sum(abs(e1 - e2) for e1, e2 in zip(a, b))

row1 = [0, 0, 0, 0, 0, 1]
row2 = [0, 0, 0, 0, 1, 0]

dist = hammingDistance(row1, row2)
print(dist)
```

可以看到字符串之间有两个差异，或者 6 个位位置中有 2 个不同，平均 (2/6) 约为 1/3 或 0.333。

```python

from scipy.spatial.distance import hamming
# define data
row1 = [0, 0, 0, 0, 0, 1]
row2 = [0, 0, 0, 0, 1, 0]

# calculate distance
dist = hamming(row1, row2)
print(dist)
```

### 欧几里得距离

欧几里得距离（`Euclidean distance`） 是计算两个点之间的距离。在计算具体的数值（例如浮点数或整数）的两行数据之间的距离时，您最有可能使用欧几里得距离。

欧几里得距离计算公式为两个向量之间的平方差之和的平方根。

$EuclideanDistance=\sqrt[]{\sum(a-b)^2}$

如果要执行数千或数百万次距离计算，通常会去除平方根运算以加快计算速度。修改后的结果分数将具有相同的相对比例，并且仍然可以在机器学习算法中有效地用于查找最相似的示例。

$EuclideanDistance = sum\ for\ i\ to\ N\ (v1[i]\ –\ v2[i])^2$

```python
# calculating euclidean distance between vectors
from math import sqrt
from scipy.spatial.distance import euclidean

# calculate euclidean distance
def euclidean_distance(a, b):
	return sqrt(sum((e1-e2)**2 for e1, e2 in zip(a,b)))
 
# define data
row1 = [10, 20, 15, 10, 5]
row2 = [12, 24, 18, 8, 7]
# calculate distance
dist = euclidean_distance(row1, row2)
print(dist)
print(euclidean(row1, row2))
```

### 曼哈顿距离

曼哈顿距离（ `Manhattan distance` ）又被称作出租车几何学 `Taxicab geometry`；用于计算两个向量之间的距离。

对于描述网格上的对象（如棋盘或城市街区）的向量可能更有用。出租车在城市街区之间采取的最短路径（网格上的坐标）。

> 粗略地说，欧几里得几何是中学常用的平面几何和立体几何 [Plane geometry](https://www.britannica.com/science/Euclidean-geometry/Plane-geometry)

曼哈顿距离可以理解为：欧几里得空间的固定直角坐标系上两点所形成的线段对轴产生的投影的距离总和。

![image-20221021234505760](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221021234505760.png)



图中： 红、蓝与黄线分别表示所有曼哈顿距离都拥有一样长度（12），绿线表示欧几里得距离 $6×\sqrt2 ≈ 8.48$

对于整数特征空间中的两个向量，应该计算曼哈顿距离而不是欧几里得距离

曼哈顿距离在二维平面的计算公式是，在X轴的亮点

$Manhattandistance\ d(x,y)=\left|x_{1}-x_{2}\right|+\left|y_{1}-y_{2}\right|$

![image-20221021234535889](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20221021234535889.png)

如果所示，描述格子和格子之间的距离可以用曼哈顿距离，如国王移动到右下角的距离是？

$King=|6-8|+|6-1| = 7$

两个向量间的距离可以表示为 $MD\ =\ Σ|Ai – Bi|$

python中的公式可以表示为 ：`sum(abs(val1-val2) for val1, val2 in zip(a,b))`

```python
from scipy.spatial.distance import cityblock
# calculate manhattan distance
def manhattan_distance(a, b):
	return sum(abs(e1-e2) for e1, e2 in zip(a,b))
 
# define data
row1 = [10, 20, 15, 10, 5]
row2 = [12, 24, 18, 8, 7]
# calculate distance
dist = manhattan_distance(row1, row2)
print(dist)
print(cityblock(row1, row2))
```

### 闵可夫斯基距离

闵可夫斯基距离（`Minkowski distance`）并不是一种距离而是对是**欧几里得距离**和**曼哈顿距离**的概括，用来计算两个向量之间的距离。

闵可夫斯基增并添加了一个参数，称为“**阶数**”或 `p`：$d(x,y) = (\sum(|x-y|)^p)^\frac{1}{p}$

在python中的公式：

```python
(sum for i to N (abs(v1[i] – v2[i]))^p)^(1/p)
```

`p` 是一个有序的参数，当 $p=1$ 时，计算的是曼哈顿距离。当 $p=2$ 时，计算的是欧几里得距离。

在实现使用距离度量的机器学习算法时，通常会使用闵可夫斯基距离，因为可以通过调整参数“ *p* ”控制用于向量的距离度量算法的类型。

```python
# calculating minkowski distance between vectors
from scipy.spatial import minkowski_distance
 
# calculate minkowski distance
def minkowski_distance(a, b, p):
	return sum(abs(e1-e2)**p for e1, e2 in zip(a,b))**(1/p)
 
# define data
row1 = [10, 20, 15, 10, 5]
row2 = [12, 24, 18, 8, 7]

# 手动实现的算法用来使用闵可夫斯基计算距离
dist = minkowski_distance(row1, row2, 1)
# 1为曼哈顿
print(dist)
# 1为欧几里得
dist = minkowski_distance(row1, row2, 2)
print(dist)

# 使用包 scipy.spatial来计算
print(minkowski_distance(row1, row2, 1))
print(minkowski_distance(row1, row2, 2))
```

## KNN算法实现

### Prerequisite

首先会用示例来实现KNN算法的每个步骤，并加以分析，然后将所有步骤关联在在一起，形成一个适用于真实数据集的实现。

KNN在实现起来主要有三个步骤：

- 计算距离（这里选择欧几里得距离）
- 获得临近邻居
- 做出预测

这三个步骤是KNN算法用以解决分类和回归预测建模问题的基础知识

### 计算距离

第一步计算数据集中两行之间的距离。在数据集中的数据行主要由数字组成，计算两行或数字向量之间的距离的一种简单方法是画一条直线。这在 2D 或 3D 平面中都是很好地选择，并且可以很好地扩展到更高的维度。

这里使用的是比较流行的计算距离的算法，**欧几里得距离**来计算两个向量之间的直线距离。欧几里得距离的公式是，两个向量的平方差的平方根，$Euclidean\ Distance=\sqrt[]{\sum(a-b)^2}$ ；在python中可以表示为：`sqrt(sum i to N (x1 – x2)^2)` ；其中 `x1` 是第一行数据，`x2` 是第二行数据，`i`  表示特定列的索引，因为可能需要对所有行进行计算。

在欧几里得距离中，值越小，两条记录就越相似； 0 表示两条记录之间没有差异。

那么使用python实现一个计算欧几里得距离的算法

```python
def euclidean_distance(row1, row2):
	distance = 0.0
	for i in range(len(row1)-1):
		distance += (row1[i] - row2[i])**2
	return sqrt(distance)
```

准备一部分测试数据，来对测试距离算法

```python
X1				X2					Y
2.7810836		2.550537003			0
1.465489372		2.362125076			0
3.396561688		4.400293529			0
1.38807019		1.850220317			0
3.06407232		3.005305973			0
7.627531214		2.759262235			1
5.332441248		2.088626775			1
6.922596716		1.77106367			1
8.675418651		-0.242068655		1
7.673756466		3.508563011			1
```

那么来测试这些数据，需要做到的是第一行与所有行之间的距离，对于第一行与自己的距离应该为**0**

```python
from math import sqrt
 
# 欧几里得距离，计算两个向量间距离的算法
def euclidean_distance(row1, row2):
	distance = 0.0
	for i in range(len(row1)-1):
		distance += (row1[i] - row2[i])**2 # 平方差
	return sqrt(distance) # 平方根
 
# 测试数据集
dataset = [
    [2.7810836,2.550537003,0],
	[1.465489372,2.362125076,0],
	[3.396561688,4.400293529,0],
	[1.38807019,1.850220317,0],
	[3.06407232,3.005305973,0],
	[7.627531214,2.759262235,1],
	[5.332441248,2.088626775,1],
	[6.922596716,1.77106367,1],
	[8.675418651,-0.242068655,1],
	[7.673756466,3.508563011,1]
]
row0 = dataset[0]
for row in dataset:
	distance = euclidean_distance(row0, row)
	print(distance)
    
# 0.0
# 1.3290173915275787
# 1.9494646655653247
# 1.5591439385540549
# 0.5356280721938492
# 4.850940186986411
# 2.592833759950511
# 4.214227042632867
# 6.522409988228337
# 4.985585382449795
```

### 获取最近邻居

数据集中新数据的邻居是k个最接近的实例（行），这个实例由距离定义。现在诞生的问题：**如何找到最近的邻居？以及怎么找到最近的邻居？**

- 为了在数据集中找到 K 的邻居，首先必须计算数据集中每条记录与新数据之间的距离。

- 有了距离之后，必须按照 **K** 的距离对训练集中的所有实例排序。然后选择前 **k** 个作为最近的邻居。

这里实现起来是通过将数据集中每条记录的距离作为一个元组来跟踪，通过对元组列表进行排序（距离降序），然后检索最近邻居。下面是一个实现这些步骤的函数

```python
# 找到最近的邻居
def get_neighbors(train, test_row, num_neighbors):
    """
    计算训练集train中所有元素到test_row的距离
    :param train: list, 数据集，可以是训练集
    :param test_row: list, 新的实例，也就是K
    :param num_neighbors:int，需要多少个邻居
    :return: None
    """
    distances = list()
    for train_row in train:
        # 计算出每一行的距离，把他添加到元组中
        dist = euclidean_distance(test_row, train_row)
        distances.append((train_row, dist))
    distances.sort(key=lambda knn: knn[1]) # 根据元素哪个字段进行排序
    neighbors = list()
    for i in range(num_neighbors):
        neighbors.append(distances[i][0])
    return neighbors
```

下面是完整的示例

```python
from math import sqrt
 
# 欧几里得距离，计算两个向量间距离的算法
def euclidean_distance(row1, row2):
	distance = 0.0
	for i in range(len(row1)-1):
		distance += (row1[i] - row2[i])**2 # 平方差
	return sqrt(distance) # 平方根
 
# 找到最近的邻居
def get_neighbors(train, test_row, num_neighbors):
    """
    计算训练集train中所有元素到test_row的距离
    :param train: list, 数据集，可以是训练集
    :param test_row: list, 新的实例，也就是K
    :param num_neighbors:int，需要多少个邻居
    :return: None
    """
    distances = list()
    for train_row in train:
        # 计算出每一行的距离，把他添加到元组中
        dist = euclidean_distance(test_row, train_row)
        distances.append((train_row, dist))
    distances.sort(key=lambda knn: knn[1]) # 根据元素哪个字段进行排序
    neighbors = list()
    for i in range(num_neighbors):
        neighbors.append(distances[i][0])
    return neighbors

# 测试数据集
dataset = [
    [2.7810836,2.550537003,0],
	[1.465489372,2.362125076,0],
	[3.396561688,4.400293529,0],
	[1.38807019,1.850220317,0],
	[3.06407232,3.005305973,0],
	[7.627531214,2.759262235,1],
	[5.332441248,2.088626775,1],
	[6.922596716,1.77106367,1],
	[8.675418651,-0.242068655,1],
	[7.673756466,3.508563011,1]
]

neighbors = get_neighbors(dataset, dataset[0], 3)
for neighbor in neighbors:
	print(neighbor)

# [2.7810836, 2.550537003, 0]
# [3.06407232, 3.005305973, 0]
# [1.465489372, 2.362125076, 0]
```

可以看到，运行后会将数据集中最相似的 3 条记录按相似度顺序打印。和预测的一样，第一个记录与其本身最相似，并且位于列表的顶部。

### 预测结果

预测结果在这里指定是，通过分类拿到了最近的邻居的实例，对邻居进行分类，找到邻居中最大类别的一类，作为预测值。这里使用的是对邻居值执行 `max()` 来实现这一点，下面是实现方式

```python
# 预测值
def predict_classification(train, test_row, num_neighbors):
    """
    计算训练集train中所有元素到test_row的距离
    :param train: list, 数据集，可以是训练集
    :param test_row: list, 新的实例，也就是K
    :param num_neighbors:int，需要多少个邻居
    :return: None
    """
    neighbors = get_neighbors(train, test_row, num_neighbors)
    output_values = [row[-1] for row in neighbors] # 拿到所属类的真实类别
    prediction = max(set(output_values), key=output_values.count)  #算出邻居类别最大的数量
    return prediction
```

下面是完整的示例

```python
from math import sqrt
 
# 欧几里得距离，计算两个向量间距离的算法
def euclidean_distance(row1, row2):
    distance = 0.0
    for i in range(len(row1)-1):
        distance += (row1[i] - row2[i])**2 # 平方差
    return sqrt(distance) # 平方根
 
# 找到最近的邻居
def get_neighbors(train, test_row, num_neighbors):
    """
    计算训练集train中所有元素到test_row的距离
    :param train: list, 数据集，可以是训练集
    :param test_row: list, 新的实例，也就是K
    :param num_neighbors:int，需要多少个邻居
    :return: None
    """
    distances = list()
    for train_row in train:
        # 计算出每一行的距离，把他添加到元组中
        dist = euclidean_distance(test_row, train_row)
        distances.append((train_row, dist))
    distances.sort(key=lambda knn: knn[1]) # 根据元素哪个字段进行排序
    neighbors = list()
    for i in range(num_neighbors):
        neighbors.append(distances[i][0])
    return neighbors

# 预测值
def predict_classification(train, test_row, num_neighbors):
    """
    计算训练集train中所有元素到test_row的距离
    :param train: list, 数据集，可以是训练集
    :param test_row: list, 新的实例，也就是K
    :param num_neighbors:int，需要多少个邻居
    :return: None
    """
    neighbors = get_neighbors(train, test_row, num_neighbors)
    output_values = [row[-1] for row in neighbors] # 拿到所属类的真实类别
    prediction = max(set(output_values), key=output_values.count)  #算出邻居类别最大的数量
    return prediction

# 测试数据集
dataset = [
    [2.7810836,2.550537003,0],
	[1.465489372,2.362125076,0],
	[3.396561688,4.400293529,0],
	[1.38807019,1.850220317,0],
	[3.06407232,3.005305973,0],
	[7.627531214,2.759262235,1],
	[5.332441248,2.088626775,1],
	[6.922596716,1.77106367,1],
	[8.675418651,-0.242068655,1],
	[7.673756466,3.508563011,1]
]

for n in range(len(dataset)):
    prediction = predict_classification(dataset, dataset[n], 5)
    print('Expected %d, Got %d.' % (dataset[n][-1], prediction))

# Expected 0, Got 0.
# Expected 0, Got 0.
# Expected 0, Got 0.
# Expected 0, Got 0.
# Expected 0, Got 0.
# Expected 1, Got 1.
# Expected 1, Got 1.
# Expected 1, Got 1.
# Expected 1, Got 1.
# Expected 1, Got 1.
```

运行结果打印了预期分类与从数据集中 3 个相进邻居预测结果是一直的。

## 鸢尾花种实例

这里使用的是 [Iris Flower Species](https://raw.githubusercontent.com/jbrownlee/Datasets/master/iris.csv) 数据集。

鸢尾花数据集是根据鸢尾花的测量值预测花卉种类。这是一个多类分类问题。每个类的观察数量是平衡的。有 150 个观测值，有 4 个输入变量和 1 个输出变量。变量名称如下：

- 萼片长度以厘米为单位。
- 萼片宽度以厘米为单位。
- 花瓣长度以厘米为单位。
- 花瓣宽度以厘米为单位。
- 真实类型

更多的关于数据集的说明可以参考：[Iris-databases数据集的说明](https://raw.githubusercontent.com/jbrownlee/Datasets/master/iris.names)

### Prerequisite

实验的步骤大概分为如下：

- 加载数据集并将数据转换为可用于均值和标准差计算的数字。将属性转为float，将类别转换为int。
- 使 5折的**K**折较差验证（**K-Fold CV**）评估该算法。

### Start

```python
from random import seed
from random import randrange
from csv import reader
from math import sqrt

# 加载CSV
def load_csv(filename):
    dataset = list()
    with open(filename, 'r') as file:
        csv_reader = reader(file)
        for row in csv_reader:
            if not row:
                continue
            dataset.append(row)
    return dataset

# 转换所有的值为float方便运算
def str_column_to_float(dataset, column):
    for row in dataset:
        row[column] = float(row[column].strip())

# 转换所有的类型为int
def str_column_to_int(dataset, column):
    class_values = [row[column] for row in dataset]
    unique = set(class_values)
    lookup = dict()
    for i, value in enumerate(unique):
        lookup[value] = i
    for row in dataset:
        row[column] = lookup[row[column]]
    return lookup



# # k-folds CV函数进行划分
def cross_validation_split(dataset, n_folds):
    dataset_split = list()
    dataset_copy = list(dataset)
    # 平均分成n_folds折数
    fold_size = int(len(dataset) / n_folds)
    for _ in range(n_folds):
        fold = list()
        while len(fold) < fold_size:
            index = randrange(len(dataset_copy))
            fold.append(dataset_copy.pop(index))
        dataset_split.append(fold)
    return dataset_split

# 计算精确度
def accuracy_metric(actual, predicted):
    correct = 0
    for i in range(len(actual)):
        if actual[i] == predicted[i]:
            correct += 1
    return correct / float(len(actual)) * 100.0

# 评估算法
def evaluate_algorithm(dataset, algorithm, n_folds, *args):
    """
    评估算法，计算算法的精确度
    :param dataset: list, 数据集
    :param algorithm: function, 算法名
    :param n_folds: int，折数
    :param args: 用于algorithm的参数
    :return: None
    """
    folds = cross_validation_split(dataset, n_folds) # 分成5折
    scores = list()
    for fold in folds:
        train_set = list(folds)
        train_set.remove(fold) # 训练集不包含本身
        train_set = sum(train_set, [])
        test_set = list() # 测试集
        for row in fold:
            row_copy = list(row)
            test_set.append(row_copy)
            row_copy[-1] = None
        predicted = algorithm(train_set, test_set, *args)
        actual = [row[-1] for row in fold]
        accuracy = accuracy_metric(actual, predicted)
        scores.append(accuracy)
    return scores

# 欧几里得距离，计算两个向量间距离的算法
def euclidean_distance(row1, row2):
    distance = 0.0
    for i in range(len(row1)-1):
        distance += (row1[i] - row2[i])**2
    return sqrt(distance)

# 确定最邻近的邻居
def get_neighbors(train, test_row, num_neighbors):
    """
    计算训练集train中所有元素到test_row的距离
    :param train: list, 数据集，可以是训练集
    :param test_row: list, 新的实例，也就是K
    :param num_neighbors:int，需要多少个邻居
    :return: None
    """
    distances = list()
    for train_row in train:
        dist = euclidean_distance(test_row, train_row)
        distances.append((train_row, dist))
    distances.sort(key=lambda tup: tup[1])
    neighbors = list()
    for i in range(num_neighbors):
        neighbors.append(distances[i][0])
    return neighbors

# 与临近值进行比较并预测
def predict_classification(train, test_row, num_neighbors):
    """
    计算训练集train中所有元素到test_row的距离
    :param train: list, 数据集，可以是训练集
    :param test_row: list, 新的实例，也就是K
    :param num_neighbors:int，需要多少个邻居
    :return: None
    """
    neighbors = get_neighbors(train, test_row, num_neighbors)
    output_values = [row[-1] for row in neighbors]
    prediction = max(set(output_values), key=output_values.count)
    return prediction

# kNN Algorithm
def k_nearest_neighbors(train, test, num_neighbors):
    predictions = list()
    for row in test:
        output = predict_classification(train, row, num_neighbors)
        predictions.append(output)
    return(predictions)

# 使用KNN算法计算鸢尾花数据集
seed(1)
filename = 'iris.csv'
dataset = load_csv(filename)
for i in range(len(dataset[0])-1):
    str_column_to_float(dataset, i)
# 转换类型为int
str_column_to_int(dataset, len(dataset[0])-1)
# 评估算法
n_folds = 5 # 5折
num_neighbors = 5 #取5个邻居
scores = evaluate_algorithm(dataset, k_nearest_neighbors, n_folds, num_neighbors)
print('Scores: %s' % scores)
print('Mean Accuracy: %.3f%%' % (sum(scores)/float(len(scores))))

# Scores: [96.66666666666667, 96.66666666666667, 100.0, 90.0, 100.0]
# Mean Accuracy: 96.667%
```

上述是对整个数据集的预测百分比，也可以对对应的类的信息进行输出

首先在类别转换函数 `str_column_to_int` 中增加打印方法

```python
for i, value in enumerate(unique):
    lookup[value] = i
    print('[%s] => %d' % (value, i))
```

然后在定义一个新的实例，这个实例是用于预测的信息 `row = [5.7,2.9,4.2,1.3]` ; 然后修改需要预测的数据，进行预测

```python
# 原来的整个数据集打分不需要了
# scores = evaluate_algorithm(dataset, k_nearest_neighbors, n_folds, num_neighbors)
# print('Scores: %s' % scores)
# print('Mean Accuracy: %.3f%%' % (sum(scores)/float(len(scores))))

# 定义一个新数据
row = [5.7,2.9,4.2,1.3]

label = predict_classification(dataset, row, num_neighbors)
print('Data=%s, Predicted: %s' % (row, label))

# Data=[5.7, 2.9, 4.2, 1.3], Predicted: 1
```

通过预测，可以看出预测结果属于第 1 类，就知道该花为 `Iris-setosa` 。



> Reference
>
> [distance measures](https://machinelearningmastery.com/distance-measures-for-machine-learning/)
>
> [k nearest neighbors implement](https://machinelearningmastery.com/tutorial-to-implement-k-nearest-neighbors-in-python-from-scratch/)

