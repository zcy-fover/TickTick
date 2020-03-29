## MySQL 数据类型

### 选择最优的数据类型

**更小的通常更好：**选择满足数据范围的最小数据类型，因为他们可以占用更少的磁盘、内存和 CPU 缓存，处理时需要的 CPU 周期也更少；

**简单就好：**简单的数据类型处理时需要更少的 CPU 周期；例如整型比字符操作代价更低；

**尽量避免 NULL 值：**可为 NULL 的列使得索引、索引统计和值比较都更复杂；

#### 整数类型

整数和实数可以用来存储整数，MySQL 有 `TINTINT(8)`、`SMALLINT(16)`、`MEDIUMINT(24)`、`INT(32)`、`BIGINT(64)`几种类型(数字表示 N 位存储空间)，值范围是 $-2^(N-1)~2^(N-1)-1$。

整数类型也可以选用 ` UNSIGNED` 无符号数，上述所有类型正数部分扩大为原来的 2 倍。

MySQL 可以为整数类型指定宽度，这不会影响实际的数据存储。

#### 实数类型

带有小数的数，`FLOAT(32)`和`DOUBLE(64)`类型使用标准的浮点数进行近似计算，`DECIMAL`用于存储精确的小数。`DECIMAL`只是一种存储格式，在实际计算中转换为 `DOUBLE`类型。MySQL 使用 `DOUBLE`作为内部的浮点数计算类型。

#### 字符串类型

`VARCHAR`、`CHAR`是两种主要的字符串类型。

##### VARCHAR

可存储变长字符串，必定长的类型更节省空间。如果 MySQL 表使用了 `ROW_FORMAT=FIXED` 创建的话，每一行会使用定长存储，这样比较浪费空间。

`VARCHAR` 需要使用额外 1 或 2 个字节记录字符串长度，最大长度小于等于 255 时，使用一个字节记录，否则使用 2 字节。`VARCHAR` 节省了存储空间能提高性能，但是这样也导致在 `UPDATE` 需要更多的额外工作。例如一个行占用的空间边长，但是当前页内空间不足，MyISAM 会将行拆成不同片段存储，InnoDB 需要分裂页来存储。

适合使用 `VARCHAR` 的情况：字符串列最大长度比平均长度大很多；列的更新较少，碎片较少；使用了像 `UTF-8` 这样复杂的字符集，每个字符使用不同的字节数存储。

##### CHAR

定长存储，`CHAR` 比 `VARCHAR` 产生更少的碎片。对于单字节的字符串更适合用 `CHAR` 用 `VARCHAR` 会占用两个字节；`CHAR` 在存储字符串时会将字符串末尾的空格去掉。

`BINARY` 和 `VARBINARY` 与上面的类似，他们存储的是二进制字符串，二进制字符串与常规字符串类似，但是存储的是字节码而不是字符，填充的时候是 `\0` (0字节)，而不是空格，检索时也不会去掉填充。

#### BLOB 和 TEXT 类型

更大的字符串数据类型，分别采用二进制和字符方式存储；两者分别对应有 `TINYTEXT`、`SMALLTEXT`、`TEXT`、`MEDIUMTEXT`、`LOGTEXT`；`TINYBLOB`、`SMALLBLOB`、`BLOB`、`MEDIUMBLOB`、`LOGBLOB`。`BLOB` 存储二进制数据，没有字符集和排序规则， `TEXT` 存储字符数据有字符集和排序规则。MySQL 在对这两个数据类型排序的时候不是针对整个字符串排序，而是没个列的最前 `max_sort_length` 字节做排序，或者使用 `ORDER BY SUSTRING(column, length)`。

#### ENUM 类型

MySQL 会将每个值在列表中的位置保存为整数，并且在 .frm 文件中保存 `数字-字符串` 的映射关系表。

```mysql
create table enum_test(ee ENUM('fish', 'apple', 'dog') not null);
insert into enum_test(ee) values('fish', 'dog', 'apple');
select ee+0 from enum_test; # 结果： 1 3 2
select ee from enum_test order by ee; # 结果：fish  apple  dog
```

可以看到在列的存储中，`enum` 存储的是对应位置的整数而不是具体枚举值；在排序中也是按照整数输出不是具体的字符串插入顺序。枚举的字符串列表是建表的时候初始化的，所以要修改的时候需要 `ALTER TABLE`。

#### 日期和时间类型

##### DATETIME

从 1001 ~ 9999 年，精度为秒。与时区无关，占用 8 个字节存储空间。

##### TIMESTAMP

保存了从 `1970-01-01 00:00:00` 以来的秒数，只使用 4 个字节存储空间。MySQL 提供 `FROM_UNIXTIME` 将时间戳转换为日期， `UNIX_TIMESTAMP`  将日期转换为时间戳。

`TIMESTAMP` 比 `DATETIME` 的空间效率更高。如果需要存储微秒级别的时间戳，可以使用 `BIGINT` 或者 `DOUBLE` 存储秒之后的小数部分，也可以使用 `MariaDB`。

#### 位数据类型

##### BIT

`BIT` 列最大长度是 64 位，MySQL 把 `BIT` 当作字符串类型，存储的是包含二进制 0 1 的字符串，而不是 ASCII 码值。检索 `BIT` 列时，例如存储的是 `b"00110011"`，在字符串环境中得到的是 ASCII 码值为 51 的字符  "9"，如果是在数字环境中则得到的是数字 51。

```mysql
create table bit_test(bb bit(8));
insert into bit_test(bb) values(b'00110011');
select bb, bb+0 from bit_test; # 输出： 3   51
```

##### SET

`SET` 数据类型可以利用 `FIND_IN_SET`、`FIELD` 方便查询，但是 `SET` 集合的修改需要使用 `ALTER TABLE` 进行修改；一般情况 `SET` 列上也无法使用索引检索。