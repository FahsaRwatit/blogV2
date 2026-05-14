---
title: "使用Golang实现视频流"
description: "使用Golang实现视频流"
date: 2021-11-26 15:10:00
slug: golang-stream-base-demo
image:
categories:
    - Go
tags: ["Go"]

---

准备：go环境， FFMPEG，一个视频

使用命令：

```shell
ffmpeg -i code.mp4 -profile:v baseline -level 3.0 -s 640x360 -start_number 0 -hls_time 10 -hls_list_size 0 -f hls index.m3u8
```

使用`ffmpeg`命令将视频分为一个`.m3u8`和多个`.ts`文件

并将文件放在目录`assets\media1\hls`下

文件 main.go 代码：

<!--more-->

```go

package main
import (
	"fmt"
	"net/http"
	"strconv"

	"github.com/gorilla/mux"

)

func main() {
	http.Handle("/", handlers())
	http.ListenAndServe(":8000",nil)
}

func handlers() *mux.Router {
	fmt.Println("handlers...")

	router := mux.NewRouter()
	router.HandleFunc("/", indexPage).Methods("GET")
	router.HandleFunc("/media/{mId:[0-9]+}/stream/", streamHandler).Methods("GET")
	router.HandleFunc("/media/{mId:[0-9]+}/stream/{segName:index[0-9]+.ts}",streamHandler).Methods("GET")

	return router
}

func indexPage(w http.ResponseWriter, r *http.Request) {
	http.ServeFile(w, r, "index.html")
}

func streamHandler(response http.ResponseWriter, request *http.Request) {
	fmt.Println("streamHandler...")

	vars := mux.Vars(request)
	mId, err := strconv.Atoi(vars["mId"])
	fmt.Println(mId)
	fmt.Println(err)

	if err != nil {
		response.WriteHeader(http.StatusNotFound)
		return
	}
	segName, ok := vars["segName"]

	if !ok {
		mediaBase := getMediaBase(mId)
		m3u8Name := "index.m3u8"
		serveHlsM3u8(response, request, mediaBase, m3u8Name)
	} else {
		mediaBase := getMediaBase(mId)
		serveHlsTs(response, request, mediaBase, segName)
	}

}
func getMediaBase(mId int) string {

	mediaRoot := "assets/media"
	return fmt.Sprintf("%s%d", mediaRoot,mId)
}

func serveHlsM3u8(w http.ResponseWriter, r *http.Request, mediaBase, m3u8Name string) {
	fmt.Println("serveHlsM3u8...")

	mediaFile := fmt.Sprintf("%s/hls/%s", mediaBase, m3u8Name)
	fmt.Println(mediaFile)
	http.ServeFile(w, r, mediaFile)
	w.Header().Set("Content-Type", "application/x-mpegURL")

}

func serveHlsTs(w http.ResponseWriter, r *http.Request, mediaBase, segName string) {
	fmt.Println("serveHlsTs...")

	mediaFile := fmt.Sprintf("%s/hls/%s", mediaBase, segName)
	fmt.Println(mediaFile)

	http.ServeFile(w, r, mediaFile)
	w.Header().Set("Content-Type", "video/MP2T")
}
```

文件`index.html`

```html
<script src="https://cdn.jsdelivr.net/npm/hls.js@latest"></script>
    <!-- Or if you want a more recent canary version -->
    <!-- <script src="https://cdn.jsdelivr.net/npm/hls.js@canary"></script> -->
    <video id="video" muted></video>
    <script>
      var video = document.getElementById('video');
      if(Hls.isSupported()) {
        var hls = new Hls();
        hls.loadSource('http://localhost:8000/media/1/stream/');
        hls.attachMedia(video);
        hls.on(Hls.Events.MANIFEST_PARSED,function() {
          video.play();
      });
     } else if (video.canPlayType('application/vnd.apple.mpegurl')) {
        video.src = 'http://localhost:8000/media/1/stream/';
        video.addEventListener('loadedmetadata',function() {
          video.play();
        });
      }
    </script>
```

执行：

```shell
go env -w GO111MODULE=on

go env -w  GOPROXY=https://goproxy.cn

go get github.com/gorilla/mux

go mod init main.go

go run main.go

```

浏览器打开：http://localhost:8000/

http://localhost:8000/media/1/stream/

结果如下：

![t](https://qnres.fahsa.cn/images/t.gif)



原文：https://www.rohitmundra.com/video-streaming-server





