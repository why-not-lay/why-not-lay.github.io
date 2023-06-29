# 数字证书简述

> 创建时间: 2023-06-29  
> 更新时间: 2023-06-29

## 对称加密

![symmetric-encryption](https://storage-1301473886.cos.ap-guangzhou.myqcloud.com/img/digital-certificate/symmetric-encryption.png)

<center style="font-size:12px;color:#C0C0C0">图 1: 对称加密</center>

对称加密的加密和解密过程均是使用同一个密钥 key。

## 非对称加密

![asymmetric-encryption](https://storage-1301473886.cos.ap-guangzhou.myqcloud.com/img/digital-certificate/asymmetric-encryption.png)

<center style="font-size:12px;color:#C0C0C0">图 2: 非对称加密</center>

非对称加密的加密和解密的过程涉及到两个不同的密钥，一般而言，用在加密端的密钥称作公钥 public key，而用在解密端的密钥称作私钥 private key。

为什么会出现非对称加密？这是因为在使用对称加密的过程中，加解密的双方会用到同一个密钥，在使用之前会有一个密钥交换的过程，这个过程会有密钥泄漏的风险。而非对称加密是两个不同的专用密钥，公钥会被交换出去并进行数据加密，私钥则永远保存在解密端，这样即便是公钥被泄漏出去，第三方无法获取明文信息。

## 哈希函数 Hash

对于哈希函数:

$$
H(X) = Y
$$

其中，$H$ 为哈希函数，$X$ 为任意长度的数据，$Y$ 为固定长度的二进制数据。

哈希函数具有不可逆性，即只能通过 $X$ 得到 $Y$，而无法通过 $Y$ 得到 $X$。

## 数字签名

数字签名的作用是用于校验数据是否被篡改以及身份认证。

![signature](https://storage-1301473886.cos.ap-guangzhou.myqcloud.com/img/digital-certificate/sign.png)

<center style="font-size:12px;color:#C0C0C0">图 3: 签名过程</center>

数字签名首先会将文档哈希化得到文档摘要，随后用私钥 private key 对文档摘要进行加密并附在文档的后面一并发送出去。

![verificate](https://storage-1301473886.cos.ap-guangzhou.myqcloud.com/img/digital-certificate/verificate.png)

<center style="font-size:12px;color:#C0C0C0">图 4: 验证过程</center>

验证的过程在用公钥 public key 对签名进行解密后得到一个哈希值，然后用这个值去与文档哈希化后的值进行比对，如果相同则签名可用。

为什么签名的过程是使用私钥而验证的过程是使用公钥，这是为了防止签名被篡改。

![hijack](https://storage-1301473886.cos.ap-guangzhou.myqcloud.com/img/digital-certificate/hijack.png)

<center style="font-size:12px;color:#C0C0C0">图 5: 传输过程被篡改</center>

由于公钥是能够被外部用户所共有的，当篡改者在篡改文档后重新签名，这个签名在接收方经过私钥解密后得到值与文档得到的哈希值同样是一致的，这样接收方是无法鉴别文档在传输过程中是否被篡改。

## 数字证书



## 参考

[数字证书原理](https://www.zhaohuabing.com/post/2020-03-19-pki/)