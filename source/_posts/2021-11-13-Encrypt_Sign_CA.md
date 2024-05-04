---
layout: post
title: 数据的加密、签名与CA证书
tag: 技术笔记
date: 2021-11-13
category: Technology
---

#### 简介

敏感数据传输过程中要防止窃取和恶意篡改。使用安全的加密算法加密传输对象可以保护数据，这就是所谓的对对象进行密封。而对密封的对象进行数字签名则可以防止对象被非法篡改，保持其完整性。

我们简单来说，发送者对明文进行加密然后生成密文，这就是加密，接受者通过一段数据来验证发送数据的用户便是正确的发送者，那么这就是签名。

#### 加密算法

##### 对称加密算法

###### 定义

对称加密算法是应用比较早的加密算法，技术成熟，在对称加密算法中，数据发信方将明文数据和加密秘钥一起经过特殊加密算法处理后，使其变成复杂的加密密文发送出去，收信方收到密文后，若想解读原文，则需要使用加密用过的密钥及相同算法的算法对密文进行解密，才能使其恢复成可读明文，在对称加密算法中，可用的密钥只有一个，发收信双方都使用这个密钥对数据进行加密和解密，这就要求解密方实现必须知道加密秘钥。

###### 优缺点

优点：

1. 算法公开、计算量小、加密速度快、加密效率高。

缺点：

1. 交易双方都使用同样钥匙，安全性得不到保证。
2. 每对用户每次使用对称加密算法时，都需要使用其他人不知道的惟一钥匙，这会使得发收信双方所拥有的钥匙数量呈几何级数增长，密钥管理成为用户的负担。对称加密算法在分布式网络系统上使用较为困难，主要是因为密钥管理困难，使用成本较高。

###### 常用的对称加密算法

基于对称密钥的加密算法主要有DES、AES、RC2、RC4、RC5和Blowfish等。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
<modelVersion>4.0.0</modelVersion>

<groupId>com.example.encrypt</groupId>
<artifactId>encrypt</artifactId>
<version>0.0.1-SNAPSHOT</version>
<name>encrypt</name>
<description>Demo project</description>

<properties>
<java.version>11</java.version>
</properties>

<dependencies>
<dependency>
<groupId>cn.hutool</groupId>
<artifactId>hutool-all</artifactId>
<version>5.1.0</version>
</dependency>
</dependencies>

</project>

```



```java
/**
* AES 加密、解密
*/
public static void aesEncrypt(String data, String secret) {
    //随机生成秘钥
    byte[] key = SecureUtil.generateKey(SymmetricAlgorithm.AES.getValue(), secret.getBytes()).getEncoded();
    //创建AES对象
    SymmetricCrypto aes = new SymmetricCrypto(SymmetricAlgorithm.AES, key);
    //加密
    String encrypt = aes.encryptHex(data);
    //解密
    String decrypt = aes.decryptStr(encrypt);
    System.out.println("AES 加密后的串：" + encrypt + "AES 解密的数据：" + decrypt);
}

/**
* DES 加密、解密
*/
public static void desEncrypt(String data, String secret) {
    //随机生成秘钥
    byte[] key = SecureUtil.generateKey(SymmetricAlgorithm.DES.getValue(), secret.getBytes()).getEncoded();
    //创建DES对象
    SymmetricCrypto des = new SymmetricCrypto(SymmetricAlgorithm.DES, key);
    //加密
    String encrypt = des.encryptHex(data);
    //解密
    String decrypt = des.decryptStr(encrypt);
    System.out.println("DES 加密后的串：" + encrypt + "DES 解密的数据：" + decrypt);
}

/**
* DESede 加密、解密
*/
public static void desedeEncrypt(String data, String secret) {
    //随机生成秘钥
    byte[] key = SecureUtil.generateKey(SymmetricAlgorithm.DESede.getValue(), secret.getBytes()).getEncoded();
    //创建DES对象
    SymmetricCrypto desede = new SymmetricCrypto(SymmetricAlgorithm.DESede, key);
    //加密
    String encrypt = desede.encryptHex(data);
    //解密
    String decrypt = desede.decryptStr(encrypt);
    System.out.println("DESede 加密后的串：" + encrypt + "DESede 解密的数据：" + decrypt);
}

