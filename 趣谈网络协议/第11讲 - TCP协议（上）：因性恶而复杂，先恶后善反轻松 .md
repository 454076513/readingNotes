# 第十一章 | TCP协议（上）：因性恶而复杂，先恶后善反轻松 

## 笔记

`TCP`天然认为网络环境是恶劣的, 丢包, 乱序, 重传, 拥塞都是常用的事情, 一言不合就可能送大不了了, **因为要从算法层面来保证可靠性**.

### TCP包头格式

![TCP包头](./img/11_01.jpg)

#### 序号

解决乱序问题, 确认哪个先来哪个后到.

#### 确认序号

确认发出去的包, 如果没有收到就应该重新发送, 知道送达. **解决不丢包的问题**

### TCP 保证可靠性

对于`TCP`来讲, `IP`层你丢不丢包, 我管不着, 但是我在我的层面上, 会努力保证可靠性.

### TCP 状态位

* `SYN`发起一个链接.
* `ACK`是回复.
* `RST`重新连接.
* `FIN`结束链接

`TCP`是面向连接的, 因而双方要维护连接的状态, 这些**带状态位的包的发送, 会引起双方的状态变更**.

**就像人与人之间的信任会经过多次交互才能建立**.

### 窗口大小(流量控制)

`TCP`要做流量控制, 通信双方各声明一个窗口, 标识自己当前能够的处理能力, 别发送的太快, 撑死我, 也别发的太慢, 饿死我.

### 拥塞控制

`TCP`拥塞控制. 控制自己, 也即**控制发送速度**. 不能改变世界, 就改变自己.

### TCP 特点

* 顺序问题, 稳重不乱.
* 丢包问题, 承诺靠谱.
* 连接维护, 有始有终.
* 流量控制, 把握分寸.
* 拥塞控制, 知进知退.

### TCP 的三次握手

`TCP`的连接建立, 称为三次握手.

```
A: 您好, 我是A.
B: 您好A, 我是B.
A: 您好 B.
```

**请求->应答->应答之应答**

* A 发起一个连接 **请求**
	* 第一个请求杳无音信
		* 包丢了
		* 包饶弯路, 超时了
		* B没有响应, 不想和我连接
	* 再次发送
	* 终于到达 B, A 暂时还不知道
* B 收到了请求包, 知道了 A 的存在, 知道 A 要和它建立链接. **应答**
	* B 不乐意建立连接, A 会重试一阵后放弃, 连接建立失败
	* B 乐意建立链接, 则会发送**应答包**给 A
		* **不能认为连接建立好**
			* 应答包会丢失
			* 应答包会饶弯路
			* A 已经挂了
	* B 发送的应答包可能会发送多次, 但是只要一次到达 A, **A 就认为连接已经建立了**, **对于A来说, 他的消息有去有回**
* A 给 B 发送**应答之应答**
	* B 也在等待这个消息, 才能确认连接的建立, 只有等到了这个消息, 对于 B 来讲, 才算他的**消息有去有回**
	* **应答之应答**也会丢失, 绕路, 甚至 B 挂了. 只要**双方的消息都有去有回, 就基本可以了**.

**需要双方发送的消息都`有去有回`**.

大部分情况下, A 和 B 建立了连接之后, A 会马上发送数据, 一旦 A 发送数据就解决了问题.

* **应答之应答**丢失. 当 A 连续发送数据的时候, B 可以认为这个连接已经建立.
* B 挂了. A 发送的数据, 会报错, 说 B 不可达, A 就知道 B 出事情了.

**keepalive**, 即使没有真实的数据包, 也有**探活包**.

如果 A 长时间不发包, B 可以主动关闭.

#### 为什么两次握手不行

* A 和 B 原来建立了连接, 做了简单通信, 结束了连接
	* 最早 A 第一次发起**请求**的时候, 重复发了很多次包, 如果这个时候**请求**到达. B 会认为这也是一个正常的请求的话, 因此**建立了连接**(如果两次握手就建立链接), 就没有终结了.	
	
