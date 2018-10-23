---
layout: post
title: "微服务下的API设计原则"
date: 2018-08-23 17:05
comments: true
categories: 微服务 
tags: [ 微服务, API, 设计 ]
---


# 微服务下的API设计原则

目的：
规范团队乃至公司的API设计。
主要参考： [Github API](https://developer.github.com/v3/)

## 1. 为了安全，请使用HTTPS

与API设计无关、为了安全请使用HTTPS。公网API,强制使用HTTPS。内网API可酌情选择。

## 2. API 地址和版本

在 `url` 中指定 API 的版本是个很好地做法。
如果 API 变化比较大，可以把API设计为子域名，比如 `https://api.ke.com/v3`；
也可以简单地把版本放在路径中，比如 `https://ke.com/api/v1`。
**不建议放入Header**，不直观。

## 3. schema请使用JSON

对于响应返回的格式，JSON 因为它的可读性、紧凑性以及多种语言支持等优点，成为了 HTTP API 最常用的返回格式。因此，最好采用**JSON**作为返回内容的格式。

**不推荐其他格式**，如果必须使用，比如 `xml`，应该在请求头部 `Accept` 中指定。
对于不支持的格式，服务端需要返回正确的 `status code`，并给出详细的说明。

<!-- more -->

## 4. 以资源为中心的URL设计

资源是 `Restful API` 的核心元素，所有的操作都是针对特定资源进行的。而资源就是 `URL`（Uniform Resoure Locator）表示的，所以简洁、清晰、结构化的 URL 设计是至关重要的。Github 可以说是这方面的典范，下面我们就拿 `repository` 来说明。

```
/users/:username/repos
/users/:org/repos
/repos/:owner/:repo
/repos/:owner/:repo/tags
/repos/:owner/:repo/branches/:branch
```

我们可以看到几个特性：

- 资源分为单个文档和集合，尽量使用复数来表示资源，单个资源通过添加 id 或者 name 等来表示
- 一个资源可以有多个不同的 URL
- 资源可以嵌套，通过类似目录路径的方式来表示，以体现它们之间的关系

**NOTE**: 根据RFC3986定义，URL是大小写敏感的。**建议全部使用小写字母**。

## 5. 使用正确的HTTP Method
有了资源的 URL 设计，所有针对资源的操作都是使用 HTTP 方法指定的。比较常用的方法有：

METHOD        |       描述
-----   |   -----
HEAD    |   只获取某个资源的头部信息。比如只想了解某个文件的大小，某个资源的修改日期等
GET     |   获取资源
POST    |   创建资源
PATCH   |   更新资源的部分属性。因为 PATCH 比较新，而且规范比较复杂，所以真正实现的比较少，一般都是用 POST 替代
PUT     |   替换资源，客户端需要提供新建资源的所有属性。如果新内容为空，要设置 `Content-Length` 为 0，以区别错误信息
DELETE  |   删除资源

比如：

```
GET /repos/:owner/:repo/issues
GET /repos/:owner/:repo/issues/:number
POST /repos/:owner/:repo/issues
PATCH /repos/:owner/:repo/issues/:number
DELETE /repos/:owner/:repo
```

建议：依据公司的实际情况，不可能所有的服务都能符合RESTFUL标准。请重点学习使用`GET,POST,DELETE`

### 不符合 CRUD 的情况

在实际资源操作中，总会有一些不符合 `CRUD`（Create-Read-Update-Delete） 的情况，一般有几种处理方法。

#### 使用 POST

为需要的动作增加一个 endpoint，使用 POST 来执行动作，比如 `POST /resend` 重新发送邮件。

#### 增加控制参数

添加动作相关的参数，通过修改参数来控制动作。比如一个博客网站，会有把写好的文章“发布”的功能，可以用上面的 `POST /articles/{:id}/publish` 方法，也可以在文章中增加 `published:boolean` 字段，发布的时候就是更新该字段 `PUT /articles/{:id}?published=true`

#### 把动作转换成资源

把动作转换成可以执行 `CRUD` 操作的资源， github 就是用了这种方法。

比如“喜欢”一个 gist，就增加一个 `/gists/:id/star` 子资源，然后对其进行操作：“喜欢”使用 `PUT /gists/:id/star`，“取消喜欢”使用 `DELETE /gists/:id/star
`。

另外一个例子是 `Fork`，这也是一个动作，但是在 gist 下面增加 `forks`资源，就能把动作变成 `CRUD` 兼容的：`POST /gists/:id/forks` 可以执行用户 fork 的动作。

## 6. Query 让查询更自由

比如查询某个 repo 下面 issues 的时候，可以通过以下参数来控制返回哪些结果：

- state：issue 的状态，可以是 `open`，`closed`，`all`
- since：在指定时间点之后更新过的才会返回
- assignee：被 assign 给某个 user 的 issues
- sort：选择排序的值，可以是 `created`、`updated`、`comments`
- direction：排序的方向，升序（asc）还是降序（desc）
- ……

## 7. 分页 Pagination

当返回某个资源的列表时，如果要返回的数目特别多，比如 github 的 `/users`，就需要使用分页分批次按照需要来返回特定数量的结果。

分页的实现会用到上面提到的 url query，通过两个参数来控制要返回的资源结果：

- size：每页返回多少资源，如果没提供会使用预设的默认值；这个数量也是有一个最大值，不然用户把它设置成一个非常大的值（比如 `99999999`）也失去了设计的初衷
- page：要获取哪一页的资源，默认是第一页

返回的资源列表为 `[(page-1)*size, page*size)`。

## 8. 选择合适的HTTP状态码

HTTP 应答中，需要带一个很重要的字段：`status code`。它说明了请求的大致情况，是否正常完成、需要进一步处理、出现了什么错误，对于客户端非常重要。状态码都是三位的整数，大概分成了几个区间：

- `2XX`：请求正常处理并返回
- `3XX`：重定向，请求的资源位置发生变化
- `4XX`：客户端发送的请求有错误
- `5XX`：服务器端错误

在 HTTP API 设计中，经常用到的状态码以及它们的意义如下表：

状态码  |   Label    | 重点关注|  解释
---     |   ---      |---   |   ---
200     |   OK        | Y |   请求成功接收并处理，一般响应中都会有 body
201     |   Created    | |   请求已完成，并导致了一个或者多个资源被创建，最常用在 POST 创建资源的时候
202     |   Accepted    | |  请求已经接收并开始处理，但是处理还没有完成。一般用在异步处理的情况，响应 body 中应该告诉客户端去哪里查看任务的状态
204     |   No Content  | |  请求已经处理完成，但是没有信息要返回，经常用在 PUT 更新资源的时候（客户端提供资源的所有属性，因此不需要服务端返回）。如果有重要的 metadata，可以放到头部返回
301     |   Moved Permanently  |Y |   请求的资源已经永久性地移动到另外一个地方，后续所有的请求都应该直接访问新地址。服务端会把新地址写在 `Location` 头部字段，方便客户端使用。允许客户端把 POST 请求修改为 GET。
304     |   Not Modified    | Y | 请求的资源和之前的版本一样，没有发生改变。用来缓存资源，和条件性请求（conditional request）一起出现
307     |   Temporary Redirect  ||   目标资源暂时性地移动到新的地址，客户端需要去新地址进行操作，但是**不能**修改请求的方法。
308     |   Permanent Redirect  | |  和 301 类似，除了客户端**不能**修改原请求的方法
400     |   Bad Request     | Y|  客户端发送的请求有错误（请求语法错误，body 数据格式有误，body 缺少必须的字段等），导致服务端无法处理
401     |   Unauthorized    | Y|  请求的资源需要认证，客户端没有提供认证信息或者认证信息不正确
403     |   Forbidden   |Y|   服务器端接收到并理解客户端的请求，但是客户端的权限不足。比如，普通用户想操作只有管理员才有权限的资源。为了安全考虑，避免攻击，对外服务，可将这个状态码改写为**404**返回给客户端
404     |   Not Found   |Y|   客户端要访问的资源不存在，链接失效或者客户端伪造 URL 的时候回遇到这个情况
405     |   Method Not Allowed  | |  服务端接收到了请求，而且要访问的资源也存在，但是不支持对应的方法。服务端**必须**返回 `Allow` 头部，告诉客户端哪些方法是允许的
415     |   Unsupported Media Type  |  | 服务端不支持客户端请求的资源格式，一般是因为客户端在 `Content-Type` 或者 `Content-Encoding` 中申明了希望的返回格式，但是服务端没有实现。比如，客户端希望收到 `xml`返回，但是服务端支持 `Json`
429     |   Too Many Requests   | Y|  客户端在规定的时间里发送了太多请求，在进行限流的时候会用到
500     |   Internal Server Error   |Y |  服务器内部错误，导致无法完成请求的内容
503     |   Service Unavailable     | Y|  服务器因为负载过高或者维护，暂时无法提供服务。服务器端应该返回 `Retry-After` 头部，告诉客户端过一段时间再来重试

上面这些状态码覆盖了 API 设计中大部分的情况，如果对某个状态码不清楚或者希望查看更完整的列表，可以参考 [HTTP Status Code](https://httpstatuses.com/) 这个网站，或者 [RFC7231 Response Status Codes](https://tools.ietf.org/html/rfc7231#section-6) 的内容。
在实际应用中，**不建议再使用更多的状态码**。

## 9. 错误处理码设计

除了使用好HTTP状态码，必须有良好的错误码设计。一个良好的状态码设计应同事给出`状态码`和对应`错误消息`。
HTTP请求返回格式(建议)：

    {
        "code":XXX,
        "message":"ABCDE...",
        "data":{
            ...
        }
    }

在正常请求时，约定`code=0,message="ok"`。

错误时，错误码规则为`ABBBBCCCC`，共9位。

A:错误级别。如1代表系统级错误，2代表服务级错误；
B:项目或模块名称代码；9999个项目或模块，够用了；
C:具体错误编号。单个项目/模块有999种错误应该够用；

A错误级别码：

|级别码|错误说明|
|---|---|
|1|依赖组件级错误|
|2|服务级错误|
|3||
|4|请求错误|
|5|系统级错误|
|...|-|

BBBB项目代码：

|级别码|错误说明|
|---|---|
|0001|search-ershou|
|0002|basic-search|
|...|-|

CCCC错误编号：

|错误编号|错误说明|
|---|---|
|1000|权限认证相关错误|
|2000|连接类错误|
|3000|超时类错误|
|4000|参数错误|
|...||


## 10. 验证和授权

一般来说，让任何人随意访问公开的 API 是不好的做法。验证和授权是两件事情：

- 验证（Authentication）是为了确定用户是其申明的身份，比如提供账户的密码。不然的话，任何人伪造成其他身份（比如其他用户或者管理员）是非常危险的
- 授权（Authorization）是为了保证用户有对请求资源特定操作的权限。比如用户的私人信息只能自己能访问，其他人无法看到；有些特殊的操作只能管理员可以操作，其他用户有只读的权限等等

如果没有通过验证（提供的用户名和密码不匹配，token 不正确等），需要返回 [**401 Unauthorized**](https://httpstatuses.com/401)状态码，并在 body 中说明具体的错误信息；而没有被授权访问的资源操作，需要返回 [**403 Forbidden**](https://httpstatuses.com/403) 状态码，还有详细的错误信息。

**NOTE**：Github API 对某些用户未被授权访问的资源操作返回 [**404 Not Found**](https://httpstatuses.com/404)，目的是为了防止私有资源的泄露（比如黑客可以自动化试探用户的私有资源，返回 403 的话，就等于告诉黑客用户有这些私有的资源）。

## 11. 限流rate limit

如果对访问的次数不加控制，很可能会造成 API 被滥用，甚至被 [DDos 攻击](https://en.wikipedia.org/wiki/Denial-of-service_attack)。根据使用者不同的身份对其进行限流，可以防止这些情况，减少服务器的压力。

对用户的请求限流之后，要有方法告诉用户它的请求使用情况，`Github API` 使用的三个相关的头部可以借鉴：

- `X-RateLimit-Limit`: 用户每个小时允许发送请求的最大值
- `X-RateLimit-Remaining`：当前时间窗口剩下的可用请求数目
- `X-RateLimit-Rest`: 时间窗口重置的时候，到这个时间点可用的请求数量就会变成 X-RateLimit-Limit的值

如果允许没有登录的用户使用 API（可以让用户试用），可以把 `X-RateLimit-Limit` 的值设置得很小，比如 Github 使用的 `60`。没有登录的用户是按照请求的 IP 来确定的，而登录的用户按照认证后的信息来确定身份。
对于超过流量的请求，可以返回 [**429 Too many requests**](https://httpstatuses.com/429) 状态码，并附带错误信息。而 `Github API` 返回的是 [**403 Forbidden**](https://httpstatuses.com/403)，虽然没有 `429` 更准确，也是可以理解的。


## 12. Hypermedia API

Restful API 的设计最好做到 Hypermedia：在返回结果中提供相关资源的链接。这种设计也被称为 [HATEOAS](http://en.wikipedia.org/wiki/HATEOAS)。这样做的好处是，用户可以根据返回结果就能得到后续操作需要访问的地址。
比如访问 [api.github.com](https://api.github.com/)，就可以看到 Github API 支持的资源操作。
Spring技术栈，可以使用[Spring Hateoas](https://spring.io/projects/spring-hateoas)；由于需要在返回body内增加内容，内部服务使用起来颇有难度，不建议。外部服务强烈推荐。

## 13. 编写优秀的文档

API 最终是给人使用的，不管是公司内部，还是公开的 API 都是一样。即使我们遵循了上面提到的所有规范，设计的 API 非常优雅，用户还是不知道怎么使用我们的 API。最后一步，但非常重要的一步是：为你的 API 编写优秀的文档。

对每个请求以及返回的参数给出说明，最好给出一个详细而完整地示例，提醒用户需要注意的地方……反正目标就是用户可以根据你的文档就能直接使用API，而不是要发邮件给你，或者跑到你的座位上问你一堆问题。
文档生成：**建议使用[SWAGGER](https://swagger.io/)**
与Spring集成：使用[SpringFox](http://springfox.github.io/springfox/)