/**
* Blowfish 加密、解密
*/
public static void blowfishEncrypt(String data, String secret) {
    //随机生成秘钥
    byte[] key = SecureUtil.generateKey(SymmetricAlgorithm.Blowfish.getValue(), secret.getBytes()).getEncoded();
    //创建Blowfish对象
    SymmetricCrypto blowfish = new SymmetricCrypto(SymmetricAlgorithm.Blowfish, key);
    //加密
    String encrypt = blowfish.encryptHex(data);
    //解密
    String decrypt = blowfish.decryptStr(encrypt);
    System.out.println("Blowfish 加密后的串：" + encrypt + "Blowfish 解密的数据：" + decrypt);
}

/**
* RC2 加密、解密
*/
public static void rc2Encrypt(String data, String secret) {
    //随机生成秘钥
    byte[] key = SecureUtil.generateKey(SymmetricAlgorithm.RC2.getValue(), secret.getBytes()).getEncoded();
    //创建Blowfish对象
    SymmetricCrypto rc2 = new SymmetricCrypto(SymmetricAlgorithm.RC2, key);
    //加密
    String encrypt = rc2.encryptHex(data);
    //解密
    String decrypt = rc2.decryptStr(encrypt);
    System.out.println("RC2 加密后的串：" + encrypt + "RC2 解密的数据：" + decrypt);
}

/**
* ARCFOUR 加密、解密
*/
public static void arcfourEncrypt(String data, String secret) {
    //随机生成秘钥
    byte[] key = SecureUtil.generateKey(SymmetricAlgorithm.ARCFOUR.getValue(), secret.getBytes()).getEncoded();
    //创建Blowfish对象
    SymmetricCrypto arcfour = new SymmetricCrypto(SymmetricAlgorithm.ARCFOUR, key);
    //加密
    String encrypt = arcfour.encryptHex(data);
    //解密
    String decrypt = arcfour.decryptStr(encrypt);
    System.out.println("ARCFOUR 加密后的串：" + encrypt + "ARCFOUR 解密的数据：" + decrypt);
}

