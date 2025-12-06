+++
date = 2025-01-01
categories = ['开发备忘']
tags = ['Java', 'Hadoop', '证书管理', 'SSL/TLS']
title = 'Java 环境下的证书生成与信任管理'
filename = 'about-java-trust-certificate.md'
showHero = true
+++

在 Hadoop 体系下，经常用到 Java 的 SSL/TLS 功能，而 Java 的 SSL/TLS 功能是基于 JSSE（Java Secure Socket Extension）实现的。

证书体系与 OpenSSL 类似，都是基于 X.509 标准的证书体系。因此，证书可以互相转换使用，证书链也可以互相兼容。我们一般使用 OpenSSL 生成证书，然后导入到 Java 的 keystore 中使用。

这样做的好处是，可以统一管理证书，避免重复生成证书，减少运维成本。

# 证书格式

OpenSSL 生成的证书一般是 PEM 格式的，而 Java 使用的证书格式是 JKS 或 PKCS12 格式的。JKS 是 Java 自有的证书格式，而 PKCS12 是一种通用的证书格式，可以被多种软件和系统支持。JKS 出现较早，现在推荐使用 PKCS12，因为它更通用且安全性更高。

证书分为公钥证书和私钥证书两种，公钥证书用于加密数据，私钥证书用于解密数据。在 Server 端，一般会同时使用公钥证书和私钥证书，而在 Client 端，一般只需要使用公钥证书。OpenSSL 生成的证书可能有很多不同的后缀名，如 .crt、.cer、.pem 等，但它们本质上都是公钥证书，只是编码格式不同。OpenSSL 生成的私钥证书一般是 .key 后缀名。

Java 的 keystore 文件可以包含多个证书和私钥，可以通过别名（alias）来区分不同的证书和私钥。Java 的 keystore 文件可以使用 keytool 命令行工具来管理，可以创建、导入、导出、删除证书和私钥。

现在（2025年底）大部分教程和文档中，仍然使用 JKS 格式的证书，但在实际生产环境中，建议使用 PKCS12 格式的证书。

#  Keystore 与 Truststore

在 Java 的 SSL/TLS 体系中，有两个重要的概念：keystore 和 truststore。

- keystore：用于存储自己的证书和私钥，一般用于 Server 端。
- truststore：用于存储信任的 CA 证书，一般用于 Client 端。

在实际使用中，keystore 和 truststore 可以是同一个文件，也可以是不同的文件。一般情况下，Server 端需要使用 keystore 来存储自己的证书和私钥，而 Client 端需要使用 truststore 来存储信任的 CA 证书。

JVM 默认的 truststore 文件是 `$JRE_HOME/lib/security/cacerts`，这个文件中包含了很多常见的 CA 证书，可以直接使用。如果需要信任自定义的 CA 证书，可以将自定义的 CA 证书导入到这个文件中。

可以查看默认的 truststore 文件中的证书：

```bash
keytool -list -keystore $JRE_HOME/lib/security/cacerts --storepass changeit | less
```

我们对证书进行管理时，一般会创建一个单独的 truststore 文件，用于存储自定义的 CA 证书。这样做的好处是，可以避免修改默认的 truststore 文件，减少对系统的影响。

也可以将自己的 CA 证书导入到系统的 truststore 中，这样所有使用该 JVM 的应用程序都可以信任该 CA 证书。

理论上，truststore 中存储的证书都是公钥证书，而 keystore 中存储的证书可以是公钥证书，也可以是私钥证书。

# 证书生成

证书生成的逻辑步骤如下：

1. 生成 CA 证书，用于签发其他证书。
2. 生成服务器证书签名请求（CSR）。
3. 使用 CA 证书对服务器证书签名，生成服务器证书。
4. 将证书导入到 keystore 中。
5. 将 CA 证书导入到系统中（可选）。

不论使用 OpenSSL 还是 keytool，生成证书的逻辑步骤都是类似的。如果我们使用 OpenSSL 生成证书，那么就需要从 OpenSSL 生成的证书中导入到 Java 的 keystore 中使用；如果使用 keytool 生成证书，那么就可以直接使用生成的 keystore 文件。

因为 OpenSSL 和 Java 信任的 CA 位置不同，所以不仅需要将 CA 证书导入到 Java 的 truststore 中，还需要将 CA 证书导入到系统的信任 CA 列表中。

生成 CA 证书文件，CA 证书也是一个普通的 X.509 证书，只不过它有一些特殊的扩展属性，用于标识它是一个 CA 证书，可以用于签发其他证书：

