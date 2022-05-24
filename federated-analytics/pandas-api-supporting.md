# Pandas API支持列表

## DataFrame API

### all

```python
delta.pandas.DataFrame.all(self, axis=0, bool_only=False, skipna=True)
```

判断每一行，或每一列上的所有元素是否都为真。

参数：

* axis: 0或1。表示聚合的轴线，0为行，1为列。当axis=0时，返回一个index为当前DataFrame的Columns的Series；当axis=1时，返回一个index为当前DataFrame的index的Series。
* bool_only: bool类型。bool_only=True时，只包含元素为bool类型的列；否则包含所有的列。
* skipna: bool类型。如果整行、整列的值都是NA，并且skina=True，那么结果为True；如果skipna=False，那么NA被视作True，因为NA是非零值。

返回值：

* delta.pandas.Series

### any

```python
delta.pandas.DataFrame.any(self, axis=0, bool_only=False, skipna=True)
```

判断每一行，或每一列上是否有任意一个元素为真。

参数：

* axis: 0或1。表示聚合的轴线，0为行，1为列。当axis=0时，返回一个index为当前DataFrame的Columns的Series；当axis=1时，返回一个index为当前DataFrame的index的Series。
* bool_only: bool类型。bool_only=True时，只包含元素为bool类型的列；否则包含所有的列。
* skipna: bool类型。如果整行、整列的值都是NA，并且skina=True，那么结果为False；如果skipna=False，那么NA被视作True，因为NA是非零值。

返回值：

* delta.pandas.Series

### count

```python
delta.pandas.DataFrame.count(self, axis=0, numeric_only=False)
```

计算每一行，或每一列上非NA元素的数量。

参数：

* axis: 0或1。表示聚合的轴线，0为行，1为列。当axis=0时，返回一个index为当前DataFrame的Columns的Series；当axis=1时，返回一个index为当前DataFrame的index的Series。
* numeric_only: bool类型。numeric_only=True时，只统计float，int，bool类型的值。

返回值：

* delta.pandas.Series

### sum

```python
delta.pandas.DataFrame.sum(self, axis=0, skipna=True, numeric_only=False, min_count=0)
```

计算每一行，或每一列上的元素的和。

参数：

* axis: 0或1。表示聚合的轴线，0为行，1为列。当axis=0时，返回一个index为当前DataFrame的Columns的Series；当axis=1时，返回一个index为当前DataFrame的index的Series。
* skipna: bool类型。skipna=True时，计算时排除NA值；否则包含NA值。
* numeric_only: bool类型。numeric_only=True时，只统计float，int，bool类型的值。
* min_count: int。所需的合法值的最小数量。如果数量不足min_count，结果会为NA。

返回值：

* delta.pandas.Series

### mean

```python
delta.pandas.DataFrame.mean(self, axis=0, skipna=True, numeric_only=False)
```

计算每一行，或每一列上的元素的平均值。

参数：

* axis: 0或1。表示聚合的轴线，0为行，1为列。当axis=0时，返回一个index为当前DataFrame的Columns的Series；当axis=1时，返回一个index为当前DataFrame的index的Series。
* skipna: bool类型。skipna=True时，计算时排除NA值；否则包含NA值。
* numeric_only: bool类型。numeric_only=True时，只统计float，int，bool类型的值。

返回值：

* delta.pandas.Series

### std

```python
delta.pandas.DataFrame.std(self, axis=0, skipna=True, ddof=1, numeric_only=False)
```

计算每一行，或每一列上的元素的标准差。

参数：

* axis: 0或1。表示聚合的轴线，0为行，1为列。当axis=0时，返回一个index为当前DataFrame的Columns的Series；当axis=1时，返回一个index为当前DataFrame的index的Series。
* skipna: bool类型。skipna=True时，计算时排除NA值；否则包含NA值。
* ddof: int类型。自由度（Delta Degrees of Freedom）。计算时的除数是N-ddof，其中N是元素的数量。
* numeric_only: bool类型。numeric_only=True时，只统计float，int，bool类型的值。

