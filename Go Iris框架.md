

# Go Iris框架

## 一、安装

需要go版本1.8以上。

Windows安装Iris：

```shell
$go get -v github.com/kataras/iris
```

但是往往因为网络原因下载失败。

解决：

打开CMD命令提示框，设置临时环境变量，临时环境变量每次都是需要重新设置。可以在本机环境变量那里设置这两个参数变量。

```powershell
$set GO111MODULE=on ## 将使用GOPATH变为使用GO MODULE
$set GOPROXY=https://goproxy.io  ##设置代理
```

之后就可以下载iris了，他会下载到GOPATH/pkg/mod目录下。

## 二、使用

因为用了GO MODULE，所以可以在除GOPATH外的文件夹中创建项目并使用。

例如：在GOPROJECT目录下创建MyProject文件夹，添加文件MyTest.go。

```go
package main

import (
	"github.com/kataras/iris"
)

func main() {
	app := iris.New()
	app.Run(iris.Addr(":8080"), iris.WithoutServerError(iris.ErrServerClosed))
	
}
```

在文件路径下，打开命令行窗口cmd。

```shell
$go mod init MyTest
```

初始化go mod包，之后会生成go.mod文件。管理项目依赖包。

```mod
module MyTest

go 1.12

require (
	github.com/BurntSushi/toml v0.3.1 // indirect
	github.com/Joker/jade v1.0.0 // indirect
	github.com/Shopify/goreferrer v0.0.0-20181106222321-ec9c9a553398 // indirect
	github.com/aymerick/raymond v2.0.2+incompatible // indirect
	github.com/eknkc/amber v0.0.0-20171010120322-cdade1c07385 // indirect
	github.com/fatih/structs v1.1.0 // indirect
	github.com/flosch/pongo2 v0.0.0-20190707114632-bbf5a6c351f4 // indirect
	github.com/gorilla/schema v1.1.0 // indirect
	github.com/iris-contrib/blackfriday v2.0.0+incompatible // indirect
	github.com/iris-contrib/formBinder v5.0.0+incompatible // indirect
	github.com/iris-contrib/go.uuid v2.0.0+incompatible // indirect
	github.com/json-iterator/go v1.1.9 // indirect
	github.com/kataras/golog v0.0.10 // indirect
	github.com/kataras/iris v11.1.1+incompatible // indirect
	github.com/klauspost/compress v1.10.1 // indirect
	github.com/microcosm-cc/bluemonday v1.0.2 // indirect
	github.com/ryanuber/columnize v2.1.0+incompatible // indirect
	github.com/shurcooL/sanitized_anchor_name v1.0.0 // indirect
	golang.org/x/crypto v0.0.0-20200221231518-2aa609cf4a9d // indirect
	gopkg.in/yaml.v2 v2.2.8 // indirect
)

```

之后就可以在项目文件目录下运行.go文件文件。

```shell
$go run MyTest.go
```



