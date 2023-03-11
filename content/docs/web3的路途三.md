# Web3去中心化存储
## IPFS
> xxxxxxxxxx ​​js

## Filecoin
> Filecoin 是专门重新为web3而设计的存储网络。ipfs提供热存储(快速+灵活数据检索)，Filecoin提供冷存储(永久保存，可验证)。

## 架构
```js
前端：web2(网页app、数据方案)，web3(NFTs、Dapp、更多...)
业务逻辑：app逻辑+智能合约
存储工具:web3.storage/Estuary/NET.storage等简化文件存储和管理的SDKs、APIs
底层节点：Filecoin&IPFS
```
## Kubo
> Kubo(go-ipfs) 是第一个 IPFS 实现，也是当今使用最广泛的一个。实施星际文件系统- 用于内容寻址的 Web3 标准，可与 HTTP 互操作。因此由 IPLD 的数据模型和用于网络通信的 libp2p 提供支持。Kubo是有go写的。安装地址：https://dist.ipfs.tech/#kubo
到官网去下载对应自己系统的kubo
### 以linux系统为例：
```bash
#下载go-ipfs 安装包
wget https://dist.ipfs.tech/kubo/v0.17.0/kubo_v0.17.0_linux-amd64.tar.gz
#下载ipfs的更新工具
wget https://dist.ipfs.tech/ipfs-update/v1.9.0/ipfs-update_v1.9.0_linux-amd64.tar.gz
#解压
tar -zxvf ipfs-update_v1.9.0_linux-amd64.tar.g
tar -zvxf kubo_v0.17.0_linux-amd64.tar.gz
cd ipfs-update/
./install.sh 
cd ..
cd kubo/
./install.sh
执行安装包自带的安装程序：./install.sh 可以自动将kubo(go-ipfs)配置到我们系统的环境变量中
```
更新ipfs
```bash
IPFS 有一个更新工具，可以通过ipfs update. 该工具没有与 IPFS 一起安装，以保持该逻辑独立于主代码库。要安装ipfs-update工具:https://dist.ipfs.tech/#ipfs-update
ipfs-update 是一个 CLI 工具，可帮助轻松更新和安装 Kubo IPFS。
```
### 启动ipfs服务
```bash
# 1. 初始化ipfs
ipfs init # 执行完后，默认初始化配置在/root/.ipfs目录下。可以通过设置环境变量自行指定初始化目录：export IPFS_PATH=/data_ipfs
cd /
mkdir data_ipfs
export IPFS_PATH=/data_ipfs
####
root@VM-16-3-ubuntu:~# ipfs init
generating ED25519 keypair...done
peer identity: 12D3KooWDYvt9FSkFs74GWUkh7qhNY9FG6ePy8ea2zTFTgdooquH
initializing IPFS node at /data_ipfs
to get started, enter:

	ipfs cat /ipfs/QmQPeNsJPyVWPFDVHb77w8G42Fvo15z4bG2X8D2GhfbSXc/readme

root@VM-16-3-ubuntu:~# 
####

# 2. 启动ipfs服务
ipfs daemon  # 此时ipfs是在前台执行，退出终端后，服务自行退出

说明：直接启动ipfs服务后，本地节点会自动连接到官网的节点，可以通过: ipfs swarm peers查看对应节点

# 3. 将ipfs加入系统服务，使用sytemctl 进行管理
vim /usr/lib/systemd/system/ipfs.service

[Unit]
Description=IPFS daemon
After=network.target
[Service]
Environment=IPFS_PATH=/data_ipfs  # 注意此处需要和init的目录保持一致
ExecStart=/usr/local/bin/ipfs daemon
Restart=always
User=root
Group=root
[Install]
WantedBy=multi-user.target

# 4. 加载系统配置
systemctl	daemon-reload

# 5. 通过systemctl 启动ipfs服务
systemctl start ipfs

# 6. 查看ipfs状态
systemctl status ipfs

# 7. 停止ipfs
systemctl stop ipfs
# 重启ipfs
systemctl restart ipfs

```
ipfs的配置文件在IPFS_PATH目录下的config。此时的ipfs服务除4001端口外不能够被其他服务器访问，因为绑定的ip地址为127.0.0.1。如果需要访问5001和8080对应端口。需要修改config中对应绑定的ip地址即可
### 验证ipfs是否安装成功
```bash
echo "hello world" > hello
ipfs add hello
# This should output a hash string that looks something like:
# QmT78zSuBmuS4z925WZfrqQ1qHaJ56DQaTfyMUF7F8ff5o
ipfs cat <that hash>
```
### ipfs常用命令
```bash
文件上传：ipfs add
上传文件ipfs add example.jpg
上传文件夹ipfs add –r dirpath
文件上传：ipfs add  [选项]  路径
主要的选项有：
-r  递归选项，用于添加文件夹
-q  添加成功后简化输出
-w  将文件或文件夹再打包成一个文件夹
-H  添加隐藏文件，跟-r一起使用
-s  规定如何切割待添加的文件
-t  用trickle-dag的形式生成dag
--pin 如果不想将自己上传的文件保留在本地，可以使用—pin=false
文件下载：ipfs cat
ipfs cat /ipfs/QmdDTor6dWzknFJPJuhJgrUYqd56WkFXYAxyxpEY7kUrEb > init.jpg
增加输出（> init.jpg），将文件保存到指定地方，否则会在命令提示符窗口输出显示

文件删除：

在其他节点存储本地节点文件之前可以通过以下命令删除文件：
（1）    删除缓存
命令：ipfs pin rm HASH
该HASH可以是文件或目录对应的HASH
（2）    删除二进制块
命令：ipfs block rm HASH
执行以上两步之后，重启ipfs站点后，便无法访问HASH对应的文件或目录了；若其他节点上存储了该相同文件，则无法删除，如空文件夹。

批量清理本地节点内容：ipfs repo gc
该命令用来清理本地节点缓存的未创建缓存（pin）的文件或文件夹，存在缓存（pin）的文件或文件夹无法通过该命令清理。
缓存（pin）操作：
添加pin：ipfs pin add ipfs/HASH
删除pin：ipfs pin rm ipfs/HASH
查看pin信息：ipfs pin ls ipfs/HASH
若不带路径（ipfs/HASH）参数，则会列出本地节点的所有pin信息
```
### ipfs数据迁移

