## pandas

### pivot_table（透视表）

```python
pandas.pivot_table(data, 
                   values=None, 
                   index=None, 
                   columns=None,
                   aggfunc='mean', 
                   fill_value=None, 
                   margins=False, 
                   dropna=True, 
                   margins_name='All')
```

> pivot_table有四个最重要的参数*index、values、columns、aggfunc*

- index参数用法

  > **Index就是层次字段，要通过透视表获取什么信息就按照相应的顺序设置字段**

  ```python
  pd.pivot_table(df,index=[u'对手'])
  ```

  根据对手字段进行索引

  ```python
  pd.pivot_table(df,index=[u'对手',u'主客场'])
  ```

  还可以有两个索引值

  

- values参数用法

  > **而Values可以对需要的计算数据进行筛选**

  ```python	
  pd.pivot_table(df,index=[u'主客场',u'胜负'],values=[u'得分',u'助攻',u'篮板'])
  ```

  values设置几个字段就筛选出几个字段

  

- columns参数用法

  > **Columns类似Index可以设置列层次字段，它不是一个必要参数，作为一种分割数据的可选方式。**

  ```python
  pd.pivot_table(df,index=[u'主客场'],columns=[u'对手'],values=[u'得分'],aggfunc=[np.sum],fill_value=0,margins=1)
  ```

  

- aggfunc参数用法

  > **aggfunc参数可以设置我们对数据聚合时进行的函数操作。**

  ```python
  pd.pivot_table(df,index=[u'主客场',u'胜负'],values=[u'得分',u'助攻',u'篮板'],aggfunc=[np.sum,np.mean])
  ```

  