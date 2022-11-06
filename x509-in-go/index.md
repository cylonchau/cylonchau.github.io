# x509 in go


本篇文章中，将描述如何使用go创建CA，并使用CA签署证书。在使用openssl创建证书时，遵循的步骤是 创建秘钥 &gt; 创建CA &gt; 生成要颁发证书的秘钥 &gt; 使用CA签发证书。这种步骤，那么我们现在就来尝试下。

### 创建证书的颁发机构

首先，会从将从创建 *CA* 开始。*CA* 会被用来签署其他证书

```go
// 对证书进行签名
ca := &amp;x509.Certificate{
	SerialNumber: big.NewInt(2019),
	Subject: pkix.Name{
        CommonName:    &#34;domain name&#34;,
		Organization:  []string{&#34;Company, INC.&#34;},
		Country:       []string{&#34;US&#34;},
		Province:      []string{&#34;&#34;},
		Locality:      []string{&#34;San Francisco&#34;},
		StreetAddress: []string{&#34;Golden Gate Bridge&#34;},
		PostalCode:    []string{&#34;94016&#34;},
	},
	NotBefore:             time.Now(),  // 生效时间
	NotAfter:              time.Now().AddDate(10, 0, 0), // 过期时间 年月日
	IsCA:                  true, // 表示用于CA
    // openssl 中的 extendedKeyUsage = clientAuth, serverAuth 字段
	ExtKeyUsage:           []x509.ExtKeyUsage{x509.ExtKeyUsageClientAuth, x509.ExtKeyUsageServerAuth},
    // openssl 中的 keyUsage 字段
	KeyUsage:              x509.KeyUsageDigitalSignature | x509.KeyUsageCertSign,
	BasicConstraintsValid: true,
}
```

接下来需要对证书生成公钥和私钥

```go
caPrivKey, err := rsa.GenerateKey(rand.Reader, 4096)
if err != nil {
	return err
}
```

然后生成证书：

```go
caBytes, err := x509.CreateCertificate(rand.Reader, ca, ca, &amp;caPrivKey.PublicKey, caPrivKey)
if err != nil {
	return err
}
```

我们看到的证书内容是PEM编码后的，现在`caBytes`我们有了生成的证书，我们将其进行 PEM 编码以供以后使用：

```go
caPEM := new(bytes.Buffer)
pem.Encode(caPEM, &amp;pem.Block{
	Type:  &#34;CERTIFICATE&#34;,
	Bytes: caBytes,
})

caPrivKeyPEM := new(bytes.Buffer)
pem.Encode(caPrivKeyPEM, &amp;pem.Block{
	Type:  &#34;RSA PRIVATE KEY&#34;,
	Bytes: x509.MarshalPKCS1PrivateKey(caPrivKey),
})
```

### 创建证书

证书的 `x509.Certificate` 与CA的 `x509.Certificate` 属性有稍微不同，需要进行一些修改

```go
cert := &amp;x509.Certificate{
	SerialNumber: big.NewInt(1658),
	Subject: pkix.Name{
        CommonName:    &#34;domain name&#34;,
		Organization:  []string{&#34;Company, INC.&#34;},
		Country:       []string{&#34;US&#34;},
		Province:      []string{&#34;&#34;},
		Locality:      []string{&#34;San Francisco&#34;},
		StreetAddress: []string{&#34;Golden Gate Bridge&#34;},
		PostalCode:    []string{&#34;94016&#34;},
	},
    IPAddresses:  []net.IP{}, // 这里就是openssl配置文件中 subjectAltName 里的 IP:/IP=
    DNSNames:     []string{}, // 这里就是openssl配置文件中 subjectAltName 里的 DNS:/DNS=
	NotBefore:    time.Now(),
	NotAfter:     time.Now().AddDate(10, 0, 0),
	SubjectKeyId: []byte{1, 2, 3, 4, 6},
    // 这里就是openssl中的extendedKeyUsage 
	ExtKeyUsage:  []x509.ExtKeyUsage{x509.ExtKeyUsageClientAuth, x509.ExtKeyUsageServerAuth},
	KeyUsage:     x509.KeyUsageDigitalSignature,
}
```