```bash
# 1. 将旧的ipfs服务拷贝到需要部署新的ipfs服务器中
# 2. 设置IPFS_PATH为拷贝的旧ipfs服务的目录
# 3. 启动新的ipfs服务即可
```
## 通过go-sdk使用ipfs服务
```go
package utils

import (
	"bytes"
	"context"
	"encoding/base64"
	"fmt"
	"io/ioutil"
	shell "github.com/ipfs/go-ipfs-api"
)

type Ipfs struct {
	Url string
	Sh  *shell.Shell
}

func NewIpfs(url string) *Ipfs {
	sh := shell.NewShell(url)
	return &Ipfs{
		Url: url,
		Sh:  sh,
	}
}

// UploadIPFS 上传数据到ipfs
func (i *Ipfs) UploadIPFS(str string) (hash string, err error) {
	hash, err = i.Sh.Add(bytes.NewBufferString(str), shell.Pin(true))
	if err != nil {
		return
	}
	return
}

// UnPinIPFS 从ipfs上删除数据
func (i *Ipfs) UnPinIPFS(hash string) (err error) {
	err = i.Sh.Unpin(hash)
	if err != nil {
		return
	}

	err = i.Sh.Request("repo/gc", hash).
		Option("recursive", true).
		Exec(context.Background(), nil)
	if err != nil {
		return
	}

	return nil
}

// CatIPFS 从ipfs下载数据
func (i *Ipfs) CatIPFS(hash string) (string, error) {
	read, err := i.Sh.Cat(hash)
	if err != nil {
		return "", err
	}
	body, err := ioutil.ReadAll(read)

	return string(body), nil
}

```
# web3.storage
> web.storage是...

