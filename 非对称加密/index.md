# 常用加密算法学习总结之非对称加密




公开密钥密码学（英语：Public-key cryptography）也称非对称式密码学（英语：Asymmetric cryptography）是密码学的一种演算法。常用的非对称加密算法有 `RSA` `DSA` `ECC` 等。[公开密钥加密](https://zh.wikipedia.org/zh-hans/%E5%85%AC%E5%BC%80%E5%AF%86%E9%92%A5%E5%8A%A0%E5%AF%86)

非对称加密算法使用**公钥**、**私钥**来加解密。
- 公钥与私钥是成对出现的。
- 多个用户（终端等）使用的密钥交公钥，只有一个用户（终端等）使用的秘钥叫私钥。
- 使用公钥加密的数据只有对应的私钥可以解密；使用私钥加密的数据只有对应的公钥可以解密。

## 非对称加密通信过程

下面我们来看一看使用公钥密码的通信流程。假设Alice要给Bob发送一条消息，Alice是发送者，Bob是接收者，而这一次窃听者Eve依然能够窃所到他们之间的通信内容。 [参考自维基百科](https://zh.wikipedia.org/zh-hans/%E5%85%AC%E5%BC%80%E5%AF%86%E9%92%A5%E5%8A%A0%E5%AF%86)

&gt; ⑴ Alice与bob事先互不认识，也没有可靠安全的沟通渠道，但Alice现在却要透过不安全的互联网向bob发送信息。
&gt; ⑵ Alice撰写好原文，原文在未加密的状态下称之为明文 `plainText`。
&gt; ⑶ bob使用密码学安全伪随机数生成器产生一对密钥，其中一个作为公钥 `publicKey`，另一个作为私钥 `privateKey`。
&gt; ⑷ bob可以用任何方法传送公钥`publicKey` 给Alice，即使在中间被窃听到也没问题。
&gt; ⑸ Alice用公钥`publicKey`把明文`plainText`进行加密，得到密文 `cipherText`
&gt; ⑹ Alice可以用任何方法传输密文给bob，即使中间被窃听到密文也没问题。
&gt; ⑺ bob收到密文，用私钥对密文进行解密，得到明文 `plainText`。
&gt; 由于其他人没有私钥，所以无法得知明文；如果Alice，在没有得到bob私钥的情况下，她将重新得到原文。

![](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/1380340-20201102205804406-1240772965.png)
![](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/1380340-20201102204939055-1888285763.png)

## RSA

RSA是一种非对称加密算法，是由罗纳德·李维斯特（Ron Rivest）、阿迪·萨莫尔（Adi Shamir）和伦纳德·阿德曼（Leonard Adleman）在1977年一起提出，并以三人姓氏开头字母拼在一起组成的。

&gt; RSA公钥和密钥的获取：随机选择两个大的素数，`p` `q`  $N = p*q$
&gt; RSA加密过程：$cipherText = plainText ^ E  mod  N$，$(N,e)$为公钥，$(N,d)$为私钥。
&gt; RSA解密过程：$plainText = cipherText^ D  mod     N$

## Go语言中RSA的应用

### 在Go语言中生成公钥与私钥

#### 生成秘钥流程

&gt; ⑴ 使用`crypto/rsa`中的`GenerateKey(random io.Reader, bits int)`方法生成私钥（结构体）
&gt; ⑵ 因为X509证书采用了[ASN1](https://wuziqingwzq.github.io/ca/2017/12/26/x509-knowledge-asn1.html)描述结构，需要通过Go语言API将的到的私钥（结构体），转换为`BER`编码规则的字符串。
&gt; ⑶ 需要将ASN1 BER 规则转回为PEM数据编码。`pem.Encode(out io.Writer, b *Block)`
&gt; ⑷ 将返回的数据保存

#### 生成私钥

```go
func GeneratePrivateKey(keySize int) (privateKey bytes.Buffer, err error) {
	// 生成私钥
	var (
		privateKeyStruct *rsa.PrivateKey
		privateStream    []byte
	)
	privateKeyStruct, err = rsa.GenerateKey(rand.Reader, keySize)

	if err != nil {
		return
	}

	privateStream = x509.MarshalPKCS1PrivateKey(privateKeyStruct)

	privateBlock := pem.Block{Type: &#34;私钥&#34;, Bytes: privateStream}

	if err = pem.Encode(&amp;privateKey, &amp;privateBlock); err != nil {
		return
	}
	return
}
```

#### 通过私钥获取公钥

通过私钥获取公钥需要将私钥生成的步骤翻转

&gt; ⑴ 私钥[]byte解码为一个pemBlock `pem.Decode()`
&gt; ⑵ pemBlock.Bytes是`BER`编码规则的字符串。将其转换为结构体 `x509.ParsePKCS1PrivateKey()`
&gt; ⑶ 转换为的结构体的属性`PublicKey`为公钥结构体，需将其转换为`BER`编码规则的字符串。`x509.MarshalPKCS1PublicKey(&amp;PublicKey)`
&gt; ⑷ 拼接公钥pemBlock，并需要将ASN1 BER规则字符串转回为PEM数据编码。`pem.Encode(out io.Writer, b *Block)`

```go
func GetPublicKey(privateKey []byte) (publicKey bytes.Buffer, err error) {
	pemBlock, _ := pem.Decode(privateKey)

	privateStream, err := x509.ParsePKCS1PrivateKey(pemBlock.Bytes)
	if err != nil {
		return
	}
	publicStream := x509.MarshalPKCS1PublicKey(&amp;privateStream.PublicKey)
	privateBlock := pem.Block{Type: &#34;公钥&#34;, Bytes: publicStream}

	if err = pem.Encode(&amp;publicKey, &amp;privateBlock); err != nil {
		return
	}
	return
}
```

![](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/1380340-20201102235640888-2142103656.png)

### 使用RSA密钥进行加解密

RSA加/解密步骤

&gt; ⑴ 因为在生成公钥与私钥时，进行了pem编码，需要先对其（一般情况下加密都使用公钥）进行解码为pemBlock。`pem.Decode()`
&gt; ⑵ pemBlock.Bytes是`BER`编码规则的字符串。将其转换为结构体 `x509.ParsePKCS1PublicKey(pemBlock.Bytes)`
&gt; ⑶ 使用 `rsa.DecryptPKCS1v15` 或 `rsa.EncryptPKCS1v15` 进行加解密，如：`rsa.DecryptPKCS1v15(rand.Reader, public|private stream, []byte plain|cipher)`，返回值即为加/解密好的数据。

```go
func RSAEncrypt(publicKey []byte, plainText string) (cipherText []byte, err error) {
	pemBlock, _ := pem.Decode(publicKey)
	publicStream, err := x509.ParsePKCS1PublicKey(pemBlock.Bytes)
	if err != nil {
		return
	}

	if cipherText, err = rsa.EncryptPKCS1v15(rand.Reader, publicStream, []byte(plainText)); err != nil {
		return
	}
	return
}

func RSADecrypt(privateKey, cipherText []byte) (plainText []byte, err error) {
	pemBlock, _ := pem.Decode(privateKey)
	privateStream, err := x509.ParsePKCS1PrivateKey(pemBlock.Bytes)
	if err != nil {
		return
	}

	if plainText, err = rsa.DecryptPKCS1v15(rand.Reader, privateStream, []byte(cipherText)); err != nil {
		return
	}
	return
}
```
![](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/1380340-20201103160438176-867986897.png)

&gt; 总结
&gt;
&gt; - Go语言接口中，明文内容的长度不能大于秘钥本身。
&gt; - RSA算法加解密速度慢，不推荐对较大数据加密。
