#### Ripemd160 加密算法

哈希值的输出值一般是`16`进制的字符串。而`16`进制字符串，每两个字符占一个字节。我们知道，一个字节`=8bit`. 

以`sha256`为例： 
- `256bit->64`位`16`进制字符。 


```
package main

import (
	"fmt"
	"crypto/sha256"
)

func main() {
	hasher := sha256.New()
	hasher.Write([]byte("The quick brown fox jumps over the lazy dog"))
	hashBytes := hasher.Sum(nil)
	hashString := fmt.Sprintf("%x", hashBytes)
	fmt.Println(hashString)
}
```

```
$ d7a8fbb307d7809469ca9abcb0082e4f8d5651e46d3cdb762d02d0bf37c9e592
```


而`ripemd`: 
- `160bit->40`位`16`进制字符。

$ cd $GOPATH/src
$ mkdir golang.org
$ cd golang.org
$ mkdir x
$ cd x
$ git clone https://github.com/golang/crypto.git

```
package main

import (
	"fmt"
	"golang.org/x/crypto/ripemd160"
)

func main() {
	hasher := ripemd160.New()
	hasher.Write([]byte("The quick brown fox jumps over the lazy dog"))
	hashBytes := hasher.Sum(nil)
	hashString := fmt.Sprintf("%x", hashBytes)
	fmt.Println(hashString)
}
```

```
$ 37f332f68db77bd9d7edd4969571ad671cf9dd3b
```