public static void main(String[] args) {
    String data = "测试数据";
    String secret1 = "abcdefghigkasdfg";
    String secret2 = "abcdefghijklmnopqrstuvwx";

    /* AES加密 */
    aesEncrypt(data, secret1);
    /* DES加密 */
    desEncrypt(data, secret1);
    /* RC2加密*/
    rc2Encrypt(data, secret1);
    /* Blowfish加密 */
    blowfishEncrypt(data, secret1);
    /* ACFOUR加密 */
    arcfourEncrypt(data, secret1);
    /* DESede加密 */
    desedeEncrypt(data, secret2);
}
```

```
运行结果：
AES 加密后的串：d10ac2728e77996e7150361870efc2cbAES 解密的数据：测试数据
DES 加密后的串：9591833211522a640e886a2b3c43f8a3DES 解密的数据：测试数据
RC2 加密后的串：cd1251b414f12dcfc35b83293c2556fbRC2 解密的数据：测试数据
Blowfish 加密后的串：d2b46322dfb30834d5580dffe42513b3Blowfish 解密的数据：测试数据
ARCFOUR 加密后的串：38681502bc3487f5a8ae9621ARCFOUR 解密的数据：测试数据
DESede 加密后的串：f8b045da6a30404d84d72f738400d799DESede 解密的数据：测试数据
```
##### 非对称加密算法

###### 定义

非对称加密需要两个密钥：公钥 (publickey) 和私钥 (privatekey)。公钥和私钥是一对，如果用公钥对数据加密，那么只能用对应的私钥解密。如果用私钥对数据加密，只能用对应的公钥进行解密。因为加密和解密用的是不同的密钥，所以称为非对称加密。

###### 优缺点

优点：

1. 算法强度复杂、安全性高

缺点：

1. 由于其算法复杂，而使得加密解密速度没有对称加密解密的速度快

###### 工作原理

1. A 要向 B 发送信息，A 和 B 都要产生一对用于加密和解密的公钥和私钥。
2. A 的私钥保密，A 的公钥告诉 B；B 的私钥保密，B 的公钥告诉 A。
3. A 要给 B 发送信息时，A 用 B 的公钥加密信息，因为 A 知道 B 的公钥。
4. A 将这个消息发给 B (已经用 B 的公钥加密消息)。
5. B 收到这个消息后，B 用自己的私钥解密 A 的消息。其他所有收到这个报文的人都无法解密，因为只有 B 才有 B 的私钥。

###### 常用的非对称加密算法

RSA、Elgamal、背包算法、Rabin、D-H、ECC (椭圆曲线加密算法)。

```java
/**
* RSA 公钥加密，私钥解密
*
*/
public static void rsaEncrypt1(String data, String privateKeyString, String publicKeyString) throws UnsupportedEncodingException {
    //公私钥，转为对象
    PrivateKey privateKey = SecureUtil.generatePrivateKey("RSA", Base64.decode(privateKeyString));
    PublicKey publicKey = SecureUtil.generatePublicKey("RSA", Base64.decode(publicKeyString));
    //获取字符串byte数组
    byte[] bytes = data.getBytes("UTF-8");
    //创建RSA对象
    RSA rsa = new RSA();
    //设置公钥，然后执行公钥加密
    rsa.setPublicKey(publicKey);
    byte[] encrypt = rsa.encrypt(bytes, KeyType.PublicKey);
    //设置私钥，然后执行私钥解密
    rsa.setPrivateKey(privateKey);
    byte[] decrypt = rsa.decrypt(encrypt, KeyType.PrivateKey);

    System.out.println("公钥加密，私钥解密，解密内容：" + new String(decrypt));
}

/**
* RSA 公钥解密，私钥加密
*/
public static void rsaEncrypt2(String data, String privateKeyString, String publicKeyString) throws UnsupportedEncodingException {
    //公私钥，转为对象
    PrivateKey privateKey = SecureUtil.generatePrivateKey("RSA", Base64.decode(privateKeyString));
    PublicKey publicKey = SecureUtil.generatePublicKey("RSA", Base64.decode(publicKeyString));
    //获取字符串byte数组
    byte[] bytes = data.getBytes("UTF-8");
    //创建RSA对象
    RSA rsa = new RSA();
    //设置私钥，然后执行私钥解密
    rsa.setPrivateKey(privateKey);
    byte[] encrypt = rsa.encrypt(bytes, KeyType.PrivateKey);
    //设置公钥，然后执行公钥加密
    rsa.setPublicKey(publicKey);
    byte[] decrypt = rsa.decrypt(encrypt, KeyType.PublicKey);

    System.out.println("公钥加密，私钥解密，解密内容：" + new String(decrypt));
}

