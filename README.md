## Summary

1. root目录下放置RootCA, RootCA仅用于签发IntermediateCA, RootCA的自签名在此目录内完成
1. intermediate目录下放置IntermediateCA, IntermediateCA的签发在此目录内完成
1. end目录下放置End Entity, End Entity的签发在此目录内完成

## Default

1. 默认示例RootCA: `LikeNetworkCoLtd`; Common Name: `Like Network Co. Ltd.`
1. 默认示例IntermediateCA: `LikeNetwork-IG1`; Common Name: `Like Network Intermediate - Generic 1`

## Notice

* 通常情况下RootCA完全不应当发生变更
* IntermediateCA发生私钥泄漏时, 或者有其它用途考虑时, 才会进行注销/新增
* End Entity则是在有新的域名签发需求时进行新增, 或者私钥发生泄漏时进行注销
* CRL默认路径为: http://pki.likeit.cn/xxx.crl, 暂不考虑部署

## Deploy

在部署End Entity之前, RootCA应当已经被正确的植入到操作系统/应用程序的根证书信任链之中

由于IntermediateCA不要求植入操作系统/应用程序, 因此, 部署End Entity的证书同时还需要额外提供签发该证书的IntermediateCA的证书(不需要IntermediateCA的私钥)

操作示例如下, 不同的HTTP服务器的要求不尽相同, 以NginX为例:

    cat end.crt intermediate.crt > deploy.crt

## New Intermediate CA

如果要启用新的IntermediateCA, 可以参照以下示例操作:

创建一个新的IntermediateCA: `LikeNetwork-IG2`; Common Name: `Like Network Intermediate - Generic 2`

    # 创建新的IntermediateCA
    ./intermediate/create LikeNetwork-IG2 'Like Network Intermediate - Generic 2'

    # 复制现成的End Entity操作目录
    cp -r end-IG1 end-IG2

    # 清理新的End Entity目录
    ./end-IG2/clean

    # 编辑 end-IG2/openssl.cnf, 将 _CANAME_ 的值修改为 LikeNetwork-IG2
    # TODO: 手工编辑

    # 使用新的IntermediateCA签发api.paadoo.net
    ./end-IG2/create api.paadoo.net api.paadoo.net

## Encrypt & Decrypt

默认状态下所有生成的私钥都使用随机密码加密, 加密用的密码会在生成私钥的过程中输出到标准输出中, 请妥善保管

### EC

默认生成的公私钥对使用ECDSA

1. 加密指令: `openssl ec -aes-256-cbc -in <plain key file path> -out <encrypted key file path>`, 根据提示输入保护密码
1. 解密指令: `openssl ec -in <encrypted key file path> -out <plain key file path>`, 根据提示输入保护密码

### RSA

1. 加密指令: `openssl rsa -aes-256-cbc -in <plain key file path> -out <encrypted key file path>`, 根据提示输入保护密码
1. 解密指令: `openssl rsa -in <encrypted key file path> -out <plain key file path>`, 根据提示输入保护密码
