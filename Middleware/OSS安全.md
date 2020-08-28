OSS 安全



## 问题

身份证图片等资料，通常存在oss上。

这面临一个问题：如果其他人知道这个图片的oss连接，那么就可以直接访问到这种私密信息。

如何避免？



## 方案

#### 加密这类信息

拿到了也没有什么用。

不过 秘钥需要客户端能拿到。app类的还好，其他的基本没有好办法。

### 重定向

客户端向服务端请求，服务端做access token授权校验。通过后，直接重定向。

不过重定向的oss url还是能拿到，最终还是被看到了。

除非重定向到自己的url。

## 访问授权

aliyun oss的acl就有private（私有读写）

#### STS token

不要使用传统的 AccesskeyID （AK）、AccesskeySecret （SK），改用 STS token 的方式替代原来的校验方式



> Ak SK

![image.png](https://ucc.alicdn.com/pic/developer-ecology/78f85935c2a84e46a234d7d43f0990ae.png)



>STS

![image.png](https://ucc.alicdn.com/pic/developer-ecology/a5830478081a4db2a0c89520b37f9e42.png)

![image.png](https://ucc.alicdn.com/pic/developer-ecology/d8870b8fd77d4312acbc3f8004e335ee.png)



#### oss加密

可以采用 OSS 内容加密，在鉴权的基础上双重加密，使用 KMS 对内容进行加密，但操作过程略微复杂



> 参考

https://developer.aliyun.com/article/756372