&gt; 注：这里会在证书中特别添加了 `DNS` 和 `IP` （这个不是必须的），这个选项的增加代表的我们的证书可以支持多域名

为该证书创建私钥和公钥：

```go
certPrivKey, err := rsa.GenerateKey(rand.Reader, 4096)
if err != nil {
	return err
}
```

### 使用CA签署证书

有了上述的内容后，可以创建证书并用CA进行签名

```go
certBytes, err := x509.CreateCertificate(rand.Reader, cert, ca, &amp;certPrivKey.PublicKey, caPrivKey)
if err != nil {
	return err
}
```

要保存成证书格式需要做PEM编码

```go
certPEM := new(bytes.Buffer)
pem.Encode(certPEM, &amp;pem.Block{
	Type:  &#34;CERTIFICATE&#34;,
	Bytes: certBytes,
})

certPrivKeyPEM := new(bytes.Buffer)
pem.Encode(certPrivKeyPEM, &amp;pem.Block{
	Type:  &#34;RSA PRIVATE KEY&#34;,
	Bytes: x509.MarshalPKCS1PrivateKey(certPrivKey),
})
```

### 把上面内容融合为一起

创建一个 `ca.go` 里面是创建ca和颁发证书的逻辑

```go
package main

import (
	&#34;bytes&#34;
	cr &#34;crypto/rand&#34;
	&#34;crypto/rsa&#34;
	&#34;crypto/x509&#34;
	&#34;crypto/x509/pkix&#34;
	&#34;encoding/pem&#34;
	&#34;math/big&#34;
	&#34;math/rand&#34;
	&#34;net&#34;
	&#34;os&#34;
	&#34;time&#34;
)

type CERT struct {
	CERT       []byte
	CERTKEY    *rsa.PrivateKey
	CERTPEM    *bytes.Buffer
	CERTKEYPEM *bytes.Buffer
	CSR        *x509.Certificate
}

func CreateCA(sub *pkix.Name, expire int) (*CERT, error) {
	var (
		ca  = new(CERT)
		err error
	)

	if expire &lt; 1 {
		expire = 1
	}
	// 为ca生成私钥
	ca.CERTKEY, err = rsa.GenerateKey(cr.Reader, 4096)
	if err != nil {
		return nil, err
	}

	// 对证书进行签名
	ca.CSR = &amp;x509.Certificate{
		SerialNumber: big.NewInt(rand.Int63n(2000)),
		Subject:      *sub,
		NotBefore:    time.Now(),                       // 生效时间
		NotAfter:     time.Now().AddDate(expire, 0, 0), // 过期时间
		IsCA:         true,                             // 表示用于CA
		// openssl 中的 extendedKeyUsage = clientAuth, serverAuth 字段
		ExtKeyUsage: []x509.ExtKeyUsage{x509.ExtKeyUsageClientAuth, x509.ExtKeyUsageServerAuth},
		// openssl 中的 keyUsage 字段
		KeyUsage:              x509.KeyUsageDigitalSignature | x509.KeyUsageCertSign,
		BasicConstraintsValid: true,
	}
	// 创建证书
	// caBytes 就是生成的证书
	ca.CERT, err = x509.CreateCertificate(cr.Reader, ca.CSR, ca.CSR, &amp;ca.CERTKEY.PublicKey, ca.CERTKEY)
	if err != nil {
		return nil, err
	}
	ca.CERTPEM = new(bytes.Buffer)
	pem.Encode(ca.CERTPEM, &amp;pem.Block{
		Type:  &#34;CERTIFICATE&#34;,
		Bytes: ca.CERT,
	})
	ca.CERTKEYPEM = new(bytes.Buffer)
	pem.Encode(ca.CERTKEYPEM, &amp;pem.Block{
		Type:  &#34;RSA PRIVATE KEY&#34;,
		Bytes: x509.MarshalPKCS1PrivateKey(ca.CERTKEY),
	})

	// 进行PEM编码，编码就是直接cat证书里面内容显示的东西
	return ca, nil
}

func Req(ca *x509.Certificate, sub *pkix.Name, expire int, dns []string, ip []net.IP) (*CERT, error) {
	var (
		cert = &amp;CERT{}
		err  error
	)
	cert.CERTKEY, err = rsa.GenerateKey(cr.Reader, 4096)
	if err != nil {
		return nil, err
	}
	if expire &lt; 1 {
		expire = 1
	}
	cert.CSR = &amp;x509.Certificate{
		SerialNumber: big.NewInt(rand.Int63n(2000)),
		Subject:      *sub,
		IPAddresses:  ip,
		DNSNames:     dns,
		NotBefore:    time.Now(),
		NotAfter:     time.Now().AddDate(expire, 0, 0),
		SubjectKeyId: []byte{1, 2, 3, 4, 6},
		ExtKeyUsage:  []x509.ExtKeyUsage{x509.ExtKeyUsageClientAuth, x509.ExtKeyUsageServerAuth},
		KeyUsage:     x509.KeyUsageDigitalSignature,
	}

	cert.CERT, err = x509.CreateCertificate(cr.Reader, cert.CSR, ca, &amp;cert.CERTKEY.PublicKey, cert.CERTKEY)
	if err != nil {
		return nil, err
	}

	cert.CERTPEM = new(bytes.Buffer)
	pem.Encode(cert.CERTPEM, &amp;pem.Block{
		Type:  &#34;CERTIFICATE&#34;,
		Bytes: cert.CERT,
	})
	cert.CERTKEYPEM = new(bytes.Buffer)
	pem.Encode(cert.CERTKEYPEM, &amp;pem.Block{
		Type:  &#34;RSA PRIVATE KEY&#34;,
		Bytes: x509.MarshalPKCS1PrivateKey(cert.CERTKEY),
	})
	return cert, nil
}

func Write(cert *CERT, file string) error {
	keyFileName := file &#43; &#34;.key&#34;
	certFIleName := file &#43; &#34;.crt&#34;
	kf, err := os.Create(keyFileName)
	if err != nil {
		return err
	}
	defer kf.Close()

	if _, err := kf.Write(cert.CERTKEYPEM.Bytes()); err != nil {
		return err
	}

	cf, err := os.Create(certFIleName)
	if err != nil {
		return err
	}
	if _, err := cf.Write(cert.CERTPEM.Bytes()); err != nil {
		return err
	}
	return nil
}
```

