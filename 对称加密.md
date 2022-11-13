# 常用加密算法学习总结之对称加密




**对称加密**，又称为 **共享密钥加密算法**，是指加密和解密方使用相同密钥的加密算法。对称加密算法的优点在于加解密的高速度和使用长密钥时的难破解性。

## 对称加密算法

![](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/1380340-20201031172028652-1698637107.png)

### DES

DES（Data Encryption Standard）：数据加密标准，速度较快，适用于加密大量数据的场合。1977年被美国联邦政府的国家标准局确定为联邦资料处理标准（FIPS）

#### DES的加密和解密

DES是一种将64bit（8Byte）的明文加密成64bit的密文的对称密码算法，==它的密钥长度是56比特==。&lt;font color=&#34;red&#34;&gt;从规格上来说，DES的密钥长度是64bit，但由于每隔7bit会设置一个用于==错误检查==的比特，因此实质上其密钥长度是56bit&lt;/font&gt;。

&lt;font color=&#34;red&#34;&gt;DES是以64bit的明文（比特序列）为一个单位来进行加密的&lt;/font&gt;，**这个64bit的单位称为分组**。一般来说，以分组为单位进行处理的密码算法称为**分组密码（blockcipher）**，DES就是分组密码的一种。

![](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/1380340-20201031173939221-127508220.png)


DES每次只能加密64比特的数据，如果要加密的明文比较长，就需要对DES加密进行迭代（反复），而迭代的具体方式就称为模式（mode）。

### 3DES

3DES（Triple DES）：是三重数据加密算法（TDEA，Triple Data Encryption Algorithm）块密码的通称。是基于DES，对一块数据用三个不同的密钥进行三次加密，强度更高。

3DES是基于计算机的运算能力的增强，基于DES算法，增强秘钥进行多绪加密，而不是一种块密码算法。

### AES

AES（Advanced Encryption Standard）：高级加密标准，是美国联邦政府采用的一种区块加密标准。

### 分组密码模式

**分组密码（`blockcipher`）**是每次只能处理特定长度的一块数据的一类密码算法，这里的一块&#34;就称为分组（block）。此外，一个分组的比特数就称为分组长度（`blocklength`）。

例如，**DES和3DES的分组长度都是64比特**。这些密码算法一次只能加密64比特的明文．并生成64比特的密文。

**AES的分组长度可以从128比特、192比特和256比特中进行选择。当选择128比特的分组长度时，AES一次可加密128比特的明文，并生成128比特的密文。**

**分组密码算法只能加密固定长度的分组，但是我们需要加密的明文长度可能会超过分组密码的分组长度，这时就需要对分组密码算法进行迭代，以便将一段很长的明文全部加密。而迭代的方法就称为分组密码的模式（mode）**。

分组密码的模式有很多种类，分组密码的主要模式有以下5种：

#### 明文与密文分组

- **明文分组: **是指分组密码算法中作为加密对象的明文。明文分组的长度与分组密码算法的分组长度是相等的。
- **密文分组: **是指使用分组密码算法将明文分组加密之后所生成的密文。

![](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/1380340-20201031180750458-1657314183.png)


####  ECB模式：Electronic Code Book mode（电子密码本模式）

ECB是最简单的加密模式，&lt;font color=&#34;red&#34;&gt;明文消息被分成固定大小的块（分组），并且每个块被单独加密。&lt;/font&gt;  每个块的加密和解密都是独立的，且使用相同的方法进行加密，所以可以进行并行计算，但是这种方法一旦有一个块被破解，使用相同的方法可以解密所有的明文数据，&lt;font color=&#34;red&#34;&gt;安全性比较差。  适用于数据较少的情形，加密前需要把明文数据填充到块大小的整倍数。&lt;/font&gt;

![](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/1380340-20201031180917418-1828892139.png)
![](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/1380340-20201031180907816-1922224853.png)

&gt; 使用ECB模式加密时，相同的明文分组会被转换为相同的密文分组，因此ECB模式也称为电子密码本模式**当最后一个明文分组的内容小于分组长度时（如一个分组8bit），需要用一特定的数据进行填充（padding），让值一个分组长度等于分组长度**。

&gt; ECB模式是所有模式中最简单的一种。ECB模式中，明文分组与密文分组是一一对应的关系，因此，如果明文中存在多个相同的明文分组，则这些明文分组最终都将被转换为相同的密文分组。这样一来，只要观察一下密文，就可以知道明文中存在怎样的重复组合，并可以以此为线索来破译密码，因此ECB模式是存在一定风险的。

####  CBC模式：Cipher Block Chaining mode（密码分组链接/密码块 模式）

