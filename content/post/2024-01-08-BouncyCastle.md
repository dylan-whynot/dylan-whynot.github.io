---
title:       "BouncyCastle简单使用"
subtitle:    "加密-解密基础使用"
description: "BouncyCastle加密解密算法API使用"
date:        "2024-01-08"
author:      "Dylan"
tags:        ["java", "BouncyCastle"]
categories:  ["编程" ]
---

近期由于工作项目需要涉及发放签名以及校验签名的功能，为此简单调研了第三方加密-解密算法库BouncyCastle。

<!--more-->
![](/post_images/BouncyCastle的简单使用/home_logo.gif)

# 介绍 BouncyCastle
[Bouncy Castle](http://www.bouncycastle.org/)是一种提供了很多哈希算法和加密算法的第三方API。BC（Bouncy Castle）目前的支持语言有java和C#。BC由澳大利亚的慈善组织（Legion of the Bouncy Castle Inc)提供支持.Bouncy Castle项目产生的理由是两个同事对于每次换到java服务端开发都要重新发明一套加密库感到疲惫，所以创建了BouncyCastle.这个项目,成立于2000年5月,最初是用Java编写的,但后来在2004年添加一个c#的API。原始的Java API包括大约27000行代码,包括测试代码并提供支持J2ME, JCA(Java Cryptography Extension) / JCE(Java Cryptography Architecture)提供者,和基础X.509 证书的生成。

Bouncy Castle架构由两个主要组件,支持基本加密功能。这些被称为“轻量级”API和Java加密扩展(JCE)提供者。JCE提供程序进一步组件建立在支持额外的功能,如PGP的支持,S / MIME等。

>The Bouncy Castle architecture consists of two main components that support the base cryptographic capabilities. These are known as the 'light-weight' API, and the [Java Cryptography Extension](https://en.wikipedia.org/wiki/Java_Cryptography_Extension) (JCE) provider. Further components built upon the JCE provider support additional functionality, such as [PGP](https://en.wikipedia.org/wiki/Pretty_Good_Privacy) support, [S/MIME](https://en.wikipedia.org/wiki/S/MIME), etc.

# The Java Provider Architecture 
Provider Architecture 与 JCA（**Java Cryptography Architecture**）提供了JVM安全服务的基础.提供了许多java相关的安全服务(Java Secure Socket Extension (JSSE) which provides access to Secure Socket Layer (SSL) and Transport Layer Security (TLS) implementations, the Java Generic Security Services (JGSS) APIs, and the Simple Authentication and Security Layer (SASL))相关的API。类似接口模式,像开发人员提供统一的接口调用，而具体的实现方式有指定的Provider提供,Provider可能是JVM也可以是第三方的提供商如 Bouncy Castle.

Provider Architecture 主要有如下两个特色.
- 把开发人员通常调用的功能和功能具体的实现分开.
- 可以指定使用哪种提供者的算法.

  ```java
  Cipher instance = Cipher.getInstance("Blowfish/CBC/PKCS5padding","BC");
  //指定由BC提供算法
  ```
service provider interface (SPI)

![](/post_images/BouncyCastle的简单使用/Snipaste_2020-04-16_20-43-08.jpg)


## JVM默认Provider
&ensp; java提供了许多默认的Provider,Provider的数量种类与JVM版本有关，java1.4有5个默认的集合,java9有12个.我安装的java8有10个默认的Provider。
```java
Provider[] providers = Security.getProviders();
for (int i = 0; i != providers.length; i++) {
            System.out.print(providers[i].getName()); System.out.print(": ");
            System.out.print(providers[i].getInfo()); System.out.println();
        }
--------------------------------------------------------------
SUN: SUN (DSA key/parameter generation; DSA signing; SHA-1, MD5 digests; SecureRandom; X.509 certificates; JKS & DKS keystores; PKIX CertPathValidator; PKIX CertPathBuilder; LDAP, Collection CertStores, JavaPolicy Policy; JavaLoginConfig Configuration)
SunRsaSign: Sun RSA signature provider
SunEC: Sun Elliptic Curve provider (EC, ECDSA, ECDH)
SunJSSE: Sun JSSE provider(PKCS12, SunX509/PKIX key/trust factories, SSLv3/TLSv1/TLSv1.1/TLSv1.2)
SunJCE: SunJCE Provider (implements RSA, DES, Triple DES, AES, Blowfish, ARCFOUR, RC2, PBE, Diffie-Hellman, HMAC)
SunJGSS: Sun (Kerberos v5, SPNEGO)
SunSASL: Sun SASL provider(implements client mechanisms for: DIGEST-MD5, GSSAPI, EXTERNAL, PLAIN, CRAM-MD5, NTLM; server mechanisms for: DIGEST-MD5, GSSAPI, CRAM-MD5, NTLM)
XMLDSig: XMLDSig (DOM XMLSignatureFactory; DOM KeyInfoFactory; C14N 1.0, C14N 1.1, Exclusive C14N, Base64, Enveloped, XPath, XPath2, XSLT TransformServices)
SunPCSC: Sun PC/SC provider
SunMSCAPI: Sun's Microsoft Crypto API provider
```

# Bouncy Castle APIs 架构

![](/post_images/BouncyCastle的简单使用/Snipaste_2020-04-16_21-32-31.jpg)

Bouncy Castle最早就是作为java平台加密项目开发的,在使用基本的API组装之后发现可以使用组件来构建更高级别的API，这些API支持JCA和JCE(Java Cryptography Extension).于是就有了像CMS, S/MIME, and X.509 等Provider.
>There were two attempts at the higher level APIs. The first attempt was closely coupled to the BC provider and the JCA. The second attempt, seen from BC 1.46 onward, introduced interfaces for providing the “gross operations” required for the different APIs. Together with the basic provider/low-level split, this resulted in the arrangement of the different APIs looking something like the following.


# 使用
##  安装Bouncy Castle
java环境安装Bouncy Castle有两种方式

### 静态安装
复制bcprov-ext-jdk15on-165.jar  放入 java安装目录下/jre/lib/ext/文件夹内,
然后编辑/jre/lib/security/java.security 文件 编辑如下 前10个是已经存在添加第11个
```java
security.provider.1=sun.security.provider.Sun
security.provider.2=sun.security.rsa.SunRsaSign
security.provider.3=sun.security.ec.SunEC
security.provider.4=com.sun.net.ssl.internal.ssl.Provider
security.provider.5=com.sun.crypto.provider.SunJCE
security.provider.6=sun.security.jgss.SunProvider
security.provider.7=com.sun.security.sasl.Provider
security.provider.8=org.jcp.xml.dsig.internal.dom.XMLDSigRI
security.provider.9=sun.security.smartcardio.SunPCSC
security.provider.10=sun.security.mscapi.SunMSCAPI
security.provider.11=org.bouncycastle.jce.provider.BouncyCastleProvider
```
### 运行时安装
```java
Security.addProvider( new BouncyCastleProvider());
或者
 Cipher.getInstance("Blowfish/ECB/NoPadding", new BouncyCastleProvider());
```

## 使用-数字加密
非对称加密.简单流程是生成公钥和私钥,发送端使用公钥加密文本，生成密文,接受端使用私钥对密文进行解密,得到实际文本。
```java
   public static void generateKeyPair() throws NoSuchProviderException, NoSuchAlgorithmException, InvalidAlgorithmParameterException {

        KeyPairGenerator generator2 = KeyPairGenerator.getInstance("RSA", "BC");
        generator2.initialize(2048);
        KeyPair kayPair2 = generator2.generateKeyPair();
        publicKey = kayPair2.getPublic();
        privateKey = kayPair2.getPrivate();
    }
public  static String encryptData(String data) throws IOException, InvalidCipherTextException, NoSuchPaddingException, NoSuchAlgorithmException, NoSuchProviderException, InvalidKeyException {
        AsymmetricBlockCipher cipher = new RSAEngine();
        AsymmetricKeyParameter priKey = PrivateKeyFactory.createKey(privateKey.getEncoded());
        cipher.init(true,priKey);
        byte[] encryptDataBytes = cipher.processBlock(data.getBytes("utf-8")
                , 0, data.getBytes("utf-8").length);
        String encryptData=ENCODER_64.encodeToString(encryptDataBytes);
        return  encryptData;
    }

    public static String decryptData(String data) throws IOException, InvalidCipherTextException {
        byte[] encryptDataBytes=DECODER_64.decode(data);
        AsymmetricBlockCipher cipher = new RSAEngine();
        AsymmetricKeyParameter pubKey = PublicKeyFactory.createKey(publicKey.getEncoded());
        //false解密
        cipher.init(false, pubKey);
        byte[] decryptDataBytes=cipher.processBlock(encryptDataBytes, 0, encryptDataBytes.length);
        String decryptData = new String(decryptDataBytes,"utf-8");
        return decryptData;
    }
    
    public static void main(String[] args) throws Exception {
        // 配置 BouncyCastle
        Security.addProvider(new BouncyCastleProvider());

        String data = "我的谁呀";
        generateKeyPair();
        String encryptData = encryptData(data);
        System.out.println(encryptData);
        String data1 = decryptData(encryptData);
        System.out.println(data1);
    }
    
输出结果
ex6YFLDUxTXJ/2N6z17//M7bEif4uY3fehZqOnM1zMEWLkFrAUyi/zQdxWL6KS8MlWzrr8nMSLeInfjSgb8dzdwfbtcol8HCesSX6ynkB82hp8yIrF6jrRFpNo51q3vRIS9WSLsDDdlq68GhYbECZ5GXxpq3wZAf71jnh+7B9PwRZ15zwzuiHDSZIaIIuM7o+t2tER+LpexIgrCbwTevQ6IfNXWYlv8bw9buT799OQOC+C3MFtP/ml8jB4NfeAnuDmq4Vakf3UJ7t7VvyqIqwJ/bRExHZcWtzqUsAMgeTi+EJgi0+NBgmx8UoMFMGixsr4+Hq0SfsYRcA7TcZEdUFg==
我的谁呀

```

## 使用-数字签名
数字签名也是使用到非对称加密技术,操作过程与数字加密相反,使用私钥进行签名,公钥进行校验签名
```java

    /**私钥*/
    private static PrivateKey privateKey;

    /**公钥*/
    private static PublicKey publicKey;
    /**Base64 转码**/
    private static final Base64.Encoder ENCODER_64 = Base64.getEncoder();

    /**Base64 解码**/
    private static final Base64.Decoder DECODER_64 = Base64.getDecoder();

    /**签名算法1*/
    private static final String SIGNATURE_ALGORITHM = X9ObjectIdentifiers.ecdsa_with_SHA512.getId();

    /***签名算法2**/
    private static final String SIGNATURE_ALGORITHM2 ="SHA256withRSA";




    /**
     * 生成一对公私钥
     *
     * @return
     */
    public static void generateKeyPair() throws NoSuchProviderException, NoSuchAlgorithmException, InvalidAlgorithmParameterException {
        
        // 算法1
        KeyPairGenerator generator = KeyPairGenerator.getInstance("EC", "BC");
        generator.initialize(new ECGenParameterSpec("P-256"));
        KeyPair keyPair1 = generator.generateKeyPair();
        publicKey = keyPair1.getPublic();
        privateKey = keyPair1.getPrivate();
        //算法2
//        KeyPairGenerator generator2 = KeyPairGenerator.getInstance("RSA", "BC");
//        generator2.initialize(2048);
//        KeyPair kayPair2 = generator2.generateKeyPair();
//        publicKey = kayPair2.getPublic();
//        privateKey = kayPair2.getPrivate();
    }

    public static String sign(String document) throws Exception {
        // 签名算法
        Signature singer = Signature.getInstance(SIGNATURE_ALGORITHM,"BC");
       // Signature singer = Signature.getInstance(SIGNATURE_ALGORITHM2,"BC");
        singer.initSign(privateKey);
        singer.update(document.getBytes("UTF-8"));
        byte[] signature = singer.sign();
        return ENCODER_64.encodeToString(signature);
    }
    
    public static boolean verifySignature(String document,String sign) throws Exception {
        Signature singer = Signature.getInstance(SIGNATURE_ALGORITHM,"BC");
      //  Signature singer = Signature.getInstance(SIGNATURE_ALGORITHM2,"BC");
        singer.initVerify(publicKey);
        singer.update(document.getBytes("utf-8"));
        return singer.verify(DECODER_64.decode(sign));
    }
    
    public static void main(String[] args) throws Exception {
        // 配置 BouncyCastle
        Security.addProvider(new BouncyCastleProvider());
        // 测试数据
        String document = "ABC";
        generateKeyPair();
        String sign = sign(document);
        System.out.println(sign);
        boolean verifySignature = verifySignature(document, sign);
        System.out.println(verifySignature);

    }
```
下面的是额外方法可以将PrivateKey 和publicKey 与String进行相互转换
```java
    public  static String toPrivateKeyString(PrivateKey privateKey ){
        byte[] encoded = privateKey.getEncoded();
        return ENCODER_64.encodeToString(encoded);

    }
    public  static String toPublicKeyString( PublicKey publicKey ){
        byte[] encoded = publicKey.getEncoded();
        return ENCODER_64.encodeToString(encoded);
    }
    
    public static PublicKey getPublicKey(String key) throws Exception {
        byte[] keyBytes;
        keyBytes = (new BASE64Decoder()).decodeBuffer(key);
        X509EncodedKeySpec keySpec = new X509EncodedKeySpec(keyBytes);
        KeyFactory keyFactory = KeyFactory.getInstance("EC","BC");
        PublicKey publicKey = keyFactory.generatePublic(keySpec);
        return publicKey;
    }
    public static PrivateKey getPrivateKey(String key) throws Exception {
        byte[] keyBytes;
        keyBytes = (new BASE64Decoder()).decodeBuffer(key);
        PKCS8EncodedKeySpec keySpec = new PKCS8EncodedKeySpec(keyBytes);
        KeyFactory keyFactory = KeyFactory.getInstance("EC","BC");
        PrivateKey privateKey = keyFactory.generatePrivate(keySpec);
        return privateKey;
    }

```

# 参考
[1] [官网](https://www.bouncycastle.org/) 

[2] [Java Cryptography: Tools and Techniques](https://leanpub.com/javacryptotoolsandtech)

[3] [wikipedia](https://en.wikipedia.org/wiki/Bouncy_Castle_(cryptography))