返回值：

* delta.pandas.Series

### var

```python
delta.pandas.DataFrame.var(self, axis=0, skipna=True, ddof=1, numeric_only=False)
```

计算每一行，或每一列上的元素的方差。

参数：

* axis: 0或1。表示聚合的轴线，0为行，1为列。当axis=0时，返回一个index为当前DataFrame的Columns的Series；当axis=1时，返回一个index为当前DataFrame的index的Series。
* skipna: bool类型。skipna=True时，计算时排除NA值；否则包含NA值。
* ddof: int类型。自由度（Delta Degrees of Freedom）。计算时的除数是N-ddof，其中N是元素的数量。
* numeric_only: bool类型。numeric_only=True时，只统计float，int，bool类型的值。

返回值：

* delta.pandas.Series

### sem

```python
delta.pandas.DataFrame.sem(self, axis=0, skipna=True, ddof=1, numeric_only=False)
```

计算每一行，或每一列上的元素的标准误差(standard error of the mean, SEM)。

参数：

* axis: 0或1。表示聚合的轴线，0为行，1为列。当axis=0时，返回一个index为当前DataFrame的Columns的Series；当axis=1时，返回一个index为当前DataFrame的index的Series。
* skipna: bool类型。skipna=True时，计算时排除NA值；否则包含NA值。
* ddof: int类型。自由度（Delta Degrees of Freedom）。计算时的除数是N-ddof，其中N是元素的数量。
* numeric_only: bool类型。numeric_only=True时，只统计float，int，bool类型的值。

返回值：

* delta.pandas.Series

### add

```python
delta.pandas.DataFrame.add(self, other, axis=1, fill_value=None)
```

将DataFrame与other逐元素相加。等同于dataFrame + other，但是支持fill_value参数来填充输入中的NA值。交换操作顺序的版本是`radd`。

参数：

* other: DataFrame, Series, 值序列或单个值。
* axis: 0或1。0为行，1为列。是否沿行或列对dataFrame与other之间的元素进行比较。
* fill_value: float或None。在计算前，填充缺失的值（NA值），以及数据对齐时需要填充的新元素。如果两个输入在同一位置的值都是缺失的，那么结果在该位置上也是缺失的。

返回值：

* delta.pandas.DataFrame

### sub

```python
delta.pandas.DataFrame.sub(self, other, axis=1, fill_value=None)
```

将DataFrame与other逐元素相减。等同于dataFrame - other，但是支持fill_value参数来填充输入中的NA值。交换操作顺序的版本是`rsub`。

参数：

* other: DataFrame, Series, 值序列或单个值。
* axis: 0或1。0为行，1为列。是否沿行或列对dataFrame与other之间的元素进行比较。
* fill_value: float或None。在计算前，填充缺失的值（NA值），以及数据对齐时需要填充的新元素。如果两个输入在同一位置的值都是缺失的，那么结果在该位置上也是缺失的。

返回值：

* delta.pandas.DataFrame

### mul

```python
delta.pandas.DataFrame.mul(self, other, axis=1, fill_value=None)
```

将DataFrame与other逐元素相乘。等同于dataFrame * other，但是支持fill_value参数来填充输入中的NA值。交换操作顺序的版本是`rmul`。

参数：

* other: DataFrame, Series, 值序列或单个值。
* axis: 0或1。0为行，1为列。是否沿行或列对dataFrame与other之间的元素进行比较。
* fill_value: float或None。在计算前，填充缺失的值（NA值），以及数据对齐时需要填充的新元素。如果两个输入在同一位置的值都是缺失的，那么结果在该位置上也是缺失的。

返回值：

* delta.pandas.DataFrame

### div

```python
delta.pandas.DataFrame.div(self, other, axis=1, fill_value=None)
```

