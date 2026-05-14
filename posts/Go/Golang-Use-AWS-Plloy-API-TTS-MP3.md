---
title: "使用AWS Plloy API 将文本合成语言并保存为MP3"
description: "使用AWS Plloy API 将文本合成语言并保存为MP3"
date: 2021-11-30 18:10:00
slug: Golang-Use-AWS-Plloy-API-TTS-MP3
image:
categories:
    - Go
tags: ["Go"]

---

官方文档：https://docs.aws.amazon.com/sdk-for-go/v1/developer-guide/using-polly-with-go-sdk.html

## 申请一个 AWS 账号

首先进入 https://aws.amazon.com/cn/polly/ 申请一个 AWS 账号

申请一个 AWS 账号中要求填写真实的地址（主要对应上就可以），因为AWS那边会对你申请的账户进行验证，验证不通过账户会被停用。

## 创建IAM用户 获取  密钥信息

<!--more-->

进入IAM控制台创建用户和权限

![image-20211130183755967](https://qnres.fahsa.cn/images/image-20211130183755967.png)

点击 用户，添加用户

![image-20211130183944977](https://qnres.fahsa.cn/images/image-20211130183944977.png)

输入用户名称test，勾选 **访问密钥 - 编程访问**

![image-20211130184043572](https://qnres.fahsa.cn/images/image-20211130184043572.png)

下一步权限，这时需要创建一个组，点击 `创建组`

![image-20211130184348702](https://qnres.fahsa.cn/images/image-20211130184348702.png)

创建组：输入组名，选择相应的权限，我们这里输入`polly`进行搜索，勾选 `AmazonPollyFullAccess`

![image-20211130184800593](https://qnres.fahsa.cn/images/image-20211130184800593.png)

![image-20211130184845389](https://qnres.fahsa.cn/images/image-20211130184845389.png)

添加标签：

![image-20211130184941118](https://qnres.fahsa.cn/images/image-20211130184941118.png)

创建用户：

![image-20211130185017406](https://qnres.fahsa.cn/images/image-20211130185017406.png)

###获取 访问密钥 ID 和 访问密钥 ID，记下来之后要用。

![image-20211130185049422](https://qnres.fahsa.cn/images/image-20211130185049422.png)

## 在 home 目录下创建一个文件`.aws/credentials`

内容如下(填入密钥信息)：

```go
[default]
aws_access_key_id = XXXXX
aws_secret_access_key = XXXXXXXXXXXXXXXXXX
```

## Golang代码

### 创建目录 golang-polly

### golang-polly下新建`service`文件夹

`service/polly-service.go` 内容如下(常量const部分可以自定义，这里我只添加了部分)：

```go
package service

import (
	"fmt"
	"github.com/aws/aws-sdk-go/aws"
	"github.com/aws/aws-sdk-go/aws/session"
	"github.com/aws/aws-sdk-go/service/polly"

	"os"
	"io"
)
// 文档 https://pkg.go.dev/github.com/aws/aws-sdk-go-v2/service/polly#SynthesizeSpeechInput
// 引擎
const (
	// EngineStandard 是一个引擎枚举值
   EngineStandard = "standard"

   // EngineNeural 是一个引擎枚举值
   EngineNeural = "neural"
)
const (
	RATE_8000  rate = 8000
	RATE_16000 rate = 16000
	RATE_22050 rate = 22050
)
type rate int

const (
	CMN_CN = "cmn-CN" // 中文普通话
	EN_US = "en-US"   // 英语美国
	JA_JP = "ja-JP"   // 日本
	KO_KR = "ko-KR"   // 韩语
	IT_IT = "it-IT"   // 意大利语
	DE_DE = "de-DE"   // 德语
	FR_FR = "fr-FR"   // 法语
	RU_RU = "ru-RU"   // 俄语
)
// 声音
const (
	// 中文普通话
	Zhiyu   = "Zhiyu" // 女

	// 英语美国
	Salli    = "Salli" // 女
	Joanna   = "Joanna"
	Ivy   = "Ivy"
	Kendra   = "Kendra"
	Kimberly   = "Kimberly"

	Kevin      = "Kevin" // 男
	Matthew   = " Matthew"
	Justin   = "Justin"
	Joey   = "Joey"
	

	// 日本
	Mizuki   = "Mizuki" // 女
	Takumi   = "Takumi" // 男
 
	// 韩语 
	Seoyeon   = "Seoyeon" // 女
	
	// 意大利语 
	Bianca   = "Bianca" // 女
	Carla   = "Carla" // 女
	Giorgio   = "Giorgio" // 男

	// 德语
	Marlene   = "Marlene" // 女
	Vicki   = "Vicki" // 女
	Hans   = "Hans" // 男

	// 法语
	Celine   = "Céline" // 女
	Lea   = "Lea" // 女
	Mathieu   = "Mathieu" // 男

	// 俄语
	Tatyana   = "Tatyana" // 女
	Maxim   = "Maxim" // 男

)
const (
	AUDIO_FORMAT = "mp3"
)
const (
	TEXT = "text"
	SSML = "ssml"
)

type PollyService interface {
	Synthesize(text string, mp3FileName string) error
}
type pollyConfig struct {
	voice string
	engine string
	texttype string
}

func NewPollyService(voice string, engine string, texttype string) PollyService {
	return &pollyConfig{
		voice: voice,
		engine: engine,
		texttype: texttype,
	}
}

// func NewZhiyuPollyService() PollyService {
// 	return &pollyConfig{
// 		voice: Zhiyu,
// 		engine: EngineStandard,
// 	}
// }

func createPollyClient() *polly.Polly {
	// Create AWS Session
	// sess := session.Must(session.NewSessionWithOptions(session.Options{
	// 	SharedConfigState: session.SharedConfigEnable,
	// }))
	//  ap-northeast-1 us-west-2
	sess, _ := session.NewSession(&aws.Config{
		Region: aws.String("ap-northeast-1")},
	)

	// Create Polly client
	return polly.New(sess)	
}

func (config *pollyConfig) Synthesize(text string, fileName string) error {
	pollyClient := createPollyClient()

	// Output to MP3
	/*
		OutputFormat types.OutputFormat
		Text *string
		VoiceId types.VoiceId
		Engine types.Engine
		LanguageCode types.LanguageCode
		LexiconNames []string
		SampleRate *string // are "8000" and "16000" The default value is "16000"
		SpeechMarkTypes []types.SpeechMarkType

		// Specifies whether the input text is plain text or SSML. The default value is
		// plain text. For more information, see Using SSML
		// (https://docs.aws.amazon.com/polly/latest/dg/ssml.html).
		TextType types.TextType

	*/
	// EngineNeural "neural"
	input := &polly.SynthesizeSpeechInput{ 
						Engine: aws.String(config.engine),
						OutputFormat: aws.String(AUDIO_FORMAT), 
						Text: aws.String(text), 
						VoiceId: aws.String(config.voice),
						TextType:aws.String(config.texttype),
					}
	// input = 
					
	output, err := pollyClient.SynthesizeSpeech(input)

	if err != nil {
		return err
	}

	outFile, err := os.Create(fileName)
	if err != nil {
		return err
	}
	
	defer outFile.Close()

	_, err = io.Copy(outFile, output.AudioStream)
	if err != nil {
		return err
	}
	
	return nil
}

func Test() {
	// sess := session.Must(session.NewSessionWithOptions(session.Options{
	// 	SharedConfigState: session.SharedConfigEnable,
	// }))

	sess, _ := session.NewSession(&aws.Config{
		Region: aws.String("ap-northeast-1")},
	)
	
	svc := polly.New(sess)
	fmt.Println(svc)

	// en-US cmn-CN

	input := &polly.DescribeVoicesInput{LanguageCode: aws.String("ru-RU")}
	
	resp, err := svc.DescribeVoices(input)

	fmt.Println(err)

	for _, v := range resp.Voices {
		fmt.Println("Name:   " + *v.Name)
		fmt.Println("Gender: " + *v.Gender)
		fmt.Println("")
	}
}
```

#### 文件`golang-polly/main.go`

新建`main.go`内容 如下：

有些注释的内容忽略即可

```go
package main

// import "gitlab.com/pragmaticreviews/golang-amazon-polly/service"
import (
	"fmt"
	"os"
	// "io"
    "io/ioutil"
	"golang-polly/service"
)
var(
	// zhiyu service.PollyService = service.NewZhiyuPollyService()
	// kimberly service.PollyService = service.NewKimberlyPollyService()
	// joey service.PollyService = service.NewJoeyPollyService()

	// 支持新闻联播
	// joanna service.PollyService = service.NewJoannaPollyService()
	// matthew service.PollyService = service.NewMatthewPollyService()
	
)
func getFileContext(fileName string) (string, error) {
	contents, err := ioutil.ReadFile(fileName)
	if err != nil {
		fmt.Println("Got error opening file " + fileName)
		fmt.Println(err.Error())
		os.Exit(1)
		return "", err
	}
	// Convert bytes to string
    s := string(contents[:])
	return s, err
}
/*
新闻播音员语音中仅提供美国英语 (en-US) 的 Matthew 和 Joanna、美国西班牙语 (es-US) 的 Lupe 和英国英语 (en-GB) 的 Amy
*/

func main() {
	// service.Test()

	// 从文件读取内容
	// fileName := os.Args[1]
	// contents, err := getFileContext(fileName)

	// fmt.Println(contents)
	// fmt.Println(err)

	zhiyu := service.NewPollyService(service.Zhiyu, service.EngineStandard,service.TEXT)

	// err := zhiyu.Synthesize("你好啊，我是知更鸟哦！", "zhiyu.mp3")
	err := zhiyu.Synthesize("臭大猫，下班没？", "mao.mp3")

	if err != nil {
		panic(err)
	}

	// joanna := service.NewPollyService(service.Joanna, service.EngineNeural, service.SSML)
	// err = joanna.Synthesize(contents, "joanna2.mp3")

	// if err != nil {
	// 	panic(err)
	// }

}

```

### 运行程序

```go
// 需要安装一些包，根据提示进行就可以
go mod  init golang-polly
go run main.go
```

生成的`go.mod`内容如下：

```go
module golang-polly

go 1.17

require (
	github.com/aws/aws-sdk-go v1.42.14 // indirect
	github.com/jmespath/go-jmespath v0.4.0 // indirect
)

```

至此，你应该学会了**亚马逊波莉**的简单使用了，最后想提个问题，你可以将此次coding应用到哪些地方呢？
