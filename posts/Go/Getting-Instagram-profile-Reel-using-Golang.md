---
title: "Golang获取Instagram用户主页的Reel信息"
description: "Golang获取Instagram用户主页的Reel信息"
date: 2021-10-08 18:10:00
slug: Getting-Instagram-profile-Reel-using-Golang
image:
categories:
    - Go
tags: ["Go"]

---

### 需要设置的参数

- LocalAgent：本地代理，查看代理：win + i 选择 "网络和Internet"，后选择代理
- UserCookie：用户的cookie
- target_user_id：用户ID，F12看一些请求参数获取

### 实例代码

获取视频链接并将链接存入`.txt`文件

```go
package main

import (
	"bufio"
	"bytes"
	"encoding/json"
	"fmt"
	"io/ioutil"
	"log"
	"net/http"
	"net/url"
	"os"
	"regexp"
	"time"
)

type InstargamReels struct {
	Items []struct {
		Media struct {
			VideoVersions []struct {
				Id     string `json:"id"`
				Width  int    `json:"width"`
				Height int    `json:"height"`
				Url    string `json:"url"`
			} `json:"video_versions"`
		} `json:"media"`
	} `json:"items"`
	PagingInfo struct {
		MaxId         string `json:"max_id"`
		MoreAvailable bool   `json:"more_available"`
	} `json:"paging_info"`
}

// POST请求接口Reels数据
func GetContent() (content string) {
	var pageUrl = "https://www.instagram.com/api/v1/clips/user/"
	// 电脑系统设置 查看代理地址和端口
	// 解析代理地址
	// proxy, err := url.Parse("http://127.0.0.1:4780") // 加载本地代理
	// proxy, err := url.Parse("http://127.0.0.1:4780") // 加载本地代理
	// proxy, err := url.Parse("http://127.0.0.1:33210") //加载本地代理
	proxy, err := url.Parse(LocalAgent) //加载本地代理

	netTransport := &http.Transport{
		Proxy:                 http.ProxyURL(proxy),
		MaxIdleConnsPerHost:   10,
		ResponseHeaderTimeout: time.Second * time.Duration(5),
	}
	httpClient := &http.Client{
		Timeout:   time.Second * 10,
		Transport: netTransport,
	}
	// 用url.values方式构造form-data参数
	formValues := url.Values{}
	formValues.Set("target_user_id", target_user_id)
	formValues.Set("page_size", page_size)
	formValues.Set("include_feed_video", include_feed_video)
	if len(max_id) > 0 {
		formValues.Set("max_id", max_id)
	}

	formDataStr := formValues.Encode()
	formDataBytes := []byte(formDataStr)
	formBytesReader := bytes.NewReader(formDataBytes)
	// request, err := http.NewRequest("GET", pageUrl, nil)
	request, err := http.NewRequest("POST", pageUrl, formBytesReader)

	if err != nil {
		log.Println("请求:", err)
		return ""
	}
	log.Println("设置头部")
	request.Header.Add("User-Agent", "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.198 Safari/537.36") //模拟浏览器User-Agent
	request.Header.Set("Cookie", UserCookie)

	request.Header.Add("Content-Type", "application/x-www-form-urlencoded")
	request.Header.Add("Origin", "https://www.instagram.com")
	request.Header.Add("Referer", "https://www.instagram.com/kedimental/reels/")

	request.Header.Add("x-csrftoken", "skhkr7NBqM7kkJVhuxBIeAfeakiX6yrw")
	request.Header.Add("x-ig-app-id", "936619743392459")
	// request.Header.Add("x-csrftoken", "skhkr7NBqM7kkJVhuxBIeAfeakiX6yrw")

	resp, err := httpClient.Do(request)
	if err != nil {
		log.Println("Http get err:", err)
		return ""
	}
	if resp.StatusCode != http.StatusOK {
		log.Println(err)
		return ""
	}
	defer resp.Body.Close()
	body, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		fmt.Println("read err:", err)
		return ""
	}
	return string(body)

}

// 正则匹配max_id
func GetMaxId(content string) string {
	regex := `paging_info":{"max_id":"(.*?)","more_available`
	rp := regexp.MustCompile(regex)
	max_id := rp.FindStringSubmatch(content)
	return max_id[1]
}

// json转Struct
func Json2Struct(strjson string) InstargamReels {
	var insReels InstargamReels
	json.Unmarshal([]byte(strjson), &insReels)
	return insReels
}

// 将url写入文件
func GetUrlToFile(insReels InstargamReels) (err error) {
	var url string
	for _, val := range insReels.Items {
		if len(val.Media.VideoVersions) > 0 {
			url = val.Media.VideoVersions[0].Url + "\n"
			err = WriteText(url)
			if err != nil {
				fmt.Println("写入失败:" + url)
				return nil
			}
		}
	}
	return nil
}

// 写入txt文件
func WriteText(txt string) error {
	f, err := os.OpenFile(savefile, os.O_RDWR|os.O_CREATE|os.O_APPEND, 0777)
	if err != nil {
		fmt.Println("os Create error: ", err)
		return err
	}
	defer f.Close()
	bw := bufio.NewWriter(f)
	bw.WriteString(txt)
	bw.Flush()
	return nil
}

// 获取用户的Reels数据
func GetUserReels() {
	var homeContent = GetContent()
	insReels := Json2Struct(homeContent)
	if len(insReels.Items) <= 0 {
		fmt.Println("无内容")
	}
	// 将视频链接写入txt文件中
	GetUrlToFile(insReels)
	// 下一页
	max_id = insReels.PagingInfo.MaxId
	fmt.Println(max_id)
	if len(max_id) > 0 {
		time.Sleep(3 * time.Second)
		GetUserReels()
	}
	fmt.Println("End...")
}

// 需要设置的参数
// 查看代理：win + i 选择 "网络和Internet"
var LocalAgent = "http://127.0.0.1:33210"
var UserCookie = ``

// 请求参数
var target_user_id = "53816783682"     // 用户ID
var page_size string = "9"             // 每页请求数
var include_feed_video string = "true" // 是否包含feed视频
var max_id string

var savefile = target_user_id + ".txt"

func main() {
	GetUserReels()
}

```
