---
title: Iris中使用JWT
date: 2019-10-30 09:19:28
categories:
- Go
tags:
- Go
---
### **JWT简介**
**JWT** 是 **Json Web Token** 的缩写，是一个签名的Json对象，可以用作身份验证。`Bearer` 在Oauth2中通常用于令牌。令牌由三部分组成，以 `.` 分割。前两部分是JSON对象，并进行`base64url`编码，最后一部分是签名，以相同格式编码。

**JWT**原理：在服务器认证后，生成一个加密的 JSON 对象，即 Token 返还给用户，之后用户与服务端通信都需要带上该 Token 令牌，服务器只依靠该令牌认定用户身份是否合法，因此服务器就不需要保存任何 session 数据了。
每当用户想要访问受保护的路由或资源时，用户代理应该使用承载模式发送JWT，通常在`Authorization Header `中。Header的内容应如下所示：
```javascript
Authorization: Bearer <token>
```
**JWT**的组成:`Header.Payload.Signature`
* Header（头部）：一个JSON对象，描述JWT的元数据，一般如下：
```go
{
    "alg":"HS256",
    "typ":"JWT"
}
```
`alg`属性表示签名的算法为 HS256(HMAC SHA256),`typ`属性表示该 Token 的类型，JWT令牌统一写为 `JWT`。最后将该对象通过 `Base64URL` 算法转为字符串
* Payload（负载）：也是JSON对象，用来存放实际需要传递的数据
```go
{
    "userId":"123"
    "exp":"3132134213"
    "iat":"1231312323"
}
```
可以在该对象中添加时间戳属性，`iat`属性用来表示当前时间戳，`exp`属性表示到期时间戳，有了这两个属性就可以验证 token 是否过期。该对象也要经过 `Base64Url`编码
* Signature（签名）：由于前两部分虽然经过编码，数据不可串改，但是是透明的，为了防止数据被恶意篡改，需要对前两部分签名。首先需要定义一个 `secret`密钥，这个密钥只有服务器知道，然后使用 Header 中指定的签名算法 生成签名，以下为 HMAC SHA256 算法创建签名:
```go
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```

### **在Iris中使用 JWT**
官方地址：<a href="https://github.com/dgrijalva/jwt-go">https://github.com/dgrijalva/jwt-go</a>

终端下载：
```
// 使用 go mod 的话直接在项目中 go get
go get "github.com/dgrijalva/jwt-go" 
```

自定义 JWT 中间件验证：
```go
import (
	"github.com/dgrijalva/jwt-go"
	jailer "github.com/iris-contrib/middleware/jwt"
)

/**
 * 验证 jwt
 * @method JwtHandler
 */
func JwtHandler() *jailer.Middleware {
	return jailer.New(jailer.Config{
		// 验证 jwt 的 token 的方法
		ValidationKeyGetter: func(token *jwt.Token) (interface{}, error) {
			// 自己加密的密钥
			return []byte("secret"), nil
		},

		SigningMethod: jwt.SigningMethodHS256,
		Expiration:true,
	})

}
```
这样一个简单的验证 token 的中间件就写好了。让我们看一下中间件的配置 jailer.Config
JWT 中 结构体 Config 源码解析
```go
type Config struct {
	//返回 JWT 验证签名 密钥的函数，默认值为nil
	ValidationKeyGetter jwt.Keyfunc
	// 请求中用户(&令牌)信息所在的属性的名称，如果是 JWT 将会被存储,默认值为 jwt
	ContextKey string
	// 令牌验证出错调用的函数
	ErrorHandler errorHandler
	// 是否需要凭证，默认值 false
	CredentialsOptional bool
	//从 request 中提取 token 的函数  默认 FromAuthHeader 从 from Authorization header 中获取 token
	Extractor TokenExtractor
	// 设置后，所有使用OPTIONS方法的请求都将使用身份验证
	//如果使用需在 路由中注册，默认值为 false
	EnableAuthOnOptions bool
	// 设置之后，middelware将验证是否使用特定的签名算法对令牌进行了签名，如果签名方法不是常量，可以使用ValidationKeyGetter回调来实现其他检查，默认值 nil
	SigningMethod jwt.SigningMethod
	// 验证 令牌是否过期，如果过期返回 过期错误，默认值 false
	Expiration bool
}
```
在登陆的时候生成 JWT Token
```go
token := jwt.New(jwt.SigningMethodHS256) // 创建 Header ,算法为 HS256
claims := make(jwt.MapClaims)
claims["exp"] = time.Now().Add(time.Hour * time.Duration(12)).Unix()// 时间戳
claims["iat"] = time.Now().Unix() // 生成时的当前时间戳
claims["user_id"] = user.Id // 自定义属性
token.Claims = claims
tokenString, err := token.SignedString([]byte("secret")) // 签名密钥
```
`tokenString` 就是我们得到的 Token 令牌了，在令牌中的第二部分加入了 `user_id`属性，方便之后用户在登陆时候的其他操作

在路由中添加 中间件验证
```go
v1.PartyFunc("/login", func(login router.Party) {
	login.Post("/user_login", controller.Login)
	login.Post("/user_reg", controller.Register)
})
v1.Use(middleware.JwtHandler().Serve)
```
JWT 自带验证方法，如果想再自行加入验证可以注册新的中间件，例如
```go
// 路由验证
v1.Use(middleware.JwtHandler().Serve, middleware.AuthToken)

// middleware.AuthToken 中间件 中部分代码
func AuthToken(ctx iris.Context) {
    u := ctx.Values().Get("jwt").(*jwt.Token) //获取 token 信息
    userId, _ := u.Claims.(jwt.MapClaims)["user_id"]
}
```
如例子，可以获取到我们之前 存在token中的 userId 属性,之后可以进行其他操作