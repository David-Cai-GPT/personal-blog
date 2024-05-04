---
layout: post
title: 密码加盐与加密
tag: 技术笔记
date: 2020-10-2
category: Technology blog
---
### 密码加盐与加密



首先，密码进行加盐加密的最终目的是：

**即使在数据库拖库，代码被泄露，请求被劫持的情况下，怎么去保障用户的密码不被泄露**



#### 密码加密方式

首先是密码散列函数，密码散列函数是一种单向散列函数，将任意长度的消息压缩到某一固定长度的消息摘要，一个理想的密码散列函数应该有四个主要的特征，对于任何一个给定的消息，它都很容易就能运算出散列数值。难以由一个已知的散列数值，去推算出原始的消息（不可逆），再不更动散列数值的前提下，修改消息内容是不可行的，对于两个不同的消息，它不能给与相同的散列数值，单向散列函数生成的信息摘要是不可预见，消息摘要看起来和原始的数据没有任何关系，而且，原始数据的微小变化都会对生成的信息摘要产生很大的影响。
h = H(M)

其中，**M**是待处理的明文，可以为任意长，**H**是单向散列函数，**h**是生成的报文摘要，它具有固定的长度，并且和**M**的长度无关，其中**H**具有以下的单向性质，给**H**和**M**，很容易计算**h**；给定**h**和**H**，很难计算**M**，甚至得不到M的任何消息；给定**H**，要找两个不同的**M1**和**M2**，使得**H(M1)= H(M2)**在计算上是不可行的。

密码的加密，我们称之为密码哈希，从上面的介绍可知，哈希算法是一种单向函数，它把任意数量的数据装换为固定长度的指纹，而且这个过程无法逆转；如果输入发生了一点改变，由此产生的哈希值便会完全不同，这些特性便很适合用来加密密码，因为我们需要一种不可逆的算法来加密存储的密码，同时保证我们也能够验证用户登录的密码是否正确。

简单的概括描述一下用户在有哈希加密算法的账号系统中，注册和认证的大致流程

1. 用户创建自己的账号和密码
2. 密码通过哈希加密后存储在数据库中
3. 当用户尝试登陆的时候，前端输入自己的密码，经过哈希加密后和后端数据库中存储的密码进行对比来认知当前用户是否有访问权限

简单介绍了一下密码的加密过程，再了解一下目前比较流行的密码加密方式吧（本文提及）

##### MD5

MD5信息摘要算法，一种被广泛使用的密码散列函数，可以产生出一个128位（16字节）的散列值，用于确保信息传输的完整一致，于1992年公开，应以取代MD4算法，这套算法的程序在RFC 1231标准中被加以规范，1996年后被证实存在弱点，可以被加以破解，对于需要高度安全性的数据，单单MD5加密自然是不够的，2004年，正式MD5算法无法防止碰撞

**SHA-2**

SHA-2，名称来自于安全散列算法2（英语：Secure Hash Algorithm 2）的缩写，一种密码散列函数算法标准，美国国家安全局研发，由美国国家标准与技术研究院（NIST）在2001年发布。属于SHA算法之一，是SHA-1的后继者。其下又可再分为六个不同的算法标准，包括了：SHA-224、SHA-256、SHA-384、SHA-512、SHA-512/224、SHA-512/256。

 **Bcrypt** 

Bcrypt 有两个特点

- 每一次 HASH 出来的值不一样
- 计算非常缓慢（这里稍后会解释）

因此使用 Bcrypt 进行加密后，攻击者想要使用算出密码的成本变得不可接受。但代价是应用自身也会性能受到影响，不过登录行为并不是随时在发生，因此能够忍受。对于攻击者来说，需要不断计算，让攻击变得不太可能。

#### 密码加密流程

经过前面知识的了解之后，我们知道只对密码进行MD5加密现在已经远远不够了。
md5(password)
那么我怎么去保障密码的安全系数呢？于是便出现了密码加盐的办法，即使用户的密码很短，只要我在他的短密码后面加上一段很长的字符，再计算md5，是不是安全性就提高了，加上的这段字符串，便是“盐”，通过这种方式加密的结果，我们成为“加盐哈希”。注意这里的**salt**，我们提议为了更加安全，每个用户的盐值最好都不一样，例如：可以为用户的用户名以及其他的唯一表示用户的字段，为了加大彩虹表破解的难度。
md5(md5(password + salt))
现在我们知道了SHA-256、SHA-512算法会比md5更加安全，更难破解，于是我们将md5算法用SHA512来代替
SHA512(SHA512(password + salt))
通过上面的加盐哈希运算，即使攻击者拿到了最终结果，也很难反推出原始的密码。不能反推，但可以正着推，假设攻击者将 salt 值也拿到了，那么他可以枚举遍历所有 6 位数的简单密码，加盐哈希，计算出一个结果对照表，从而破解出简单的密码。这就是通常所说的暴力破解。

