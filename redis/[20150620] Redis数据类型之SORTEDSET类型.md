#**Redis数据类型之SORTEDSET类型**
>*Web程序猿博客：[http://blog.csdn.net/thinkercode](http://blog.csdn.net/thinkercode)*

####**sorted set类型-特点**
sorted set 是 set 的一个升级版本，它在 set 的基础上增加了一个顺序属性，这一属性在添加修改元素的时候可以指定，每次指定后，zset 会自动重新按新的值调整顺序。可以理解为有两列的 mysql 表，一列存 value，一列存顺序。操作中 key 理解为 zset 的名字。
和 set 一样 sorted set 也是 string 类型元素的集合，不同的是每个元素都会关联一个 double类型的 score。sorted set 的实现是 skip list 和 hash table 的混合体。
当元素被添加到集合中时，一个元素到 score 的映射被添加到 hash table 中，所以给定一个元素获取 score 的开销是 O(1),另一个 score 到元素的映射被添加到 skip list，并按照 score 排序，所以就可以有序的获取集合中的元素。添加，删除操作开销都是 O(log(N))和 skip list 的开销一致,redis 的 skip list 实现用的是双向链表,这样就可以逆序从尾部取元素。sorted set 最经常的使用方式应该是作为索引来使用.我们可以把要排序的字段作为 score 存储， 对象的 id当元素存储。下面是 sorted set 相关命令。

####**sorted set常见命令**
1. ZADD
	ZADD key score member [[score member] [score member] ...]
	将一个或多个 member 元素及其 score 值加入到有序集 key 当中。
	如果某个 member 已经是有序集的成员，那么更新这个 member 的 score 值，并通过重新插入这个 member元素，来保证该 member 在正确的位置上。
	score 值可以是整数值或双精度浮点数。
	如果 key 不存在，则创建一个空的有序集并执行ZADD 操作。
	当 key 存在但不是有序集类型时，返回一个错误。
	时间复杂度: O(M*log(N))，N 是有序集的基数，M 为成功添加的新成员的数量。
	返回值: 被成功添加的新成员的数量，不包括那些被更新的、已经存在的成员。
	```shell
	# 添加单个元素
	127.0.0.1:6379[15]> ZADD page_rank 10 google.com
	(integer) 1
	# 添加多个元素
	127.0.0.1:6379[15]> ZADD page_rank 9 baidu.com 8 bing.com
	(integer) 2
	127.0.0.1:6379[15]> ZRANGE page_rank 0 -1 WITHSCORES
	1) "bing.com"
	2) "8"
	3) "baidu.com"
	4) "9"
	5) "google.com"
	6) "10"
	# 添加已存在元素
	127.0.0.1:6379[15]> ZADD page_rank 10 google.com
	(integer) 0
	127.0.0.1:6379[15]> ZRANGE page_rank 0 -1 WITHSCORES 
	1) "bing.com"
	2) "8"
	3) "baidu.com"
	4) "9"
	5) "google.com"
	6) "10"
	# 添加已存在元素，但是改变 score 值
	127.0.0.1:6379[15]> ZADD page_rank 6 bing.com
	(integer) 0
	127.0.0.1:6379[15]> ZRANGE page_rank 0 -1 WITHSCORES
	1) "bing.com"
	2) "6"
	3) "baidu.com"
	4) "9"
	5) "google.com"
	6) "10"
	```

2. ZRANGE
	ZRANGE key start stop [WITHSCORES]
	返回有序集 key 中，指定区间内的成员。
	其中成员的位置按 score 值递增 (从小到大) 来排序。
	具有相同 score 值的成员按字典序 (lexicographical order ) 来排列。
	如果你需要成员按 score 值递减 (从大到小) 来排列，请使用ZREVRANGE 命令。
	下标参数 start 和 stop 都以 0 为底，也就是说，以 0 表示有序集第一个成员，以 1 表示有序集第二个成员，以此类推。
	你也可以使用负数下标，以 -1 表示最后一个成员，-2 表示倒数第二个成员，以此类推。超出范围的下标并不会引起错误。
	比如说，当 start 的值比有序集的最大下标还要大，或是 start > stop 时，ZRANGE 命令只是简单地返回一个空列表。
	另一方面，假如 stop 参数的值比有序集的最大下标还要大，那么 Redis 将 stop 当作最大下标来处理。
	可以通过使用 WITHSCORES 选项，来让成员和它的 score 值一并返回，返回列表以 value1,score1, ...,valueN,scoreN 的格式表示。
	客户端库可能会返回一些更复杂的数据类型，比如数组、元组等。
	时间复杂度: O(log(N)+M)，N 为有序集的基数，而 M 为结果集的基数。
	返回值: 指定区间内，带有 score 值 (可选) 的有序集成员的列表。
	```shell
	127.0.0.1:6379[15]> ZADD salary 3500 jack 5000 tom 10086 boss
	(integer) 3
	127.0.0.1:6379[15]> ZRANGE salary 1 2 WITHSCORES
	1) "tom"
	2) "5000"
	3) "boss"
	4) "10086"
	127.0.0.1:6379[15]> ZRANGE salary 0 200000 WITHSCORES 
	1) "jack"
	2) "3500"
	3) "tom"
	4) "5000"
	5) "boss"
	6) "10086"
	127.0.0.1:6379[15]> ZRANGE salary 0 -1 WITHSCORES 
	1) "jack"
	2) "3500"
	3) "tom"
	4) "5000"
	5) "boss"
	6) "10086"
	127.0.0.1:6379[15]> ZRANGE salary 200000 3000000 WITHSCORES
	(empty list or set)
	```
	
3. ZREM
	ZREM key member [member ...]
	移除有序集 key 中的一个或多个成员，不存在的成员将被忽略。
	当 key 存在但不是有序集类型时，返回一个错误。
	时间复杂度: O(M*log(N))，N 为有序集的基数，M 为被成功移除的成员的数量。
	返回值: 被成功移除的成员的数量，不包括被忽略的成员。
	```shell
	127.0.0.1:6379[15]> ZRANGE page_rank 0 -1 WITHSCORES
	1) "bing.com"
	2) "6"
	3) "baidu.com"
	4) "9"
	5) "google.com"
	6) "10"
	# 移除单个元素
	127.0.0.1:6379[15]> ZREM page_rank google.com
	(integer) 1
	127.0.0.1:6379[15]> ZRANGE page_rank 0 -1 WITHSCORES
	1) "bing.com"
	2) "6"
	3) "baidu.com"
	4) "9"
	# 移除多个元素
	127.0.0.1:6379[15]> ZREM page_rank baidu.com bing.com
	(integer) 2
	127.0.0.1:6379[15]> ZRANGE page_rank 0 -1 WITHSCORES
	(empty list or set)
	# 移除不存在的元素
	127.0.0.1:6379[15]> ZREM page_rank non-exists-element
	(integer) 0
	```

4. ZREMRANGEBYRANK
	ZREMRANGEBYRANK key start stop
	移除有序集 key 中，指定排名 (rank) 区间内的所有成员。
	区间分别以下标参数 start 和 stop 指出，包含 start 和 stop 在内。
	下标参数 start 和 stop 都以 0 为底，也就是说，以 0 表示有序集第一个成员，以 1 表示有序集第二个成员，以此类推。
	你也可以使用负数下标，以 -1 表示最后一个成员，-2 表示倒数第二个成员，以此类推。
	时间复杂度: O(log(N)+M)，N 为有序集的基数，而 M 为被移除成员的数量。
	返回值: 被移除成员的数量。
	```shell
	127.0.0.1:6379[15]> ZADD salary 2000 jack
	(integer) 1
	127.0.0.1:6379[15]> ZADD salary 5000 tom
	(integer) 1
	127.0.0.1:6379[15]> ZADD salary 3500 peter
	(integer) 1
	127.0.0.1:6379[15]> ZREMRANGEBYRANK salary 0 1 
	(integer) 2
	127.0.0.1:6379[15]> ZRANGE salary 0 -1 WITHSCORES
	1) "tom"
	2) "5000"
	```

5. ZREMRANGEBYSCORE
	ZREMRANGEBYSCORE key min max
	移除有序集 key 中，所有 score 值介于 min 和 max 之间 (包括等于 min 或 max ) 的成员。
	自版本 2.1.6 开始，score 值等于 min 或 max 的成员也可以不包括在内。
	时间复杂度: O(log(N)+M)，N 为有序集的基数，而 M 为被移除成员的数量。
	返回值: 被移除成员的数量。
	```shell
	127.0.0.1:6379[15]> ZADD salary 2000 tom 3500 peter 5000 jack 
	(integer) 3
	127.0.0.1:6379[15]> ZRANGE salary 0 -1 WITHSCORES
	1) "tom"
	2) "2000"
	3) "peter"
	4) "3500"
	5) "jack"
	6) "5000"
	127.0.0.1:6379[15]> ZREMRANGEBYSCORE salary 1500 3500
	(integer) 2
	127.0.0.1:6379[15]> ZRANGE salary 0 -1 WITHSCORES
	1) "jack"
	2) "5000"
	```
	
6. ZCOUNT
	ZCOUNT key min max
	返回有序集 key 中，score 值在 min 和 max 之间 (默认包括 score 值等于 min 或 max ) 的成员的数量。
	时间复杂度: O(log(N)+M)，N 为有序集的基数，M 为值在 min 和 max 之间的元素的数量。
	返回值: score 值在 min 和 max 之间的成员的数量。
	```shell
	127.0.0.1:6379[15]> ZADD salary 2000 tom 3500 peter 5000 jack 
	(integer) 2
	127.0.0.1:6379[15]> ZRANGE salary 0 -1 WITHSCORES
	1) "tom"
	2) "2000"
	3) "peter"
	4) "3500"
	5) "jack"
	6) "5000"
	127.0.0.1:6379[15]> ZCOUNT salary 2000 5000
	(integer) 3
	127.0.0.1:6379[15]> ZCOUNT salary 3000 5000
	(integer) 2
	```

7. ZRANK
	ZRANK key member
	返回有序集 key 中成员 member 的排名。其中有序集成员按 score 值递增 (从小到大) 顺序排列。
	排名以 0 为底，也就是说，score 值最小的成员排名为 0 。
	使用ZREVRANK 命令可以获得成员按 score 值递减 (从大到小) 排列的排名。
	时间复杂度: O(log(N))
	返回值:
	　　如果 member 是有序集 key 的成员，返回 member 的排名。
	　　如果 member 不是有序集 key 的成员，返回 nil 。
	```shell
	127.0.0.1:6379[15]> ZRANGE salary 0 -1 WITHSCORES
	1) "tom"
	2) "2000"
	3) "peter"
	4) "3500"
	5) "jack"
	6) "5000"
	127.0.0.1:6379[15]> ZRANK salary tom
	(integer) 0
	```

8. ZREVRANK
	ZREVRANK key member
	返回有序集 key 中成员 member 的排名。其中有序集成员按 score 值递减 (从大到小) 排序。
	排名以 0 为底，也就是说，score 值最大的成员排名为 0 。
	使用ZRANK 命令可以获得成员按 score 值递增 (从小到大) 排列的排名。
	时间复杂度: O(log(N))
	返回值:
	　　如果 member 是有序集 key 的成员，返回 member 的排名。
	　　如果 member 不是有序集 key 的成员，返回 nil 。
	```shell
	127.0.0.1:6379[15]>  ZRANGE salary 0 -1 WITHSCORES
	1) "jack"
	2) "3500"
	3) "tom"
	4) "5000"
	5) "boss"
	6) "10086"
	127.0.0.1:6379[15]> ZREVRANK salary tom
	(integer) 1
	127.0.0.1:6379[15]> ZREVRANK salary boss
	(integer) 0
	```
	
9. ZREVRANGE
	ZREVRANGE key start stop [WITHSCORES]
	返回有序集 key 中，指定区间内的成员。
	其中成员的位置按 score 值递减 (从大到小) 来排列。
	具有相同 score 值的成员按字典序的逆序 (reverse lexicographical order) 排列。
	除了成员按 score 值递减的次序排列这一点外，ZREVRANGE 命令的其他方面和ZRANGE 命令一样。
	```shell
	127.0.0.1:6379[15]> ZRANGE salary 0 -1 WITHSCORES 
	1) "tom"
	2) "2000"
	3) "peter"
	4) "3500"
	5) "jack"
	6) "5000"
	127.0.0.1:6379[15]> ZREVRANGE salary 0 -1 WITHSCORES
	1) "jack"
	2) "5000"
	3) "peter"
	4) "3500"
	5) "tom"
	6) "2000"
	```

10. ZRANGEBYSCORE
	ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]
	返回有序集 key 中，所有 score 值介于 min 和 max 之间 (包括等于 min 或 max ) 的成员。有序集成员按score 值递增 (从小到大) 次序排列。
	具有相同 score 值的成员按字典序 (lexicographical order) 来排列 (该属性是有序集提供的，不需要额外的计算)。
	可选的 LIMIT 参数指定返回结果的数量及区间 (就像 SQL 中的 SELECT LIMIT offset, count )，注意当offset 很大时，定位 offset 的操作可能需要遍历整个有序集，此过程最坏复杂度为 O(N) 时间。
	可选的 WITHSCORES 参数决定结果集是单单返回有序集的成员，还是将有序集成员及其 score 值一起返回。
	区间及无限,min 和 max 可以是 -inf 和 +inf ，这样一来，你就可以在不知道有序集的最低和最高 score 值的情况下，使用ZRANGEBYSCORE 这类命令。
	默认情况下，区间的取值使用闭区间 (小于等于或大于等于)，你也可以通过给参数前增加 ( 符号来使用可选的开区间 (小于或大于)。
	举个例子：
	ZRANGEBYSCORE zset (1 5
	返回所有符合条件 1 < score <= 5 的成员，而
	ZRANGEBYSCORE zset (5 (10
	则返回所有符合条件 5 < score < 10 的成员。
	时间复杂度: O(log(N)+M)，N 为有序集的基数，M 为被结果集的基数。
	返回值: 指定区间内，带有 score 值 (可选) 的有序集成员的列表。
	```shell
	127.0.0.1:6379[15]> ZADD salary 2500 jack 
	(integer) 1
	127.0.0.1:6379[15]> ZADD salary 5000 tom
	(integer) 1
	127.0.0.1:6379[15]> ZADD salary 12000 peter
	(integer) 1
	127.0.0.1:6379[15]> ZRANGEBYSCORE salary -inf +inf 
	1) "jack"
	2) "tom"
	3) "peter"
	127.0.0.1:6379[15]> ZRANGEBYSCORE salary -inf +inf WITHSCORES
	1) "jack"
	2) "2500"
	3) "tom"
	4) "5000"
	5) "peter"
	6) "12000"
	127.0.0.1:6379[15]> ZRANGEBYSCORE salary -inf 5000 WITHSCORES 
	1) "jack"
	2) "2500"
	3) "tom"
	4) "5000"
	127.0.0.1:6379[15]> ZRANGEBYSCORE salary (5000 400000 
	1) "peter"
	```

11. ZREVRANGEBYSCORE
	ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count]
	返回有序集 key 中，score 值介于 max 和 min 之间 (默认包括等于 max 或 min ) 的所有的成员。有序集成员按 score 值递减 (从大到小) 的次序排列。
	具有相同 score值的成员按字典序的逆序 (reverse lexicographical order ) 排列。
	除 了成员按score值递减的次序排列这一点外， ZREVRANGEBYSCORE命令的其他方面和ZRANGEBYSCORE 命令一样。
	时间复杂度: O(log(N)+M)，N 为有序集的基数，M 为结果集的基数。
	返回值: 指定区间内，带有 score 值 (可选) 的有序集成员的列表。
	```shell
	127.0.0.1:6379[15]> ZADD salary 10086 jack 5000 tom 7000 peter 3500 joe
	(integer) 4
	127.0.0.1:6379[15]> ZREVRANGEBYSCORE salary +inf -inf
	1) "jack"
	2) "peter"
	3) "tom"
	4) "joe"
	127.0.0.1:6379[15]> ZREVRANGEBYSCORE salary +inf -inf
	1) "jack"
	2) "peter"
	3) "tom"
	4) "joe"
	127.0.0.1:6379[15]> ZREVRANGEBYSCORE salary 10000 2000
	1) "peter"
	2) "tom"
	3) "joe"
	```

12. ZCARD
	ZCARD key
	返回有序集 key 的基数。
	时间复杂度: O(1)
	返回值:
	　　当 key 存在且是有序集类型时，返回有序集的基数。
	　　当 key 不存在时，返回 0 。
	```shell
	127.0.0.1:6379[15]> ZADD salary 2000 tom
	(integer) 1
	127.0.0.1:6379[15]> ZCARD salary
	(integer) 1
	127.0.0.1:6379[15]> ZADD salary 5000 jack 
	(integer) 1
	127.0.0.1:6379[15]> ZCARD salary
	(integer) 2
	127.0.0.1:6379[15]> EXISTS non_exists_key 
	(integer) 0
	127.0.0.1:6379[15]> ZCARD non_exists_key
	(integer) 0
	```
	
13. ZSCORE
	ZSCORE key member
	返回有序集 key 中，成员 member 的 score 值。
	如果 member 元素不是有序集 key 的成员，或 key 不存在，返回 nil 。
	时间复杂度: O(1)
	返回值: member 成员的 score 值，以字符串形式表示。
	```shell
	127.0.0.1:6379[15]> ZADD salary 2000 tom 3500 peter 5000 jack
	(integer) 3
	127.0.0.1:6379[15]>  ZSCORE salary peter
	"3500"
	```

14. ZINCRBY
	ZINCRBY key increment member
	为有序集 key 的成员 member 的 score 值加上增量 increment 。
	可以通过传递一个负数值 increment ，让 score 减去相应的值，比如 ZINCRBY key -5 member，就是让member的 score 值减去 5 。
	当 key 不存在，或 member 不是 key 的成员时，ZINCRBY key increment member 等同于 ZADD key increment member 。
	当 key 不是有序集类型时，返回一个错误。
	score 值可以是整数值或双精度浮点数。
	时间复杂度: O(log(N))
	返回值: member 成员的新 score 值，以字符串形式表示。
	```shell
	127.0.0.1:6379[15]> ZADD salary 2000 tom 3500 peter 5000 jack
	(integer) 3
	127.0.0.1:6379[15]>  ZSCORE salary tom
	"2000"
	127.0.0.1:6379[15]>  ZINCRBY salary 2000 tom
	"4000"
	127.0.0.1:6379[15]>  ZSCORE salary tom
	"4000"
	```
	
15. ZUNIONSTORE
	ZUNIONSTORE destination numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM|MIN|MAX]
	计算给定的一个或多个有序集的并集，其中给定 key的数量必须以 numkeys 参数指定，并将该并集 (结果集) 储存到 destination 。
	默认情况下，结果集中某个成员的 score 值是所有给定集下该成员 score 值之 和 。
	使用 WEIGHTS 选项，你可以为每 给定有序集分别指定一个乘法因子 (multiplication factor)，每个给定有序集的所有成员的 score值在传递给聚合函数 (aggregation function) 之前都要先乘以该有序集的因子。如果没有指定 WEIGHTS 选项，乘法因子默认设置为 1 。
	使用 AGGREGATE 选项，你可以指定并集的结果集的聚合方式。默认使用的参数 SUM ，可以将所有集合中某个成员的 score值之和作为结果集中该成员的 score值；使用参数 MIN ，可以将所有集合中某个成员的最小score值作为结果集中该成员的 score值；而参数 MAX则是将所有集合中某个成员的 最大 score值作为结果集中该成员的 score 值。
	时间复杂度: O(N)+O(M log(M))，N 为给定有序集基数的总和，M 为结果集的基数。
	返回值: 保存到 destination 的结果集的基数。
	```shell
	127.0.0.1:6379[15]> ZADD manager 2000 herry 3500 mary 4000 bob 
	(integer) 3
	127.0.0.1:6379[15]> ZADD programmer 2000 peter 3500 jack 5000 tom
	(integer) 3
	127.0.0.1:6379[15]> ZUNIONSTORE salary 2 programmer manager WEIGHTS 1 3 
	(integer) 6
	127.0.0.1:6379[15]> ZRANGE salary 0 -1 WITHSCORES
	 1) "peter"
	 2) "2000"
	 3) "jack"
	 4) "3500"
	 5) "tom"
	 6) "5000"
	 7) "herry"
	 8) "6000"
	 9) "mary"
	10) "10500"
	11) "bob"
	12) "12000"
	```

16. ZINTERSTORE
	ZINTERSTORE destination numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGRE-GATE SUM|MIN|MAX]
	计算给定的一个或多个有序集的交集，其中给定 key 的数量必须以 numkeys 参数指定，并将该交集 (结果集) 储存到 destination 。
	默认情况下，结果集中某个成员的 score 值是所有给定集下该成员 score 值之和.关于 WEIGHTS 和 AGGREGATE 选项的描述，参见ZUNIONSTORE 命令。
	时间复杂度: O(N*K)+O(M*log(M))，N 为给定 key 中基数最小的有序集，K 为给定有序集的数量，M 为结果集的基数。
	返回值: 保存到 destination 的结果集的基数。
	```shell
	127.0.0.1:6379[15]>  ZADD mid_test 70 "Li Lei"
	(integer) 1
	127.0.0.1:6379[15]>  ZADD mid_test 70 "Han Meimei"
	(integer) 1
	127.0.0.1:6379[15]>  ZADD mid_test 99.5 "Tom"
	(integer) 1
	127.0.0.1:6379[15]>  ZADD fin_test 88 "Li Lei"
	(integer) 1
	127.0.0.1:6379[15]>  ZADD fin_test 75 "Han Meimei"
	(integer) 1
	127.0.0.1:6379[15]>  ZADD fin_test 99.5 "Tom"
	(integer) 1
	127.0.0.1:6379[15]> ZINTERSTORE sum_point 2 mid_test fin_test
	(integer) 3
	127.0.0.1:6379[15]> ZRANGE sum_point 0 -1 WITHSCORES 
	1) "Han Meimei"
	2) "145"
	3) "Li Lei"
	4) "158"
	5) "Tom"
	6) "199"
	127.0.0.1:6379[15]> ZINTERSTORE sum_point 2 mid_test fin_test  WEIGHTS 1 2
	(integer) 3
	127.0.0.1:6379[15]> ZRANGE sum_point 0 -1 WITHSCORES
	1) "Han Meimei"
	2) "220"
	3) "Li Lei"
	4) "246"
	5) "Tom"
	6) "298.5"
	127.0.0.1:6379[15]> ZINTERSTORE sum_point 2 mid_test fin_test  AGGREGATE min
	(integer) 3
	127.0.0.1:6379[15]> ZRANGE sum_point 0 -1 WITHSCORES
	1) "Han Meimei"
	2) "70"
	3) "Li Lei"
	4) "70"
	5) "Tom"
	6) "99.5"
	```