### TIPS

> ##### ON DUPLICATE KEY UPDATE使用：
>
> **作用：**向数据库以相同unique或者primary key插入一条记录，若已经存在该key则更新这条记录，否则则新增一条记录；
>
> **示例：**
>
> > ```mysql
> > INSERT INTO TABLE_NAME('id', 'column2') VALUES('', '') ON DUPLICATE KEY UPDATE id = '111'
> > ```
>

> ##### MySQL字符串函数：
>
> - ##### ASCII(str)：
>
> > **作用：**返回第一个字符串的ASCII码值；如果字符串为空则返回0；
> >
> > **示例：**
> >
> > > `````mysql
> > > SELECT ASCII('2')
> > > `````
>
> - ##### ORD(str)：
>
> > **作用：**如果字符串str句首是单字节返回与ASCII()函数返回的相同值。如果是一个多字节字符,以格式返回((first byte ASCII code)*256+(second byte ASCII code))[*256+third byte ASCII code...]  
> >
> > **示例：**
> >
> > > ``````mysql
> > > SELECT ORD('2')
> > > ``````
>
