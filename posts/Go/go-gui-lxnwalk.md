---
title: "Golang Windows GUI之lxn/walk"
description: "Golang Windows GUI之lxn/walk"
date: 2021-12-09 15:10:00
slug: "go-gui-lxnwalk"
image:
categories:
    - Go
tags: ["Go", "lxn/walk"]


---

lxn/walk文档：https://pkg.go.dev/github.com/lxn/walk#section-readme



#### main.go文件

```go
package main

import (
	"strings"

	"github.com/lxn/walk"
	. "github.com/lxn/walk/declarative"
)

// https://pkg.go.dev/github.com/lxn/walk#section-readme
func main() {
	var inTE, outTE *walk.TextEdit // 声明两个文本编辑控件

	//主窗口对象
	MainWindow{
		Title:   "windows程序",    // 窗口标题设置
		MinSize: Size{600, 400}, //窗体的大小
		Layout:  VBox{},         // 窗体的布局形式
		//定义vbox的所有控件
		Children: []Widget{ //定义控件
			HSplitter{ //水平分割控件
				Children: []Widget{ //定义子控件
					TextEdit{AssignTo: &inTE},
					TextEdit{AssignTo: &outTE, ReadOnly: true},
				},
			},
			PushButton{ //按钮控件
				Text: "确定",
				OnClicked: func() {
					outTE.SetText(strings.ToUpper(inTE.Text()))
				},
			},
		},
	}.Run()
}

```



#### 文件：main.manifest

- 和main.go保存在同级目录下
- `main.manifest`是应用程序配置元数据的清单文件
- 直接复制以下内容即可，不用修改

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<assembly xmlns="urn:schemas-microsoft-com:asm.v1" manifestVersion="1.0">
    <assemblyIdentity version="1.0.0.0" processorArchitecture="*" name="SomeFunkyNameHere" type="win32"/>
    <dependency>
        <dependentAssembly>
            <assemblyIdentity type="win32" name="Microsoft.Windows.Common-Controls" version="6.0.0.0" processorArchitecture="*" publicKeyToken="6595b64144ccf1df" language="*"/>
        </dependentAssembly>
    </dependency>
    <application xmlns="urn:schemas-microsoft-com:asm.v3">
        <windowsSettings>
            <dpiAwareness xmlns="http://schemas.microsoft.com/SMI/2016/WindowsSettings">PerMonitorV2, PerMonitor</dpiAwareness>
            <dpiAware xmlns="http://schemas.microsoft.com/SMI/2005/WindowsSettings">True</dpiAware>
        </windowsSettings>
    </application>
</assembly>
```

#### 下载相应工具

```sh
# 下载walk
go get github.com/lxn/walk

# 下载rsrc工具
go get github.com/akavel/rsrc

```

生成.syso文件

rsrc -manifest main.manifest -o main.syso

生成.exe文件

go build -ldflags="-H windowsgui"













