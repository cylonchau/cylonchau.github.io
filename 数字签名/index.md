# 常用加密算法学习总结之数字签名




数字签名（Digital Signature），通俗来讲是基于非对称加密算法，用秘钥对内容进行散列值签名，在对内容与签名一起发送。

[更详细的解说](http://www.youdzone.com/signature.html)
[更详细的解说 - 中文](http://www.ruanyifeng.com/blog/2011/08/what_is_a_digital_signature.html)

## 数字签名的生成个验证

&gt; **签名**
&gt;
&gt; ⑴ 对数据进行散列值运算。 
&gt; ⑵ 签名：使用签名者的私钥对数据的散列值进行加密。
&gt; ⑶ 数字签名数据：签名与原始数据。

![how do digital signatures and digital certificates work together in ssl](https://cdn.jsdelivr.net/gh/CylonChau/imgbed/img/how-do-digital-signatures-and-digital-certificates-work-together-in-ssl.png)



&lt;center class=&#34;podsc&#34;&gt;图：数字签名&lt;/center&gt;
&lt;center&gt;&lt;em&gt;Source：&lt;/em&gt;https://cheapsslsecurity.com/blog/digital-signature-vs-digital-certificate-the-difference-explained/&lt;/center&gt;

&gt; **验证**
&gt; ⑴ 接收数据：原始数据&amp;数字签名。
&gt; ⑵ 使用公钥进行解密得到散列值。
&gt; ⑶ 将原始数据的散列值与解密后的散列值进行对比。

## Go语言中使用RSA进行数字签名


&gt; ⑴ pem解码：使用pem对私钥进行解码, 得到pem.Block结构体
&gt; ⑵ 获得私钥：使用GO x509接口`pem.Block`据解析成私钥结构体
&gt; ⑶ 计算hash值：对明文进行散列值计算
&gt; ⑷ 使用秘钥对散列值签名

```go
package main

import (
	&#34;crypto&#34;
	&#34;crypto/rand&#34;
	&#34;crypto/rsa&#34;
	&#34;crypto/sha256&#34;
	&#34;crypto/x509&#34;
	&#34;encoding/pem&#34;
	&#34;fmt&#34;
)

var (
	private = `-----BEGIN 私钥-----
MIICXQIBAAKBgQDc73afIxqYOHg80puDIMYrqUAiTi8EiTVDEiO9YE3&#43;VxRvN0sa
pe3zx1UdhgIn3iCPUzyI2vwNADId3LjuIjkdCcdB2fHrBTbcy6u0545HnY42F9aQ
7cAr168bHcqhQoKcna9i9nukO&#43;w7So1J9C6Wr8J4e4923q7&#43;T7z7bZeXywIDAQAB
AoGBAItX5KLdywoyo3MJCdgcNaCX8MEyOmlL&#43;HHC4ROxx78gQN0cLJw0Bu33zHEA
ch&#43;e8z4yKz3Nj6bLdtBqw6A9qXLBCfWfD/p9YKDZNFP/6&#43;u9teUirOgiBSq7kXWy
mtBm0I3pz33EomCuSJzLj/Mj/fkKs&#43;425jPFcZboJdZpCyBhAkEA8mtGUGYuAZwV
RKBDkf1bz5EyPBGV&#43;9CyXa6pd6md61APY0j&#43;qhb1w9ADfHKkAzfoilhpucznRhaz
kAheqMPAMwJBAOlQEx2Ytc8TxfFqhF8RPTODe2N0jBBvsvJ85k7vNiQ&#43;hnmaAray
XS6pCbZdvmGHYKlz3MVGeis/UJKDdSzE0gkCQQCoZijkNPcEmz6S&#43;5m00oFywXRa
EgVUdndRaMHEpIlVK7pkyBJQab60Fc42JxUUP0RExoI7VcHbCG4YQhgvuDvNAkBQ
CUolcwebe/sBcDrsqetGyqn/WjHaSZcnnDUdiu4VzOUwveaEafeRVCeiydHPfzNn
rflkK2MphtTLDhGaRAKRAkASKlhV8aTBzTty/V3XMQfFVIAdHCyEIGMdjDDSzPly
shZCn66IyIze8j5Q4ZLcRz6GPglHdrkBnyt4QFuGurpl
-----END 私钥-----`

	public = `-----BEGIN 公钥-----
MIGJAoGBANzvdp8jGpg4eDzSm4MgxiupQCJOLwSJNUMSI71gTf5XFG83Sxql7fPH
VR2GAifeII9TPIja/A0AMh3cuO4iOR0Jx0HZ8esFNtzLq7TnjkedjjYX1pDtwCvX
rxsdyqFCgpydr2L2e6Q77DtKjUn0Lpavwnh7j3berv5PvPttl5fLAgMBAAE=
-----END 公钥-----`
)

func digitalSign(privateKey, plainText string) (signText []byte, err error) {
	var (
		pemBlock, _   = pem.Decode([]byte(privateKey))
		privateStream *rsa.PrivateKey
		plainHash     = sha256.Sum256([]byte(plainText))
	)

	if privateStream, err = x509.ParsePKCS1PrivateKey(pemBlock.Bytes); err != nil {
		return
	}
	if signText, err = rsa.SignPKCS1v15(rand.Reader, privateStream, crypto.SHA256, plainHash[:]); err != nil {
		return
	}
	return
}

func digitalVerify(publicKeyByte, plainText string, signText []byte) (ok bool, err error) {
	var (
		pemBlock, _  = pem.Decode([]byte(publicKeyByte))
		publicStream *rsa.PublicKey
		plainHash    = sha256.Sum256([]byte(plainText))
	)

	if publicStream, err = x509.ParsePKCS1PublicKey(pemBlock.Bytes); err != nil {
		return
	}

	if err = rsa.VerifyPKCS1v15(publicStream, crypto.SHA256, plainHash[:], signText); err != nil {
		return
	}
	return true, nil
}

func main() {
	text, err := digitalSign(private, &#34;张三李四王五赵柳&#34;)
	ok, err := digitalVerify(public, &#34;张三李四王五赵柳&#34;, text)
	fmt.Println(ok)
	fmt.Println(err)
}
```

&gt; 总结
&gt; 在Go语言API中公钥私钥的注释头尾也需要加上
