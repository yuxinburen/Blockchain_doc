##### MPT结构源码分析

包trie实现了Merkle Patricia Tries，通畅习惯用简称MPT来表示这种数据结构，这种数据结构实际上是一种Tie树变种，MPT是以太坊中一种非常重要的数据结构。在以太坊中使用MPT这种数据结构来存储账户的状态及其变更，交易信息，交易的收据信息等。MPT实际上是三种数据结构的组合，分别是Tire树，Patricia Trie和Merkle树。

##### Trie树介绍

Trie树，又称为字典树，单词查找树或者前缀树，是一种用于快速检索的多叉树结构。比如，英文字母的字典树是一个26叉树，数字的字典树是一个10叉树。

Trie树可以利用字符串的公共前缀来节省存储空间。如下图，一个Trie树用10个节点保存了6个字符串分别是：tea，ten，to，in，inn，int；

![image-20180830203603457](/var/folders/_n/0wz2n7dx27gcl3hd2sk1cfth0000gn/T/abnerworks.Typora/image-20180830203603457.png)

在Trie树中，字符串in，inn和int的公共前缀是“in”。因此可以只存储一份‘in’以节省空间。当然，如果系统中存在大量字符串且这些字符串基本没有公共前缀，则相应的trie树将非常消耗内存，这就是trie树结构的两面性。

综上，Trie树的基本性质可以归纳为：

- 根节点不包含字符，除根节点以外每个节点只包含一个字符。
- 从根节点到某一个节点，路径上经过的字符连接起来，为该节点对应的字符串。
- 每个节点的所有子节点包含的字符串不相同。

##### Patricia Tries（前缀树）介绍

前缀树根Trie树不同之处在于Trie树给每一个字符串分配一个节点，这样将使那些很长但是又没有公共节点的字符串的Trie树退化成数组。在以太坊里面会由黑客构造很多的这种节点造成拒绝服务攻击。前缀树的不同之处在于如果节点公共前缀，安么就使用公共前缀，否则就把剩下的所有节点插入同一个节点。Paticia相对Trie的优化如下：

![image-20180830204600338](/var/folders/_n/0wz2n7dx27gcl3hd2sk1cfth0000gn/T/abnerworks.Typora/image-20180830204600338.png)

##### Merkle树介绍

Merkle Tree，通畅也被称作Hash Tree，顾名思义，就是存储hash值的一棵树。Merkele树的叶子是数据块（例如，文件或者文件的集合）的hash值。非叶节点是其对应子节点串联字符串的hash。

![image-20180830225355748](/var/folders/_n/0wz2n7dx27gcl3hd2sk1cfth0000gn/T/abnerworks.Typora/image-20180830225355748.png)

Merkle Tree的主要作用是当我拿到Top Hash的时候，这个hash值代表了整棵树的信息摘要，当树里面任何一个数据发生变化，都会导致Top Hash的值发生变化。而Top Hash的值发生。而Top Hash的值是会存储到区块链的区块头里面的，区块头是必须经过工作量证明。这也就是说我只要拿到一个区块头，就可以对区块信息进行验证。

##### 以太坊的MPT

每一个以太坊的区块头包含三颗MPT，分别是：

- 交易树
- 收据树（交易执行过程中的一些数据）
- 状态树（账号信息，合约账户和用户账户）

##### 源码实现：trie/encoding.go

encodeing.go文件主要是处理trie树中的三种编码格式的相互转换的工作。三种编码格式分别为以下三种编码格式：

- KEYBYTES encoding这种编码格式就是原生的key字节数组，大部分的Trie的API都是使用这样的编码格式
- HEX encoding这种编码格式每一个字节包含了Key的一个半字节，尾部接上一个可选的‘终结符’，‘终结符’代表这个节点到底是叶子节点还是扩展节点。
- COMPACT encoding：这种编码格式就是Hex－Prefix Encoding，这种编码格式可以看成是*Hex encoding*这种编码格式的另外一个版本，可以在存储到数据库的时候节约磁盘空间。


简单理解：将普通的字节序列keybytes编码为带有t标志与奇数个半个字节nibble标志位的keybytes