## Install web3.storage
```bash
go get github.com/web3-storage/go-w3s-client
```
## 使用web3.storage
> 要使用web3.storage我们首先需要在web3.storage的官网(https://web3.storage/)注册一个账号,然后使用api与我们的账户进行沟通。手动注册时它提供应该1TB的免费存储空间，关联github账号登入的话它给了5G的免费存储空间，所以该怎么做不用再多说了吧。
申请完账号后，创建一个api token
## 例子
```bash
#获取依赖
go get github.com/web3-storage/go-w3s-client
```
```go
//上代码
package main

import (
	"context"
	"fmt"
	"io/fs"
	"os"
	"path"

	"github.com/ipfs/go-cid"
	"github.com/web3-storage/go-w3s-client"
)

func main() {
	//1. 创建客户端实例，使用WithToken选项传入 API 令牌
	 var AUTH_TOKEN = "<AUTH_TOKEN>"
	client, err := w3s.NewClient(w3s.WithToken(AUTH_TOKEN))
	if err != nil {
		fmt.Printf("请检查 API 令牌 err: %v\n", err)
		return
	}
	//2.打开准备上传的文件
	filename := "./images/4.jpg"
	file, err := os.Open(filename)
	if err != nil {
		fmt.Printf("文件打开失败err: %v\n", err)
		return
	}
	//或者打开应该文件目录(已存在的文件目录)
	//   file, _ := os.Open("images")
	//或者创建应该文件目录
	//   img0, _ := os.Open("aliens.jpg")
	//   img1, _ := os.Open("donotresist.jpg")
	//依次打开当个文件后，组装到指定目录下
	//要上传目录，您需要从中导入w3fs包github.com/web3-storage/go-w3s-client/fs并使用该NewDir函数指定目录内容。
	//上传多个文件时，尽量给每个文件一个唯一的名称。请求中的所有文件put将被捆绑到一个内容存档中，如果每个文件都有一个唯一的、人类可读的名称，则链接到内部文件会容易得多。
	//   f := w3fs.NewDir("images", []fs.File{img0, img1})

	//3.上传文件/目录
	//Go 客户端的Client接口定义了一个Put的方法接受一个context.Context和fs.File，它可以是单个文件或目录。
	//每次Put上传文件后，会返回文件所在文件夹的根目录，如果这个目录这的文件发生改变，那么根目录也会跟着改变，
	//不管一次传多少文件，他都只会给你应该根目录，这样需要文件时就可以从这个根目录下载
	cid, _ := client.Put(context.Background(), file)
	if err != nil {
		fmt.Printf("c.Put err: %v\n", err)
		return
	}

	//4.浏览器访问我们上传的文件
	/*
		默认情况下，上传到 web3.storage 的文件将被包装在 IPFS 目录列表中。这会保留原始文件名，并使链接比 CID 字符串更人性化，后者看起来像是随机的乱码。
		你上传时从客户端取回的CID是目录的CID，不是文件本身！要使用 IPFS URI 链接到文件本身，只需将文件名添加到 CID，用/这样的分隔符：ipfs://<cid>/<filename>。
		要建立网关链接，请使用https://<cid>.ipfs.<gateway-host>/<filename>或https://<gateway-host>/ipfs/<cid>/<filename>，其中<gateway-host>是 HTTP 网关的地址，例如dweb.link。
	*/

	//获取文件名
	basename := path.Base(filename)
	fmt.Printf("https://%v.ipfs.dweb.link\n", cid) //这样加才能让普通的浏览器访问到我们上传的文件
	gatewayURL := fmt.Sprintf("https://%s.ipfs.dweb.link/%s\n", cid.String(), basename)
	fmt.Printf("gatewayURL: %v\n", gatewayURL) //直接访问上传的文件
	//https://bafybeibvut434226iocm7ezis5ikewtqpjqj3hrg5zrpnsidkz3cs4ngoq.ipfs.dweb.link/3.jpg
	//gatewayURL: https://bafybeiernrtk2zvyxeyvyijv35m7qet4wkjb2prjv4g3eb7jcewge5m4sy.ipfs.dweb.link/4.jpg
}

//5. 获取我们上传的文件/目录

//注意:目前2023.01.08 在官网上有一条提示说明，使用客户端库不稳定,建议使用 HTTP 网关，如w3link或ipfs 命令行工具。但目前我用着还没发现什么问题。
/*
	Go 客户端go-cid库的Client接口提供了一个Get的方法,它接受一个context.Context和一个Cid。
	该Get方法返回一个w3http.Web3Response，这是一个http.Response带有附加Files方法的标准，该方法提供对下载文件的访问。
*/
func retrieveFiles(client w3s.Client, cidString string) (fs.File, fs.FS, error) {

	parseCid, err := cid.Parse(cidString) //"github.com/ipfs/go-cid"
	if err != nil {
		return nil, nil, err
	}
	res, err := client.Get(context.Background(), parseCid)
	if err != nil {
		return nil, nil, err
	}

	if res.StatusCode != 200 {
		return nil, nil,
			fmt.Errorf("request for %s was unsuccessful: [%d]: %s",
				cidString, res.StatusCode, res.Status)
	}
	/*
		该Files方法返回一个fs.File可能是单个文件或目录的，具体取决于您请求的 CID。
		区分可以调用Stat文件，查看IsDir返回的方法fs.FileInfo。
		如果它是一个目录，您可以类型转换为fs.ReadDirFile接口并使用ReadDirFile的ReadDir方法列出内容.
	*/
	return res.Files()
}

func listDirectory(f fs.File) error {
	//确保文件实际上是一个支持列表内容的目录
	info, err := f.Stat()
	if err != nil {
		return err
	}

	d, ok := f.(fs.ReadDirFile)
	if !ok || !info.IsDir() {
		return fmt.Errorf("not a directory")
	}
	/*
		如果n > 0, ReadDir最多返回n个DirEntry结构。
			在这种情况下，如果ReadDir返回一个空片，它将返回一个解释原因的非nil错误。
			在目录的末尾，错误是io.EOF。(ReadDir必须返回io.EOF本身，而不是错误包装io.EOF。)
		如果n <= 0, ReadDir在一个切片中返回目录中的所有DirEntry值。
			在这种情况下，如果ReadDir成功(一直读取到目录的末尾)，它将返回切片和nil错误。
			如果它在目录结束之前遇到错误，ReadDir返回读到该点的DirEntry列表和一个非nil错误。
	*/
	dirEntries, err := d.ReadDir(0)
	if err != nil {
		return err
	}
	//打印每个文件或目录的名称。注意这不会遍历嵌套目录。
	for _, entry := range dirEntries {
		fmt.Println(entry.Name())
	}
	return nil
}

/*
或者，您可以使用该Files方法的第二个返回值，
它是一个fs.FS“文件系统”，表示下载中包含的所有文件。
该fs.ReadDir函数接受一个fs.FS和一个目录名来读取，可以是"/"读取根目录：
*/
func listDirectoryUsingFilesystem(fsys fs.FS) error {
	entries, err := fs.ReadDir(fsys, "/")
	if err != nil {
		return err
	}
	for _, entry := range entries {
		fmt.Println(entry.Name())
	}
	return nil
}

/*
上面的示例仅列出目录的直接内容，而没有进入嵌套的子目录。
您可以将返回fs.FS的传递fs.WalkDir给遍历整个结构，包括所有嵌套文件夹：
*/
func walkDirectory(fsys fs.FS) {
	// Walk whole directory contents (including nested directories)
	fs.WalkDir(fsys, "/", func(path string, d fs.DirEntry, err error) error {
		info, _ := d.Info()
		fmt.Printf("%s (%d bytes)\n", path, info.Size())
		return err
	})
}

// 查询状态信息
func getStatusForCidString(client w3s.Client, cidString string) error {
	c, err := cid.Parse(cidString)
	if err != nil {
		return err
	}
	/*
		Go 客户端的go-cid库的Client接口定义了一个Status的方法，它接受 一个context.Context和一个Cid。
	*/
	s, err := client.Status(context.Background(), c)
	if err != nil {
		return err
	}

	fmt.Printf("Status: %+v", s)
	return nil
}

//列出上传到 web3.storage 的文件
//使用web3.storage存储了一些文件后，您将希望看到已上传内容的列表。登入到web3.storage的网站管理吧，今天的内容太多了

```

## 参考
>  https://dist.ipfs.tech/#ipfs-update
https://github.com/ipfs/kubo
https://docs.ipfs.tech/
https://docs.ipfs.tech/basics/go/command-line/#retrieve-a-file
https://web3.storage/docs/how-tos/store/?lang=go
https://blog.csdn.net/youngZ_H/article/details/127469818
http://t.zoukankan.com/HandyLi-p-9156279.html
https://pkg.go.dev/github.com/web3-storage/go-w3s-client#section-readme