public static void main(String[] args) throws UnsupportedEncodingException {
    /* 生成钥匙对 */
    KeyPair pair = SecureUtil.generateKeyPair("RSA");
    PrivateKey privateKey = pair.getPrivate();
    PublicKey publicKey = pair.getPublic();

    //将公钥私钥转换成Base64字符串
    String privateKeyString = Base64.encode(privateKey.getEncoded());
    String publicKeyString = Base64.encode(publicKey.getEncoded());

    /* 设置要加密的内容 */
    String data = "测试数据";
    /* RSA公钥加密，私钥解密 */
    rsaEncrypt1(data, privateKeyString, publicKeyString);
    /* RSA私钥加密，公钥解密 */
    rsaEncrypt2(data, privateKeyString, publicKeyString);

}
```

```
运行结果
公钥加密，私钥解密，解密内容：测试数据
公钥加密，私钥解密，解密内容：测试数据
```
#### 签名

##### 定义

数字签名是通过一个单项函数(hash函数)对要传送的信息进行处理，得到一个用于认证信息来源并核实信息在传递过程中是否发生变化的一个字符串。数字签名提供了对信息来源的确定并能检测信息是否被篡改。

##### 基本特征

- **报文鉴别**——接收者能够核实发送者对报文的签名；
- **报文的完整性**——接收者不能伪造对报文的签名或更改报文内容。
- **不可否认**——发送者事后不能抵赖对报文的签名；

##### 散列(哈希)函数

- 通常有MD5、SHA1、SHA256、SHA512
- 实质是抽取特征码，这样一般不会重复！是的，不同的文本它的哈希结果是有可能相同的，但概率很小。
（举例：比如想要识别一个人，我们可以通过他的指纹来锁定他，指纹出现相同的概率很低吧！在这里，人就相当于数据，而指纹就相当于对人这个数据进行hash后得到的结果）
- 对任意一个二进制数据进行哈希，可以得到定长的字符串结果，例如MD5哈希结果是128位，更多是以32个字符的十六进制格式哈希输出
- 还有就是不可逆的

##### 数字签名的验证过程

- 首先，文件经过单向散列函数的处理得到一份占128位的摘要（无论文件多大，经过单向散列函数的处理，生成的摘要都是128位），这份摘要相当于该文件的"指纹"，能够唯一地识别文件。**注意：**只要文件发生改动，经过单向散列函数处理后得到地摘要都会不一样。所以，文件和文件的摘要具有很强的对应关系
- 随后，用户A使用自己地私钥对这份128位地摘要进行加密，得到一份加密地摘要。
- 然后，用户A把文件、加密的摘要和公钥打包一起发给用户B。传输的过程中并没有对文件进行加密处理。
- 用户B将收到的文件经过单向散列函数处理得出一份128位摘要，这份摘要是通过收到的文件得到的，存在被更改的可能；使用A提供的公钥对收到的"加密的摘要"进行解密得到另一份128位摘要，这份摘要是通过原始文件得到的，一般认为代表真正的文件；然后将两份摘要进行比较。
- 如果两份摘要相等，说明文件经过用户A签名之后，在传输的过程中没有被更改；若不相等，说明文件在传输过程中被更改了，或者说已经不是原来的文件了，此时用户A的签名失效。

#### 一般流程

假设A、B双方均拥有一对公私钥(`PUB_A`、`PRI_A`、`PUB_B`、`PRI_B`)。

A向B发送Message的整个签名和加密的过程如下：

1. A先使用HASH对Message生成一个固定长度的信息摘要`Message_hash_A`
2. A使用A的私钥`PRI_A`对`Message_hash_A`进行签名得到`Message_sign`（这里为什么不直接对Message进行签名，而要对`Message_hash_A`进行签名呢？因为Message的长度可能很长，而`Message_hash_A`的长度则是固定的，这样性能更高，格式也固定，况且hash的结果一般不会出现重复的可能）
3. A接着使用B的公钥`PUB_B`对信息`Message`和信息`Message_sign`进行加密得到`Message_RSA`，这时A将`Message_RSA`发送给B。

当B接收到A的信息`Message_RSA`后，获取`Message`的步骤如下：

1. B用自己的私钥`PRI_B`解密得到明文：`Message`和`Message_sign`；
2. 然后B使用A的公钥`PUB_A`解`Message_sign`得到`Message_hash_A`；同时，B再对`Message`使用与A相同的HASH得到`Message_hash_B`；
3. 如果`Message_hash_A`与`Message_hash_B`相同，则说明`Message`没有被篡改过。

```java

/**
* RSA 私钥签名
*/
public static String rsaSign(String data, String privateKeyString) {
    //将私钥字符串转换为私钥对象
    PrivateKey privateKey = SecureUtil.generatePrivateKey("RSA", Base64.decode(privateKeyString));
    //设置签名对象以及加密算法
    Sign sign = SecureUtil.sign(SignAlgorithm.MD5withRSA);
    sign.setPrivateKey(privateKey);
    byte[] bytes = sign.sign(data.getBytes());
    return Base64.encode(bytes);
}

