---
sidebar_position: 1
---
# go-oauth2/oauth
该库是 Go 语言下的 OAuth2 授权服务器核心库，遵循 [RFC 6749](https://datatracker.ietf.org/doc/html/rfc6749) 标准。
提供 token 颁发、验证、存储等核心功能，适合开发自己的 OAuth2 Server。

## 实现端点
go-oauth2/oauth2 实现了标准的 OAuth2 服务端（Authorization Server）常见端点，主要包括：

### 1. 授权端点（/authorize 或 /oauth/authorize）
处理 OAuth2 的授权请求（Authorization Request），支持 Authorization Code、Implicit 等授权模式。
实现方法：`srv.HandleAuthorizeRequest(w, r)`
示例路由:
```go
http.HandleFunc("/authorize", func(w, r) { srv.HandleAuthorizeRequest(w, r) })
```

### 2. 令牌端点（/token 或 /oauth/token）
处理 Access Token 的颁发与刷新，支持授权码、密码、客户端凭证、刷新令牌等模式。
实现方法：`srv.HandleTokenRequest(w, r)`
示例路由：
```go
http.HandleFunc("/token", func(w, r) { srv.HandleTokenRequest(w, r) })
```

### 3. 受保护资源端点（示例为 /test）
通过 Bearer Token 校验访问受保护资源。
通常需要在业务代码中调用 srv.ValidationBearerToken(r) 进行 Token 校验。
示例路由:
```go
http.HandleFunc("/test", func(w, r) {
    token, err := srv.ValidationBearerToken(r)
    // 校验通过后返回资源
})
```

## 端点参数及返回
### 1. /authorize 授权端点
请求参数（GET 或 POST）：

| 参数                  | 必填 | 说明                                               |
|-----------------------|------|----------------------------------------------------|
| response_type         | 是   | 授权类型，通常为 code（授权码模式）或 token（隐式模式） |
| client_id             | 是   | 客户端唯一标识                                     |
| redirect_uri          | 否   | 授权后重定向回客户端的 URI，需与注册一致           |
| scope                 | 否   | 申请的权限范围                                     |
| state                 | 否   | 客户端自定义参数，授权后会原样返回，用于防 CSRF 攻击 |
| code_challenge        | 否   | PKCE 扩展字段，增强安全性（如用 PKCE）             |
| code_challenge_method | 否   | PKCE 方法（如 S256）                               |

示例：
```bash
GET /authorize?client_id=000000&response_type=code&redirect_uri=http://localhost&scope=read&state=xyz
```
返回内容：
- 授权码模式（response_type=code）
  成功后重定向到 redirect_uri，带上 code 和 state
  ```
  HTTP/1.1 302 Found
  Location: http://localhost?code=AUTH_CODE&state=xyz
  ```  
- 隐式模式（response_type=token）
  重定向并带上 access_token
- 失败时
  重定向并带 error 参数
  ```bash
  Location: http://localhost?error=access_denied&state=xyz
  ```
### 2. /token 令牌端点
请求参数（POST，Content-Type: application/x-www-form-urlencoded）

常见模式参数如下：

**（1）授权码模式 grant_type=authorization_code**
|参数|	必填|	说明|
|---|---|---|
|grant_type |	是|	固定为 authorization_code|
|code |	是|	从 /authorize 获取的授权码|
|redirect_uri |	是|	必须与授权请求时一致|
|client_id |	是|	客户端ID|
|client_secret |	是|	客户端密钥|

**（2）客户端模式 grant_type=client_credentials**
|参数|	必填|	说明|
|---|---|---|
|grant_type|	是|	client_credentials|
|client_id|	是|	
|client_secret|	是|	

**（3）密码模式 grant_type=password**
|参数|	必填|	说明|
|---|---|---|
|grant_type|	是|	password|
|username|	是|	用户名|
|password|	是|	用户密码|
|client_id|	是|	|
|client_secret|	是|	|

**（4）刷新令牌模式 grant_type=refresh_token**
|参数|	必填|	说明|
|---|---|---|
|grant_type|	是|	refresh_token|
|refresh_token|	是|	刷新令牌|
|client_id|	是|	|
|client_secret|	是|	|
