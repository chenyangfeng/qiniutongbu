**使用前请先配置空间信息！**

#### 配置说明：

+ `AccessKey`：密钥组合
+ `SecretKey`：密钥组合
+ `Bucket`：空间名
+ `Domain`：使用哪个域名来访问资源
+ `Private`：是否为私有空间
+ `Prefix`：资源前缀，如有填写则只下载匹配的文件，留空则下载所有文件
+ `SaveAs`：本地保存路径
+ `OverWrite`：覆盖本地文件

#### 项目依赖：

+ 本程序依赖 `QiniuBackup.exe.config` 来记录配置信息，请确保运行目录中含有该文件；
+ 本程序依赖 `LitJson.dll` 来处理 JSON，请确保运行目录中含有该文件；
