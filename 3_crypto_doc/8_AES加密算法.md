# AES加密算法原理

AES是作为DES的替代标准出现的，全称Advanced Encryption Standard，即：高级加密标准。AES加密算法，经历了公开的选拔，最终2000年，由比利时密码学家Joan Daemen和Vincent Rijmen设计的Rijndael算法被选中，成为了AES标准。
**AES明文分组长度为128位，即16个字节，密钥长度可以为16个字节、24个字节、或32个字节，即128位密钥、192位密钥、或256位密钥。**

## 总体结构
　　AES中没有使用Feistel网络，其结构称为SPN结构。
　　
　　和DES相同，AES也由多个轮组成，其中每个轮分为SubBytes、ShiftRows、MixColumns、AddRoundKey 4个步骤，即：字节代替、行移位、列混淆和轮密钥加。根据密钥长度不同，所需轮数也不同，128位、192位、256位密钥，分别需要10轮、12轮和14轮。第1轮之前有一次AddRoundKey，即轮密钥加，可以视为第0轮；之后1至N-1轮，执行SubBytes、ShiftRows、MixColumns、AddRoundKey；最后一轮仅包括：SubBytes、MixColumns、AddRoundKey。
　　
AES总体结构示意图：
　　
![结构示意图](http://olgjbx93m.bkt.clouddn.com/201801010001.png)

### 加密整体流程

加密的代码实现：

```
func encryptBlockGo(xk []uint32, dst, src []byte) {
	var s0, s1, s2, s3, t0, t1, t2, t3 uint32
	//按4x4矩阵排列
	s0 = uint32(src[0])<<24 | uint32(src[1])<<16 | uint32(src[2])<<8 | uint32(src[3])
	s1 = uint32(src[4])<<24 | uint32(src[5])<<16 | uint32(src[6])<<8 | uint32(src[7])
	s2 = uint32(src[8])<<24 | uint32(src[9])<<16 | uint32(src[10])<<8 | uint32(src[11])
	s3 = uint32(src[12])<<24 | uint32(src[13])<<16 | uint32(src[14])<<8 | uint32(src[15])

	// 第1次轮密钥加
	s0 ^= xk[0]
	s1 ^= xk[1]
	s2 ^= xk[2]
	s3 ^= xk[3]

	//nr为中间重复轮数
    //例如128位密钥，44字子密钥，此处为9轮
    //-2位去除开头和结尾轮
	nr := len(xk)/4 - 2 
	
	//4表示已使用了4个字子密钥
	k := 4
	for r := 0; r < nr; r++ {
		//此处代码包括字节代替、行移位、列混淆、轮密钥加
		t0 = xk[k+0] ^ te0[uint8(s0>>24)] ^ te1[uint8(s1>>16)] ^ te2[uint8(s2>>8)] ^ te3[uint8(s3)]
		t1 = xk[k+1] ^ te0[uint8(s1>>24)] ^ te1[uint8(s2>>16)] ^ te2[uint8(s3>>8)] ^ te3[uint8(s0)]
		t2 = xk[k+2] ^ te0[uint8(s2>>24)] ^ te1[uint8(s3>>16)] ^ te2[uint8(s0>>8)] ^ te3[uint8(s1)]
		t3 = xk[k+3] ^ te0[uint8(s3>>24)] ^ te1[uint8(s0>>16)] ^ te2[uint8(s1>>8)] ^ te3[uint8(s2)]
		k += 4
		s0, s1, s2, s3 = t0, t1, t2, t3
	}

	//最后一轮仅包括字节代替、行移位、轮密钥加
    //此处为字节代替和行移位
	s0 = uint32(sbox0[t0>>24])<<24 | uint32(sbox0[t1>>16&0xff])<<16 | uint32(sbox0[t2>>8&0xff])<<8 | uint32(sbox0[t3&0xff])
	s1 = uint32(sbox0[t1>>24])<<24 | uint32(sbox0[t2>>16&0xff])<<16 | uint32(sbox0[t3>>8&0xff])<<8 | uint32(sbox0[t0&0xff])
	s2 = uint32(sbox0[t2>>24])<<24 | uint32(sbox0[t3>>16&0xff])<<16 | uint32(sbox0[t0>>8&0xff])<<8 | uint32(sbox0[t1&0xff])
	s3 = uint32(sbox0[t3>>24])<<24 | uint32(sbox0[t0>>16&0xff])<<16 | uint32(sbox0[t1>>8&0xff])<<8 | uint32(sbox0[t2&0xff])

	//轮密钥加
	s0 ^= xk[k+0]
	s1 ^= xk[k+1]
	s2 ^= xk[k+2]
	s3 ^= xk[k+3]
	
    //输出
	dst[0], dst[1], dst[2], dst[3] = byte(s0>>24), byte(s0>>16), byte(s0>>8), byte(s0)
	dst[4], dst[5], dst[6], dst[7] = byte(s1>>24), byte(s1>>16), byte(s1>>8), byte(s1)
	dst[8], dst[9], dst[10], dst[11] = byte(s2>>24), byte(s2>>16), byte(s2>>8), byte(s2)
	dst[12], dst[13], dst[14], dst[15] = byte(s3>>24), byte(s3>>16), byte(s3>>8), byte(s3)
}
```

### 密钥扩展

Nk表示初始密钥的字数，一个字为4个字节，以16字节初始密钥为例，初始密钥共计4个字。AES加密过程共计需Nk+7个子密钥，即4(Nk + 7)个字，以16字节初始密钥为例，共计需11个子密钥，44个字。其中前Nk个字作为种子密钥，由初始密钥填充。以16字节初始密钥为例，前4个字，由初始密钥填充。之后的每个字，W[i]等于前边一个字W[i-1]与Nk个字之前的字W[i-Nk]异或。但是对于Nk整数倍位置的字，在异或之前先对W[i-1]做如下变换：字节循环移位、S盒变换，并异或轮常数。

密钥扩展示意图

![](http://olgjbx93m.bkt.clouddn.com/201801010002.jpg)

```
func expandKeyGo(key []byte, enc, dec []uint32) {
	// Encryption key setup.
	var i int
	nk := len(key) / 4
	for i = 0; i < nk; i++ {
		//其中前Nk个字作为种子密钥，由初始密钥填充
        //以16字节初始密钥为例，前4个字，由初始密钥填充
		enc[i] = uint32(key[4*i])<<24 | uint32(key[4*i+1])<<16 | uint32(key[4*i+2])<<8 | uint32(key[4*i+3])
	}
	for ; i < len(enc); i++ {
		t := enc[i-1]
		if i%nk == 0 {
			//但是对于Nk整数倍位置的字，在异或之前先对W[i-1]做如下变换：字节循环移位、S盒变换，并异或轮常数
			t = subw(rotw(t)) ^ (uint32(powx[i/nk-1]) << 24)
		} else if nk > 6 && i%nk == 4 {
			t = subw(t)
		}
		//之后的每个字，W[i]等于前边一个字W[i-1]与Nk个字之前的字W[i-Nk]异或
		enc[i] = enc[i-nk] ^ t
	}

	//enc为加密子密钥组
    //dec为解密子密钥组，dec为enc逆序
	if dec == nil {
		return
	}
	n := len(enc)
	for i := 0; i < n; i += 4 {
		ei := n - i - 4
		for j := 0; j < 4; j++ {
			x := enc[ei+j]
			if i > 0 && i+4 < n {
				x = td0[sbox0[x>>24]] ^ td1[sbox0[x>>16&0xff]] ^ td2[sbox0[x>>8&0xff]] ^ td3[sbox0[x&0xff]]
			}
			dec[i+j] = x
		}
	}
}
```

### 字节代替SubBytes

AES定义了一个S盒，它由16x16个字节组成的矩阵，包含了8位所能表示的256个数的一个置换表。AES的分组长度128位，即16字节，每个字节高4位作为行值、低4位作为列值，从S盒中查找指定行、列的值作为输出。AES输入的16个字节，每个字节按上述方式映射为新的字节，即字节代替。

### 行移位ShiftRows

AES输入的16个字节组成4x4字节矩阵，行移位即：第1行保持不变，第2行循环左移1个字节，第3行循环左移2个字节，第4行循环左移3个字节。逆向行移位为后3行做行移位的反向操作，如第2行循环右移1个字节，其他行类似处理。

行移位示意图如下
![](http://olgjbx93m.bkt.clouddn.com/201801010003.jpg)

### 列混淆MixColumns

　　列混淆和逆向列混淆，实际上是使用乘法矩阵，但是其加法和乘法均为定义在有限域上的加法和乘法。

　　如下为正向列混淆：
　　
　　![](http://olgjbx93m.bkt.clouddn.com/201801010005.jpg)
　　
　　如下为逆向列混淆：
　　
　　![](http://olgjbx93m.bkt.clouddn.com/201801010004.jpg)
　　
### 轮密钥加AddRoundKey

轮密钥加，即128位输入与128位轮密钥做异或运算。


## 动画演示AES

https://coolshell.cn/articles/3161.html