如果需要使用的话，可以引用这些函数

```go
package main

import (
	&#34;crypto/x509/pkix&#34;
	&#34;log&#34;
	&#34;net&#34;
)

func main() {
	subj := &amp;pkix.Name{
		CommonName:    &#34;chinamobile.com&#34;,
		Organization:  []string{&#34;Company, INC.&#34;},
		Country:       []string{&#34;US&#34;},
		Province:      []string{&#34;&#34;},
		Locality:      []string{&#34;San Francisco&#34;},
		StreetAddress: []string{&#34;Golden Gate Bridge&#34;},
		PostalCode:    []string{&#34;94016&#34;},
	}
	ca, err := CreateCA(subj, 10)
	if err != nil {
		log.Panic(err)
	}

	Write(ca, &#34;./ca&#34;)

	crt, err := Req(ca.CSR, subj, 10, []string{&#34;test.default.svc&#34;, &#34;test&#34;}, []net.IP{})

	if err != nil {
		log.Panic(err)
	}

	Write(crt, &#34;./tls&#34;)
}
```

### 遇到的问题

**panic: x509: unsupported public key type: rsa.PublicKey**

这里是因为 `x509.CreateCertificate` 的参数 `privatekey` 需要传入引用变量，而传入的是一个普通变量

&gt; 注：**x509: only RSA and ECDSA public keys supported**

### 一些参数的意思

`extendedKeyUsage` ：增强型密钥用法(参见&#34;new_oids&#34;字段)：服务器身份验证、客户端身份验证、时间戳。

```cnf
extendedKeyUsage = critical,serverAuth, clientAuth, timeStamping
```

`keyUsage `： 密钥用法，防否认(nonRepudiation)、数字签名(digitalSignature)、密钥加密(keyEncipherment)。

```conf
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
```

&gt; Reference
&gt;
&gt; [golang ca and signed cert go](https://shaneutt.com/blog/golang-ca-and-signed-cert-go/)
&gt;
&gt; [package x509](https://golang.google.cn/pkg/crypto/x509/)


