---
title: "Golang请求pixabay API下载视频"
description: "Golang请求pixabay API下载视频"
date: 2021-12-09 15:10:00
slug: golang-download-pixabayapi-video
image:
categories:
    - Go
tags: ["Go"]

---

因为需要用到一些视频素材，加上最近在学`Golang`，所以就想着能不能写个 Golang 程序 实现简单的下载视频的需求。

免版权网站：`https://pixabay.com/`

API 文档：`https://pixabay.com/api/docs/`

思路：传递关键字 -> 请求视频搜索接口 -> 获取相对应的视频Url，保存到 map 中 -> 批量下载

## 代码

工具：json:https://oktools.net/json2go 用来将`json `转 `struct `类型。(ps：golang中处理json竟然如此麻烦-_-)

需要注册个`pixabay`账号，因为请求接口使用了`key`。

```go
package main
// json:https://oktools.net/json2go
import (
	"os"
	"io"
	"fmt"
	"io/ioutil"
	"net/http"
	"net/url"
	"encoding/json"
	"strconv"
	// "github.com/bitly/go-simplejson"
)
// 解析JSON类型的返回结果
type result struct {
    Args string `json:"args"`
    Headers map[string]string `json:"headers"`
    Origin string `json:"origin"`
    Url string `json:"url"`
}
func getType1(v interface{}) string {
	return fmt.Sprintf("%T", v)
}

func print_json(m map[string]interface{}) {
    for k, v := range m {
        switch vv := v.(type) {
        case string:
            fmt.Println(k, "is string", vv)
        case float64:
            fmt.Println(k, "is float", int64(vv))
        case int:
            fmt.Println(k, "is int", vv)
        case []interface{}:
            fmt.Println(k, "is an array:")
            for i, u := range vv {
                fmt.Println(i, u)
            }
        case nil:
            fmt.Println(k, "is nil", "null")
        case map[string]interface{}:
            fmt.Println(k, "is an map:")
            print_json(vv)
        default:
            fmt.Println(k, "is of a type I don't know how to handle ", fmt.Sprintf("%T", v))
        }
    }
}

type PixabayVideos struct {
	Total int `json:"total"`
	TotalHits int `json:"totalHits"`
	Hits []Hits `json:"hits"`
}
type Large struct {
	URL string `json:"url"`
	Width int `json:"width"`
	Height int `json:"height"`
	Size int `json:"size"`
}
type Medium struct {
	URL string `json:"url"`
	Width int `json:"width"`
	Height int `json:"height"`
	Size int `json:"size"`
}
type Small struct {
	URL string `json:"url"`
	Width int `json:"width"`
	Height int `json:"height"`
	Size int `json:"size"`
}
type Tiny struct {
	URL string `json:"url"`
	Width int `json:"width"`
	Height int `json:"height"`
	Size int `json:"size"`
}
type Videos struct {
	Large Large `json:"large"`
	Medium Medium `json:"medium"`
	Small Small `json:"small"`
	Tiny Tiny `json:"tiny"`
}
type Hits struct {
	ID int `json:"id"`
	PageURL string `json:"pageURL"`
	Type string `json:"type"`
	Tags string `json:"tags"`
	Duration int `json:"duration"`
	PictureID string `json:"picture_id"`
	Videos Videos `json:"videos"`
	Views int `json:"views"`
	Downloads int `json:"downloads"`
	Likes int `json:"likes"`
	Comments int `json:"comments"`
	UserID int `json:"user_id"`
	User string `json:"user"`
	UserImageURL string `json:"userImageURL"`
}

// 通过链接下载文件
func DownFileByUrl(link string, savePath string) error {
	fmt.Println(link)
	resp, err := http.Get(link)
	defer resp.Body.Close()
	out, err := os.Create(savePath)
	if err != nil {
		fmt.Println(err)
		return err
	}
	defer out.Close()
	_, err = io.Copy(out, resp.Body)
	if err != nil {
		fmt.Println(err)
		return err
	}
	fmt.Println(savePath + " -> success...")
	return nil
}

// https://pixabay.com/api/?key=[my_key]f&q=nature&image_type=photo&orientation=horizontal&min_width=1920&min_height=1080&page=1&per_page=100
func SearchVideo(keywords string) {
	targetUrl := "https://pixabay.com/api/videos/"
	params := url.Values{}
	weburl, err := url.Parse(targetUrl)
	if err != nil {
		fmt.Println("请求失败！")
		return
	}
	params.Set("key", "xxxxxxxxxxxxxxxxxxxx")
	params.Set("q", keywords)
	params.Set("per_page", "30")

	// 如果参数中有中文参数,这个方法会进行URLEncode
	weburl.RawQuery = params.Encode()
	urlPath := weburl.String()
	fmt.Println(urlPath)

	resp, err := http.Get(urlPath)
	defer resp.Body.Close()
	body, _ := ioutil.ReadAll(resp.Body)

	var videos PixabayVideos
	// 将请求体中的JSON数据解析到结构体中
	err = json.Unmarshal(body, &videos)
	if err != nil {
		fmt.Println(err)
		return
	}
	videoList := videos.Hits

	var video Hits
	for _,video = range videoList {
		strID := strconv.Itoa(video.ID)
		largeVideoUrls[strID] = video.Videos.Large.URL

		if largeVideoUrls[strID] == "" {
			largeVideoUrls[strID] = video.Videos.Medium.URL
		}
		// largeVideoUrls = append(largeVideoUrls, video.Videos.Large.URL)
		// fmt.Println(video.ID)
		// fmt.Println(video.Videos.Large.URL)
	}
}

// var largeVideoUrls []string
var largeVideoUrls map[string]string

func main() {
	largeVideoUrls = make(map[string]string)
	fmt.Println(largeVideoUrls)
	var keywords string
	keywords = "wealth"
	savePath := "pixabay-video/"

	//  搜索
	SearchVideo(keywords)

	for k, link := range largeVideoUrls {
		// fmt.Println(k,link)
		filePath := savePath + k + ".mp4"
		fmt.Println(filePath)
		err := DownFileByUrl(link, filePath)
		if err != nil {
			fmt.Println(err)
		}
	}
}
```

