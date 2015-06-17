#**Redis数据类型之strings类型**
>*Web程序猿博客：[http://blog.csdn.net/thinkercode](http://blog.csdn.net/thinkercode)*

####**string类型-特点**
string 是最简单的类型，你可以理解成与 Memcached 是一模一样的类型，一个 key对应一个value，其上支持的操作与 Memcached 的操作类似。但它的功能更丰富。
string 类型是二进制安全的。意思是redis的string可以包含任何数据，比如jpg图片或者序列化的对象；从内部实现来看其实string可以看作byte数组，最大上限是1G字节。
另外string类型可以被部分命令按 int处理，比如 incr等命令，如果只用 string类型，redis就可以被看作加上持久化特性的 memcached。 当然 redis对string类型的操作比 memcached还是多很多的。
在list、set和zset中包含的独立元素类型都是string元素。

####**string常见命令**
1. SET
	SET key value [EX seconds] [PX milliseconds] [NX|XX]
	将字符串值value关联到key。
	如果key已经持有其他值，SET就覆写就值，无视类型。
	对于某个原本带有生存时间（TTL）的键来说，当SET命令成功在这个键上执行时，这个键原有的TTL将被清除。
	SET 命令的行为可以通过一系列参数来修改：
	　　 EX second ：设置键的过期时间为 second 秒。SET key value EX second 效果等同于 SETEX key second value 。
	　　 PX millisecond ：设置键的过期时间为 millisecond 毫秒。SET key value PX millisecond 效果等同于 PSETEX key millisecond value 。
	　　 NX ：只在键不存在时，才对键进行设置操作。SET key value NX 效果等同于 SETNX key value 。
	　　 XX ：只在键已经存在时，才对键进行设置操作。
	时间复杂度： O(1)
	返回值：SET 在设置操作成功完成时，才返回 OK 。如果设置了 NX 或者 XX ，但因为条件没达到而造成设置操作未执行，那么命令返回空批量回复（NULL Bulk Reply） 。
	```shell
	# 对不存在的key进行设置
	127.0.0.1:6379[15]> SET key_1 1
	OK
	127.0.0.1:6379[15]> GET key_1
	"1"
	
	# 对已存在的key进行设置
	127.0.0.1:6379[15]> SET key_1 "key_1"
	OK
	127.0.0.1:6379[15]> GET key_1
	"key_1"
	
	# 使用EX选项
	127.0.0.1:6379[15]> SET key_2 2 EX 86400
	OK
	127.0.0.1:6379[15]> GET key_2
	"2"
	127.0.0.1:6379[15]> TTL key_2
	(integer) 86387
	127.0.0.1:6379[15]> SET key_3 3
	OK
	
	# 使用PX选项
	127.0.0.1:6379[15]> SET key_3 3 PX 86400
	OK
	127.0.0.1:6379[15]> GET key_3
	"3"
	127.0.0.1:6379[15]> PTTL key_3
	(integer) 68219
	
	# 使用NX选项
	127.0.0.1:6379[15]> SET key_4 4 NX
	OK
	127.0.0.1:6379[15]> GET key_4
	"4"
	127.0.0.1:6379[15]> SET key_4 "key_4" NX 
	(nil)
	127.0.0.1:6379[15]> GET key_4
	"4"
	
	# 使用XX选项
	127.0.0.1:6379[15]> EXISTS key_5
	(integer) 0
	127.0.0.1:6379[15]> SET key_5 5 XX
	(nil)
	127.0.0.1:6379[15]> SET key_5 5
	OK
	127.0.0.1:6379[15]> SET key_5 "key_5" XX
	OK
	127.0.0.1:6379[15]> GET key_5
	"key_5"
	
	# NX 或 XX 可以和 EX 或者 PX 组合使用
	127.0.0.1:6379[15]>  SET key-with-expire-and-NX "hello" EX 10086 NX
	OK
	127.0.0.1:6379[15]> GET key-with-expire-and-NX
	"hello"
	127.0.0.1:6379[15]>  TTL key-with-expire-and-NX
	(integer) 10075
	127.0.0.1:6379[15]> SET key-with-pexpire-and-XX "old value"
	OK
	127.0.0.1:6379[15]> SET key-with-pexpire-and-XX "new value" PX 123321 XX
	OK
	127.0.0.1:6379[15]>  GET key-with-pexpire-and-XX
	"new value"
	127.0.0.1:6379[15]>  PTTL key-with-pexpire-and-XX
	(integer) 107932
	
	# EX 和 PX 可以同时出现，但后面给出的选项会覆盖前面给出的选项
	127.0.0.1:6379[15]>  SET key "value" EX 1000 PX 5000000
	OK
	127.0.0.1:6379[15]> TTL key
	(integer) 4993	# 这是 PX 参数设置的值
	127.0.0.1:6379[15]>  SET another-key "value" PX 5000000 EX 1000
	OK
	127.0.0.1:6379[15]>  TTL another-key
	(integer) 995	# 这是 EX 参数设置的值
	```

2. SETNX
	SETNX key value
	将 key 的值设为 value ，当且仅当 key 不存在。
	若给定的 key 已经存在，则SETNX 不做任何动作。
	SETNX 是『SET if Not eXists』(如果不存在，则 SET) 的简写。
	时间复杂度： O(1)
	返回值：
	　　设置成功，返回 1 。
	　　设置失败，返回 0 。
	```shell
	127.0.0.1:6379[15]> EXISTS key_1
	(integer) 0
	127.0.0.1:6379[15]> SETNX key_1 1
	(integer) 1
	127.0.0.1:6379[15]> SETNX key_1 "key_1"
	(integer) 0
	127.0.0.1:6379[15]> GET key_1
	"1"
	```
	
3. SETEX
	SETEX key seconds value
	将值 value 关联到 key ，并将 key 的生存时间设为 seconds (以秒为单位)。
	如果 key 已经存在，SETEX 命令将覆写旧值。
	这个命令类似于以下两个命令：
	SET key value
	EXPIRE key seconds # 设置生存时间
	不同之处是，SETEX 是一个原子性 (atomic) 操作，关联值和设置生存时间两个动作会在同一时间内完成，该命令在 Redis 用作缓存时，非常实用。
	时间复杂度： O(1)
	返回值：
	　　设置成功时返回 OK 。
	　　当 seconds 参数不合法时，返回一个错误。
	```shell
	# 在 key 不存在时进行 SETEX
	127.0.0.1:6379[15]> SETEX key_1 86400 1
	OK
	127.0.0.1:6379[15]> GET key_1
	"1"
	127.0.0.1:6379[15]> TTL key_1
	(integer) 86390		# 剩余生存时间
	
	# key 已经存在时，SETEX 覆盖旧值
	127.0.0.1:6379[15]> SET key_2 2
	OK
	127.0.0.1:6379[15]> SETEX key_2 86400 "key_2"
	OK
	127.0.0.1:6379[15]> GET key_2
	"key_2"
	127.0.0.1:6379[15]> TTL key_2
	(integer) 86388
	```
	
4. PSETEX
	PSETEX key milliseconds value
	这个命令和SETEX 命令相似，但它以毫秒为单位设置 key 的生存时间，而不是像SETEX 命令那样，以秒为单位。
	时间复杂度： O(1)
	返回值： 设置成功时返回 OK 。
	```shell
	127.0.0.1:6379[15]> PSETEX key_1 80000 1
	OK
	127.0.0.1:6379[15]> PTTL key_1
	(integer) 70167
	127.0.0.1:6379[15]> GET key_1
	"1"
	```

5. SETRANGE
	SETRANGE key offset value
	用 value 参数覆写 (overwrite) 给定 key 所储存的字符串值，从偏移量 offset 开始。
	不存在的 key 当作空白字符串处理。
	SETRANGE 命令会确保字符串足够长以便将 value 设置在指定的偏移量上，如果给定 key 原来储存的字符串长度比偏移量小 (比如字符串只有 5 个字符长，但你设置的 offset 是 10 )，那么原字符和偏移量之间的空白将用零字节 (zerobytes, "\x00" ) 来填充。
	注意你能使用的最大偏移量是 2^29-1(536870911) ，因为 Redis 字符串的大小被限制在 512 兆 (megabytes)以内。如果你需要使用比这更大的空间，你可以使用多个 key 。
	>当生成一个很长的字符串时，Redis 需要分配内存空间，该操作有时候可能会造成服务器阻塞 (block)。在 2010 年的 Macbook Pro 上，设置偏移量为 536870911(512MB 内存分配)，耗费约 300 毫秒，设置偏移量为 134217728(128MB 内存分配)，耗费约 80 毫秒，设置偏移量 33554432(32MB 内存分配)，耗费约 30 毫秒，设置偏移量为 8388608(8MB 内存分配)，耗费约 8 毫秒。注意若首次内存分配成功之后，再对同一个 key 调用SETRANGE 操作，无须再重新内存。
	时间复杂度：对小 (small) 的字符串，平摊复杂度 O(1)。(关于什么字符串是” 小” 的，请参考APPEND 命令)否则为 O(M)，M 为 value 参数的长度。
	返回值： 被SETRANGE 修改之后，字符串的长度。
	因为有了SETRANGE 和GETRANGE 命令，你可以将 Redis 字符串用作具有 O(1) 随机访问时间的线性数组，这在很多真实用例中都是非常快速且高效的储存方式，具体请参考APPEND 命令的『模式：时间序列』部分。
	```shell
	127.0.0.1:6379[15]> SET greeting "hello world"
	OK
	127.0.0.1:6379[15]> SETRANGE greeting 6 "Redis"
	(integer) 11
	127.0.0.1:6379[15]> GET greeting
	"hello Redis"
	127.0.0.1:6379[15]> EXISTS empty_string
	(integer) 0
	127.0.0.1:6379[15]> SETRANGE empty_string 5 "Redis!" 
	(integer) 11
	127.0.0.1:6379[15]> GET empty_string
	"\x00\x00\x00\x00\x00Redis!"
	```	
	
6. GETRANGE
	GETRANGE key start end
	返回 key 中字符串值的子字符串，字符串的截取范围由 start 和 end 两个偏移量决定 (包括 start 和 end在内)。
	负数偏移量表示从字符串最后开始计数，-1 表示最后一个字符，-2 表示倒数第二个，以此类推。GETRANGE 通过保证子字符串的值域 (range) 不超过实际字符串的值域来处理超出范围的值域请求。
	时间复杂度：
	　　O(N)，N 为要返回的字符串的长度。
	　　复杂度最终由字符串的返回值长度决定，但因为从已有字符串中取出子字符串的操作非常廉价(cheap)，所以对于长度不大的字符串，该操作的复杂度也可看作 O(1)。
	返回值： 截取得出的子字符串。
	```shell
	127.0.0.1:6379[15]> SET greeting "hello, my friend"
	OK
	127.0.0.1:6379[15]>  GETRANGE greeting 0 4 		# 返回索引 0-4 的字符，包括 4。
	"hello"
	127.0.0.1:6379[15]>  GETRANGE greeting -1 -5	# 不支持回绕操作
	""
	127.0.0.1:6379[15]>  GETRANGE greeting -3 -1	# 负数索引
	"end"
	127.0.0.1:6379[15]> GETRANGE greeting 0 -1 		# 从第一个到最后一个
	"hello, my friend"
	127.0.0.1:6379[15]>  GETRANGE greeting 0 1008611	# 值域范围不超过实际字符串，超过部分自动被忽略
	"hello, my friend"
	```
	>注意：GETRANGE和SETRANGE都是单字节操作，对中文支持并不友好。

7. APPEND
	APPEND key value
	如果 key 已经存在并且是一个字符串，APPEND 命令将 value 追加到 key 原来的值的末尾。
	如果 key 不存在，APPEND 就简单地将给定 key 设为 value ，就像执行 SET key value 一样。
	时间复杂度： 平摊 O(1)
	返回值： 追加 value 之后，key 中字符串的长度。
	```shell
	127.0.0.1:6379[15]> EXISTS myphone
	(integer) 0
	127.0.0.1:6379[15]> APPEND myphone "nokia"
	(integer) 5
	127.0.0.1:6379[15]>  APPEND myphone " - 1110" 
	(integer) 12
	127.0.0.1:6379[15]> GET myphone
	"nokia - 1110"
	```
	
	<font color=blue>时间序列 (Time series)</font>
	APPEND 可以为一系列定长 (fixed-size) 数据 (sample) 提供一种紧凑的表示方式，通常称之为时间序列。每当一个新数据到达的时候，执行以下命令：
	APPEND timeseries "fixed-size sample"
	然后可以通过以下的方式访问时间序列的各项属性：
	　　STRLEN 给出时间序列中数据的数量
	　　GETRANGE 可以用于随机访问。只要有相关的时间信息的话，我们就可以在 Redis 2.6 中使用 Lua脚本和GETRANGE 命令实现二分查找。
	　　SETRANGE 可以用于覆盖或修改已存在的的时间序列。
	这个模式的唯一缺陷是我们只能增长时间序列，而不能对时间序列进行缩短，因为 Redis 目前还没有对字符串进行修剪 (tirm) 的命令，但是，不管怎么说，这个模式的储存方式还是可以节省下大量的空间。
	>Note: 可以考虑使用 UNIX 时间戳作为时间序列的键名，这样一来，可以避免单个 key 因为保存过大的时间序列而占用大量内存，另一方面，也可以节省下大量命名空间。
	```shell
	127.0.0.1:6379[15]> APPEND ts "0043"
	(integer) 4
	127.0.0.1:6379[15]> APPEND ts "0035"
	(integer) 8
	127.0.0.1:6379[15]> GETRANGE ts 0 3
	"0043"
	127.0.0.1:6379[15]> GETRANGE ts 4 7
	"0035"
	```

8. GET
	GET key
	返回 key 所关联的字符串值。
	如果 key 不存在那么返回特殊值 nil 。
	假如 key 储存的值不是字符串类型，返回一个错误，因为GET 只能用于处理字符串值。
	时间复杂度： O(1)
	返回值：
	　　当 key 不存在时，返回 nil ，否则，返回 key 的值。
	　　如果 key 不是字符串类型，那么返回一个错误。
	```shell
	127.0.0.1:6379[15]> GET key_1
	(nil)
	127.0.0.1:6379[15]> SET key_1 1
	OK
	127.0.0.1:6379[15]> get key_1
	"1"	
	```

9. MSET
	MSET key value [key value ...]
	同时设置一个或多个 key-value 对。
	如果某个给定 key 已经存在，那么MSET 会用新值覆盖原来的旧值，如果这不是你所希望的效果，请考虑使用MSETNX 命令：它只会在所有给定 key 都不存在的情况下进行设置操作。
	MSET 是一个原子性 (atomic) 操作，所有给定 key 都会在同一时间内被设置，某些给定 key 被更新而另一些给定 key 没有改变的情况，不可能发生。
	时间复杂度： O(N)，N 为要设置的 key 数量。
	返回值： 总是返回 OK (因为 MSET 不可能失败)
	```shell
	127.0.0.1:6379[15]> MSET key_1 1 key_2 2 key_3 3 
	OK
	127.0.0.1:6379[15]> MGET key_1 key_2 key_3
	1) "1"
	2) "2"
	3) "3"
	```
	
10. MGET
	MGET key [key ...]
	返回所有 (一个或多个) 给定 key 的值。
	如果给定的 key 里面，有某个 key 不存在，那么这个 key 返回特殊值 nil 。因此，该命令永不失败。
	时间复杂度: O(N) , N 为给定 key 的数量。
	返回值： 一个包含所有给定 key 的值的列表。
	```shell
	127.0.0.1:6379[15]> MSET key_1 1 key_2 2 key_3 3
	OK
	127.0.0.1:6379[15]> MGET key_1 key_2 key_4
	1) "1"
	2) "2"
	3) (nil)	
	```

11. MSETNX
	MSETNX key value [key value ...]
	同时设置一个或多个 key-value 对，当且仅当所有给定 key 都不存在。
	即使只有一个给定 key 已存在，MSETNX 也会拒绝执行所有给定 key 的设置操作。
	MSETNX 是原子性的，因此它可以用作设置多个不同 key 表示不同字段 (field) 的唯一性逻辑对象 (uniquelogic object)，所有字段要么全被设置，要么全不被设置。
	时间复杂度： O(N)，N 为要设置的 key 的数量。
	返回值：
	　　当所有 key 都成功设置，返回 1 。
	　　如果所有给定 key 都设置失败 (至少有一个 key 已经存在)，那么返回 0 。
	```shell
	127.0.0.1:6379[15]> MSETNX rmdbs "MySQL" nosql "MongoDB" key-value-store "redis"
	(integer) 1
	127.0.0.1:6379[15]> MGET rmdbs nosql key-value-store
	1) "MySQL"
	2) "MongoDB"
	3) "redis"
	127.0.0.1:6379[15]> MSETNX rmdbs "Sqlite" language "python"
	(integer) 0
	127.0.0.1:6379[15]> EXISTS language
	(integer) 0
	127.0.0.1:6379[15]> GET rmdbs
	"MySQL"
	```
	
12. GETSET
	GETSET key value	
	将给定 key 的值设为 value ，并返回 key 的旧值 (old value)。
	当 key 存在但不是字符串类型时，返回一个错误。
	时间复杂度： O(1)
	返回值：
	　　返回给定 key 的旧值。
	　　当 key 没有旧值时，也即是，key 不存在时，返回 nil 。
	```shell
	127.0.0.1:6379[15]> EXISTS db
	(integer) 0
	127.0.0.1:6379[15]>  GETSET db mongodb
	(nil)
	127.0.0.1:6379[15]>  GET db
	"mongodb"
	127.0.0.1:6379[15]>  GETSET db redis
	"mongodb"
	127.0.0.1:6379[15]>  GET db
	"redis"
	```
	>GETSET 可以和INCR 组合使用，实现一个有原子性 (atomic) 复位操作的计数器 (counter)。
	举例来说，每次当某个事件发生时，进程可能对一个名为 mycount 的 key 调用INCR 操作，通常我们还要在一个原子时间内同时完成获得计数器的值和将计数器值复位为 0 两个操作。
	可以用命令 GETSET mycounter 0 来实现这一目标。
	```shell
	127.0.0.1:6379[15]> INCR mycount
	(integer) 12
	127.0.0.1:6379[15]> GETSET mycount 0
	"12"
	127.0.0.1:6379[15]> GET mycount
	"0"
	```
	
13. STRLEN
	STRLEN key
	返回 key 所储存的字符串值的长度。
	当 key 储存的不是字符串值时，返回一个错误。
	复杂度： O(1)
	返回值：
	　　字符串值的长度。
	　　当 key 不存在时，返回 0 。
	```shell
	127.0.0.1:6379[15]> SET mykey "Hello world"
	OK
	127.0.0.1:6379[15]> STRLEN mykey
	(integer) 11
	127.0.0.1:6379[15]>  STRLEN nonexisting
	(integer) 0
	```

14. INCR
	INCR key
	将 key 中储存的数字值增一。
	如果 key 不存在，那么 key 的值会先被初始化为 0 ，然后再执行INCR 操作。
	如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。
	本操作的值限制在 64 位 (bit) 有符号数字表示之内。
	时间复杂度： O(1)
	返回值： 执行INCR 命令之后 key 的值。
	```shell
	127.0.0.1:6379[15]>  SET page_view 20
	OK
	127.0.0.1:6379[15]>  INCR page_view
	(integer) 21
	127.0.0.1:6379[15]>  GET page_view
	"21"
	```
	<font color=blue>模式：计数器</font>
	计数器是 Redis 的原子性自增操作可实现的最直观的模式了，它的想法相当简单：每当某个操作发生时，向Redis 发送一个INCR 命令。
	比如在一个 web 应用程序中，如果想知道用户在一年中每天的点击量，那么只要将用户 ID 以及相关的日期信息作为键，并在每次用户点击页面时，执行一次自增操作即可。
	比如用户名是 peter，点击时间是 2012 年 3 月 22 日，那么执行命令 INCR peter::2012.3.22 。
	可以用以下几种方式扩展这个简单的模式：
	　　可以通过组合使用INCR 和EXPIRE ，来达到只在规定的生存时间内进行计数 (counting) 的目的。
	　　客户端可以通过使用GETSET 命令原子性地获取计数器的当前值并将计数器清零，更多信息请参考GETSET命令。
	 　　使用其他自增/自减操作，比如DECR 和INCRBY ，用户可以通过执行不同的操作增加或减少计数器的值，比如在游戏中的记分器就可能用到这些命令。
	<font color=blue> 模式：限速器</font>
	限速器是特殊化的计算器，它用于限制一个操作可以被执行的速率 (rate)。
	限速器的典型用法是限制公开 API的请求次数，以下是一个限速器实现示例，它将 API的最大请求数限制在每个 IP地址每秒钟十个之内：
	```shell
	FUNCTION LIMIT_API_CALL(ip)
	ts = CURRENT_UNIX_TIME()
	keyname = ip+":"+ts
	current = GET(keyname)
	IF current != NULL AND current > 10 THEN
	ERROR "too many requests per second"
	END
	
	IF current == NULL THEN
	MULTI
	INCR(keyname, 1)
	EXPIRE(keyname, 1)
	EXEC
	ELSE
	INCR(keyname, 1)
	END
	PERFORM_API_CALL()
	```
	这个实现每秒钟为每个 IP 地址使用一个不同的计数器，并用EXPIRE 命令设置生存时间 (这样 Redis 就会负责自动删除过期的计数器)。
	注意，我们使用事务打包执行INCR 命令和EXPIRE 命令，避免引入竞争条件，保证每次调用 API时都可以正确地对计数器进行自增操作并设置生存时间。
	以下是另一个限速器实现：
	```shell
	FUNCTION LIMIT_API_CALL(ip):
	current = GET(ip)
	IF current != NULL AND current > 10 THEN
	ERROR "too many requests per second"
	ELSE
	value = INCR(ip)
	IF value == 1 THEN
	EXPIRE(ip,1)
	END
	PERFORM_API_CALL()
	END
	```
	这个限速器只使用单个计数器，它的生存时间为一秒钟，如果在一秒钟内，这个计数器的值大于 10 的话，那么访问就会被禁止。
	这个新的限速器在思路方面是没有问题的，但它在实现方面不够严谨，如果我们仔细观察一下的话，就会发现在INCR 和EXPIRE 之间存在着一个竞争条件，假如客户端在执行INCR 之后，因为某些原因 (比如客户端失败) 而忘记设置EXPIRE 的话，那么这个计数器就会一直存在下去，造成每个用户只能访问 10 次，噢，这简直是个灾难！
	要消灭这个实现中的竞争条件，我们可以将它转化为一个 Lua 脚本，并放到 Redis 中运行 
	```shell
	local current
	current = redis.call("incr",KEYS[1])
	if tonumber(current) == 1 then
	redis.call("expire",KEYS[1],1)
	end
	```
	通过将计数器作为脚本放到 Redis上运行，我们保证了INCR 和EXPIRE 两个操作的原子性，现在这个脚本实现不会引入竞争条件，它可以运作的很好。
	还有另一种消灭竞争条件的方法，就是使用 Redis 的列表结构来代替INCR 命令，这个方法无须脚本支持，因此它在 Redis 2.6 以下的版本也可以运行得很好：
	```shell
	FUNCTION LIMIT_API_CALL(ip)
	current = LLEN(ip)
	IF current > 10 THEN
	ERROR "too many requests per second"
	ELSE
	IF EXISTS(ip) == FALSE
	MULTI
	RPUSH(ip,ip)
	EXPIRE(ip,1)
	EXEC
	ELSE
	RPUSHX(ip,ip)
	END
	PERFORM_API_CALL()
	END
	```
	新的限速器使用了列表结构作为容器，LLEN 用于对访问次数进行检查，一个事务包裹着RPUSH和EXPIRE 两个命令，用于在第一次执行计数时创建列表，并正确设置地设置过期时间，最后，RPUSHX在后续的计数操作中进行增加操作。
	
15. INCRBY
	INCRBY key increment
	将 key 所储存的值加上增量 increment 。
	如果 key 不存在，那么 key 的值会先被初始化为 0 ，然后再执行INCRBY 命令。
	如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。
	本操作的值限制在 64 位 (bit) 有符号数字表示之内。
	关于递增 (increment) / 递减 (decrement) 操作的更多信息，参见INCR 命令。
	时间复杂度： O(1)
	返回值： 加上 increment 之后，key 的值。
	```shell
	127.0.0.1:6379[15]> SET rank 50
	OK
	127.0.0.1:6379[15]> INCRBY rank 20
	(integer) 70
	127.0.0.1:6379[15]> EXISTS counter
	(integer) 0
	127.0.0.1:6379[15]> INCRBY counter 30
	(integer) 30
	```
	
16. INCRBYFLOAT
	INCRBYFLOAT key increment
	为 key 中所储存的值加上浮点数增量 increment 。
	如果 key 不存在，那么INCRBYFLOAT 会先将 key 的值设为 0 ，再执行加法操作。
	如果命令执行成功，那么 key 的值会被更新为（执行加法之后的）新值，并且新值会以字符串的形式返回给调用者。
	无论是 key 的值，还是增量 increment ，都可以使用像 2.0e7 、3e5 、90e-2 那样的指数符号 (exponentialnotation) 来表示，但是，执行 INCRBYFLOAT 命令之后的值总是以同样的形式储存，也即是，它们总是由一个数字，一个（可选的）小数点和一个任意位的小数部分组成（比如 3.14 、69.768 ，诸如此类)，小数部分尾随的 0 会被移除，如果有需要的话，还会将浮点数改为整数（比如 3.0 会被保存成 3 ） 。
	除此之外，无论加法计算所得的浮点数的实际精度有多长，INCRBYFLOAT 的计算结果也最多只能表示小数点的后十七位。
	当以下任意一个条件发生时，返回一个错误：
	　　key 的值不是字符串类型 (因为 Redis 中的数字和浮点数都以字符串的形式保存，所以它们都属于字符串类型）
	　　key 当前的值或者给定的增量 increment 不能解释 (parse) 为双精度浮点数 (double precision floatingpoint number）
	时间复杂度： O(1)
	返回值： 执行命令之后 key 的值。
	```shell
	127.0.0.1:6379[15]> SET mykey 10.50
	OK
	127.0.0.1:6379[15]> INCRBYFLOAT mykey 0.1
	"10.6"
	
	# 值和增量都是指数符号
	127.0.0.1:6379[15]>  SET mykey 314e-2
	OK
	127.0.0.1:6379[15]> GET mykey	# 用 SET 设置的值可以是指数符号
	"314e-2"
	127.0.0.1:6379[15]> INCRBYFLOAT mykey 0		# 但执行 INCRBYFLOAT 之后格式会被改成非指数符号
	"3.14"
	127.0.0.1:6379[15]>  SET mykey 3
	OK
	127.0.0.1:6379[15]>  INCRBYFLOAT mykey 1.1
	"4.1"
	127.0.0.1:6379[15]>  SET mykey 3.0	# 后跟的 0 会被移除
	OK
	127.0.0.1:6379[15]>  GET mykey
	"3.0"
	127.0.0.1:6379[15]>  INCRBYFLOAT mykey 1.000000000000000000000 	# 但 INCRBYFLOAT 会将无用的 0 忽略掉，有需要的话，将浮点变为整数
	"4"
	127.0.0.1:6379[15]>  GET mykey
	"4"
	```
	
17. DECR
	DECR key
	将 key 中储存的数字值减一。
	如果 key 不存在，那么 key 的值会先被初始化为 0 ，然后再执行DECR 操作。
	如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。
	本操作的值限制在 64 位 (bit) 有符号数字表示之内。
	关于递增 (increment) / 递减 (decrement) 操作的更多信息，请参见INCR 命令。
	时间复杂度： O(1)
	返回值： 执行DECR 命令之后 key 的值。
	```shell
	127.0.0.1:6379[15]>  SET failure_times 10
	OK
	127.0.0.1:6379[15]> DECR failure_times
	(integer) 9
	127.0.0.1:6379[15]>  EXISTS count
	(integer) 0
	127.0.0.1:6379[15]> DECR count
	(integer) -1
	```
	
18. DECRBY
	DECRBY key decrement
	将 key 所储存的值减去减量 decrement 。
	如果 key 不存在，那么 key 的值会先被初始化为 0 ，然后再执行DECRBY 操作。
	如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。
	本操作的值限制在 64 位 (bit) 有符号数字表示之内。
	关于更多递增 (increment) / 递减 (decrement) 操作的更多信息，请参见INCR 命令。
	时间复杂度： O(1)
	返回值： 减去 decrement 之后，key 的值。
	```shell
	127.0.0.1:6379[15]> SET count 100
	OK
	127.0.0.1:6379[15]>  DECRBY count 20
	(integer) 80
	127.0.0.1:6379[15]>  EXISTS pages
	(integer) 0
	127.0.0.1:6379[15]>  DECRBY pages 10
	(integer) -10
	```

19. SETBIT
	SETBIT key offset value
	对 key 所储存的字符串值，设置或清除指定偏移量上的位 (bit)。
	位的设置或清除取决于 value 参数，可以是 0 也可以是 1 。
	当 key 不存在时，自动生成一个新的字符串值。
	字符串会进行伸展 (grown) 以确保它可以将 value 保存在指定的偏移量上。当字符串值进行伸展时，空白位置以 0 填充。
	offset 参数必须大于或等于 0 ，小于 2^32 (bit 映射被限制在 512 MB 之内)。
	时间复杂度: O(1)
	返回值： 指定偏移量原来储存的位。
	```shell
	127.0.0.1:6379[15]> SETBIT bit 10086 1
	(integer) 0
	127.0.0.1:6379[15]>  GETBIT bit 10086
	(integer) 1
	127.0.0.1:6379[15]>  GETBIT bit 100
	(integer) 0
	```
	>Warning: 对使用大的 offset 的SETBIT 操作来说，内存分配可能造成 Redis 服务器被阻塞。具体参考SETRANGE 命令，warning(警告) 部分。
	
20. GETBIT
	GETBIT key offset
	对 key 所储存的字符串值，获取指定偏移量上的位 (bit)。
	当 offset 比字符串值的长度大，或者 key 不存在时，返回 0 。
	时间复杂度： O(1)
	返回值： 字符串值指定偏移量上的位 (bit)。
	```shell
	127.0.0.1:6379[15]> EXISTS bit
	(integer) 0
	127.0.0.1:6379[15]> GETBIT bit 10086
	(integer) 0
	127.0.0.1:6379[15]> SETBIT bit 10086 1
	(integer) 0
	127.0.0.1:6379[15]> GETBIT bit 10086
	(integer) 1
	```
	
21. BITOP
	BITOP operation destkey key [key ...]
	对一个或多个保存二进制位的字符串 key 进行位元操作，并将结果保存到 destkey 上。
	operation 可以是 AND 、OR 、NOT 、XOR 这四种操作中的任意一种：
	　　BITOP AND destkey key [key ...] ，对一个或多个 key 求逻辑并，并将结果保存到 destkey 。
	　　BITOP OR destkey key [key ...] ，对一个或多个 key 求逻辑或，并将结果保存到 destkey 。
	　　BITOP XOR destkey key [key ...] ，对一个或多个 key 求逻辑异或，并将结果保存到 destkey 。
	　　BITOP NOT destkey key ，对给定 key 求逻辑非，并将结果保存到 destkey 。
	除了 NOT 操作之外，其他操作都可以接受一个或多个 key 作为输入。
	处理不同长度的字符串
	当BITOP 处理不同长度的字符串时，较短的那个字符串所缺少的部分会被看作 0 。
	空的 key 也被看作是包含 0 的字符串序列。
	时间复杂度： O(N)
	返回值： 保存到 destkey 的字符串的长度，和输入 key 中最长的字符串长度相等。
	```shell
	127.0.0.1:6379[15]> SETBIT bits-1 0 1
	(integer) 1
	127.0.0.1:6379[15]>  SETBIT bits-1 3 1
	(integer) 0
	127.0.0.1:6379[15]>  SETBIT bits-2 0 1 
	(integer) 0
	127.0.0.1:6379[15]>  SETBIT bits-2 1 1
	(integer) 0
	127.0.0.1:6379[15]>  SETBIT bits-2 3 1
	(integer) 0
	127.0.0.1:6379[15]>  BITOP AND and-result bits-1 bits-2
	(integer) 1
	127.0.0.1:6379[15]>  GETBIT and-result 0
	(integer) 1
	127.0.0.1:6379[15]> GETBIT and-result 1
	(integer) 0
	127.0.0.1:6379[15]>  GETBIT and-result 2
	(integer) 0
	127.0.0.1:6379[15]>  GETBIT and-result 3
	(integer) 1
	```
	>Note: BITOP 的复杂度为 O(N) ，当处理大型矩阵 (matrix) 或者进行大数据量的统计时，最好将任务指派到附属节点 (slave) 进行，避免阻塞主节点。
	
22. BITCOUNT
	BITCOUNT key [start] [end]
	计算给定字符串中，被设置为 1 的比特位的数量。
	一般情况下，给定的整个字符串都会被进行计数，通过指定额外的 start 或 end 参数，可以让计数只在特定的位上进行。
	start 和 end 参数的设置和GETRANGE 命令类似，都可以使用负数值：比如 -1 表示最后一个位，而 -2表示倒数第二个位，以此类推。
	不存在的 key 被当成是空字符串来处理，因此对一个不存在的 key 进行 BITCOUNT 操作，结果为 0 。
	时间复杂度： O(N)
	返回值： 被设置为 1 的位的数量。
	```shell
	127.0.0.1:6379[15]>  BITCOUNT bits
	(integer) 0
	127.0.0.1:6379[15]> SETBIT bits 0 1
	(integer) 0
	127.0.0.1:6379[15]>  BITCOUNT bits
	(integer) 1
	127.0.0.1:6379[15]>  SETBIT bits 3 1
	(integer) 0
	127.0.0.1:6379[15]> BITCOUNT bits
	(integer) 2
	```
	<font color=blue>模式：使用 bitmap 实现用户上线次数统计</font>
	Bitmap 对于一些特定类型的计算非常有效。
	假设现在我们希望记录自己网站上的用户的上线频率，比如说，计算用户 A 上线了多少天，用户 B 上线了多少天，诸如此类，以此作为数据，从而决定让哪些用户参加 beta 测试等活动——这个模式可以使用SETBIT 和BITCOUNT 来实现。
	比如说，每当用户在某一天上线的时候，我们就使用SETBIT ，以用户名作为 key ，将那天所代表的网站的上线日作为 offset 参数，并将这个 offset 上的为设置为 1 。
	举个例子，如果今天是网站上线的第 100 天，而用户 peter 在今天阅览过网站，那么执行命令 SETBIT peter 100 1 ；如果明天 peter 也继续阅览网站，那么执行命令 SETBIT peter 101 1 ，以此类推。
	当要计算 peter 总共以来的上线次数时，就使用BITCOUNT 命令：执行 BITCOUNT peter ，得出的结果就是 peter 上线的总天数。
	<font color=blue>性能</font>
	前面的上线次数统计例子，即使运行 10 年，占用的空间也只是每个用户 10*365 比特位 (bit)，也即是每个用户 456 字节。对于这种大小的数据来说，BITCOUNT 的处理速度就像GET 和INCR 这种 O(1) 复杂度的操作一样快。
	如果你的 bitmap 数据非常大，那么可以考虑使用以下两种方法：
	　　1. 将一个大的 bitmap 分散到不同的 key 中，作为小的 bitmap 来处理。使用 Lua 脚本可以很方便地完成这一工作。
	　　2. 使用BITCOUNT 的 start 和 end 参数，每次只对所需的部分位进行计算，将位的累积工作 (accumu-lating) 放到客户端进行，并且对结果进行缓存 (caching)。