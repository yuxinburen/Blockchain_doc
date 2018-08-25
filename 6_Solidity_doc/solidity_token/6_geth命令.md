#### Geth命令介绍

##### 命令：

account 管理账户

attach 启动交互式JavaScript环境（连接到节点）

bug（上报bug）

console 启动交互式JavaScript环境

copydb从文件夹创建本地链

dump Dump（分析）一个特定的块存储

dumpconfig 显示配置值

export 导出区块链到文件

import 导入一个区块链文件

init 启动并初始化一个新的创世纪块

js 执行指定的JavaScript文件

license 显示许可信息

makecache 生成ethash验证缓存（用于测试）

makedag 生成ethash 挖矿DAG（用于测试）

monitor 监控和可视化节点指标

removedb 删除区块链和状态数据库

version 打印版本号

wallet 管理Ethereum预售钱包

help，h 显示一个命令或帮助一个命令列表

##### ETHEREUM选项：

--config value TOML配置文件

--datadir xxx 数据库和keytore密钥的数据目录

--keystore keystore存放目录(默认在datadir内)

--nousb 禁用监控和管理USB硬件钱包

--networkid value 网络标识符（整形，1=Frontier，2=Morden（弃用），3=Ropsten，4=Rinkeby）（默认：1）

--testnet Ropsten网络：预先配置的POW（proof－of－work）测试网络

--rinkeby Rinkeby网络：预先配置的POA（proof－of－authority）测试网络

--syncmode "fast" 同步模式（“fast”，“full”，or “light“）

--ethstats value 上报ethstats service URL(codename:secrett@host:port)

--identity value 自定义节点名

--lightserv value 允许LES请求时间最大百分比（0-90）默认值为：0

--lightpeers value 最大LES client peers数量（默认值：20）

--lightkdf 在KDF强度消费时降低key-derivation RAM&CPU使用

##### 开发者（模式）选项：

--dev 使用POA共识网络，默认预分配一个开发者账户并且会自动开启挖矿；

--dev.period value 开发者模式下挖矿周期（0=仅在交易时）默认值为：0

##### ETHASH选项：

--ethash.cachedir ethash验证缓存目录（默认＝datadir目录内）

--ethash.cachesinmem value 在内存保存的最近的ethash缓存个数（每个缓存16MB）默认为：2

--ethash.cachesondisk value 在磁盘保存的最近的ethash缓存个数（每个缓存16MB）默认值为：3

--ethash.dagdir "" 存ethash DAGs目录（默认＝用户hom目录）

--ethash.dagsinmem value 在内存保存的最近的ethash DAGs个数 （每个1GB以上）默认值为：1

--ethash.dagsondisk value 在磁盘保存的最近的ethash DAGs个数（每个1GB以上）默认值为：2

##### 交易池选项：

--txpool.nolocals 为本地提交交易禁用价格豁免

--txpool.journal value 本地交易的磁盘日志：用于节点重启（默认：“transactions.rlp")

--txpool.rejournal value 重新生成本地交易日志的时间间隔（默认：1小时）

--txpool.pricelimit value 加入交易池的最小的gas价格限制（默认：1）

--txpool.pricebump value 价格波动百分比(相对之前已有交易)(默认值:10)

--txpool.accountslots value 每个账户保证可执行的最少的交易槽数量（默认值：16）

--txpool.globalslots value 所有账户可执行的最大交易槽数量（默认值：4096）

--txpool.accountqueue value 每个账户允许的最多非可执行文件交易槽数量(默认：64)

--txpool.globalqueue value 所有账户非可执行文件交易最大槽数量（默认值：1024）

--txpool.lifetime value 非可执行交易最大入队时间（默认值：3小时）

##### 性能调优的选项：

--cache value 分配给内部缓存的内存MB数量，缓存值（最低值：16MB／数据库强制要求）（默认值：128）

--trie-cache-gens value 保存在内存中产生的trie node 数量（默认值：120）

##### 账户选项：

--unlock value 需要解锁账户；用逗号分割

--password value 用于非交互式密码输入的密码文件

##### API和控制台选项：

--rpc 启动HTTP－RPC服务器