- keybytes为按完整节点（8bit）存储的正常信息
- hex为按照半字节nibble（4bit）存储信息的格式。供compact使用
- 为了便于作Modified Merkle Patricia Tree的节点key，编码为偶数字节长度的hex格式。其第一个半字节nibble会在低的2个bit位中，由高到低分别存放t标志与奇数标志。经compact编码的keybytes，在增加了hex的t标志与半字节nibble为偶数个的情况下，便于存储。

```
func hexToCompact(hex []byte) []byte {
	terminator := byte(0)
	if hasTerm(hex) {
		terminator = 1
		hex = hex[:len(hex)-1]
	}
	buf := make([]byte, len(hex)/2+1)
	buf[0] = terminator << 5 // the flag byte
	if len(hex)&1 == 1 {
		buf[0] |= 1 << 4 // odd flag
		buf[0] |= hex[0] // first nibble is contained in the first byte
		hex = hex[1:]
	}
	decodeNibbles(hex, buf[1:])
	return buf
}

func compactToHex(compact []byte) []byte {
	base := keybytesToHex(compact)
	// delete terminator flag
	if base[0] < 2 {
		base = base[:len(base)-1]
	}
	// apply odd flag
	chop := 2 - base[0]&1
	return base[chop:]
}

func keybytesToHex(str []byte) []byte {
	l := len(str)*2 + 1
	var nibbles = make([]byte, l)
	for i, b := range str {
		nibbles[i*2] = b / 16
		nibbles[i*2+1] = b % 16
	}
	nibbles[l-1] = 16
	return nibbles
}

// hexToKeybytes turns hex nibbles into key bytes.
// This can only be used for keys of even length.
func hexToKeybytes(hex []byte) []byte {
	if hasTerm(hex) {
		hex = hex[:len(hex)-1]
	}
	if len(hex)&1 != 0 {
		panic("can't convert hex key of odd length")
	}
	key := make([]byte, len(hex)/2)
	decodeNibbles(hex, key)
	return key
}

// prefixLen returns the length of the common prefix of a and b.
func prefixLen(a, b []byte) int {
	var i, length = 0, len(a)
	if len(b) < length {
		length = len(b)
	}
	for ; i < length; i++ {
		if a[i] != b[i] {
			break
		}
	}
	return i
}
```

##### 数据结构

node的结构，可以在trie包下的node.go文件中，看到node的结构类型的声明，node分为4种类型：fullNode为分支节点，shortNode代表扩展节点和叶子节点，还有hashNode，valueNode两个变量。

```
type node interface {
	fstring(string) string
	cache() (hashNode, bool)
	canUnload(cachegen, cachelimit uint16) bool
}

type (
	fullNode struct {
		Children [17]node // Actual trie node data to encode/decode (needs custom encoder)
		flags    nodeFlag
	}
	shortNode struct {
		Key   []byte
		Val   node
		flags nodeFlag
	}
	hashNode  []byte
	valueNode []byte
)
```

trie结构中，root包含了当前的root节点，db是后端的KV存储，trie的结构最终都是需要通过KV的形式存储到数据库里面去，然后启动的时候是需要从数据库里面加载的。originalRoot启动加载的时候的hash值，通过这个hash值可以在数据库里面恢复出整颗的trie树。cachegen字段指示了当前trie树的cache，每次调用commit的时候，会增加Trie树的cache时代。cache时代会被附加在node节点上面，如果当前的cache时代-cachelimit参数大于node的cache时代，那么node会从cache里面卸载，以便节约内存。其实本质就是LRU算法。

##### Trie树的插入

Trie树的插入，查找和删除Trie树的初始化调用New函数，函数接受一个hash和一个Database参数，如果hash值不是空值的变化，就说明是从数据库加载一个已经存在的Trie树，就调用trie.resolveHash方法来加载整颗Trie树。如果root是空，那么就新建一颗Trie树进行返回。

```
func New(root common.Hash, db Database) (*Trie, error) {
	trie := &Trie{db: db, originalRoot: root}
	if (root != common.Hash{}) && root != emptyRoot {
		if db == nil {
			panic("trie.New: cannot use existing root without a database")
		}
		rootnode, err := trie.resolveHash(root[:], nil)
		if err != nil {
			return nil, err
		}
		trie.root = rootnode
	}
	return trie, nil
}
```