/**
* RSA 公钥验签
*/
public static boolean rsaSignVerify(String data, String publicKeyString, String signString) {
    //将公钥字符串转换为私钥对象
    PublicKey publicKey = SecureUtil.generatePublicKey("RSA", Base64.decode(publicKeyString));
    //设置签名对象以及加密算法
    Sign sign = SecureUtil.sign(SignAlgorithm.MD5withRSA);
    //将签名字符串转换成byte数组
    byte[] bytes = Base64.decode(signString);
    //设置公钥，然后执行验签
    sign.setPublicKey(publicKey);
    return sign.verify(data.getBytes(), bytes);
}

public static void main(String[] args) {
    KeyPair pair = SecureUtil.generateKeyPair("RSA");
    PrivateKey privateKey = pair.getPrivate();
    PublicKey publicKey = pair.getPublic();
    //将公私钥转换成Base64字符串
    String privateKeyString = Base64.encode(privateKey.getEncoded());
    String publicKeyString = Base64.encode(publicKey.getEncoded());

    String data = "测试数据";
    String sign = rsaSign(data, privateKeyString);
    boolean verify = rsaSignVerify(data, publicKeyString, sign);
    System.out.println("签名验证结果" + verify);
}
```

##### ps

注意，一般流程为先签名后加密，如果先将对象加密然后为其签名，任何恶意的第三方可以截获原始加密签名后的数据，剔除原始的签名，并对密封的数据加上自己的签名，这样一来，由于对象被加密和签名（只有在签名验证后才可以解密对象），恶意第三方和正常的接收者均无法得到原始的消息内容，接收者无法确认发件人的身份。

#### CA数字证书

在使用数字签名的情况下，有这个问题，如果不怀好意的人将你的公钥替换成自己的公钥了该怎么办呢？

上面说了，如果你用公钥将文件解密了，那么你就确定这个文件是由对方发过来的，但是如果这个公钥被人做了手脚了呢？比如C偷偷使用了B的电脑，用自己的公钥替换了A的公钥，然后C用自己的私钥做成数字签名，再写信给B，B就用假的公钥解密了假的数字签名。为了解决这个现象，数字证书就诞生了。

##### 定义

(digital certificate)是由CA(Certification Authority)签发的一种较为权威与公正的电子文档。证书的内容包括：CA的信息，公钥用户的信息，公钥，CA的签名和有效期等等。
CA是颁发数字证书的权威机构，是受大家信任的第三方。如果一个用户想要得到一份属于自己的数字证书，他应先向CA提出申请。在CA判明申请者的身份后，便为他分配一个公钥，并且CA将该公钥与申请者的身份信息绑定在一起，并为之签字，便形成证书发给申请者。
由于B不确定自己拥有的公钥是否来自于A，所以他要求A去权威中心为公钥做认证，CA用自己的私钥，对A的公钥和一些相关信息一起加密，生成数字证书并签名。所以之后如果A再想给B写信，他只需要在签名的同时，再附上数字证书就可以了。B收到信件后，用CA的公钥解开数字证书，就可以拿到A真实的公钥了，然后就能证明信件确实是A发送的。



#### 参考文档

> 网络安全——数据的加密与签名,RSA介绍 https://www.jianshu.com/p/003b85fd3e36
>
> 加解密篇 - 非对称加密算法 (RSA、DSA、ECC、DH) https://blog.csdn.net/u014294681/article/details/86705999
>
> Java 中的加密与签名 https://blog.csdn.net/qq_42298793/article/details/108431662
>
> 常见对称加密算法 https://www.cnblogs.com/Terry-Wu/p/10314315.html
>
> 数字签名是什么？详细介绍数字签名！https://www.cnblogs.com/itps/p/12359865.html
>
> 数字签名、数字证书与CA https://blog.csdn.net/weixin_38418878/article/details/98487188