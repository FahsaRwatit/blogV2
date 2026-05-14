---
title: "Gin中使用JWT的示例代码"
description: "Gin中使用JWT的示例代码"
date: 2021-10-08 18:10:00
slug: gin-use-jwt
image:
categories:
    - Go
tags: ["Go"]

---



### JWT代码封装：

```go
package jwt

import (
	"errors"
	"time"

	"github.com/dgrijalva/jwt-go"
)

// MyClaims 自定义声明结构体并内嵌jwt.StandardClaims
// jwt包自带的jwt.StandardClaims只包含了官方字段
// 我们这里需要额外记录一个UserID字段，所以要自定义结构体
// 如果想要保存更多信息，都可以添加到这个结构体中
type MyClaims struct {
	UserID uint64 `json:"user_id"`
	// 只包含了官方字段
	jwt.StandardClaims
}

var mySecret = []byte("特立独行的猪")

func keyFunc(_ *jwt.Token) (i interface{}, err error) {
	return mySecret, nil
}

const TokenExpireDuration = time.Hour * 24 * 365

// GenToken 生成 AccessToken 和 refresh token
func GenToken(userID uint64) (aToken, rToken string, err error) {
	// 创建一个我们自己的声明
	c := MyClaims{
		userID,
		jwt.StandardClaims{
			// 设置过期时间
			ExpiresAt: time.Now().Add(TokenExpireDuration).Unix(),
			// 设置签发人
			Issuer: "web-app",
		},
	}
	// 加密并获得完整的编码后的字符串token
	// 使用指定的secret签名并获得完整的编码后的字符串token
	aToken, err = jwt.NewWithClaims(jwt.SigningMethodHS256, c).SignedString(mySecret)

	// refresh token 不需要存在 任何 自定义数据
	// 使用指定的secret签名并获得完整的编码后的字符串token
	rToken, err = jwt.NewWithClaims(jwt.SigningMethodHS256, jwt.StandardClaims{
		ExpiresAt: time.Now().Add(time.Second * 30).Unix(), // 过期时间
		Issuer:    "web-app",                               // 签发人
	}).SignedString(mySecret)

	return
}

// 刷新 AccessToken
func RefreshToken(aToken, rToken string) (newAToken, newRToken string, err error) {
	// refresh token 无效直接返回
	if _, err = jwt.Parse(rToken, keyFunc); err != nil {
		return
	}
	// 从旧access token 中解析claims数据
	var claims MyClaims
	_, err = jwt.ParseWithClaims(aToken, &claims, keyFunc)
	v, _ := err.(*jwt.ValidationError)
	// 当access token 是过期错误 并且 refresh token 没有过期时就创建一个新的AccessToken
	if v.Errors == jwt.ValidationErrorExpired {
		return GenToken(claims.UserID)
	}
	return
}

// 解析JWT
func ParseToken(tokenString string) (claims *MyClaims, err error) {
	// 解析token
	var token *jwt.Token
	claims = new(MyClaims)
	token, err = jwt.ParseWithClaims(tokenString, claims, keyFunc)
	if err != nil {
		return
	}
	// 校验 token
	if !token.Valid {
		err = errors.New("invalid token")
	}
	return
}

```



### 生成Token

登录时根据一些信息生成AccessToken 和 RefreshToken

```go
// 生成Token
	aToken, rToken, _ := jwt.GenToken(um.UserID)
	ResponseSuccess(c, gin.H{
		"accessToken":  aToken,
		"refreshToken": rToken,
		"userID":       um.UserID,
		"username":     um.UserName,
	})
```

### 基于JWT的认证中间件

```go
package controller

import (
	"errors"
	"fmt"
	"strings"

	"web-app/pkg/jwt"

	"github.com/gin-gonic/gin"
)

const (
	ContextUserIDKey = "userID"
)

var (
	ErrorUserNotLogin = errors.New("当前用户未登录")
)

// 基于JWT的认证中间件
func JWTAuthMiddleware() func(c *gin.Context) {
	return func(c *gin.Context) {
		// 客户端携带Token有三种方式 1.放在请求头 2.放在请求体 3.放在URI
		// 这里假设Token放在Header的Authorization中，并使用Bearer开头
		// 这里的具体实现方式要依据你的实际业务情况决定
		authHeader := c.Request.Header.Get("Authorization")
		if authHeader == "" {
			ResponseErrorWithMsg(c, CodeInvalidToken, "请求头缺少Auth Token")
			// Abort() 阻止调用后续的处理函数
			c.Abort()
			return
		}
		// 按空格分割
		parts := strings.SplitN(authHeader, " ", 2)
		if !(len(parts) == 2 && parts[0] == "Bearer") {
			ResponseErrorWithMsg(c, CodeInvalidToken, "Token 格式不正确")
			c.Abort()
			return
		}
		// parts[1]是获取到的tokenString，我们使用之前定义好的解析JWT的函数来解析它
		mc, err := jwt.ParseToken(parts[1])
		if err != nil {
			fmt.Println(err)
			ResponseError(c, CodeInvalidToken)
			c.Abort()
			return
		}
		// 将当前请求的username信息保存到请求的上下文c上
		c.Set(ContextUserIDKey, mc.UserID)
		c.Next() // 后续的处理函数可以用过c.Get("userID")来获取当前请求的用户信息
	}

}

```



### 路由使用中间件

访问需要带token的路由中使用JWT验证中间件

```go
// JWT 认证中间件
	v1.Use(controller.JWTAuthMiddleware())
	{
		v1.GET("/test", controller.CommunityHandler)

	}
```

### 更新Token

token过期时 进行更新

```go

func (u *UserController) RefreshTokenHandler(c *gin.Context) {
	rt := c.Query("refresh_token")
	// 客户端携带Token有三种方式 1.放在请求头 2.放在请求体 3.放在URI
	// 这里假设Token放在Header的Authorization中，并使用Bearer开头
	// 这里的具体实现方式要依据你的实际业务情况决定
	authHeader := c.Request.Header.Get("Authorization")
	if authHeader == "" {
		ResponseErrorWithMsg(c, CodeInvalidToken, "请求头缺少 Auth Token")
		c.Abort()
		return
	}
	// 按空格分割
	parts := strings.SplitN(authHeader, " ", 2)
	if !(len(parts) == 2 && parts[0] == "Bearer") {
		ResponseErrorWithMsg(c, CodeInvalidToken, "Token格式不正确")
		c.Abort()
		return
	}
	aToken, rToken, err := jwt.RefreshToken(parts[1], rt)
	fmt.Println(err)
	c.JSON(http.StatusOK, gin.H{
		"access_token":  aToken,
		"refresh_token": rToken,
	})
}
```

以上使用JWT的简单流程