1976年，IBM发明了密码分组链接CBC。CBC&lt;font color=&#34;red&#34;&gt;模式中每一个分组要先和前一个分组加密后的数据进行XOR异或操作，然后再进行加密&lt;/font&gt;。 这样每个密文块依赖该块之前的所有明文块，为了保持每条消息都具有唯一性，&lt;font color=&#34;red&#34;&gt;在**第一个块**进行加密之前需要用初始化向量 `IV` 进行异或操作&lt;/font&gt;。  &lt;font color=&#34;blue&#34;&gt;CBC模式是一种最常用的加密模式，它主要缺点是加密是连续的，不能并行处理，并且与ECB一样消息块必须填充到块大小的整倍数。&lt;/font&gt;

![](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/1380340-20201031194830815-323142789.png)

![](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/1380340-20201031194838945-4927688.png)

**当加密第一个明文分组时，由于不存在 “前一个密文分组&#34;，因此&lt;font color=&#34;red&#34;&gt;需要事先准备一个长度为一个分组的比特序列来代替“前一个密文分组&lt;/font&gt;&#34;，这个比特序列称为初始化向量（initialization vector）**通常缩写为 `IV`。一般来说，每次加密时都会随机产生一个不同的比特序列来作为初始化向量。

####  CFB模式：Cipher FeedBack mode（密文反馈模式）

密文反馈模式 CFB；在CFB模式中，&lt;font color=&#34;red&#34;&gt;前一个分组的密文加密后和当前分组的明文XOR异或操作生成当前分组的密文&lt;/font&gt;。所谓反馈，这里指的就是返回输入端的意思，即前一个密文分组会被送回到密码算法的输入端。

![](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/1380340-20201031200320644-9430185.png)
![](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/1380340-20201031200333802-435217010.png)

在ECB和CBC中，明文分组都是通过密码算法进行加密的，然而，在CFB模式中，明文分组和密文分组之间并没有经过&#34;加密&#34;这一步骤，明文分和密文分组之间只有一个XOR。

####  OFB模式：Output FeedBack mode（输出反馈模式）

输出反馈模式, OFB。在OFB模式中，上一个分组密码算法的输出是当前分组密码算法的输入（下图）

![](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/1380340-20201031201815366-1110323402.png)

![](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/1380340-20201031201835300-606590872.png)

####  **CTR模式**：CounTeR mode（计数器模式）  

&lt;font color=&#34;red&#34;&gt;CTR是一种通过将逐次累加的计数器进行加密来生成密钥流的流密码&lt;/font&gt;；即每个分组对应一个逐次累加的计数器，并通过对计数器进行加密来生成密钥流。也就是说，最终的密文分组是通过将计数器加密得到的比特序列，与明文分组进行XOR而得到的。

![](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/1380340-20201031202045165-1475648471.png)

![](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/1380340-20201031202056336-731754020.png)

**CTR模式的特点**

&gt; CTR模式的加密和解密使用了完全相同的结构，因此在程序实现上比较容易。这一特点和同为流密码的OFB模式是一样的。
&gt; CTR模式中可以以任意顺序对分组进行加密和解密，因此在加密和解密时需要用到的“计数器&#34;的值可以由nonce和分组序号直接计算出来。这一性质是OFB模式所不具备的。
&gt; CTR模式能够以任意顺序处理分组，就意味着能够实现并行计算。在支持并行计算的系统中，CTR模式的速度是非常快的。

总结

![](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/1380340-20201031202458031-1831855010.png)

&gt; 初始化向量 - IV
&gt; - ecb, ctr模式不需要初始化向量
&gt; - cbc, ofc, cfb需要初始化向量

&gt; 最后一个明文分组的填充
&gt; - 使用cbc, ecb需要填充
&gt;   - 明文分组中进行了填充, 然后加密
&gt;   - 解密密文得到明文, 需要把填充的字节删除
&gt; - 使用 ofb, cfb, ctr不需要填充


## 对称加密在Go语言中的实现方式

### CBC分组模式

```go
/*
 *  @brief DES加密函数，
 *  @param1 加密的明文
 *  @param2 秘钥
 *  @return，得到的密文
 */

func DesEncrypt(plainText, key string) ([]byte, error) {

	var (
		// 创建一个des加密的接口
		block, err = des.NewCipher([]byte(key))
		// 分组加密 需要对最后进行填充
		padText    = LastPadding([]byte(plainText), block.BlockSize())
		cipherText = make([]byte, len(padText))
	)

	if err != nil {
		return nil, err
	}
	// 创建使用cbc分组模式加密接口
	mode := cipher.NewCBCEncrypter(block, []byte(&#34;12345678&#34;))
	// 加密
	mode.CryptBlocks(cipherText, padText)
	return cipherText, nil
}

/*
 *  @brief DES解密函数，
 *  @param1 加密的明文
 *  @param2 秘钥
 *  @return，得到的密文
 */

func DesDecrypt(cipherText, key string) ([]byte, error) {

	var (
		// 创建一个des加密的接口
		block, err = des.NewCipher([]byte(key))
		// 创建使用cbc分组模式解密接口
		mode           = cipher.NewCBCDecrypter(block, []byte(&#34;12345678&#34;))
		byteCipherText = []byte(cipherText)
		// 明文存储变量
		plainText = make([]byte, len(byteCipherText))
	)

	if err != nil {
		return nil, err
	}
	// 解密，无返回值
	mode.CryptBlocks(plainText, byteCipherText)
	// 将填充的内容删除
	return LastUnPadding(plainText, des.BlockSize), nil
}
```
![](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/1380340-20201101190422825-4035893.png)