将DataFrame与other逐元素相除。等同于dataFrame / other，但是支持fill_value参数来填充输入中的NA值。交换操作顺序的版本是`rdiv`。

参数：

* other: DataFrame, Series, 值序列或单个值。
* axis: 0或1。0为行，1为列。是否沿行或列对dataFrame与other之间的元素进行比较。
* fill_value: float或None。在计算前，填充缺失的值（NA值），以及数据对齐时需要填充的新元素。如果两个输入在同一位置的值都是缺失的，那么结果在该位置上也是缺失的。

返回值：

* delta.pandas.DataFrame

### truediv

```python
delta.pandas.DataFrame.truediv(self, other, axis=1, fill_value=None)
```

将DataFrame与other逐元素相除。等同于dataFrame / other，但是支持fill_value参数来填充输入中的NA值。交换操作顺序的版本是`rtruediv`。

参数：

* other: DataFrame, Series, 值序列或单个值。
* axis: 0或1。0为行，1为列。是否沿行或列对dataFrame与other之间的元素进行比较。
* fill_value: float或None。在计算前，填充缺失的值（NA值），以及数据对齐时需要填充的新元素。如果两个输入在同一位置的值都是缺失的，那么结果在该位置上也是缺失的。

返回值：

* delta.pandas.DataFrame

### floordiv

```python
delta.pandas.DataFrame.floordiv(self, other, axis=1, fill_value=None)
```

将DataFrame与other逐元素相除。等同于dataFrame // other，但是支持fill_value参数来填充输入中的NA值。交换操作顺序的版本是`rfloordiv`。

参数：

* other: DataFrame, Series, 值序列或单个值。
* axis: 0或1。0为行，1为列。是否沿行或列对dataFrame与other之间的元素进行比较。
* fill_value: float或None。在计算前，填充缺失的值（NA值），以及数据对齐时需要填充的新元素。如果两个输入在同一位置的值都是缺失的，那么结果在该位置上也是缺失的。

返回值：

* delta.pandas.DataFrame

### mod

```python
delta.pandas.DataFrame.mod(self, other, axis=1, fill_value=None)
```

将DataFrame与other逐元素取余。等同于dataFrame % other，但是支持fill_value参数来填充输入中的NA值。交换操作顺序的版本是`rmod`。

参数：

* other: DataFrame, Series, 值序列或单个值。
* axis: 0或1。0为行，1为列。是否沿行或列对dataFrame与other之间的元素进行比较。
* fill_value: float或None。在计算前，填充缺失的值（NA值），以及数据对齐时需要填充的新元素。如果两个输入在同一位置的值都是缺失的，那么结果在该位置上也是缺失的。

返回值：

* delta.pandas.DataFrame

### pow

```python
delta.pandas.DataFrame.pow(self, other, axis=1, fill_value=None)
```

计算DataFrame与other逐元素的幂。等同于dataFrame ** other，但是支持fill_value参数来填充输入中的NA值。交换操作顺序的版本是`rpow`。

参数：

* other: DataFrame, Series, 值序列或单个值。
* axis: 0或1。0为行，1为列。是否沿行或列对dataFrame与other之间的元素进行比较。
* fill_value: float或None。在计算前，填充缺失的值（NA值），以及数据对齐时需要填充的新元素。如果两个输入在同一位置的值都是缺失的，那么结果在该位置上也是缺失的。

返回值：

* delta.pandas.DataFrame

## Series API

### all

```python
delta.pandas.Series.all(self, axis=0, bool_only=False, skipna=True)
```

判断所有元素是否都为真。

参数：

* axis: 0。与DataFrame的api兼容。
* bool_only: bool类型。bool_only=True时，只包含元素为bool类型的值；否则包含所有的值。
* skipna: bool类型。如果整行、整列的值都是NA，并且skina=True，那么结果为True；如果skipna=False，那么NA被视作True，因为NA是非零值。