--rpcaddr value HTTP-RPC服务器接口地址（默认值：“localhost”）

--rpcport value HTTP-RPC服务器监听接口（默认值：8545）

--rpcapi value 基于HTTP－RPC接口提供的API

--ws 启用WS－RPC服务器

--wsaddr value WS-RPC服务器监听接口地址（默认值：“localhost”）

--wsport value WS-RPC服务器监听接口端口(默认值：8546)

--wsapi value 基于WS－RPC的接口提供的API

--wsorigins value web socket请求允许的源

--ipcdisable 禁用IPC－RPC服务器

--ipcpath 包含在datadir里的IPC socket／pipe文件名（转义过的显示路径）

--rpccorsdomain value 允许跨域请求的域名列表

--jspath loadScript JavaScript加载脚本的根路径（默认值：”.")

--exec value 执行JavaScript语句（只能结合console/attach使用）

--preload value 预加载到控制台的JavaScript文件列表（逗号分隔）

##### 网络选项：

--bootnodes value 用于P2P发现引导的enode urls（逗号分隔）（对于light servers用v4＋v5代替）

--bootnodesv4 value 用于P2P v4发现引导的enode urls（逗号分隔）（light，server，全节点）

--bootnodesv5 value 用于P2P v5发现引导的enode urls（逗号分隔）（light，server，轻节点）

--port value 网卡监听端口（默认值：30303）

--maxpeers value 最大的网络节点数量（如果设置为0，网络将被禁用）（默认值为25）

--maxpendpeers value 最大尝试连接的数量（如果设置为0，则将使用默认值）（默认值为：0）

--nat value NAT端口映射机制（any｜none｜upnp｜pmp ｜extip：<ip>)（默认值：“any”）

--nodiscover  禁用节点发现机制（手动添加节点）

--v5disc  启用试验性的RLPx V5（Topic发现）机制

--nodekey value P2P节点密钥文件

--nodekeyhex value 十六进制的P2P节点密钥（用于测试）

##### 矿工选项：

--mine 打开挖矿

--minerthreads value 挖矿使用的CPU线程数量（默认值：8）

--etherbase value 挖矿奖励地址（默认-第一个创建的账户）（默认值：“0”）

--targetgaslimit value 目标gas机制：设置最低gas机制（低于这个不会被挖矿）（默认值：“4712388“）

--gasprice value 挖矿接受交易的最低gas价格

--extradata value 矿工设置的额外块数据（默认-client version)

##### GAS价格选项：

--gpoblocks value 用于检查gas价格的最近块的个数（默认值：10）

--gpopercentile value 建议gas价参考最近交易的gas价的百分位数（默认值：50）

##### 虚拟机的选项：

--vmdebug 记录VM及合约调试信息

##### 日志和调试选项：

--metrice 启用metrics收集和报告

--fakepow 禁用proof-of-work验证

--verbosity value 日志详细度：0-silent，1-error，2-warn，3－info，4-debug，5－detail（default：3）

--vmodule value 每个模块详细度。

##### WHISPER实验选项：

--shh 启用Whisper

--shh.maxmessagesize value 可接受的最大的消息大小（默认值：1048576）

--shh.pow value  可接受的最小的POW（默认值：0.2）

##### 弃用选项：

--fast 开启快速同步

--light 启用轻客户端模式

##### 常用命令：

- geth --datadir xxx --dev ：创建一个xxx的文件目录
- geth --dev console 2>>file_to_log_output ：打开控制台，在目录下生成一个叫file_to_log_output的日志文件
- eth.accounts ：查看当前有哪些账户
- personal.newAccount("密码")：创建一个新的账户，需要自己设置账户密码
- tail -f file_to_log_output：该命令能够打开日志文件
- miner.start()：启动挖矿命令
- miner.stop()：停止挖矿命令
- 系统默认挖矿所得的以太币传入第一个账户
- eth.sendTransaction({from:user1,to:user2,value:web3.toWei(3,"ether")})：从user1账户转账到user2账户3个eth
- eth.getBalance(user1)：查询user1用户的余额
- personal.unlockAccount(keystoredata,password)：解锁账户
- txpool.status：查看状态

