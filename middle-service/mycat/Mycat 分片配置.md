## Mycat 分片配置 
mycat 将表分为两种概念：对于数据量小且不需要做数据切分的表成为非分片表；数据量大到单库性能、容量不足以支撑，数据需要通过水平切分均匀分布到不同的数据库表称之为分片表
### rule.xml
rule.xml 为分片规则配置文件， schema.xml 中 table 标签中的 rule 属性的值需要在 rule.xml 中配置
##### function 标签
```
<function name="mod-long" class="io.mycat.route.function.PartitionByMod">
	<property name="count">3</property>
</function>
```
- **name** 指定算法的名称，在该文件中唯一
- **class** 属性对应具体的分片算法，指定具体的算法类
- **property** 根据算法要求指定
##### tableRule 标签
```
<tableRule name="my-mod-long">
	<rule>
		<columns>id</columns>
		<algorithm>mod-long</algorithm>
	</rule>
</tableRule>
```
- **name** 指定分片唯一算法名称
- **rule** 分片算法具体内容
- **columns** 对应表中用于分片的列名
- **algorithm** function 中定义的算法名称
### 取模分片
```
<tableRule name="my-mod-long">
	<rule>
		<columns>id</columns>
		<algorithm>mod-long</algorithm>
	</rule>
</tableRule>
<function name="mod-long" class="io.mycat.route.function.PartitionByMod">
	<property name="count">3</property>
</function>
```
概算法根据 ID 进行十进制求模计算，相比固定的分片 hash，这种分片算法在批量插入会增加事务一致性的难度
### 枚举分片
通过在配置文件中配置可能的枚举ID，指定数据分布到不同的物理节点上，本规则适合按照省份或县区来拆分数据类业务
```
<tableRule name="sharding-byintfile">
	<rule>
		<columns>sharding_id</columns>
		<algorithm>hash-init</algorithm>
	</rule>
</tableRule>
<function name="hash-init" class="io.mycat.route.function.PartitionByFileMap">
	<property name="mapFile">partion-hash-init.txt</property>
	<property name="type">0</property>
	<property name="defaultNode">0</property>
</function>
```
其中 partition-hash-init.txt 内容：
```
10000=0
10010=1
DEFAULT_NODE=1
```
- **type** 默认值为 0，0 表示 Integer，非 0 表示 String
- **defaultNode** 小于 0 表示不设置默认节点，大于等于 0 表示设置默认节点。默认节点的作用：枚举分片时，如果有不认识的枚举值就路由到默认节点。如果不配置默认节点，遇到无法识别的枚举值，就会报错。
### 范围分片
```
<tableRule name="auto-sharding-long">
	<rule>
		<columns>id</columns>
		<algorithm>range-long</algorithm>
	</rule>
</tableRule>
<function name="range-long" class="io.mycat.route.function.AutoPartitionByLong">
	<property name="mapFile">auto-partition-long.txt</property>
	<property name="defaultNode">0</property>
</function>
```
auto-partition-long.txt 配置如下：
```
# range start-end，data node index
0-10000=0
10001-20000=1
20000-60000=2
```
### 范围求模算法
先范围分片，然后组内求模。范围求模可以保证组内数据分布比较均匀，避免热点数据问题；范围分片在水平扩展时，原有数据不需要迁移。
```
<tableRule name="auto-sharding-rang-mod">
	<rule>
		<columns>id</columns>
		<algorithm>range-mod</algorithm>
	</rule>
</tableRule>
<function name="range-mod" class="io.mycat.route.function.PartitionByRangeMod">
	<property name="mapFile">partition-range-mod.txt</property>
	<property name="defaultNode">0</property>
</function>
```
partiton-range-mod.txt 配置如下，等号前面的范围代表一个分片组，等号后面的数字代表该分片组所拥有的分片数量
```
0-200M=5 //5个分片节点
200M1-400M=1
400M1-600M=3
```
### 固定分片 hash 算法
```
<tableRule name="sharding-hash">
	<rule>
		<columns>userId</columns>
		<algorithm>sharding-hash</algorithm>
	</rule>
</tableRule>
<function name="sharding-hash" class="io.mycat.route.function.PartitionByLong">
	<property name="partitionCount">2,1</property>
	<property name="partitionLength">256,512</property>
</function>
```
- **partitionCount** 分片个数列表
- **partitionLength** 分片范围列表，分区长度默认最大为 2^n = 1024，最大支持 1024 个分区