返回值：

* 单个值

### any

```python
delta.pandas.Series.any(self, axis=0, bool_only=False, skipna=True)
```

判断是否有任意一个元素为真。

参数：

* axis: 0。与DataFrame的api兼容。
* bool_only: bool类型。bool_only=True时，只包含元素为bool类型的值；否则包含所有的值。
* skipna: bool类型。如果整行、整列的值都是NA，并且skina=True，那么结果为True；如果skipna=False，那么NA被视作True，因为NA是非零值。

返回值：

* 单个值

### count

```python
delta.pandas.Series.count(self, axis=0, numeric_only=False)
```

计算非NA元素的数量。

参数：

* axis: 0。与DataFrame的api兼容。
* numeric_only: bool类型。numeric_only=True时，只统计float，int，bool类型的值。

返回值：

* 单个值

### sum

```python
delta.pandas.Series.sum(self, axis=0, skipna=True, numeric_only=False, min_count=0)
```

计算所有元素的和。

参数：

* axis: 0。与DataFrame的api兼容。
* skipna: bool类型。skipna=True时，计算时排除NA值；否则包含NA值。
* numeric_only: bool类型。numeric_only=True时，只统计float，int，bool类型的值。
* min_count: int。所需的合法值的最小数量。如果数量不足min_count，结果会为NA。

返回值：

* 单个值

### mean

```python
delta.pandas.Series.mean(self, axis=0, skipna=True, numeric_only=False)
```

计算所有元素的平均值。

参数：

* axis: 0。与DataFrame的api兼容。
* skipna: bool类型。skipna=True时，计算时排除NA值；否则包含NA值。
* numeric_only: bool类型。numeric_only=True时，只统计float，int，bool类型的值。

返回值：

* 单个值

### std

```python
delta.pandas.Series.std(self, axis=0, skipna=True, ddof=1, numeric_only=False)
```

计算所有元素的标准差。

参数：

* axis: 0。与DataFrame的api兼容。
* skipna: bool类型。skipna=True时，计算时排除NA值；否则包含NA值。
* ddof: int类型。自由度（Delta Degrees of Freedom）。计算时的除数是N-ddof，其中N是元素的数量。
* numeric_only: bool类型。numeric_only=True时，只统计float，int，bool类型的值。

返回值：

* 单个值

### var

```python
delta.pandas.Series.var(self, axis=0, skipna=True, ddof=1, numeric_only=False)
```

计算所有元素的方差。

参数：

* axis: 0。与DataFrame的api兼容。
* skipna: bool类型。skipna=True时，计算时排除NA值；否则包含NA值。
* ddof: int类型。自由度（Delta Degrees of Freedom）。计算时的除数是N-ddof，其中N是元素的数量。
* numeric_only: bool类型。numeric_only=True时，只统计float，int，bool类型的值。

返回值：

* 单个值

### sem

```python
delta.pandas.Series.sem(self, axis=0, skipna=True, ddof=1, numeric_only=False)
```

计算所有元素的标准误差(standard error of the mean, SEM)。

参数：

* axis: 0。与DataFrame的api兼容。
* skipna: bool类型。skipna=True时，计算时排除NA值；否则包含NA值。
* ddof: int类型。自由度（Delta Degrees of Freedom）。计算时的除数是N-ddof，其中N是元素的数量。
* numeric_only: bool类型。numeric_only=True时，只统计float，int，bool类型的值。

返回值：

* delta.pandas.Series

### add

```python
delta.pandas.Series.add(self, other, fill_value=None)
```

将Series与other逐元素相加。等同于Series + other，但是支持fill_value参数来填充输入中的NA值。交换操作顺序的版本是`radd`。

参数：

* other: Series, 值序列或单个值。
* fill_value: float或None。在计算前，填充缺失的值（NA值），以及数据对齐时需要填充的新元素。如果两个输入在同一位置的值都是缺失的，那么结果在该位置上也是缺失的。

