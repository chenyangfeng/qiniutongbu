
 配置参数
+ `AccessKey`：AK
+ `SecretKey`：SK
+ `Bucket`：空间
+ `Domain`：域名 示例  value="http://obxxxx.bkt.clouddn.com/"
+ `Private`：是否私有
+ `Prefix`：资源前缀
+ `SaveAs`：本地保存路径 示例 value="D:\pic\"
+ `OverWrite`：覆盖本地文件

外部依赖
+  `qiniutongbu.exe.config` 来记录配置信息.
+  `LitJson.dll` 来处理 JSON，确保目录中含有该文件；


参考资料
+ http://developer.qiniu.com/code/v6/sdk/csharp.html
+ https://github.com/qiniu/csharp-sdk
+ https://github.com/abelyao/qiniu-backup
+ https://github.com/wzyuliyang/qiniu4blog


官方文档摘取：
+ https://support.qiniu.com/hc/kb/article/112823/
qLocalBackup 本地备份工具
1. 客户端应用（如 App）向七牛云（Qiniu）发起上传（此处省略上传授权过程）；

2. 七牛云成功保存文件后，向业务服务器（Biz Server）发起回调通知；

3. 业务服务器向 qLocalBackup 发起记录请求；

4. 业务服务器在适当时候向 qLocalBackup 发起备份请求；

5. qLocalBackup 计算差分增量，自动从七牛云拖回未备份的文件；

6. 拖回的文件将保存至本地存储系统中，根据文件的 Key 还原组织成文件夹结构。


功能特性

记录单个文件的上传记录

可作为后台服务运行

异步执行备份操作

支持记录备份进度


下载


使用方法


前提

客户端应用在上传策略中指定 callbackUrl 字段；

业务服务器监听 callbackUrl 指定接口，并在收到通知时，转调用 qLocalBackup 的新增记录接口。


配置文件

qLocalBackup 使用 JSON 格式作为配置文件的标准书写格式，内容如下：

{
    "bind_host":    "<BindHost      string>",
    "bucket":       "<Bucket        string>",
    "domain":       "<Domain        string>",
    "referer":      "<Referer       string>",
    "base_dir":     "<BaseDir       string>",
    "access_key":   "<AccessKey     string>",
    "secret_key":   "<SecretKey     string>",
    "sync_step":     <SyncStep      int64>,
    "try_times":     <TryTimes      int64>,
    "get_expires":   <GetExpires    int64>,
    "debug_level":   <DebugLevel    int64>
}
字段	必填	说明	范例
bind_host	是	服务器绑定的 IP 及端口。
出于安全考虑，不建议监听公网 IP。	127.0.0.1:51234
bucket	是	文件所处的存储空间名。	some-bucket
domain	是	空间绑定的域名， 不含 http:// 部分， 可在空间设置-域名设置 中查看。	some-bucket.qiniudn.com
referer	
如果已为空间设置防盗链，需要提供允许的 Referer 头部值。	http://www.abc.com
base_dir	是	备份存储的本地文件夹。	Linux 环境：
/home/user/databackup
Windows 环境：
D:/databackup
access_key	是	密钥组中的AccessKey, 可以在 账号设置-密钥 中查看。	iguImegxd6hbwF8J6ij2dlLIgycyU4thjg-xxxxx
secret_key	是	密钥组中的SecretKey, 可以在 账号设置-密钥 中查看。	ejqDluRblcUIW0ZIqP1gxxxxxxxxxxxxxxxxxxxx
sync_step	
记录备份进度的间隔，默认为 16 次	20
try_times	
同一个文件下载重试次数，默认为 3 次	3
get_expires	
生成的私有下载链接的有效时间，默认为 3600 秒	1800
debug_level	
日志等级，默认为 0（输出 DEBUG 级别及以上的日志）	1

使用步骤

下载适用于本地操作系统的 qLocalBackup ；

在 qLocalBackup 所在文件夹中，按上述格式编辑一个配置文件（比如命名为 backup.conf）：

{
    "bind_host":    "127.0.0.1:51234",
    "bucket":       "some-bucket",
    "domain":       "some-bucket.qiniudn.com",
    "base_dir":     "home/user/databackup",
    "access_key":   "<AccessKey>",
    "secret_key":   "<SecretKey>"
}
直接启动：

./qlocalbackup backup.conf
或者启动为后台服务模式：

nohup ./qlocalbackup backup.conf >> backup.log 2>&1 &
注意：配置文件和日志文件可以放置在别的文件夹中。


测试步骤

使用控制管理台的 内容管理-上传 ，向指定 Bucket 上传一张图片（本例中为 foo.jpg）；

在获取该图片的下载外链，在浏览器中打开检查是否可下载，比如：

http://some-bucket.qiniudn.com/foo.jpg
取出外链中的 Path 部分（本例中为 foo.jpg），调用 qLoaclBackup 的新增记录接口：

curl -v "http://127.0.0.1:51234/addkey?key=foo.jpg"
调用 qLoaclBackup 的异步备份接口：

curl -v "http://127.0.0.1:51234/backup"
过一会后检查文件是否已备份：

ls -1 /home/user/databackup/foo.jpg

开发接口

接口	参数	说明	范例	状态码
/addkey	key	新增一个 key。	curl -v "http://127.0.0.1:51234/addkey?key=abcd"	200 新增成功
400 没有提供 key 参数
500 新增失败
/backup	无	启动备份任务。	curl -v "http://127.0.0.1:51234/backup"	200 启动成功，备份任务会在异步模式下执行
/get	key	获取已备份文件。	curl -v "http://127.0.0.1:51234/get?key=abcd"	200 返回文件内容
400 没有提供 key 参数
404 该 key 没有对应的本地文件
注意：参数都需要进行 URL 转义。



