# powercli The SSL connection could not be established, see inner exception. 问题解决



```
Connect-VIServer -Server 
```

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20210801143141181-1047906479.png)


这里是“SSL连接不能建立......”这实际上意味着你没有一个有效的证书。如果你想连接到vCenter没有一个有效的证书，您必须允许可以改变你的vCenter证书到受信任的一个，这是正确的解决方案，也可以忽略无效的证书，以规避所有的安全性，但使它现在的工作。 

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20210801143200019-1825949412.png)

忽略无效证书
```
Set-PowerCLIConfiguration -InvalidCertificateAction:ignore
```

再次登陆可正常登陆
![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20210801143232415-1036597499.png)

 