返回值：

* delta.pandas.Series

### sub

```python
delta.pandas.Series.sub(self, other, fill_value=None)
```

将Series与other逐元素相减。等同于Series - other，但是支持fill_value参数来填充输入中的NA值。交换操作顺序的版本是`rsub`。

参数：

* other: Series, 值序列或单个值。
* fill_value: float或None。在计算前，填充缺失的值（NA值），以及数据对齐时需要填充的新元素。如果两个输入在同一位置的值都是缺失的，那么结果在该位置上也是缺失的。

返回值：

* delta.pandas.Series

### mul

```python
delta.pandas.Series.mul(self, other, fill_value=None)
```

将Series与other逐元素相乘。等同于Series * other，但是支持fill_value参数来填充输入中的NA值。交换操作顺序的版本是`rmul`。

参数：

* other: Series, 值序列或单个值。
* fill_value: float或None。在计算前，填充缺失的值（NA值），以及数据对齐时需要填充的新元素。如果两个输入在同一位置的值都是缺失的，那么结果在该位置上也是缺失的。

返回值：

* delta.pandas.Series

### div

```python
delta.pandas.Series.div(self, other, fill_value=None)
```

将Series与other逐元素相除。等同于Series / other，但是支持fill_value参数来填充输入中的NA值。交换操作顺序的版本是`rdiv`。

参数：

* other: Series, 值序列或单个值。
* fill_value: float或None。在计算前，填充缺失的值（NA值），以及数据对齐时需要填充的新元素。如果两个输入在同一位置的值都是缺失的，那么结果在该位置上也是缺失的。

返回值：

* delta.pandas.Series

### truediv

```python
delta.pandas.Series.truediv(self, other, fill_value=None)
```

将Series与other逐元素相除。等同于Series / other，但是支持fill_value参数来填充输入中的NA值。交换操作顺序的版本是`rtruediv`。

参数：

* other: Series, 值序列或单个值。
* fill_value: float或None。在计算前，填充缺失的值（NA值），以及数据对齐时需要填充的新元素。如果两个输入在同一位置的值都是缺失的，那么结果在该位置上也是缺失的。

返回值：

* delta.pandas.Series

### floordiv

```python
delta.pandas.Series.floordiv(self, other, fill_value=None)
```

将Series与other逐元素相除。等同于Series // other，但是支持fill_value参数来填充输入中的NA值。交换操作顺序的版本是`rfloordiv`。

参数：

* other: Series, Series, 值序列或单个值。
* fill_value: float或None。在计算前，填充缺失的值（NA值），以及数据对齐时需要填充的新元素。如果两个输入在同一位置的值都是缺失的，那么结果在该位置上也是缺失的。

返回值：

* delta.pandas.Series

### mod

```python
delta.pandas.Series.mod(self, other, fill_value=None)
```

将Series与other逐元素取余。等同于Series % other，但是支持fill_value参数来填充输入中的NA值。交换操作顺序的版本是`rmod`。

参数：

* other: Series, 值序列或单个值。
* fill_value: float或None。在计算前，填充缺失的值（NA值），以及数据对齐时需要填充的新元素。如果两个输入在同一位置的值都是缺失的，那么结果在该位置上也是缺失的。

返回值：

* delta.pandas.Series

### pow

```python
delta.pandas.Series.pow(self, other, fill_value=None)
```

计算Series与other逐元素的幂。等同于Series ** other，但是支持fill_value参数来填充输入中的NA值。交换操作顺序的版本是`rpow`。

参数：

* other: Series, 值序列或单个值。
* fill_value: float或None。在计算前，填充缺失的值（NA值），以及数据对齐时需要填充的新元素。如果两个输入在同一位置的值都是缺失的，那么结果在该位置上也是缺失的。

返回值：

* delta.pandas.Series