count 和 length 两个数组的长度必须一致。1024 = SUM((count[i]*length[i]))，count 和 length两个向量的点击恒等于 1024。
### 取模范围算法
先取模，然后范围分片
```
<tableRule name="sharding-by-pattern">
	<rule>
		<columns>userId</columns>
		<algorithm>sharding-by-pattern</algorithm>
	</rule>
</tableRule>
<function name="sharding-by-pattern" class="io.mycat.route.function.PartitionByPattern">
	<property name="patternValue">256</property>
	<property name="defaultNode">2</property>
	<property name="mapFile">partition-pattern.txt</property>
</function>
```
partition-pattern.txt 配置，dataNode 是默认节点，如果采用默认配置则不进行求模运算，如果 id 不是数据则会分配在默认节点上
```
1-32=0
33-64=1
65-96=2
97-128=3
```
- **patternValue** 求模基数
### 字符串 hash 求模范围算法
与上相同，该算法支持述职、符号、字母取模
```
<tableRule name="sharding-by-prefixpattern">
	<rule>
		<columns>userId</columns>
		<algorithm>sharding-by-prefixpattern</algorithm>
	</rule>
</tableRule>
<function name="sharding-by-prefixpattern" class="io.mycat.route.function.PartitionByPrefixPattern">
	<property name="patternValue">256</property>
	<property name="prefixLength">5</property>
	<property name="defaultNode">2</property>
	<property name="mapFile">partition-prefixpattern.txt</property>
</function>
```
partition-prefixpattern.txt，截取长度为 prefixLength 的子串，再对字符串中每个字符的 ASCII 码进行求和得出 sum 值，最后对 sum 进行求模运算 (sum % patternValue)，可以算出分片数
```
1-4=0
5-8=1
9-12=2
13-16=3
```
### 应用指定的算法
```
<tableRule name="sharding-by-substring">
	<rule>
		<columns>userId</columns>
		<algorithm>sharding-by-substring</algorithm>
	</rule>
</tableRule>
<function name="sharding-by-substring" class="io.mycat.route.function.PartitionDirectBySubString">
	<property name="patternCount">8</property>
	<property name="size">2</property>
	<property name="defaultPattern">0</property>
</function>
```
根据字符串子串(必须是数字)计算分区号，例如 id = 05-0001，其中 id 是从 startIndex = 0 开始，截取长度为 2 位，即分区号为 05，分配到默认分区。
### 字符串 hash 解析算法
```
<tableRule name="sharding-by-stringhash">
	<rule>
		<columns>userId</columns>
		<algorithm>sharding-by-stringhash</algorithm>
	</rule>
</tableRule>
<function name="sharding-by-stringhash" class="io.mycat.route.function.PartitionByString">
	<property name="length">512</property>
	<property name="count">2</property>
	<property name="hashSlice">0:2</property>
</function>
```
- **length** 字符串 hash 的求模基数
- **count** 分区数
- **hashSlice** 预算位，根据子字符串中的 int 值进行 hash 运算
### 一致性 hash 算法
```
<tableRule name="sharding-by-murmur">
	<rule>
		<columns>userId</columns>
		<algorithm>sharding-by-murmur</algorithm>
	</rule>
</tableRule>
<function name="sharding-by-murmur" class="io.mycat.route.function.PartitionByMurMueHash">
	<property name="seed">0</property>
	<property name="count">2</property>
	<property name="virtualBucketTimes">160</property>
	<property name="weightMapFile">weightMapFile</property>
	<property name="bucketMapPath">/etc/mycat/bucketMapPath</property>
</function>
```
- **count** 要分片的数据库节点数量
- **virtualBucketTimes** 一个实际的数据库节点被映射后的虚拟节点，默认是 160 倍，即虚拟节点是物理节点的 160 倍
- **weightMapFile** 节点的权重，没有指定权重的节点默认是 1，以 properties 文件的格式编辑，以从 0 开始到 count - 1 的整数值也就是节点索引 key，以节点权重值为值。所有权重必须是正整数，否则以 1 代替
- **bucketMapPath** 测试时观察各物理节点与虚拟节点的分布情况。如果指定该属性，则会把虚拟节点的 murmur hash 值与物理节点的映射输出到这个文件，没有默认值，不配置则不输出。
### 按日期(天)分片算法
```
<tableRule name="sharding-by-date">
	<rule>
		<columns>create_time</columns>
		<algorithm>sharding-by-date</algorithm>
	</rule>
</tableRule>
<function name="sharding-by-date" class="io.mycat.route.function.PartitionByDate">
	<property name="dateFormat">yyyy-MM-dd</property>
	<property name="sBeginDate">2019-08-08</property>
	<property name="sEndDate">2019-08-09</property>
	<property name="sPartionDay">10</property>
</function>
```
- **sPartitionDay** 分区天数，默认从开始日期算起，每隔 10 天一个分区
- **sEndDate** 如果配置该属性，则数据达到这个日期的分片后会重复从开始分片插入
### 单月小时算法
```
<tableRule name="sharding-by-hour">
	<rule>
		<columns>create_time</columns>
		<algorithm>sharding-by-hour</algorithm>
	</rule>
</tableRule>
<function name="sharding-by-hour" class="io.mycat.route.function.LatestMonthPartition">
	<property name="sliptOneDay">24</property>
</function>
```
- **columns** 为拆分字段，字符串类型(yyyyMMddHH)
- **sliptOneDay** 为一天切分的分片数
### 自然月分片算法
```
<tableRule name="sharding-by-month">
	<rule>
		<columns>create_time</columns>
		<algorithm>sharding-by-month</algorithm>
	</rule>
</tableRule>
<function name="sharding-by-month" class="io.mycat.route.function.PartitionByMonth">
	<property name="dateFormat">yyyy-MM-dd</property>
	<property name="sBeginDate">2019-08-08</property>
</function>
```
### 日期范围 hash 算法
现根据日期分组，再根据时间 hash 使得短期内数据分布更加均匀，可在一定程度上避免范围分片的数据热点问题
```
<tableRule name="range-date-hash">
	<rule>
		<columns>create_time</columns>
		<algorithm>range-date-hash</algorithm>
	</rule>
</tableRule>
<function name="range-date-hash" class="io.mycat.route.function.PartitionByRangeDateHash">
	<property name="dateFormat">yyyy-MM-dd</property>
	<property name="sBeginDate">2019-08-08</property>
	<property name="sPartionDay">12</property>
	<property name="groupPartionSize">6</property>
</function>
```
- **sPartionDay** 代表多少天一组
- **groupPartionSize** 每组的分片数量

<div style="text-align:center;margin-top:50px">
    <img src="https://note.youdao.com/yws/api/personal/file/C2C6FCFDC10942B6A3532E6F0928E455?method=download&shareKey=c554dacfc5193c29d4b35682aa1226d9" />
</div>