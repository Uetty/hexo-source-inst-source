---
title: Gitlab Api
date: 2020-01-08 19:02:24
tags: api
permalink: gitlab-api
keywords: gitlab, api
rId: MB-20010801
---

## 认证([官网文档地址](https://docs.gitlab.com/ee/api/README.html#authentication))

四种方式

1. OAuth2 tokens
2. Personal access tokens
3. Session cookie
4. GitLab CI job token (Specific endpoints only)

### OAuth2 tokens

`curl https://gitlab.example.com/api/v4/projects?access_token=OAUTH-TOKEN`

或

`curl --header "Authorization: Bearer OAUTH-TOKEN" https://gitlab.example.com/api/v4/projects`

### Personal access tokens

`curl https://gitlab.example.com/api/v4/projects?private_token=<your_access_token>`

或

`curl --header "Private-Token: <your_access_token>" https://gitlab.example.com/api/v4/projects`

或

`curl --header "Authorization: Bearer <your_access_token>" https://gitlab.example.com/api/v4/projects`



## 响应状态码([官网文档地址](https://docs.gitlab.com/ee/api/README.html#status-codes))

成功的状态码主要会有三种，失败的情况有多种，具体官网文档查看

| 状态码 | 描述                                                         | 
| :-----: | :-----------------------------------------------------------: | 
| `200`  | 对于 `GET`, `PUT` or `DELETE` 请求，成功时返回的响应头中的状态码，并且返回的响应体包含json数据 | 
| `201`  | 对于`POST`请求，成功时返回的响应头中的状态码，并且返回的响应体包含json数据 | 
| `204`  | 表示服务器成功执行了请求，但返回的响应体无内容（即无响应体部分） | 

## 分页([官网文档地址](https://docs.gitlab.com/ee/api/README.html#pagination))

提供两种分页方式

1. 键集分页（性能原因考虑，官网推荐这种方式）
2. 偏移分页



### 偏移分页

请求参数控制分页

| 参数名     | 描述                        | 
| :---------: | :--------------------------: | 
| `page`     | 页码（默认1）               | 
| `per_page` | 每页数量（默认20，最大100） | 

### 键集分页

键集分页可以更有效地检索页面，并且与基于偏移的分页不同，运行时与集合的大小无关（翻译自官网）。

请求参数控制分页

| 参数名       | 描述                              | 
| :-----------: | :--------------------------------: | 
| `pagination` | `keyset` (为了允许键集分页而设定) | 
| `per_page`   | 每页数量（默认20，最大100）       | 

例（**官网文档案例**）：

请求（官网文档上`--request PUT`应该是错的，实际请求应该去掉）

```
curl --request PUT --header "PRIVATE-TOKEN: <your_access_token>" "https://gitlab.example.com/api/v4/projects?pagination=keyset&per_page=50&order_by=id&sort=asc"
```

响应信息（**官网文档案例**中响应中包含到下一页的链接）

```
HTTP/1.1 200 OK
...
Link: <https://gitlab.example.com/api/v4/projects?pagination=keyset&per_page=50&order_by=id&sort=asc&id_after=42>; rel="next"
Status: 200 OK
...
```

**按官网文档介绍**，下一页链接中包含额外的过滤器`id_after`，过滤器类型取决于`order_by`选项，当没有更多的记录时，响应头不包含`Link`字段。`rel="next"`，并不是唯一的情况，还可能有`rel="first"`、rel="last"，`Link`字段中也可能同时存在多个`<....>; rel="..."`，解码时需要注意。

仅对特定的资源支持键集分页(实际上应该不止这个接口)

