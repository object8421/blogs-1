#**Redis数据类型之LIST类型**
>*Web程序猿博客：[http://blog.csdn.net/thinkercode](http://blog.csdn.net/thinkercode)*

####**list类型-特点**
list 是一个链表结构，主要功能是 push、pop、获取一个范围的所有值等等，操作中 key理解为链表的名字。
Redis 的 list类型其实就是一个每个子元素都是 string 类型的双向链表。链表的最大长度是(2的 32 次方)。我们可以通过 push,pop 操作从链表的头部或者尾部添加删除元素。这使得 list既可以用作栈，也可以用作队列。
有意思的是 list 的 pop 操作还有阻塞版本的，当我们[lr]pop 一个 list 对象时，如果 list 是空，或者不存在，会立即返回 nil。但是阻塞版本的 b[lr]pop 可以则可以阻塞，当然可以加超时时间，超时后也会返回 nil。为什么要阻塞版本的 pop 呢，主要是为了避免轮询。举个简单的例子如果我们用 list 来实现一个工作队列。 执行任务的 thread 可以调用阻塞版本的 pop 去获取任务这样就可以避免轮询去检查是否有任务存在。当任务来时候工作线程可以立即返回，也可以避免轮询带来的延迟。

####**list类型-应用场景**
Redis list应用场景非常多，也是Redis最重要的数据结构之一，比如微博的关注列表，粉丝列表等都可以用Redis的list结构来实现；博客实现中，可为每篇日志设置一个list，在该list中推入进博客评论；也可以使用Redis list实现消息队列。
list中的数据逻辑上存在顺序关系（数组的下表），适合存储带有顺序特性（空间、时间属性）的数据。比如，记录用户在网站上浏览商品id，这种带时间属性的数据可以用来分析用户的购物行为。
list作为queue，一端加入数据，另一端读取数据，最典型的就是生产者/消费者模型，比如：商品秒杀，从已经初始化队列中pop出数据，直到队列为空。
list作为stack，只操作list的一端，数据先进后出。

####**list常见命令**
1. LPUSH
	LPUSH key value [value ...]
	将一个或多个值 value 插入到列表 key 的表头如果有多个 value 值，那么各个 value 值按从左到右的顺序依次插入到表头：比如说，对空列表 mylist执行命令 LPUSH mylist a b c ，列表的值将是 c b a ，这等同于原子性地执行 LPUSH mylist a 、LPUSH	mylist b 和 LPUSH mylist c 三个命令。
	如果 key 不存在，一个空列表会被创建并执行LPUSH 操作。
	当 key 存在但不是列表类型时，返回一个错误。
	时间复杂度： O(1)
	返回值： 执行LPUSH 命令后，列表的长度。
	```shell
	127.0.0.1:6379[15]> LPUSH languages python
	(integer) 1
	127.0.0.1:6379[15]> LPUSH languages python
	(integer) 2
	127.0.0.1:6379[15]> LRANGE languages 0 -1
	1) "python"
	2) "python"
	127.0.0.1:6379[15]> LPUSH mylist a b c
	(integer) 3
	127.0.0.1:6379[15]> LRANGE mylist 0 -1
	1) "c"
	2) "b"
	3) "a"
	```

2. LPUSHX
	LPUSHX key value
	将值 value 插入到列表 key 的表头，当且仅当 key 存在并且是一个列表。
	和LPUSH 命令相反，当 key 不存在时，LPUSHX 命令什么也不做。
	时间复杂度： O(1)
	返回值： LPUSHX 命令执行之后，表的长度。
	```shell
	127.0.0.1:6379[15]> LLEN greet
	(integer) 0
	127.0.0.1:6379[15]> LPUSHX greet "hello"
	(integer) 0
	127.0.0.1:6379[15]> LPUSH greet "hello"
	(integer) 1
	127.0.0.1:6379[15]> LPUSHX greet "good morning"
	(integer) 2
	127.0.0.1:6379[15]> LRANGE greet 0 -1
	1) "good morning"
	2) "hello"
	```

3. RPUSH
	RPUSH key value [value ...]
	将一个或多个值 value 插入到列表 key 的表尾 (最右边)。
	如果有多个 value 值，那么各个 value 值按从左到右的顺序依次插入到表尾：比如对一个空列表 mylist执行 RPUSH mylist a b c ，得出的结果列表为 a b c ，等同于执行命令 RPUSH mylist a 、RPUSH mylist	b 、RPUSH mylist c 。
	如果 key 不存在，一个空列表会被创建并执行RPUSH 操作。
	当 key 存在但不是列表类型时，返回一个错误。
	时间复杂度： O(1)
	返回值： 执行RPUSH 操作后，表的长度。
	```shell
	127.0.0.1:6379[15]> RPUSH languages c
	(integer) 1
	127.0.0.1:6379[15]> RPUSH languages c
	(integer) 2
	127.0.0.1:6379[15]> LRANGE languages 0 -1
	1) "c"
	2) "c"
	127.0.0.1:6379[15]> RPUSH mylist a b c
	(integer) 3
	127.0.0.1:6379[15]> LRANGE mylist 0 -1
	1) "a"
	2) "b"
	3) "c"
	```
	
4. RPUSHX
	RPUSHX key value
	将值 value 插入到列表 key 的表尾，当且仅当 key 存在并且是一个列表。
	和RPUSH 命令相反，当 key 不存在时，RPUSHX 命令什么也不做。
	时间复杂度： O(1)
	返回值： RPUSHX 命令执行之后，表的长度。
	```shell
	127.0.0.1:6379[15]> LLEN greet
	(integer) 0
	127.0.0.1:6379[15]> LPUSHX greet "hello" 
	(integer) 0
	127.0.0.1:6379[15]> LPUSH greet "hello" 
	(integer) 1
	127.0.0.1:6379[15]> LPUSHX greet "good morning"
	(integer) 2
	127.0.0.1:6379[15]> LRANGE greet 0 -1
	1) "good morning"
	2) "hello"
	```

5. LPOP
	LPOP key
	移除并返回列表 key 的头元素。
	时间复杂度： O(1)
	返回值：
	　　列表的头元素。
	　　当 key 不存在时，返回 nil 。
	```shell
	127.0.0.1:6379[15]> LLEN course
	(integer) 0
	127.0.0.1:6379[15]> RPUSH course algorithm001
	(integer) 1
	127.0.0.1:6379[15]> RPUSH course c++101
	(integer) 2
	127.0.0.1:6379[15]> LPOP course
	"algorithm001"
	```
	
6. RPOP
	RPOP key
	移除并返回列表 key 的尾元素。
	时间复杂度： O(1)
	返回值：
	　　列表的尾元素。
	　　当 key 不存在时，返回 nil 。
	```shell
	127.0.0.1:6379[15]> RPUSH mylist "one"
	(integer) 1
	127.0.0.1:6379[15]> RPUSH mylist "two"
	(integer) 2
	127.0.0.1:6379[15]> RPUSH mylist "three"
	(integer) 3
	127.0.0.1:6379[15]> RPOP mylist
	"three"
	127.0.0.1:6379[15]> LRANGE mylist 0 -1
	1) "one"
	2) "two"
	```

7. BLPOP
	BLPOP key [key ...] timeout
	BLPOP 是列表的阻塞式 (blocking) 弹出原语。
	它是LPOP 命令的阻塞版本，当给定列表内没有任何元素可供弹出的时候，连接将被BLPOP 命令阻塞，直到等待超时或发现可弹出元素为止。
	当给定多个 key 参数时，按参数 key 的先后顺序依次检查各个列表，弹出第一个非空列表的头元素。非阻塞行为当BLPOP 被调用时，如果给定 key 内至少有一个非空列表，那么弹出遇到的第一个非空列表的头元素，并和被弹出元素所属的列表的名字一起，组成结果返回给调用者。
	当存在多个给定 key 时，BLPOP 按给定 key 参数排列的先后顺序，依次检查各个列表。
	假设现在有 job 、command 和 request 三个列表，其中 job 不存在，command 和 request 都持有非空列表。考虑以下命令：
	BLPOP job command request 0
	BLPOP 保证返回的元素来自 command，因为它是按” 查找 job -> 查找 command -> 查找 request “这样的顺序，第一个找到的非空列表。
	```shell
	127.0.0.1:6379[15]> DEL job command request
	(integer) 0
	127.0.0.1:6379[15]> LPUSH command "update system..."
	(integer) 1
	127.0.0.1:6379[15]> LPUSH request "visit page" 
	(integer) 1
	127.0.0.1:6379[15]> BLPOP job command request 0
	1) "command"
	2) "update system..."
	```
	
8. BRPOP
	BRPOP key [key ...] timeout
	BRPOP 是列表的阻塞式 (blocking) 弹出原语。
	它是RPOP 命令的阻塞版本，当给定列表内没有任何元素可供弹出的时候，连接将被BRPOP 命令阻塞，直	到等待超时或发现可弹出元素为止。
	当给定多个 key 参数时，按参数 key 的先后顺序依次检查各个列表，弹出第一个非空列表的尾部元素。
	关于阻塞操作的更多信息，请查看BLPOP 命令，BRPOP 除了弹出元素的位置和BLPOP 不同之外，其他表现一致。
	时间复杂度： O(1)
	返回值：
	　　假如在指定时间内没有任何元素被弹出，则返回一个 nil 和等待时长。
	　　反之，返回一个含有两个元素的列表，第一个元素是被弹出元素所属的 key ，第二个元素是被弹出元素的值。
	```shell
	127.0.0.1:6379[15]> LLEN course
	(integer) 0
	127.0.0.1:6379[15]> RPUSH course algorithm001
	(integer) 1
	127.0.0.1:6379[15]> RPUSH course c++101
	(integer) 2
	127.0.0.1:6379[15]> BRPOP course 30
	1) "course"
	2) "c++101"
	```
	
9. LINSERT
	LINSERT key BEFORE|AFTER pivot value
	将值 value 插入到列表 key 当中，位于值 pivot 之前或之后。
	当 pivot 不存在于列表 key 时，不执行任何操作。
	当 key 不存在时，key 被视为空列表，不执行任何操作。
	如果 key 不是列表类型，返回一个错误。
	时间复杂度: O(N)，N 为寻找 pivot 过程中经过的元素数量。
	返回值:
	　　如果命令执行成功，返回插入操作完成之后，列表的长度。
	　　如果没有找到 pivot ，返回 -1 。
	　　如果 key 不存在或为空列表，返回 0 。
	```shell
	127.0.0.1:6379[15]> RPUSH mylist "Hello"
	(integer) 1
	127.0.0.1:6379[15]> RPUSH mylist "World"
	(integer) 2
	127.0.0.1:6379[15]> LINSERT mylist BEFORE "World" "There"
	(integer) 3
	127.0.0.1:6379[15]> LRANGE mylist 0 -1
	1) "Hello"
	2) "There"
	3) "World"
	127.0.0.1:6379[15]> LINSERT mylist BEFORE "go" "let's"
	(integer) -1
	127.0.0.1:6379[15]> EXISTS fake_list
	(integer) 0
	127.0.0.1:6379[15]> LINSERT fake_list BEFORE "nono" "gogogog"
	(integer) 0
	```
	
10. LRANGE
	LRANGE key start stop
	返回列表 key 中指定区间内的元素，区间以偏移量 start 和 stop 指定。
	下标 (index) 参数 start 和 stop 都以 0 为底，也就是说，以 0 表示列表的第一个元素，以 1 表示列表的第二个元素，以此类推。
	你也可以使用负数下标，以 -1 表示列表的最后一个元素，-2 表示列表的倒数第二个元素，以此类推。
	注意 LRANGE 命令和编程语言区间函数的区别假如你有一个包含一百个元素的列表，对该列表执行 LRANGE list 0 10 ，结果是一个包含 11 个元素的列表，这表明 stop下标也在LRANGE命令的取值范围之内 (闭区间)，这和某些语言的区间函数可能不一致，比如 Ruby 的 Range.new 、Array#slice 和 Python 的 range() 函数。
	超出范围的下标值不会引起错误。
	如果 start 下标比列表的最大下标 end ( LLEN list 减去 1 ) 还要大，那么LRANGE 返回一个空列表。
	如果 stop 下标比 end 下标还要大，Redis 将 stop 的值设置为 end 。
	时间复杂度: O(S+N)，S 为偏移量 start ，N 为指定区间内元素的数量。
	返回值: 一个列表，包含指定区间内的元素。
	```shell
	127.0.0.1:6379[15]>  RPUSH fp-language lisp
	(integer) 1
	127.0.0.1:6379[15]>  LRANGE fp-language 0 0
	1) "lisp"
	127.0.0.1:6379[15]>  RPUSH fp-language scheme
	(integer) 2
	127.0.0.1:6379[15]>  LRANGE fp-language 0 1
	1) "lisp"
	2) "scheme"
	```
	
11. LREM
	LREM key count value
	根据参数 count 的值，移除列表中与参数 value 相等的元素。
	count 的值可以是以下几种：
	　　count > 0 : 从表头开始向表尾搜索，移除与 value 相等的元素，数量为 count 。
	　　count < 0 : 从表尾开始向表头搜索，移除与 value 相等的元素，数量为 count 的绝对值。
	　　count = 0 : 移除表中所有与 value 相等的值。
	时间复杂度： O(N)，N 为列表的长度。
	返回值：
	　　被移除元素的数量。
	　　因为不存在的 key 被视作空表 (empty list)，所以当 key 不存在时，LREM 命令总是返回 0 。
	```shell
	127.0.0.1:6379[15]> LPUSH greet "morning"
	(integer) 1
	127.0.0.1:6379[15]> LPUSH greet "hello"
	(integer) 2
	127.0.0.1:6379[15]> LPUSH greet "morning"
	(integer) 3
	127.0.0.1:6379[15]> LPUSH greet "hello"
	(integer) 4
	127.0.0.1:6379[15]> LPUSH greet "morning"
	(integer) 5
	127.0.0.1:6379[15]> LRANGE greet 0 4 
	1) "morning"
	2) "hello"
	3) "morning"
	4) "hello"
	5) "morning"
	127.0.0.1:6379[15]> LREM greet 2 morning
	(integer) 2
	127.0.0.1:6379[15]> LLEN greet
	(integer) 3
	127.0.0.1:6379[15]> LRANGE greet 0 2
	1) "hello"
	2) "hello"
	3) "morning"
	127.0.0.1:6379[15]> LREM greet -1 morning 
	(integer) 1
	127.0.0.1:6379[15]> LLEN greet
	(integer) 2
	127.0.0.1:6379[15]> LRANGE greet 0 1
	1) "hello"
	2) "hello"
	127.0.0.1:6379[15]> LREM greet 0 hello
	(integer) 2
	127.0.0.1:6379[15]> LLEN greet
	(integer) 0
	```
	
12. LTRIM
	LTRIM key start stop
	对一个列表进行修剪 (trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。
	举个例子，执行命令 LTRIM list 0 2 ，表示只保留列表 list 的前三个元素，其余元素全部删除。
	下标 (index) 参数 start 和 stop 都以 0 为底，也就是说，以 0 表示列表的第一个元素，以 1 表示列表的第二个元素，以此类推。
	你也可以使用负数下标，以 -1 表示列表的最后一个元素，-2 表示列表的倒数第二个元素，以此类推。当 key 不是列表类型时，返回一个错误。
	LTRIM 命令通常和LPUSH 命令或RPUSH 命令配合使用，举个例子：
	LPUSH log newest_log
	LTRIM log 0 99
	这个例子模拟了一个日志程序，每次将最新日志 newest_log 放到 log 列表中，并且只保留最新的 100 项。注意当这样使用 LTRIM 命令时，时间复杂度是 O(1)，因为平均情况下，每次只有一个元素被移除。
	注意 LTRIM 命令和编程语言区间函数的区别
	假如你有一个包含一百个元素的列表 list ，对该列表执行 LTRIM list 0 10 ，结果是一个包含 11 个元素的列表，这表明 stop 下标也在LTRIM 命令的取值范围之内 (闭区间)，这和某些语言的区间函数可能不一致，比如 Ruby 的 Range.new 、Array#slice 和 Python 的 range() 函数。
	超出范围的下标值不会引起错误。
	如果 start 下标比列表的最大下标 end ( LLEN list 减去 1 ) 还要大，或者 start > stop ，LTRIM 返回一个空列表 (因为LTRIM 已经将整个列表清空)。
	如果 stop 下标比 end 下标还要大，Redis 将 stop 的值设置为 end 。
	时间复杂度: O(N)，N 为被移除的元素的数量。
	返回值:命令执行成功时，返回 ok 。
	```shell
	# 情况 1： 常见情况， start 和 stop 都在列表的索引范围之内
	127.0.0.1:6379[15]> LPUSH alpha h e l l o
	(integer) 5
	127.0.0.1:6379[15]> LRANGE alpha 0 -1
	1) "o"
	2) "l"
	3) "l"
	4) "e"
	5) "h"
	127.0.0.1:6379[15]>  LTRIM alpha 1 -1 
	OK
	127.0.0.1:6379[15]> LRANGE alpha 0 -1	# o被删除了
	1) "l"
	2) "l"
	3) "e"
	4) "h"
	# 情况 2： stop 比列表的最大下标还要大
	127.0.0.1:6379[15]>  LTRIM alpha 1 10086 
	OK
	127.0.0.1:6379[15]>  LRANGE alpha 0 -1 # 只有索引 0 上的元素 "l" 被删除了，其他元素还在
	1) "l"
	2) "e"
	3) "h"
	127.0.0.1:6379[15]> LTRIM alpha 10086 123321
	OK
	# 情况 4： start 和 stop 都比列表的最大下标要大，并且 start > stop
	127.0.0.1:6379[15]> LRANGE alpha 0 -1
	(empty list or set)
	127.0.0.1:6379[15]> RPUSH new-alpha "h" "e" "l" "l" "o"
	(integer) 5
	127.0.0.1:6379[15]> LRANGE new-alpha 0 -1
	1) "h"
	2) "e"
	3) "l"
	4) "l"
	5) "o"
	127.0.0.1:6379[15]> LTRIM new-alpha 123321 10086 
	OK
	127.0.0.1:6379[15]> LRANGE new-alpha 0 -1 
	(empty list or set)
	```
	
13. LSET
	LSET key index value
	将列表 key 下标为 index 的元素的值设置为 value 。
	当 index 参数超出范围，或对一个空列表 ( key 不存在) 进行LSET 时，返回一个错误。
	时间复杂度：
	　　对头元素或尾元素进行LSET 操作，复杂度为 O(1)。
	　　其他情况下，为 O(N)，N 为列表的长度。
	返回值： 操作成功返回 ok ，否则返回错误信息。
	```shell
	127.0.0.1:6379[15]> EXISTS list
	(integer) 0
	127.0.0.1:6379[15]> LSET list 0 item
	(error) ERR no such key
	127.0.0.1:6379[15]> LPUSH job "cook food"
	(integer) 1
	127.0.0.1:6379[15]> LRANGE job 0 0
	1) "cook food"
	127.0.0.1:6379[15]> LSET job 0 "play game"
	OK
	127.0.0.1:6379[15]> LRANGE job 0 0
	1) "play game"
	127.0.0.1:6379[15]> LPUSH list a
	(integer) 1
	127.0.0.1:6379[15]> llen list
	(integer) 1
	127.0.0.1:6379[15]> LSET list 3 b
	(error) ERR index out of range
	```
	
14. LINDEX
	LINDEX key index
	返回列表 key 中，下标为 index 的元素。
	下标 (index) 参数 start 和 stop 都以 0 为底，也就是说，以 0 表示列表的第一个元素，以 1 表示列表的第二个元素，以此类推。
	你也可以使用负数下标，以 -1 表示列表的最后一个元素，-2 表示列表的倒数第二个元素，以此类推。如果 key 不是列表类型，返回一个错误。
	时间复杂度：
	　　O(N)，N 为到达下标 index 过程中经过的元素数量。
	　　因此，对列表的头元素和尾元素执行LINDEX 命令，复杂度为 O(1)。
	返回值:
	　　列表中下标为 index 的元素。
	　　如果 index 参数的值不在列表的区间范围内 (out of range)，返回 nil 。
	```shell
	127.0.0.1:6379[15]>  LPUSH mylist "World"
	(integer) 1
	127.0.0.1:6379[15]> LPUSH mylist "Hello"
	(integer) 2
	127.0.0.1:6379[15]>  LINDEX mylist 0
	"Hello"
	127.0.0.1:6379[15]>  LINDEX mylist -1
	"World"
	127.0.0.1:6379[15]>  LINDEX mylist 3 
	(nil)
	```
	
15. LLEN
	LLEN key
	返回列表 key 的长度。
	如果 key 不存在，则 key 被解释为一个空列表，返回 0 .
	如果 key 不是列表类型，返回一个错误。
	时间复杂度： O(1)
	返回值： 列表 key 的长度。
	```shell
	127.0.0.1:6379[15]> LLEN job
	(integer) 0
	127.0.0.1:6379[15]> LPUSH job "cook food"
	(integer) 1
	127.0.0.1:6379[15]> LPUSH job "have lunch"
	(integer) 2
	127.0.0.1:6379[15]> LLEN job
	(integer) 2
	```

16. RPOPLPUSH
	RPOPLPUSH source destination
	命令RPOPLPUSH 在一个原子时间内，执行以下两个动作：
	　　将列表 source 中的最后一个元素 (尾元素) 弹出，并返回给客户端。
	　　将 source 弹出的元素插入到列表 destination ，作为 destination 列表的的头元素。
	举个例子，你有两个列表 source 和 destination ，source 列表有元素 a, b, c ，destination 列表有元素 x, y, z ，执行 RPOPLPUSH source destination 之后，source 列表包含元素 a, b ，destination 列表包含元素 c, x, y, z ，并且元素 c 会被返回给客户端。
	如果 source 不存在，值 nil 被返回，并且不执行其他动作。
	如果 source 和 destination 相同，则列表中的表尾元素被移动到表头，并返回该元素，可以把这种特殊情况视作列表的旋转 (rotation) 操作。
	时间复杂度： O(1)
	返回值： 被弹出的元素。
	```shell
	127.0.0.1:6379[15]> LPUSH alpha d c b a
	(integer) 4
	127.0.0.1:6379[15]> LRANGE alpha 0 -1
	1) "a"
	2) "b"
	3) "c"
	4) "d"
	127.0.0.1:6379[15]> RPOPLPUSH alpha reciver
	"d"
	127.0.0.1:6379[15]> LRANGE alpha 0 -1
	1) "a"
	2) "b"
	3) "c"
	127.0.0.1:6379[15]> LRANGE reciver 0 -1
	1) "d"
	127.0.0.1:6379[15]> RPOPLPUSH alpha reciver
	"c"
	127.0.0.1:6379[15]> LRANGE alpha 0 -1
	1) "a"
	2) "b"
	127.0.0.1:6379[15]> LRANGE reciver 0 -1
	1) "c"
	2) "d"
	source 和 destination 相同
	127.0.0.1:6379[15]> RPUSH number 1 2 3 4
	(integer) 4
	127.0.0.1:6379[15]> LRANGE number 0 -1 
	1) "1"
	2) "2"
	3) "3"
	4) "4"
	127.0.0.1:6379[15]> RPOPLPUSH number number
	"4"
	127.0.0.1:6379[15]> LRANGE number 0 -1 
	1) "4"
	2) "1"
	3) "2"
	4) "3"
	127.0.0.1:6379[15]> RPOPLPUSH number number
	"3"
	127.0.0.1:6379[15]> LRANGE number 0 -1 
	1) "3"
	2) "4"
	3) "1"
	4) "2"
	```
	
17. BRPOPLPUSH
	BRPOPLPUSH source destination timeout
	BRPOPLPUSH 是RPOPLPUSH 的阻塞版本，当给定列表 source 不为空时，BRPOPLPUSH 的表现和RPOPLPUSH 一样。
	当列表 source 为空时，BRPOPLPUSH 命令将阻塞连接，直到等待超时，或有另一个客户端对 source 执行LPUSH 或RPUSH 命令为止。
	超时参数 timeout 接受一个以秒为单位的数字作为值。超时参数设为 0 表示阻塞时间可以无限期延长(block indefinitely) 。
	时间复杂度： O(1)
	返回值：
	　　假如在指定时间内没有任何元素被弹出，则返回一个 nil 和等待时长。
	　　反之，返回一个含有两个元素的列表，第一个元素是被弹出元素的值，第二个元素是等待时长。
	```shell
	127.0.0.1:6379[15]> BRPOPLPUSH msg reciver 500
	"hello"
	(84.71s)
	127.0.0.1:6379[15]> LLEN reciver
	(integer) 1
	127.0.0.1:6379[15]> LRANGE reciver 0 0
	1) "hello"
	127.0.0.1:6379[15]> BRPOPLPUSH msg reciver 1
	(nil)
	(1.82s)
	```