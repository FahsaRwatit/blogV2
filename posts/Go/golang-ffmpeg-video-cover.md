---
title: "Golang使用ffmpeg命令制作视频封面"
description: "Golang使用ffmpeg命令制作视频封面"
date: 2021-12-09 15:10:00
slug: golang-ffmpeg-video-cover
image:
categories:
    - Go
tags: ["Go"]

---

整体思路：抽取视频中的一张图片作为封面，然后在这张图片上加上文字即可。

使用到的`ffmpeg`命令:

```sh
# 抽取一张图片
ffmpeg -ss 0.5 -i input.mp4 -vframes 1 -s 1080x720 -f image2 input_pic.jpg

# 图片中加入居中文本
ffmpeg -i input_pic.jpg -vf drawtext=font='Impact':text='Text':fontcolor=yellow@0.8:box=1:boxcolor=red@0.2:borderw=3:fontsize-75:x=(w-tw)/2:y=((h-text_h)/2) cover.jpg
```

<!--more-->

### 具体代码

```go
var (
	wg	sync.WaitGroup
)
// 命令行调用
func Cmd(commandName string, params []string) (string, error) {
	fmt.Println("命令行调用")
	cmd := exec.Command(commandName, params...)
	//fmt.Println("Cmd", cmd.Args)
	var out bytes.Buffer
	cmd.Stdout = &out
	cmd.Stderr = os.Stderr
	err :=  cmd.Start()
	if err != nil {
		return "", err
	}
	err = cmd.Wait()
	return out.String(), err
}
// 封面
func getVideoCover(video string, cover string, fonttxt string) {
	defer wg.Done()
	// 抽取一张图片，然后图片中间添加文字
	// ffmpeg -ss 0.5 -i r1.mp4 -vframes 1 -s 1080x720 -f image2 r2.jpg
	fmt.Println("Make a cover: " + video)

	vname := strings.Split(video, ".")
	videoPic := strings.Join(vname[:len(vname) - 1],"") + ".jpg"

	fmt.Println(videoPic)

	cmdStr1 := fmt.Sprintf("ffmpeg -ss 0.5 -i %s -vframes 1 -s 1080x720 -f image2 %s", video, videoPic)
	fmt.Println("制作封面-getVideoCover命令1: " + cmdStr1)
	fmt.Println(cmdStr1)
	args := strings.Fields(cmdStr1)
	msg, err := Cmd(args[0], args[1:])
	if err != nil {
		fmt.Printf("getVideoCover1 videofailed, %v, output: %v\n", err, msg)
        return
	}
	// 添加文字
	// ffmpeg -i r2.jpg -vf "drawtext=font='Impact':text='TEXT':fontcolor=yellow@0.8:box=1:boxcolor=red@0.2:borderw=3:fontsize-75:x=(w-tw)/2:y=((h-text_h)/2)" output2.jpg
	
	cmdStr2 := fmt.Sprintf("ffmpeg -i %s -vf drawtext=font='Impact':text='%s':fontcolor=yellow@0.8:box=1:boxcolor=red@0.2:borderw=3:fontsize-78:x=(w-tw)/2:y=((h-text_h)/2) %s", videoPic, fonttxt, cover)
	
	
	fmt.Println("制作封面-getVideoCover命令2: " + cmdStr2)
	fmt.Println(cmdStr2)
	args = strings.Fields(cmdStr2)
	msg, err = Cmd(args[0], args[1:])
	if err != nil {
		fmt.Printf("getVideoCover1 videofailed, %v, output: %v\n", err, msg)
        return
	}
}

// 执行
wg.Add(1)
go getVideoCover("one/xue.mp4", "one/cover_xue.jpg", "Music")
wg.Wait()

```

