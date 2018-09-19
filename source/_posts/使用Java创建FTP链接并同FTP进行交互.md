---
title: 使用Java创建FTP链接并同FTP进行交互
date: 2018-09-09 19:14:21
comments: true
tags:
	 - Java
---


## 实现方式

1. 使用 JDK 自带的 ftpclient 进行 FTP 的交互
2. 使用 Apache 的 commons-net 进行 FTP 交互
3. 手动创建 Socket 使用 FTP 命令进行交互

## 使用 JDK 自带的 ftpclient 进行 FTP 的交互

JDK 自带的 ftpclient 大家说会有一些问题，不是很好解决，这里我就没有再尝试。

## 使用 Apache 的 commons-net 进行 FTP 交互

使用 Apache 的 comment-net 是比较主流的做法，可以方便的进行目录切换、目录下文件获取、上传、下载等功能。

下面是上传的简单代码，其中有几点要说明的是：
1. FTP传输协议支持的编码为 iso-8859-1 ，我们接受文件的 FTP 服务要求编码为 gb2312 所以要进行 `new String(fullFile.getBytes("gb2312"),"iso-8859-1")` 对编码进行处理。
2. 由于通常情况下为了安全考虑上传文件的服务器并不会开放太多的端口，并且很多 FTP 默认的模式为被动模式，所以需要进行模式切换 `ftpClient.enterLocalPassiveMode()`。

_客户端通过21端口同服务器的指定端口进行连接，连接后支持发送FTP命令获取目录下文件、切换目录、删除目录等操作，这个连接的通道称为命令连接通过；但是如果需要传输文件，那么就需要开启额外的端口进行文件数据的交互，这个连接的通道称为数据连接通道。_
_主动模式(PORT): 当要传输文件时，客户端主动开启一个随机 >1024 的端口进行监听，等待服务端连接，然后通过命令连接通道告知服务端自己开启的端口，服务端通过20端口同客户端开启的端口进行连接，连接后进行文件的传输。_
_被动模式(PASV): 当要传输文件时，服务端主动开启一个随机 >1024 的端口进行监听，等待客户端连接，然后通过命令连接通道告知客户端自己开启的端口，客户端通过20端口同客户端开启的端口进行连接，连接后进行文件的传输。_

```
FTPClient ftpClient = new FTPClient();
try {
        ftpClient.connect(host);
        int reply = ftpClient.getReplyCode();
        //是否连接成功
        if ( !FTPReply.isPositiveCompletion( reply ) )         {
            throw new ConnectException( ftpInfoDto.getFtpHost()+" 服务器拒绝连接" );
        }
        //登陆
        if (!ftpClient.login(user, pass)) {
            throw new ConnectException( "用户名或密码错误" );
        }
        ftpClient.enterLocalPassiveMode();
        ftpClient.setFileType(FTPClient.BINARY_FILE_TYPE);
        ftpClient.setDefaultTimeout(2*60*1000);
        ftpClient.setConnectTimeout(4*60*1000);
        ftpClient.setDataTimeout(8*60*1000);

        final String filename =
                    Objects.equals(fullFile.lastIndexOf("/"), -1) ?
                            new String(fullFile.getBytes("gb2312"), "iso-8859-1"):
                            new String(fullFile.substring(fullFile.lastIndexOf("/") + 1).getBytes("gb2312"), "iso-8859-1");

        InputStream localFile = new FileInputStream(fullFile);

        InputStream in = ftpClient.retrieveFileStream(filename);
        if (in != null) {
            log.info("file {} 已存在",fullFile);
            return true;
        }
        Boolean flag = ftpClient.storeFile(filename, localFile);
        localFile.close();
        log.info("flag::{}:", flag);

        ftpClient.logout();
        ftpClient.disconnect();
        return flag;
    } catch (Exception e) {
            log.error(Throwables.getStackTraceAsString(e));
            throw e;
    } finally {
        if(ftpClient.isConnected()){
            try{
                ftpClient.disconnect();
            } catch (Exception e){
                log.error("disconnect ftp failed");
            }
        }
    }        
``` 

有一个需要额外的说明的监听器，这个 ftpclient 支持进行监听器配置，将客户端同服务端交互的命令以及服务端的返回结果输出到一个流当中,我在测试环境进行命令的采集，并且通过追加的方式写入到一个txt文件中
`tpClient.addProtocolCommandListener( new PrintCommandListener( new   PrintWriter(
                    new   BufferedWriter(
                            new   FileWriter(homePath+"ftpCommand.txt",true)))) );`