为了应对暴力破解，我使用了加盐的慢哈希。慢哈希是指执行这个哈希函数非常慢，这样暴力破解需要枚举遍历所有可能结果时，就需要花上非常非常长的时间。比如：bcrypt 就是这样一个慢哈希函数：
bcrypt (SHA512(password) + salt ,cost)


#### 代码实现

这里我们包装一个密码加密的工具类

```java
import org.springframework.security.crypto.bcrypt.BCrypt;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;

public class EncryptionUtils {
    //public static MessageDigest getInstance(String algorithm, String provider)
    //支持MD5\SHA-1\SHA-256\SHA-384\SHA-512
    public static String SHA(final String strText, final String strType){
        //这里定义返回值
        String strResult = null;
        //判断一下字符串是否有效
        if(strText != null && strText.length() > 0){
            //SHA加密
            try {
                //获取MessageDigest对象，传入需要使用的加密算法
                MessageDigest messageDigest = MessageDigest.getInstance(strType);
                //加密字符串做处理获得byte类型，使用指定的 byte 数组更新摘要。
                messageDigest.update(strText.getBytes());
                // 得到 byte 类型结果
                byte byteBuffer[] = messageDigest.digest();
                StringBuffer strHexString = new StringBuffer();
                for(int i = 0; i < byteBuffer.length; i++){
                    //返回的字符串表示的无符号整数参数所表示的值以十六进制
                    String hex = Integer.toHexString(0xff & byteBuffer[i]);
                    if(hex.length() == 1){
                        strHexString.append('0');
                    }
                    strHexString.append(hex);
                }
                //得到返回结果
                strResult = strHexString.toString();
            } catch (NoSuchAlgorithmException e) {
                e.printStackTrace();
            }
        }
        return strResult;
    }

    public static String Bcyrpt(String strText){
        String strResult = null;
        if (strText != null && strText.length() >= 0) {
            //使用BCrypt
            strResult = BCrypt.hashpw(strText, BCrypt.gensalt());
        }
        return strResult;
    }
}

```

```java
public class TestPasswordSalt {
    public static void main(String[] args) {
        String uid = "David";
        String pwd = "123456";
        //首先做SHA-512加密
        String afterSHA = EncryptionUtils.SHA(pwd,"SHA-512");
        StringBuffer stringBuffer = new StringBuffer(afterSHA);
        //将用户名拼接进去做随机盐值
        stringBuffer.append(uid);
        //Bcyrpt加密
        String result = EncryptionUtils.Bcyrpt(stringBuffer.toString());
        System.out.println(result);
    }
}

输出结果：$2a$10$cYLWOFwv2BWQsTkLIEjjTuarLe7VU3WDwcEeEHR36jYK6xcakyJG2

```

![3111490-d12e59731291f022.bng](3111490-d12e59731291f022.bng.webp)

#### Q&A

**黑客使用彩虹表进行hash碰撞？**

bcrypt一个密码出来的时间比较长，需要0.3秒，而MD5只需要一微秒（百万分之一秒），一个40秒可以穷举得到明文的MD5，在bcrypt需要12年，时间成本太高

**为什么要进行随机加盐？**

在这里，只是初步使用用户用户名来作为随机盐，其实有更多更复杂的加盐方式能够提高安全性，这里不再赘述，只是强调随机盐的重要性，随机盐的话，即使对方知道算法，也很难搞，因为对应于每一份盐，都得花时间空间去生成对应的密码表，也就是说一个密码表只能破解一个用户的密码，这还是建立在已经得到数据库和加密算法的情况下，成本瞬间高了几个数量级

> 参考
>
> [java实现SHA256、SHA512、MD5加密](https://www.jianshu.com/p/ecb3b85adf6f)
> [加盐密码保存的最通用方法是？](https://www.zhihu.com/question/20299384)