Trie树的插入，这个是一个递归调用的方法，从根节点开始，一直往下找，直到找到可以插入的点，进行插入操作。参数node是当前插入的节点，prefix是当前已经处理完的部分key，key是还没有处理完的部分key，完整的key＝prefix+key。value是需要插入的值。返回值bool是操作是否改变了Trie树（dirty），node是插入完成后的子树的根节点，error是错误信息。

- 如果节点类型是nil，这个时候整棵树都是空的，直接返回shortNode{key,value,t.newFlag()},这个时候整棵树的根就含有了一个shortNode节点。
- 如果当前的根节点类型是shortNode（也就是叶子节点），首先计算公共前缀，如果公共前缀就等于key，那么说明这两个key是一样的，如果value也一样的（dirty＝＝false），那么返回错误。如果没有错误直接更新shortNode的值然后返回。如果公共前缀不完全匹配，那么就需要把公共前缀提取出来形成一个独立的节点（扩展节点），扩展节点后面连接一个branch节点，branch节点后面看情况连接两个short节点。首先构建一个branch节点（brnch：＝&fullNode{flags:t.newFlag()})，然后再branch节点的Children位置调用t.insert插入剩下的两个short节点。
- 如果当前的节点是fullNode（也就是branch节点），那么直接往对应的孩子节点调用insert方法，然后把对应的孩子节点指向新的生成的节点。
- 如果当前节点是hashNode，hashNode的意思是当前节点还没有加载到内存里面来，还是存放在数据库里面，那么首先调用t.resolveHash(n,prefix)来加载到内存，然后对加载出来的节点调用insert方法来进行插入。

```
func (t *Trie) insert(n node, prefix, key []byte, value node) (bool, node, error) {
	if len(key) == 0 {
		if v, ok := n.(valueNode); ok {
			return !bytes.Equal(v, value.(valueNode)), value, nil
		}
		return true, value, nil
	}
	switch n := n.(type) {
	case *shortNode:
		matchlen := prefixLen(key, n.Key)
		// If the whole key matches, keep this short node as is
		// and only update the value.
		if matchlen == len(n.Key) {
			dirty, nn, err := t.insert(n.Val, append(prefix, key[:matchlen]...), key[matchlen:], value)
			if !dirty || err != nil {
				return false, n, err
			}
			return true, &shortNode{n.Key, nn, t.newFlag()}, nil
		}
		// Otherwise branch out at the index where they differ.
		branch := &fullNode{flags: t.newFlag()}
		var err error
		_, branch.Children[n.Key[matchlen]], err = t.insert(nil, append(prefix, n.Key[:matchlen+1]...), n.Key[matchlen+1:], n.Val)
		if err != nil {
			return false, nil, err
		}
		_, branch.Children[key[matchlen]], err = t.insert(nil, append(prefix, key[:matchlen+1]...), key[matchlen+1:], value)
		if err != nil {
			return false, nil, err
		}
		// Replace this shortNode with the branch if it occurs at index 0.
		if matchlen == 0 {
			return true, branch, nil
		}
		// Otherwise, replace it with a short node leading up to the branch.
		return true, &shortNode{key[:matchlen], branch, t.newFlag()}, nil

	case *fullNode:
		dirty, nn, err := t.insert(n.Children[key[0]], append(prefix, key[0]), key[1:], value)
		if !dirty || err != nil {
			return false, n, err
		}
		n = n.copy()
		n.flags = t.newFlag()
		n.Children[key[0]] = nn
		return true, n, nil

	case nil:
		return true, &shortNode{key, value, t.newFlag()}, nil

	case hashNode:
		// We've hit a part of the trie that isn't loaded yet. Load
		// the node and insert into it. This leaves all child nodes on
		// the path to the value in the trie.
		rn, err := t.resolveHash(n, prefix)
		if err != nil {
			return false, nil, err
		}
		dirty, nn, err := t.insert(rn, prefix, key, value)
		if !dirty || err != nil {
			return false, rn, err
		}
		return true, nn, nil

	default:
		panic(fmt.Sprintf("%T: invalid node: %v", n, n))
	}
}
```