这当然时理想的情况，并且在本地测试，测试环境测试，代码运行的都是很好的，但是生产环境就出了莫名奇妙的问题，上传文件的时候，在FTP服务器上文件名可以创建出来，但是文件大小始终时 0kB，说明无法进行文件数据传输，也就是说数据连接通道存在文件。由于生产环境的特殊性，历史遗留系统，无法复制相应配置进行测试，最终采用第三种方式才解决问题。

## 手动创建 Socket 使用 FTP 命令进行交互

[代码链接](https://github.com/chaiyunhao/FtpSocket)

通过手动创建 Socket 的方式同 FTP交互，需要了解 FTP 的命令字和应答码

#### 命令字

|命令       	|描述 |
|--------------|:--------------------------|
|ABOR	|中断数据连接程序|
|ACCT <account>	|系统特权帐号|
|ALLO <bytes> 	|为服务器上的文件存储器分配字节|
|APPE <filename>	|添加文件到服务器同名文件|
|CDUP <dir path>	|改变服务器上的父目录|
|CWD <dir path>	|改变服务器上的工作目录|
|DELE <filename>	|删除服务器上的指定文件|
|HELP <command>	|返回指定命令信息|
|LIST <name>	|如果是文件名列出文件信息，如果是目录则列出文件列表|
|MODE <mode>	|传输模式（S=流模式，B=块模式，C=压缩模式）|
|MKD <directory>	|在服务器上建立指定目录|
|NLST <directory>	|列出指定目录内容|
|NOOP	|无动作，除了来自服务器上的承认|
|PASS <password>	|系统登录密码|
|PASV	|请求服务器等待数据连接|
|PORT <address>	|IP 地址和两字节的端口 ID|
|PWD	|显示当前工作目录|
|QUIT	|从 FTP 服务器上退出登录|
|REIN	|重新初始化登录状态连接|
|REST <offset>	|由特定偏移量重启文件传递|
|RETR <filename>	|从服务器上找回（复制）文件|
|RMD <directory>	|在服务器上删除指定目录|
|RNFR <old path>	|对旧路径重命名|
|RNTO <new path>	|对新路径重命名|
|SITE <params>	|由服务器提供的站点特殊参数|
|SMNT <pathname>	|挂载指定文件结构|
|STAT <directory>	|在当前程序或目录上返回信息|
|STOR <filename>	|储存（复制）文件到服务器上|
|STOU <filename>	|储存文件到服务器名称上|
|STRU <type>	|数据结构（F=文件，R=记录，P=页面）|
|SYST	|返回服务器使用的操作系统|
|TYPE <data type>	|数据类型（A=ASCII，E=EBCDIC，I=binary）|
|USER <username>>	|系统登录的用户名|


#### 应答码

|响应代码 	|解释说明 |
|--------------|:--------------------------|
|110|	新文件指示器上的重启标记|
|120|	服务器准备就绪的时间（分钟数）|
|125|	打开数据连接，开始传输|
|150|	打开连接|
|200|	成功|
|202|	命令没有执行|
|211|	系统状态回复|
|212|	目录状态回复|
|213|	文件状态回复|
|214|	帮助信息回复|
|215|	系统类型回复|
|220|	服务就绪|
|221|	退出网络|
|225|	打开数据连接|
|226|	结束数据连接|
|227|	进入被动模式（IP 地址、ID 端口）|
|230|	登录因特网|
|250|	文件行为完成|
|257|	路径名建立|
|331|	要求密码|
|332|	要求帐号|
|350|	文件行为暂停|
|421|	服务关闭|
|425|	无法打开数据连接|
|426|	结束连接|
|450|	文件不可用|
|451|	遇到本地错误|
|452|	磁盘空间不足|
|500|	无效命令|
|501|	错误参数|
|502|	命令没有执行|
|503|	错误指令序列|
|504|	无效命令参数|
|530|	未登录网络|
|532|	存储文件需要帐号|
|550|	文件不可用|
|551|	不知道的页类型|
|552|	超过存储分配|
|553|	文件名不允许|


## 更新
最终发现自己还是太天真了，给我的参考代码是 `.net` 的，原先的系统是运行在 windows 环境，现在 `java` 是部署在 linux 上的，生产的 ftp 服务是使用的 `serv-U` ,版本号为 6.4.0.6,`serv-U` 的配置中有一个允许被动模式传输数据的选项，这个选项是勾选的，但是后面的 ip 地址中并没有填写任何 ip,这样导致 liunx 环境连接时被动模式完全无法传输数据，但是有没有任何错误码抛出来。
切换为主动模式后一切正常

```
//ftpClient.enterLocalPassiveMode();
ftpClient.enterLocalActiveMode();
```