| Resource                                                     | Order              | 
| :-----------------------------------------------------------: | :-----------------: | 
| [/api/v4/projects](https://docs.gitlab.com/ee/api/projects.html) | `order_by=id` only | 

**实际验证之后发现**，与文档描述是不一样的，响应头`Link`字段中的链接增加的参数不是`order_by`字段，而是`page=2`参数，以及增加了`/api/v4/projects`接口的其他参数，如下所示：

```
HTTP/1.1 200 OK
...
Link: <https://gitlab.com/api/v4/projects?membership=false&order_by=id&owned=false&page=2&pagination=keyset&per_page=50&repository_checksum_failed=false&simple=false&sort=asc&starred=false&statistics=false&wiki_checksum_failed=false&with_custom_attributes=false&with_issues_enabled=false&with_merge_requests_enabled=false>; rel="next", <https://gitlab.com/api/v4/projects?membership=false&order_by=id&owned=false&page=1&pagination=keyset&per_page=50&repository_checksum_failed=false&simple=false&sort=asc&starred=false&statistics=false&wiki_checksum_failed=false&with_custom_attributes=false&with_issues_enabled=false&with_merge_requests_enabled=false>; rel="first"
...
```

可能是官网文档较长时间没更新了吧



### 分页响应头

分页响应头中包含下面字段

| 响应头          | 描述                                                         | 
| :--------------: | :-----------------------------------------------------------: | 
| `X-Total`       | 结果总数量（并不一定返回，官网介绍出于性能原因考虑结果集大于1000时就不会返回，实际上不返回的情况可能更多） | 
| `X-Total-Pages` | 结果总页数（并不一定返回，官网介绍出于性能原因考虑结果集大于1000时就不会返回，实际上不返回的情况可能更多） | 
| `X-Per-Page`    | 每页多少条                                                   | 
| `X-Page`        | 当期所在页                                                   | 
| `X-Next-Page`   | 下一页                                                       | 
| `X-Prev-Page`   | 前一页                                                       | 

分页这里还是有不少坑的，实际表现与官方文档描述诸多不符

键集分页响应头中另外还包含如下字段

| 响应头 | 描述                                                         | 
| :-----: | :-----------------------------------------------------------: | 
| `Link` | 可能包含下一页的请求地址、上一页的请求地址、第一页的请求地址、最后一页的请求地址。仅仅是可能包含，不一定包含，比如，当结果总条数特别多时，就不会包含最后一页的请求地址（可能是性能原因考虑，gitlab服务器上实际上没有查询完所有结果就响应了）；当页面位于第一页时，不会包含上一页的请求地址。 | 



## 频率限制([官网文档地址](https://docs.gitlab.com/ee/security/rate_limits.html))

实际上频率限制这一块文档，基本是面向2B客户（即在自己服务器上部署gitlab的客户）的，API查询从官网文档中得不到多少信息。`Rack Attach`章节，可能会有些许借鉴意义，它旨在限制来自大量请求的IP地址的请求。

### Rack Attach

API查询的请求头中返回的字段会包含如下几个([此处官网链接](https://docs.gitlab.com/ee/user/gitlab_com/index.html#haproxy-api-throttle))：

```
RateLimit-Limit: 600
RateLimit-Observed: 6
RateLimit-Remaining: 594
RateLimit-Reset: 1563325137
RateLimit-ResetTime: Wed, 17 Jul 2019 00:58:57 GMT
```

其中`RateLimit-Remaining`表示到重置时间`RateLimit-Reset`为止，能进行多少次请求

对于某些受保护的接口，每分钟超过10次的情况下，会触发code `429`，并且响应头中会包含如下信息。表明当前IP被服务器限制请求，可在60秒后重新发起请求

```
Retry-After: 60
```



默认情况下，受保护的接口有

```
default['gitlab']['gitlab-rails']['rack_attack_protected_paths'] = [
  '/users/password',
  '/users/sign_in',
  '/api/#{API::API.version}/session.json',
  '/api/#{API::API.version}/session',
  '/users',
  '/users/confirmation',
  '/unsubscribes/',
  '/import/github/personal_access_token',
  '/admin/session'
]
```



### 设置项目的请求频率

这一块与API请求无关，是2B客户设置请求频率相关的内容，可以在[User and IP rate limits](https://docs.gitlab.com/ee/user/admin_area/settings/user_and_ip_rate_limits.html)和[Rate limits on raw endpoints](https://docs.gitlab.com/ee/user/admin_area/settings/rate_limits_on_raw_endpoints.html)章节中查看



## 查询相关接口[官网文档地址](https://docs.gitlab.com/ee/api/README.html#status-codes)

### search接口

全局（不限项目project/不限组织group）search接口路径

```
/api/v4/search
```

参数

| 参数     | 描述         | 
| :-------: | :-----------: | 
| `scope`  | 搜索范围     | 
| `search` | 搜索的字符串 | 

scope取值

| 取值 | 描述 |
| :---- | :---- |
| projects | 从项目名称和项目描述中搜索 |
| merge_requests | 从merge记录搜索 |
| issues | 从issue搜索 |
| milestones | 从发布的里程碑版本搜索 |
| snippet_titles | 从代码片（与项目没有关系，是另一种与项目同级别的产品）标题搜索 |
| snippet_blobs | 从代码片的内容搜索 |
| wiki_blobs | 从项目的文件内容搜索（在gitlab官方代码服务器上没法使用，需要在自己服务器部署gitlab服务器，并安装elasticsearch才能使用），在官网描述中看不出wiki_blobs与blobs有何区别 |
| commits | 从commit中搜索（在gitlab官方代码服务器上没法使用，需要在自己服务器部署gitlab服务器，并安装elasticsearch才能使用） |
| blobs | 从项目的文件内容搜索（在gitlab官方代码服务器上没法使用，需要在自己服务器部署gitlab服务器，并安装elasticsearch才能使用） |
| users | 从用户名搜索 |



### projects接口

接口路径

```
/api/v4/projects
```

参数

| 参数                          | 类型    | 是否必传 | 描述                                                         |
| :----------------------------: | :------: | :-------: | :-----------------------------------------------------------: |
| `archived`                    | boolean | no       | 过滤打包状态                                                 |
| `visibility`                  | string  | no       | 过滤可见范围 `public`, `internal`, `private`                 |
| `order_by`                    | string  | no       | 排序依据字段，可以为`id`, `name`, `path`, `created_at`, `updated_at`, `last_activity_at` 字段，默认是 `created_at` |
| `sort`                        | string  | no       | 排序，`asc` / `desc` ，默认是 `desc`                         |
| `search`                      | string  | no       | 搜索字符串                                                   |
| `simple`                      | boolean | no       | true时，只返回少量简单字段                                   |
| `owned`                       | boolean | no       | 过滤，仅当前用户拥有的项目                                   |
| `membership`                  | boolean | no       | 过滤，仅当前用户所属的项目                                   |
| `starred`                     | boolean | no       | 过滤，仅当前用户加星的项目                                   |
| `statistics`                  | boolean | no       | 包括项目统计                                                 |
| `with_custom_attributes`      | boolean | no       | 过滤，包含[自定义属性 custom_attributes](https://docs.gitlab.com/ee/api/custom_attributes.html)设置（仅管理员） |
| `with_issues_enabled`         | boolean | no       | 过滤，根据issues_enabled值                                   |
| `with_merge_requests_enabled` | boolean | no       | 过滤，根据merge_requests_enabled值                           |
| `with_programming_language`   | string  | no       | 过滤，按编程语言过滤                                         |
| `wiki_checksum_failed`        | boolean | no       | wiki校验值计算失败的项目([GitLab Premium](https://about.gitlab.com/pricing/) 11.2中的[介绍](https://gitlab.com/gitlab-org/gitlab/merge_requests/6137)) |
| `repository_checksum_failed`  | boolean | no       | repository校验值计算失败的项目([GitLab Premium](https://about.gitlab.com/pricing/) 11.2中的[介绍](https://gitlab.com/gitlab-org/gitlab/merge_requests/6137)) |
| `min_access_level`            | integer | no       | 过滤，仅当前用户的 [最低访问权限级别](https://docs.gitlab.com/ee/api/members.html) |
| `id_after`                    | integer | no       | 过滤，限制为ID大于指定ID的项目                               |
| `id_before`                   | integer | no       | 过滤，限制为ID小于指定ID的项目                               |

返回值中有两个需要注意的参数`repository_access_level`和`visibility`，只有`repository_access_level`参数值为`enabled`且`visibility`参数值为`public`时才可以clone该仓库



该接口，经身份验证的用户和未经身份验证的用户均可使用，但未经身份验证的用户响应内容中的信息会更少，与`simple=true`的时候相同。

当请求中包含自定义属性时，请求url传参如下所示

```
GET /api/v4/projects?custom_attributes[key]=value&custom_attributes[other_key]=other_value
```





阅读文档后，没怎么弄懂上面`/api/v4/projects`和`/api/v4/search?scope=projects`这两个接口有什么重要的区别，姑且认为它们是可以相互取代的，使用任意一个皆可。

