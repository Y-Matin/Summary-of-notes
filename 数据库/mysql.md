### 
- MySQL InnoDB 引擎下，delete/update操作where后面的条件如果没有走索引，会锁表


一、区别1：取结果的并集

1、union: 对两个结果集进行并集操作, 不包括重复行,相当于distinct, 同时进行默认规则的排序;

2、union all: 对两个结果集进行并集操作, 包括重复行, 即所有的结果全部显示, 不管是不是重复;

二、区别2：获取结果后的操作

1、union: 会对获取的结果进行排序操作

2、union all: 不会对获取的结果进行排序操作


### 报错
- Deadlock found when trying to get lock; try restarting transaction