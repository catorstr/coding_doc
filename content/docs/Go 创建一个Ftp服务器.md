# Go 创建一个Ftp服务器

> FTP，即文件传输协议，这个功能在很多场合都在使用，
>
> 不过，大家有没有想过在自己的程序中集成这个功能呢
>
> 这样我们在上传文件时，就可以任意的进行各种控制了，
>
> 那么本节我们就一起来实现这个功能

```go
package main

import (
  "io/ioutil"
  "os"
  "time"
  ftpserver "github.com/fclairamb/ftpserverlib"
  "github.com/fclairamb/go-log/gokit"

  "github.com/fclairamb/ftpserver/config"
  "github.com/fclairamb/ftpserver/server"
)

var (
  ftpServer *ftpserver.FtpServer
  driver    *server.Server
)

func confFileContent() []byte {
  str := `{
  "version": 1,
  "accesses": [
    {
      "user": "test",
      "pass": "test",
      "fs": "os",
      "params": {
        "basePath": "./ftp"
      }
    }
  ],
  "passive_transfer_port_range": {
    "start": 2122,
    "end": 2130
  }
}`
  return []byte(str)
}

func startFtpServer() {
  // Arguments vars
  var confFile = "ftp.json"
  // Setting up the logger
  logger := gokit.NewGKLoggerStdout().With(
    "ts", gokit.GKDefaultTimestampUTC,
    "caller", gokit.GKDefaultCaller,
  )
  if _, err := os.Stat(confFile); err != nil && os.IsNotExist(err) {
    logger.Warn("No conf file, creating one", "confFile", confFile)
    if err := ioutil.WriteFile(confFile, confFileContent(), 0600); err != nil { // nolint: gomnd
      logger.Warn("Couldn't create conf file", "confFile", confFile)
    }
  }
  conf, errConfig := config.NewConfig(confFile, logger)
  if errConfig != nil {
    logger.Error("Can't load conf", "err", errConfig)
    return
  }
  // Loading the driver
  var errNewServer error
  driver, errNewServer = server.NewServer(conf, logger.With("component", "driver"))
  if errNewServer != nil {
    logger.Error("Could not load the driver", "err", errNewServer)
    return
  }
  // Instantiating the server by passing our driver implementation
  ftpServer = ftpserver.NewFtpServer(driver)
  // Overriding the server default silent logger by a sub-logger (component: server)
  ftpServer.Logger = logger.With("component", "server")
  if err := ftpServer.ListenAndServe(); err != nil {
    logger.Error("Problem listening", "err", err)
  }
  // We wait at most 1 minutes for all clients to disconnect
  if err := driver.WaitGracefully(time.Minute); err != nil {
    ftpServer.Logger.Warn("Problem stopping server", "err", err)
  }
}

func stop() {
  driver.Stop()
  if err := ftpServer.Stop(); err != nil {
    ftpServer.Logger.Error("Problem stopping server", "err", err)
  }
}

func main() {
  startFtpServer()
}

```

**扩展阅读：**

> 文件传输协议（File Transfer Protocol，FTP）
>
> 是用于在网络上进行文件传输的一套标准协议，
>
> 它工作在 OSI 模型的第七层，
>
> TCP 模型的第四层， 
>
> 即应用层，
>
> 使用 TCP 传输而不是 UDP， 
>
> 客户在和服务器建立连接前要经过一个“三次握手”的过程，
>
> 保证客户与服务器之间的连接是可靠的， 
>
> 而且是面向连接， 
>
> 为数据传输提供可靠保证。 
>
>  
>
> FTP允许用户以文件操作的方式
>
> （如文件的增、删、改、查、传送等）
>
> 与另一主机相互通信。
>
> 用户并不真正登录到自己想要存取的计算机上面而成为完全用户， 
>
> 可用FTP程序访问远程资源，
>
> 实现用户往返传输文件、目录管理以及访问电子邮件等等， 
>
> 即使双方计算机可能配有不同的操作系统和文件存储方式。