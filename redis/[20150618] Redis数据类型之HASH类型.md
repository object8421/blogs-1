#**Redis数据类型之HASH类型**
>*Web程序猿博客：[http://blog.csdn.net/thinkercode](http://blog.csdn.net/thinkercode)*

####**HASH类型-特点**
Redis hash 是一个 string 类型的 field 和 value 的映射表.它的添加、 删除操作都是 O(1) （平均） 。
hash 特别适合用于存储对象。 相较于将对象的每个字段存成单个 string 类型。 将一个对象存储在 hash 类型中会占用更少的内存，并且可以更方便的存取整个对象。省内存的原因是新建一个 hash 对象时开始是用 zipmap（又称为 small hash）来存储的。这个 zipmap 其实并不是 hash table，但是 zipmap 相比正常的 hash 实现可以节省不少 hash 本身需要的一些元数据存储开销。尽管 zipmap 的添加，删除，查找都是 O(n)，但是由于一般对象的 field 数量都不太多。所以使用 zipmap 也是很快的,也就是说添加删除平均还是 O(1)。如果 field 或者 value的大小超出一定限制后， Redis 会在内部自动将 zipmap 替换成正常的 hash 实现. 这个限制可以在配置文件中指定hash-max-zipmap-entries 64 #配置字段最多 64 个hash-max-zipmap-value 512 #配置 value 最大为 512 字节

####**HASH常见命令**
1. HSET
	HSET key field value
	将哈希表 key 中的域 field 的值设为 value 。
	如果 key 不存在，一个新的哈希表被创建并进行HSET 操作。
	如果域 field 已经存在于哈希表中，旧值将被覆盖。
	时间复杂度： O(1)
	返回值：
	　　如果 field 是哈希表中的一个新建域，并且值设置成功，返回 1 。
	　　如果哈希表中域 field 已经存在且旧值已被新值覆盖，返回 0 。
	```shell
	127.0.0.1:6379[15]> HSET website google "www.g.cn"
	(integer) 1
	127.0.0.1:6379[15]>  HSET website google "www.google.com"
	(integer) 0
	```

2. HSETNX
	HSETNX key field value
	将哈希表 key 中的域 field 的值设置为 value ，当且仅当域 field 不存在。若域 field 已经存在，该操作无效。
	如果 key 不存在，一个新哈希表被创建并执行HSETNX 命令。
	时间复杂度： O(1)
	返回值：
	　　设置成功，返回 1 。
	　　如果给定域已经存在且没有操作被执行，返回 0 。
	```shell
	127.0.0.1:6379[15]> HSETNX nosql key-value-store redis
	(integer) 1
	127.0.0.1:6379[15]>  HSETNX nosql key-value-store Memcached
	(integer) 0
	127.0.0.1:6379[15]> HGET nosql key-value-store
	"redis"
	```

3. HMSET
	HMSET key field value [field value ...]
	同时将多个 field-value (域 -值) 对设置到哈希表 key 中。此命令会覆盖哈希表中已存在的域。
	如果 key 不存在，一个空哈希表被创建并执行HMSET 操作。
	时间复杂度： O(N)，N 为 field-value 对的数量。
	返回值：
	　　如果命令执行成功，返回 OK 。
	　　当 key 不是哈希表 (hash) 类型时，返回一个错误。
	```shell
	127.0.0.1:6379[15]> HMSET website google www.google.com yahoo www.yahoo.com
	OK
	127.0.0.1:6379[15]> HGET website google
	"www.google.com"
	127.0.0.1:6379[15]> HGET website yahoo
	"www.yahoo.com"
	```
	
4. HGET
	HGET key field
	返回哈希表 key 中给定域 field 的值。
	时间复杂度： O(1)
	返回值：
	　　给定域的值。
	　　当给定域不存在或是给定 key 不存在时，返回 nil 。
	```shell
	127.0.0.1:6379[15]> HSET site redis redis.com
	(integer) 1
	127.0.0.1:6379[15]> HGET site redis
	"redis.com"
	127.0.0.1:6379[15]> HGET site mysql
	(nil)
	```

5. HMGET
	返回哈希表 key 中，一个或多个给定域的值。
	如果给定的域不存在于哈希表，那么返回一个 nil 值。
	因为不存在的 key 被当作一个空哈希表来处理，所以对一个不存在的 key 进行HMGET 操作将返回一个只	带有 nil 值的表。
	时间复杂度： O(N)，N 为给定域的数量。
	返回值： 一个包含多个给定域的关联值的表，表值的排列顺序和给定域参数的请求顺序一样。
	```shell
	127.0.0.1:6379[15]>  HMSET pet dog "doudou" cat "nounou"
	OK
	127.0.0.1:6379[15]>  HMGET pet dog cat fake_pet
	1) "doudou"
	2) "nounou"
	3) (nil)
	```

6. HEXISTS
	HEXISTS key field
	查看哈希表 key 中，给定域 field 是否存在。
	时间复杂度： O(1)
	返回值：
	　　如果哈希表含有给定域，返回 1 。
	　　如果哈希表不含有给定域，或 key 不存在，返回 0 。
	```shell
	127.0.0.1:6379[15]> HEXISTS phone myphone
	(integer) 0
	127.0.0.1:6379[15]> HSET phone myphone nokia-1110
	(integer) 1
	127.0.0.1:6379[15]> HEXISTS phone myphone
	(integer) 1
	```

7. HDEL
	HDEL key field [field ...]
	删除哈希表 key 中的一个或多个指定域，不存在的域将被忽略。
	时间复杂度: O(N)，N 为要删除的域的数量。
	返回值: 被成功移除的域的数量，不包括被忽略的域。
	```shell
	127.0.0.1:6379[15]> HMSET key f1 "v1" f2 "v2" f3 "v3" f4 "v4"
	OK
	127.0.0.1:6379[15]> HGETALL key
	1) "f1"
	2) "v1"
	3) "f2"
	4) "v2"
	5) "f3"
	6) "v3"
	7) "f4"
	8) "v4"
	127.0.0.1:6379[15]> HDEL key f1
	(integer) 1
	127.0.0.1:6379[15]> HDEL key not-field
	(integer) 0
	127.0.0.1:6379[15]> HDEL key f2 f3
	(integer) 2
	127.0.0.1:6379[15]> HDEL key f4 f1
	(integer) 1
	```
	
8. HKEYS
	HKEYS key
	返回哈希表 key 中的所有域。
	时间复杂度： O(N)，N 为哈希表的大小。
	返回值：
	　　一个包含哈希表中所有域的表。
	　　当 key 不存在时，返回一个空表。
	```shell
	127.0.0.1:6379[15]> HMSET website google www.google.com yahoo www.yahoo.com
	OK
	127.0.0.1:6379[15]> HKEYS website
	1) "google"
	2) "yahoo"
	127.0.0.1:6379[15]> EXISTS fake_key
	(integer) 0
	127.0.0.1:6379[15]> HKEYS fake_key
	(empty list or set)
	```

9. HVALS
	HVALS key
	返回哈希表 key 中所有域的值。
	时间复杂度： O(N)，N 为哈希表的大小。
	返回值：
	　　一个包含哈希表中所有值的表。
	　　当 key 不存在时，返回一个空表。
	```shell
	127.0.0.1:6379[15]> HMSET website google www.google.com yahoo www.yahoo.com
	OK
	127.0.0.1:6379[15]> HVALS website
	1) "www.google.com"
	2) "www.yahoo.com"
	127.0.0.1:6379[15]> EXISTS not_exists
	(integer) 0
	127.0.0.1:6379[15]> HVALS not_exists
	(empty list or set)
	```

10. HGETALL
	HGETALL key
	返回哈希表 key 中，所有的域和值。
	在返回值里，紧跟每个域名 (field name) 之后是域的值 (value)，所以返回值的长度是哈希表大小的两倍。	
	时间复杂度： O(N)，N 为哈希表的大小。
	返回值：
	　　以列表形式返回哈希表的域和域的值。
	　　若 key 不存在，返回空列表。
	```shell
	127.0.0.1:6379[15]> HSET people jack "Jack Sparrow"
	(integer) 1
	127.0.0.1:6379[15]> HSET people gump "Forrest Gump"
	(integer) 1
	127.0.0.1:6379[15]> HGETALL people
	1) "jack"
	2) "Jack Sparrow"
	3) "gump"
	4) "Forrest Gump"
	```
	
11. HINCRBY
	HINCRBY key field increment
	为哈希表 key 中的域 field 的值加上增量 increment 。
	增量也可以为负数，相当于对给定域进行减法操作。
	如果 key 不存在，一个新的哈希表被创建并执行HINCRBY 命令。
	如果域 field 不存在，那么在执行命令前，域的值被初始化为 0 。
	对一个储存字符串值的域 field 执行HINCRBY 命令将造成一个错误。
	本操作的值被限制在 64 位 (bit) 有符号数字表示之内。	
	时间复杂度： O(1)
	返回值： 执行HINCRBY 命令之后，哈希表 key 中域 field 的值。
	```shell
	127.0.0.1:6379[15]> HEXISTS counter page_view
	(integer) 0
	127.0.0.1:6379[15]>  HINCRBY counter page_view 200
	(integer) 200
	127.0.0.1:6379[15]> HGET counter page_view
	"200"
	127.0.0.1:6379[15]>  HINCRBY counter page_view -50
	(integer) 150
	127.0.0.1:6379[15]>  HGET counter page_view
	"150"
	```
	
12. HINCRBYFLOAT
	HINCRBYFLOAT key field increment
	为哈希表 key 中的域 field 加上浮点数增量 increment 。
	如果哈希表中没有域 field ，那么HINCRBYFLOAT 会先将域 field 的值设为 0 ，然后再执行加法操作。
	如果键 key 不存在，那么HINCRBYFLOAT 会先创建一个哈希表，再创建域 field ，最后再执行加法操作。
	当以下任意一个条件发生时，返回一个错误：
	　　域 field 的值不是字符串类型 (因为 redis 中的数字和浮点数都以字符串的形式保存，所以它们都属于字符串类型）
	　　域 field 当前的值或给定的增量 increment 不能解释 (parse) 为双精度浮点数 (double precision floating point number)	
	返回值： 执行加法操作之后 field 域的值。
	```shell
	127.0.0.1:6379[15]> HSET mykey field 10.50
	(integer) 0
	127.0.0.1:6379[15]> HINCRBYFLOAT mykey field 0.1
	"10.6"
	127.0.0.1:6379[15]> HSET mykey field 5.0e3
	(integer) 0
	127.0.0.1:6379[15]> HINCRBYFLOAT mykey field 2.0e2
	"5200"
	127.0.0.1:6379[15]> EXISTS price
	(integer) 0
	127.0.0.1:6379[15]> HINCRBYFLOAT price milk 3.5
	"3.5"
	127.0.0.1:6379[15]> HGETALL price
	1) "milk"
	2) "3.5"
	```

13. HLEN
	HLEN key
	返回哈希表 key 中域的数量。
	返回值：
	　　哈希表中域的数量。
	　　当 key 不存在时，返回 0 。
	```shell
	127.0.0.1:6379[15]> HSET db redis redis.com
	(integer) 1
	127.0.0.1:6379[15]> HSET db mysql mysql.com
	(integer) 1
	127.0.0.1:6379[15]> HLEN db
	(integer) 2
	127.0.0.1:6379[15]> HSET db mongodb mongodb.org
	(integer) 1
	127.0.0.1:6379[15]> HLEN db
	(integer) 3
	```