#### TCP 包的序号问题

* A 要告诉 B, 发起包的**序号**
* B 同样要高速 A, 发起包的**序号**

**序号不能从1开始**, 这样往往会冲突.

```
A 发送 1 2 3. 

丢了 或者 绕路了.

A 重新发送 1 2 (这次不发3）

但是上次绕路的3 又回来了, 发给了B. B 就会错误的认为是下一个包. 

# 不能每次从1开始
```	

* 每个连接都要有不同的序号. 
* 这个序号的起始序号是随着时间变化的.
	* 32位的计数器, 每`4ms`加一.如果到重复, 需要4个多小时, 绕路的包早都死了. 以为`IP`包头里有个`TTL`, 也即生存时间.

#### 连接连接过程的状态变化

![连接状态变化](./img/11_02.jpg)

### TCP 四次挥手

* A: B 不玩了
* B: 你不玩了, 我知道了

B 不能在 ACK 的时候, 直接关闭. **有可能A是发完了最后的数据就准备不玩了, 但是 B 还没做完自己的事情, 还是可能在发送数据的**, 所以称为**半关闭**的状态.

A 可以选择不再接收数据, 也可以选择最后再接收一段数据, 等待 B 也主动关闭.

* B: A 好啊, 我也不玩了, 拜拜
* A: 好的, 拜拜

#### 解释

* A : B 不玩了 
* B: 你不玩了, 我知道了
* A 没收到回复
	* 重新发送"不玩了"
* A 收到回复
	* A 跑路了, B 发起的请求得不到A的应答
	* B 跑路了, A 不知道 B 是还有事情要处理, 还是过一会会发送结束

#### 断开时序图

![断开时序图](./img/11_03.jpg)

* A : B 不玩了, **FIN-WAIT-1**
* B: 你不玩了, 我知道了 **CLOSE-WAIT**
* A 收到回复 **FIN-WAIT-2**
	* B 跑路了, `Linux`调整`tcp_fin_timeout`这个参数, 设置超时时间
* B: A 好啊, 我也不玩了, 拜拜 **LAST_ACK**
* A: 好的, 拜拜 
	* 发送 ACK, **FIN-WAIT-2**结束. 
	* 如果这个 ACK, B 收不到.
		* B 重新发送一个 `A 好啊, 我也不玩了`, 如果这个时候 A 已经跑路了, B 就再也收不到 ACK 了.
	* `TCP`协议要求 A 最后等待一段时间**TIME-WAIT**, 这个时间要足够长到如果 B 没收到 ACK, B说 `A 好啊, 我也不玩了`会重发的
		* A 会重新发一个 ACK 并且足够时间到达 B
	* 如果 A 直接跑路, 端口就直接空出来了, 但是 B 不知道, B 原来发过的很多包可能都还在路上. 如果 A 的端口被一个新的应用占用了, 就会收到 B 发过来的包(虽然需要会重新生成). **双保险, 为了防止混乱, 需要等足够长的时间, 等待原来B发送的所有包都死翘翘, 再空出端口**.

#### 等待时间

等待时间设为`2MSL`.

**MSL 是 Maximum Segment Liftetime**, 报文最大生存时间. 它是任何报文在网络上存在的最长时间, 超过这个时间报文将被丢弃.

`IP`头中有一个`TTL`, 是`IP`数据报可以经过的**最大路由数**, 每经过一个处理他的路由器此值就减1, 当此值为0则数据报将被丢弃, 同时发送`ICMP`报文通知主机.

协议规定`MSL`为2分钟, 实际应用中常用的是30秒, 1分钟和2分钟等.

#### 查过了2MSL

如果 B 超过了 2MSL 的时间, 依然没收到它发的`FIN`的`ACK`.

* 按照 TCP 的原理, B 重发`FIN`
* A 收到`FIN`, 表示超过时间了. 直接发送 `RST`

### TCP 状态机

![TCP状态机](./img/11_04.jpg)

* 数字是连接状态变化
* 虚线是 A 的连接
* 实线是 B 的连接

## 扩展