&gt; 总结
&gt; - DES使用64bit钥对数据块进行加密
&gt; - 在Go语言中`iv`的长须需要与密钥对长度一致。
&gt; - CBC使用的流密码算法
&gt; - CBC需要对最后明文分组填充


### OFB分组模式

```go
func OFBEncrypt(plainText, key string) ([]byte, error) {
	var (
		// 创建一个des加密的接口

		block, err = des.NewCipher([]byte(key))
		// 分组加密 需要对最后进行填充

		cipherText = make([]byte, len(plainText))
	)

	if err != nil {
		return nil, err
	}
	// 创建使用cbc分组模式加密接口
	mode := cipher.NewOFB(block, []byte(&#34;12345678&#34;))
	//mode := cipher.NewCBCEncrypter(block, []byte(&#34;12345678&#34;))
	// 加密
	mode.XORKeyStream(cipherText, []byte(plainText))
	return cipherText, nil
}

func OFBDecrypt(cipherText, key string) ([]byte, error) {

	var (
		// 创建一个des加密的接口
		block, err = des.NewCipher([]byte(key))
		// 创建使用cbc分组模式解密接口
		mode           = cipher.NewOFB(block, []byte(&#34;12345678&#34;))
		byteCipherText = []byte(cipherText)
		// 明文存储变量
		plainText = make([]byte, len(byteCipherText))
	)

	if err != nil {
		return nil, err
	}
	// 解密，无返回值
	mode.XORKeyStream(plainText, byteCipherText)
	// 将填充的内容删除
	return LastUnPadding(plainText, des.BlockSize), nil
}
```
![](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/1380340-20201101214408337-854797848.png)

### 填充方式

```go
/*
 *  填充函数，如果最后一个分组字节数不够则填充，填充的字节数为缺少的字节数
 *  如果最后一个字节数正好的话，则新建一个分组
 */
func LastPadding(plainText []byte, blockSize int) []byte {

	var (
		// 获得明文的长度，以判断时候需要补充
		paddingLength = blockSize - len(plainText)%blockSize
		// 初始化填充的内容
		padText = bytes.Repeat([]byte{byte(paddingLength)}, paddingLength)
	)
	//将填充的内容追加到明文后
	return append(plainText, padText...)
}

/*
 *  删除填充函数，如果最后一个分组字节数不够则填充，填充的字节数为缺少的字节数
 *  如果最后一个字节数正好的话，则新建一个分组
 */
func LastUnPadding(plainText []byte, blockSize int) []byte {

	var (
		// 获得明文的长度，以判断时候需要补充
		paddingLength = len(plainText)
		// 获得尾部填充的字节数量
		lastChar = int(plainText[paddingLength-1])
	)
	return bytes.TrimFunc(plainText, func(r rune) bool {
		return r == rune(lastChar)
	})

}
```

&gt; 总结
&gt; - ofb不需要最后为明文分组填充
&gt; - 加密算法Go语言API已经提供，但算法的分组业务流程需要自己实现

### AES

```go
func AESEncrypt(cipherText, key string) ([]byte, error) {

	var (
		// 创建一个AES加密的接口
		block, err     = aes.NewCipher([]byte(key))
		byteCipherText = []byte(cipherText)
		// 明文存储变量
		plainText = make([]byte, len(byteCipherText))
	)

	if err != nil {
		return nil, err
	}

	// 创建使用cbc分组模式解密接口
	mode := cipher.NewOFB(block, []byte(&#34;1234567812345678&#34;))

	// 解密，无返回值
	mode.XORKeyStream(plainText, byteCipherText)
	// 将填充的内容删除
	return LastUnPadding(plainText, aes.BlockSize), nil
}
```
![](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/1380340-20201102002205863-975724083.png)

&gt; **总结**
&gt;
&gt; - AES秘钥为 16,24,32 Byte 即 128,196,256 bit
&gt; - 在无需明文填充的分组模式下，`ofb` `cfb`  `ctr`，加密解密的业务逻辑处理是一样的。