```bash
keytool -genkeypair -keyalg RSA -keysize 2048 -keystore bigdata.ca.jks \
-alias bigdata_ca -dname "C=CN,CN=BIGDATA.CAS.CN" -storepass secret -keypass secret \
-validity 36500 -ext KeyUsage=keyCertSign,digitalSignature \
-ext BasicConstraints=CA:true
```

这一步中，我们引入了 BasicConstraints 扩展属性，指定该证书是一个 CA 证书，可以用于签发其他证书。另外，KeyUsage 扩展属性指定了该证书的使用范围。

```bash
keytool -exportcert -alias bigdata_ca \
-keystore bigdata.ca.jks -storepass secret -file bigdata.ca.crt
keytool -importcert -alias bigdata_ca \
-keystore $JRE_HOME/lib/security/cacerts -storepass changeit -file bigdata.ca.crt
```

bigdata.ca.crt 文件就是生成的 CA 证书（格式为 PEM 格式），可以将该证书分发给需要信任该 CA 证书的客户端使用。然后通过 importcert 命令将 X.509 证书导入到系统的 truststore 中。

不可以直接将 jks 文件导入到系统的 truststore 中，原因如下：

1. 系统的 truststore 一般是 cacerts 文件，直接导入 jks 文件会覆盖掉原有的 cacerts 文件，导致系统中其他应用程序无法使用原有的 CA 证书。
2. 如果直接导入 jks 文件，那么 jks 文件中的所有证书都会被导入到系统的 truststore 中，可能会导致信任的 CA 证书过多，增加安全风险。

所以使用 importcert 命令将 X.509 证书导入到系统的 truststore 中是比较安全和合理的做法。

生成服务器证书签名请求（CSR），keytool 会直接生成公私钥对，然后根据公钥生成 CSR：

```bash
keytool -genkeypair -keyalg RSA -keysize 2048 -keystore openlookeng.jks \
-alias server -dname "C=CN,CN=OPENLOOKENG.BIGDATA.CAS.CN" -storepass secret -keypass secret
keytool -certreq -alias server -keystore openlookeng.jks -storepass secret -keypass secret -file openlookeng.csr
```

使用 CA 证书对服务器证书签名，在签名过程中可以指定证书的扩展属性，如 KeyUsage、ExtendedKeyUsage、SubjectAlternativeName 等；

指定 SubjectAlternativeName 可以让证书支持多个域名或 IP 地址，也就是说当客户端访问服务器时，可以使用这些域名或 IP 地址进行验证；

指定有效期为 100 年（36500 天），实际生产环境中一般不会这么长，通常为 1 年或 2 年：

```bash
keytool -gencert -alias bigdata_ca -keystore bigdata.ca.jks -storepass secret \
-infile openlookeng.csr -outfile openlookeng.crt \
-validity 36500 \
-ext KeyUsage=digitalSignature,dataEncipherment,keyEncipherment \
-ext ExtendedKeyUsage=serverAuth,clientAuth \
-ext SubjectAlternativeName:c=DNS:node26,DNS:node27,IP:172.20.9.26,IP:172.20.9.27
```

然后将签发完成的证书导入到 keystore 中：

```bash
keytool -importcert -trustcacerts -alias server \
-keystore openlookeng.jks -storepass secret \
-file openlookeng.crt
```

到了这一步，服务器端的 keystore 文件就生成完成了，可以用于配置 Java 应用程序的 SSL/TLS 功能。当前系统的 JVM 已经信任了 bigdata_ca 这个 CA 证书，可以用于客户端验证服务器端的证书。

将 CA 证书导入到 Linux 系统中，确保 curl、wget 等工具可以信任该 CA 证书：

```bash
update-ca-trust -f -i bigdata.ca.crt
```

# 总结

相对于其他自签名证书的生成说明，我这里使用了自己的 CA 证书来签发服务器证书，这样做的好处是可以统一管理证书，避免重复生成证书，减少运维成本。另外，CA 证书可以导入到系统的 truststore 中，确保所有使用该 JVM 的应用程序都可以信任该 CA 证书。

在 Hadoop 集群中，建议使用统一的 CA 证书来签发各个节点的服务器证书，这样可以简化证书管理，提高安全性。

另外，也有一些集成方案例如 freeipa，可以用于集中管理证书和密钥，简化证书的生成和分发过程。在实际生产环境中，可以根据具体需求选择合适的证书管理方案。
