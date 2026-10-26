# COROS开放平台接口文档 V2.1.1

文档版本：V2.1.1 · 文档整理日期：2026-10-26

[文档首页](README.zh.md) · [English](api-reference.md)

示例中花括号内的内容（如 `{open_id}`、`{access_token}`）为占位符，使用前请连同花括号一起替换为实际值；带编号的占位符用于区分不同用户或记录，同一占位符代表同一值。字段名称保持不变。数字类型保留虚构示例数字，调用前也需替换为实际值；日期请使用接口允许范围内的值，查询日期格式为 `yyyyMMdd`。授权两步使用的回调地址必须完全一致；放入 URL 时请对实际回调地址进行 URL 编码。example.com 下的文件地址仅为占位示例，不能用于实际下载。签名示例中的令牌和请求参数需替换为双方约定的值及实际收到的值。标记为 `jsonc` 的示例含解释性注释，提交请求前需移除注释。

## 目录

- [快速开始](#quick-start)
- [下载地址说明](#download-notes)
- [常见问题](#faq)
- [1. 接入前准备](#section-1)
  - [1.1 文档说明](#section-1-1)
  - [1.2 接入申请](#section-1-2)
  - [1.3 接口请求说明](#section-1-3)
- [2. 授权协议说明](#section-2)
  - [2.1 授权流程](#section-2-1)
- [3. 授权接口](#section-3)
  - [3.1 用户同意授权，获得code](#section-3-1)
  - [3.2 通过code换取accessToken](#section-3-2)
  - [3.3 刷新accessToken](#section-3-3)
  - [3.4 取消授权](#section-3-4)
  - [3.5 获取绑定状态](#section-3-5)
- [4. 数据接口](#section-4)
  - [4.1 获取用户信息](#section-4-1)
  - [4.2 获取运动记录(按指定日期区间)](#section-4-2)
  - [4.3 获取用户日常数据](#section-4-3)
- [5. 运动数据和日常数据推送](#section-5)
  - [5.1 介绍](#section-5-1)
  - [5.2 第三方需要提供的信息](#section-5-2)
  - [5.3 签名校验](#section-5-3)
  - [5.4 推送运动数据概要信息](#section-5-4)
  - [5.5 日常数据推送](#section-5-5)
  - [5.6 获取fit格式的运动数据详情](#section-5-6)
- [6 训练计划接口](#section-6)
  - [6.1 接收第三方平台的训练日程](#section-6-1)
  - [6.2 第三方平台根据workoutId删除训练课程](#section-6-2)
  - [6.3 接收第三方平台的训练课程](#section-6-3)
- [7 路线功能接口](#section-7)
  - [7.1 接收第三方平台的路线数据](#section-7-1)
- [8 第三方上传运动数据](#section-8)
  - [8.1 第三方平台同步上传运动数据](#section-8-1)
  - [8.2 第三方平台异步上传运动数据](#section-8-2)
  - [8.3 查询第三方平台异步上传运动数据状态](#section-8-3)
- [附录](#appendix)
  - [1. 请求频率限制](#appendix-1)
  - [2. HTTP响应码](#appendix-2)
  - [3. 运动类型说明](#appendix-3)


<a id="quick-start"></a>

## 快速开始

1. 按[接入申请](#section-1-2)准备公司、应用和回调域名资料，提交给 COROS 对接人，取得 clientId 和 clientSecret。
2. 选择环境：测试环境 `https://opentest.coros.com`，正式环境 `https://open.coros.com`。按各接口说明配置请求，示例中的账号、授权信息、文件路径和日期需要替换。
3. 按[用户授权](#section-3-1)引导用户授权。保存发起请求时的 state，并在回调时核对；取得的 code 只能使用一次，30 分钟内有效。
4. 按[换取授权凭证](#section-3-2)使用 code 换取 access_token、refresh_token、openId，并保存 expires_in。不要把 clientSecret 放在公开代码或反馈内容中。
5. 调用[用户信息](#section-4-1)确认授权可用，再调用[运动记录](#section-4-2)查询数据。查询区间最多 30 天，不能早于当前日期前三个月；使用用户确实有运动数据的日期。
6. 从返回内容取得 fitUrl 并下载文件。组合运动的下载地址在各单项中；没有 fitUrl 时，参见[下载说明](#download-notes)。需要自动接收数据时，再配置[运动推送](#section-5-4)或[日常数据推送](#section-5-5)。

<a id="download-notes"></a>

## 下载地址说明

- fitUrl 是完整文件下载地址，长度不超过 500 个字符，包含签名及其他参数。接收后应完整保留，不要截断、重新拼接或删除参数。
- 链接可能带有效期，不应作为永久地址保存。取得后建议及时下载；下载失败时先检查返回错误，再通过[获取运动详情](#section-5-6)重新查询下载地址。
- 本文未约定统一的链接有效时长，也未承诺每次查询都会延长有效期；具体规则请联系 COROS 对接人确认。

<a id="faq"></a>

## 常见问题

### 授权过期或解绑后怎么办？

accessToken 默认有效期为 30 天，可按[刷新授权](#section-3-3)延长有效时间。[取消授权](#section-3-4)后，两种凭证均失效，必须重新授权；是否仍绑定可通过[绑定状态](#section-3-5)查询。

### 为什么查不到运动记录或下载地址？

先确认用户授权和查询日期。查询区间最多 30 天，最早为当前日期前三个月。组合运动需查看各单项的 fitUrl；仍没有地址时，按[运动详情接口](#section-5-6)查询。不要假定空结果表示请求失败，应同时检查 result 和 message。

### 收到重复推送怎么办？

运动推送可能重复。建议使用 openId 和 labelId 识别同一用户的同一运动，避免重复创建记录。日常数据会推送最近三天的数据，建议按 openId 和 happenDay 更新已有记录，避免每天重复累加。以上是接收方处理建议。

### 怎样回复推送？

先按对应推送章节校验 client、secret 和签名，再处理数据，并返回该章节规定的成功响应。运动推送的重试规则见[推送步骤](#section-5-2)。本文未规定统一的响应超时时长，接入时请与 COROS 对接人确认。

### 请求超过次数限制怎么办？

默认每个 clientId 每分钟最多 1000 次，超过后返回 HTTP 429，下一分钟恢复。暂停继续重试，降低请求频率；需要更高额度时联系 COROS 对接人申请。

### 接口返回失败时怎样排查？

同时记录 HTTP 状态、result 和 message。参数错误先核对必填项和提交格式；授权无效先检查授权；无权限需联系 COROS 对接人。重复上传错误不要继续反复上传；异步上传成功受理后，还需通过[状态查询](#section-8-3)确认处理结果。

### 如何申请接入或反馈问题？

接入申请和业务问题提交给 COROS 对接人。文档纠错方式及尚无对接人时的处理方法见首页的[接入与反馈](README.zh.md#access-and-support)。反馈时提供文档版本、接口名称、环境、发生时间和已去除敏感信息的错误响应；不要公开 clientSecret、token、签名下载链接或用户数据。

**修订历史记录**

以下记录按已记载的日期排列，历史版本号和日期保留原记录，不应仅凭版本号推断时间先后。如需更正历史记录，应先向 COROS 核实。

| 版本 | 日期 | AMD | 修订者 | 说明 |
| :-------- | :----------- | :--- | :--- | :--------------------- |
| V2.0 | 2020年06月08日 | M |  | 1. 2.0版本接口，支持更多运动类型数据 |
| V2.1 | 2021年08月26日 | A |  | 每分钟请求数量限制说明 |
| V2.2 | 2022年01月07日 | A |  | 新增路线上传功能 |
| V2.3 | 2023年06月06日 | M |  | 6.1 接收第三方平台的训练日程,时间限制由未来7天改为未来1年，并且限制单次只能上传30个训练。支持越野跑运动 |
| V2.4 | 2023年10月19日 | A |  | 新增 6.2 接收第三方平台根据workoutId删除训练 |
| V2.5 | 2023年10月25日 | M |  | 2024年1月1日开始，限制第三方查询日期最早为当前日期前三个月,例如当天为2023年10月30日，则仅能查询2023年7月30日后的数据 |
| V2.6 | 2024年5月23日 | M |  | 4.2 获取运动记录 增加返回运动计划id,5.4 推送运动数据概要信息接口增加推送运动计划id |
| V2.7 | 2024年10月29日 | M |  | 6.1 接收第三方平台的训练日程 目标类型增加"手动结束" |
| V2.0.7 | 2025年07月29日 | A |  | 新增：5.3运动数据推送接口签名校验 |
| V2.0.8 | 2025年09月16日 | A |  | 新增：附录 3.0运动类型说明 |
| V2.1.0 | 2025年12月2日 | A |  | 6.1 接收第三方平台的训练日程 返回参数增加在coros平台已被删除的id列表deletedIds |
| V2.0.10 | 2026年03月04日 | A |  | 新增：5.5日常数据推送 |
| V2.1.1 | 2026年05月23日 | A |  | 新增：6.3 接收第三方平台的训练课程 |
| V2.1.1（文档整理） | 2026年10月26日 | M | | 修复示例、链接与中英文差异，补充快速开始、下载说明及常见问题；不涉及接口行为变更 |

（A-添加，M-修改，D-删除）


<a id="section-1"></a>

## 1. 接入前准备

<a id="section-1-1"></a>

### 1.1 文档说明

本文档制定了COROS开放平台的接入流程及相关技术标准，主要面向与COROS进行开放性技术和数据协作的合作伙伴技术人员。测试环境域名：opentest.coros.com，正式环境域名为：open.coros.com 。

<a id="section-1-2"></a>

### 1.2 接入申请

请填写详细的接入资料，提交给COROS相关对接人审核。您需要提供以下信息：

| **字段** | **说明** |
| :------: | :------------------------------------------------------------------------------------------------------------------: |
| 公司名称 |  |
| 公司官网网址 |  |
| 应用名称 | 50字以内 |
| 应用LOGO | 需要两个尺寸：144px * 144px 和 102px * 102px，png格式 |
| 应用简介 | 100字以内 |
| 授权回调域名 | 1~2个，http://或https://开头，如 `https://example.com`；授权域应与回调链接 redirect_uri 一致 |
| 数据接口 | 合作商通过调用COROS数据接口拉取COROS用户的运动数据。 |

COROS审核通过后，将会颁发应用ID(clientId)和接入秘钥(clientSecret)给接入申请人,这将是您接入COROS的唯一凭证。请妥善保管应用ID和接入秘钥，一旦发现秘钥使用异常，COROS有权随时回收秘钥中止使用。

COROS推送服务需要提供以下信息：

| 信息 | 说明 |
| :--------- | :----------------------------------------------------------------------------------- |
| 运动数据接收地址 | 使用https的post服务接口，需要兼容COROS服务推送的重复运动数据（COROS推送超时，无法判断第三方平台是否已经接收到运动数据，因此可能推送重复的运动数据） |
| 服务健康检查API | 用于检查第三方平台接口正常提供服务（GET请求，校验http status code是否为200） |

<a id="section-1-3"></a>

### 1.3 接口请求说明

1. 第三方APP需支持http协议的跳转，支持这三个状态码： 301、302、303

<a id="section-2"></a>

## 2. 授权协议说明

<a id="section-2-1"></a>

### 2.1 授权流程

COROS开放平台的授权流程基于OAuth 2.0标准，授权过程大致可分为以下4步：

1. 引导用户进入授权页面同意授权，获取code (有效期30分钟，一次有效)
2. 通过code换取accessToken
3. accessToken默认有效期30天，如果需要，开发者可以刷新授权accessToken，避免过期
4. 通过accessToken调用开放的API
请求格式以各接口说明为准：授权换取及刷新使用 `application/x-www-form-urlencoded`，文件上传使用 `multipart/form-data`，数据推送使用 JSON。

授权顺序：第三方引导用户授权 → COROS 返回 code → 第三方换取 accessToken → 调用数据接口。

<a id="section-3"></a>

## 3. 授权接口

<a id="section-3-1"></a>

### 3.1 用户同意授权，获得code

**业务描述**

1. 用户同意授权后

用户同意授权，页面将跳转至redirect\_uri?code=CODE&state=STATE。

若用户禁止授权，则重定向后不会带上code参数，仅会带上state参数redirect\_uri?state=STATE

1. code说明

code作为换取accessToken的票据，每次用户授权带上的code将不一样，code只能使用一次，30分钟未被使用自动过期。

**请求URL示例：**

请求方式：GET

示例：

```http
https://open.coros.com/oauth2/authorize?client_id={client_id}&redirect_uri={redirect_uri}&state={state}&response_type=code
```

**请求参数说明：**

| **参数名** | **必须?** | **参数类型** | **长度** | **参数描述** | **说明** |
| :-------------- | :-------: | :--------: | :------: | :---------------------------------------------- | :-------------------------------------------------------------------------- |
| client\_id | 是 | String | 32 | client唯一标识 |  |
| redirect\_uri | 是 | String | 200 | 重定向url | `　`授权后重定向的回调链接地址，后面可以带参数，请使用urlencode对链接进行处理，并且与申请时填写的域名保持一致 |
| state | 是 | String | 128 | 重定向后会带上state参数，开发者可以填写a-z A-Z 0-9的参数值，最多128字节。 | 我们建议开发者可以用这个参数验证请求有效性，也可以记录用户请求授权页前的位置。这个参数可用于防止跨站请求伪造（CSRF）攻击，回调时应校验 state 与发起授权时保存的值一致 |
| response\_type | 是 | String |  | 授权返回类型 | 请填写code |

<a id="section-3-2"></a>

### 3.2 通过code换取accessToken

**业务描述**

1. accessToken有效期为30天。

**请求URL示例：**

请求方式：POST

Content-Type：application/x-www-form-urlencoded

示例：

```shell
curl --location --request POST 'https://open.coros.com/oauth2/accesstoken' \
--data-urlencode 'client_id={client_id}' \
--data-urlencode 'redirect_uri={redirect_uri}' \
--data-urlencode 'code={code}' \
--data-urlencode 'client_secret={client_secret}' \
--data-urlencode 'grant_type=authorization_code' \
--data-urlencode 'user_id={user_id}'
```

**请求参数说明：**

| **参数名** | **必须?** | **参数类型** | **长度** | **参数描述** | **说明** |
| :-------------- | :-------: | :--------: | :------: | :----------: | :-------------------------------------- |
| client\_id | 是 | String | 32 | client唯一标识 |  |
| redirect\_uri | 是 | String | 200 | 重定向url | 上一步请求code时填写的<br>redirect\_uri，请保持完全一致 |
| code | 是 | String | 64 | 访问凭证 | 上一步获取的code |
| client\_secret | 是 | String | 以实际发放为准 | client秘钥 |  |
| grant\_type | 是 | String |  | 授权类型 | 请填写authorization\_code |
| user\_id | 否 | String |  | 第三方平台用户ID |  |

**返回参数说明**

| **参数名** | **参数类型** | **参数描述** | **说明** |
| :------------ | :-------------- | :------------ | :--------------------- |
| expires_in | signed 32位 int | 超时时间 | accessToken超时时间，单位（秒） |
| refresh_token | String | 授权码 | 刷新accessToken的授权码 |
| access_token | String | token | 接口调用授权凭证 |
| openId | String | COROS用户的唯一标识 |  |

返回结果示例：

```json
{
    "expires_in":2592000,
    "refresh_token":"{refresh_token}",
    "access_token":"{access_token}",
    "openId":"{open_id}"
}
```

<a id="section-3-3"></a>

### 3.3 刷新accessToken

**业务描述**

刷新成功后会更新当前 accessToken 的有效时间。成功响应格式见本节返回示例。

refreshToken 无固定过期时间；用户取消授权后，accessToken 和 refreshToken 均失效，需要重新授权。

举例：假如token有效期是30天，第三方APP会在本地保存token和过期时间，APP可以调用该接口刷新token，延长token的有效期

**请求URL示例：**

请求方式：POST

Content-Type：application/x-www-form-urlencoded

示例：

```shell
curl --location --request POST 'https://open.coros.com/oauth2/refresh-token' \
--data-urlencode 'client_id={client_id}' \
--data-urlencode 'refresh_token={refresh_token}' \
--data-urlencode 'client_secret={client_secret}' \
--data-urlencode 'grant_type=refresh_token'

```


**请求参数说明：**

| **参数名** | **必须?** | **参数类型** | **长度** | **参数描述** | **说明** |
| :--------------: | :-------: | :--------: | :------: | :----------: | :-----------------: |
| client\_id | 是 | String | 32 | client唯一标识 |  |
| refresh\_token | 是 | String | 64 | 授权码 | 刷新accessToken的授权码 |
| client\_secret | 是 | String | 以实际发放为准 | client秘钥 |  |
| grant\_type | 是 | String |  | 授权类型 | 请填写refresh\_token |

**返回参数说明**

| **参数名** | **参数类型** | **参数描述** | **说明** |
| :-------: | :--------: | :--------: | :------: |
| result | String | 返回编码 |  |
| message | String | 返回信息 |  |

`  `返回结果示例：

```json
{
  "result": "0000",
  "message": "OK"
}
```

<a id="section-3-4"></a>

### 3.4 取消授权

#### 业务描述

​        取消授权后，刷新令牌和令牌都会被设置为无效。需要恢复使用，就必须要重新授权。

#### 请求URL示例：

请求方式：POST

url：

```http
https://open.coros.com/oauth2/deauthorize
```

请求参数说明：

#### HTTP Header参数

| 参数名 | 必须? | 参数类型 | 长度 | 参数描述 | 说明 |
| :-----: | :---: | :------: | :---: | :----: | :-------------: |
| token | 是 | String | 32 | 令牌 | 参数放在http请求的头域 |

请求示例

```shell
curl --location --request POST 'https://open.coros.com/oauth2/deauthorize' \
--header 'token: {access_token}'
```

#### 返回参数说明

| **参数名** | **参数类型** | **参数描述** | **说明** |
| :-------: | :--------: | :--------: | :------: |
| result | String | 返回编码 |  |
| message | String | 返回信息 |  |

#### 返回结果示例：

```json
{
  "result": "0000",
  "message": "OK"
}
```

<a id="section-3-5"></a>

### 3.5 获取绑定状态

#### 业务描述

与token是否过期无关，传入的token存在且用户未手动解绑，返回的bindState为1。建议接入方进入绑定列表时可请求此接口获取当前的绑定状态。

#### 请求URL示例：

请求方式：GET

示例

```shell
curl --location --request GET 'https://open.coros.com/coros/bindState?token={access_token}&openId={open_id}'
```

**请求参数说明：**

| 参数名 | 必须? | 参数类型 | 长度 | 参数描述 | 说明 |
| :-----: | :---: | :------: | :---: | :----: | :-------------: |
| token | 是 | String | 64 | 授权凭证accessToken |  |
| openId | 是 | String | 32 | COROS用户唯一标识 |  |

**返回参数说明**

| **参数名** | **参数类型** | **参数描述** | **说明** |
| :-------------- | :-------- | :--------- | :------------------------------------------------------------------------------- |
| result | String | 返回编码 |  |
| message | String | 返回信息 |  |
| data | 返回结果 |  |  |
| - bindState | number | 绑定状态 | 1：绑定，0：解绑 |

返回结果示例：

```json
{
    "data": {
        "bindState": 1
    },
    "message": "OK",
    "result": "0000"
}
```

<a id="section-4"></a>

## 4. 数据接口

<a id="section-4-1"></a>

### 4.1 获取用户信息

**业务描述**

**请求URL示例：**

请求方式： GET

示例：

`https://open.coros.com/coros/userinfosim?token={access_token}&openId={open_id}`

**请求参数说明：**

| **参数名** | **必须?** | **参数类型** | **长度** | **参数描述** | **说明** |
| :------- | :-------: | :--------: | :------: | :--------------- | :------: |
| token | 是 | String | 64 | 授权凭证accessToken |  |
| openId | 是 | String | 32 | COROS用户唯一标识 |  |

**返回参数说明**

| **参数名** | **参数类型** | **参数描述** | **说明** |
| :-------------- | :-------- | :--------- | :------------------------------------------------------------------------------- |
| result | String | 返回编码 |  |
| message | String | 返回信息 |  |
| data | 返回结果 |  |  |
| - nick | String | 昵称 | 用户在COROS的昵称 |
| - openId | String | 用户唯一标识 |  |
| - profilePhoto | String | 用户头像 |  |
| - runCalorie | String | 累计跑步消耗卡路里 | 单位：大卡 |
| - runDistance | String | 累计跑步距离 | 单位：米 |
| - runTotalTime | String | 累计跑步时间 | 单位：秒 |
| - isGray | Boolean | 是否为灰度用户 | true:是灰度用户<br>false:不是灰度用户<br>需要联系COROS平台管理员，<br>在COROS平台配置。<br>目前只有指定第三方用到了该字段 |

返回结果示例：

```json
{
  "data": {
    "nick": "Example user",
    "openId": "{open_id}",
    "profilePhoto": "https://example.com/avatar.png",
    "runCalorie": "296",
    "runDistance": "33602",
    "runTotalTime": "9096",
    "isGray": true
  },
  "message": "OK",
  "result": "0000"
}
```

<a id="section-4-2"></a>

### 4.2 获取运动记录(按指定日期区间)

**业务描述**

1. 接口说明：查询指定日期范围内的运动数据记录，一次查询的日期范围最大为30天，查询日期不早于当天之前的三个月(例如当天为2023年10月30日，则仅能查询2023年7月30日后的数据)

**请求URL示例：**

请求方式： GET

示例：

`https://open.coros.com/v2/coros/sport/list?token={access_token}&openId={open_id}&startDate={start_date}&endDate={end_date}`

**请求参数说明：**

| **参数名** | **必须?** | **参数类型** | **长度** | **参数描述** | **说明** |
| :---------: | :-------: | :--------------: | :------: | :---------------: | :-------------------------: |
| token | 是 | String | 64 | 授权凭证accessToken |  |
| openId | 是 | String | 32 | COROS用户唯一标识 |  |
| startDate | 是 | signed 32位 int | 32 | 查询运动开始日期 | 开始日期（月首日），int类型，如20170101 |
| endDate | 是 | signed 32位 int | 32 | 查询运动结束日期 | 结束日期（月首日），int类型，如20170101 |

<a id="section-4-2-1"></a>

#### 4.2.1 返回参数说明

| **参数名** | **参数类型** | **参数描述** | **说明** |
| :------------------- | :-------------- | :----------- | :----------------------- |
| result | String | 返回编码 |  |
| message | String | 返回信息 |  |
| data | 返回结果 |  |  |
| - planWorkoutId | Signed 64 int type | 运动计划的ID |  |
| - labelId | String | 运动ID |  |
| - mode | signed 32位 int | 运动类型 | 查看[运动类型说明](#sport-types) |
| - subMode | signed 32位 int | 运动子类型 | 查看[运动类型说明](#sport-types) |
| - deviceName | String | 设备名称 |  |
| - distance | 浮点数 | 距离 | 单位：米 |
| - calorie | 浮点数 | 卡路里 | 单位：小卡 |
| - avgSpeed | signed 32位 int | 平均配速 | 单位：秒/千米 |
| - avgFrequency | signed 32位 int | 平均步频 | 单位：步/分钟 |
| - step | signed 32位 int | 总步数 |  |
| - startTime | signed 32位 int | 运动开始时间戳 | 精确到秒 |
| - endTime | signed 32位 int | 运动结束时间戳 | 精确到秒 |
| - startTimezone | signed 32位 int | 开始时间时区 | 15分钟时区制，32表示东八区 |
| - endTimezone | signed 32位 int | 结束时间时区 | 15分钟时区制，32表示东八区 |
| - fitUrl | String | fit文件下载地址；完整下载地址长度不超过500个字符（含签名及其他参数） | 组合运动fit文件下载地址在每个单项运动数据中 |
| - triathlonItemList | 列表 | 铁人三项数据 |  |
| - - mode | signed 32位 int | 铁人三项单项运动类型 | 查看[运动类型说明](#sport-types) |
| - - subMode | signed 32位 int | 铁人三项单项运动子类型 | 查看[运动类型说明](#sport-types) |
| - - distance | 浮点数 | 铁人三项单项运动距离 | 单位：米 |
| - - calorie | 浮点数 | 铁人三项单项卡路里 | 单位：小卡 |
| - - duration | signed 32位 int | 铁人三项单项运动时长 | 单位：秒 |
| - - step | signed 32位 int | 步数 |  |
| - - fitUrl | String | fit文件下载地址；完整下载地址长度不超过500个字符（含签名及其他参数） | 组合运动只有每个单项运动才有fit文件下载地址 |


返回结果示例：

```json
{
    "data":[
        {
            "mode":8,
            "avgFrequency":8,
            "avgSpeed":163,
            "calorie":9553,
            "deviceName":"COROS PACE",
            "distance":3014,
            "duration":491,
            "endTime":1516097362,
            "labelId":"{label_id_1}",
            "subMode":1,
            "startTime":1516096869,
            "step":52,
            "startTimezone":32,
            "endTimezone":32,
            "fitUrl":"https://example.com/fit/example-workout.fit"
        },
        {
            "mode":13,
            "avgSpeed":414,
            "calorie":34058,
            "deviceName":"COROS PACE",
            "distance":3620,
            "duration":1497,
            "endTime":1516195402,
            "labelId":"{label_id_2}",
            "startTime":1516193903,
            "startTimezone":32,
            "subMode":1,
            "endTimezone":32,
            "triathlonItemList":[
                {
                    "mode":8,
                    "subMode":1,
                    "calorie":1000,
                    "distance":1000,
                    "duration":500,
                    "fitUrl":"https://example.com/fit/example-workout.fit"
                },
                {
                    "mode":9,
                    "subMode":1,
                    "calorie":1000,
                    "distance":1000,
                    "duration":500,
                    "fitUrl":"https://example.com/fit/example-workout.fit"
                },
                {
                    "mode":10,
                    "subMode":1,
                    "calorie":1000,
                    "distance":1000,
                    "duration":500,
                    "step":200,
                    "fitUrl":"https://example.com/fit/example-workout.fit"
                }
            ]
        }
    ],
    "message":"OK",
    "result":"0000"
}
```

<a id="section-4-3"></a>

### 4.3 获取用户日常数据

**业务描述**

1. 接口说明：查询指定日期范围内的日常数据记录，一次查询的日期范围最大为30天，查询日期不早于当天之前的三个月(例如当天为2023年10月30日，则仅能查询2023年7月30日后的数据)

**请求URL示例：**

请求方式： GET

示例：

`https://open.coros.com/coros/daily/query?token={access_token}&openId={open_id}&startDate={start_date}&endDate={end_date}`

**请求参数说明：**

| **参数名** | **必须?** | **参数类型** | **长度** | **参数描述** | **说明** |
| :---------: | :-------: | :--------------: | :------: | :---------------: | :-------------------------: |
| token | 是 | String | 64 | 授权凭证accessToken |  |
| openId | 是 | String | 32 | COROS用户唯一标识 |  |
| startDate | 是 | signed 32位 int | 32 | 查询开始日期 | 开始日期（月首日），int类型，如20170101 |
| endDate | 是 | signed 32位 int | 32 | 查询结束日期 | 结束日期（月首日），int类型，如20170101 |

1. ### **返回参数说明**

| **参数名** | **参数类型** | **参数描述** | **说明** |
| :------------------ | :--------: | :--------: | :---------------------- |
| result | String | 返回编码 |  |
| message | String | 返回信息 |  |
| data | 返回结果 |  |  |
| - dailyList | 列表 | 日常数据列表 |  |
| -- happenDay | 32位 int | 数据天 | 格式：yyyyMMdd |
| -- sleepStartTime | String | 睡觉开始时间 | 格式：yyyy-MM-dd HH:mm:ss |
| -- sleepEndTime | String | 睡眠结束时间 | 格式：yyyy-MM-dd HH:mm:ss |
| -- calorie | 浮点数 | 卡路里 | 单位：小卡 |
| -- step | 32位 int | 总步数 |  |
| -- rhr | 32位int | 用户静息心率 | 如果用户没有静息心率，就不返回该信息 |
| -- hrvList | 列表 | 用户带当天hrv原始值 |  |
| --- hrv | 32位int | hrv原始值 |  |
| --- timestamp | 32位int | 产生hrv的时间戳 |  |
| --- hr | 32位int | 平均心率 |  |
| -- ppgHrv | 32位int | hrv评估平均值 | Overnight HRV |
| -- sleepAvgHr | 32位int | 睡眠平均心率 |  |

返回结果示例：

```json
{
    "data":{
        "dailyList":[
            {
                "happenDay":20200615,
                "sleepStartTime":"2020-06-14 22:00:01",
                "sleepEndTime":"2020-06-15 08:00:01",
                "calorie":9553,
                "step":52,
                "rhr":56,
                "hrvList":[
                    {
                        "hrv":25,
                        "hr":60,
                        "timestamp":1592098222
                    },
                    {
                        "hrv":30,
                        "timestamp":1592101822
                    }
                ],
                "ppgHrv": 50,
                "sleepAvgHr": 70
            },
            {
                "happenDay":20200616,
                "sleepStartTime":"2020-06-15 22:00:01",
                "sleepEndTime":"2020-06-18 08:00:01",
                "calorie":9553,
                "step":52,
                "rhr":56,
                "hrvList":[
                    {
                        "hrv":21,
                        "hr":68,
                        "timestamp":1592247630
                    }
                ],
                "ppgHrv": 50,
                "sleepAvgHr": 76
            }
        ]
    },
    "message":"OK",
    "result":"0000"
}
```

<a id="section-5"></a>

## 5. 运动数据和日常数据推送

<a id="section-5-1"></a>

### 5.1 介绍

1. COROS服务推送运动数据概要信息给第三方，第三方平台再根据运动数据ID获取指定的运动数据详情。
2. COROS服务支持.fit格式的运动数据详情文件，也支持json格式的运动数据详情
3. 希望第三方能够支持.fit格式的运动数据详情文件。后期运动数据扩展，.fit格式比json格式要灵活

<a id="section-5-2"></a>

### 5.2 第三方需要提供的信息

1. 第三方需要准备的信息：
   1) 接收运动数据的https服务接口，用于接口签名验证的token（允许16-32长度，由数字、大小写字母总计62个字符随机组成），详细的签名鉴权参考5.3
   2) 第三方收到运动数据后，校验推送请求头域信息的client、secret，确认正确后接收运动数据信息
   3) 第三方平台需要兼容COROS服务推送的重复运动数据（COROS推送超时，无法判断第三方平台是否已经接收到运动数据，因此可能推送重复的运动数据）

| 信息 | 说明 |
| :--------- | :----------------------------------------------------------------------------------- |
| 运动数据接收地址 | 使用https的post服务接口，需要兼容COROS服务推送的重复运动数据（COROS推送超时，无法判断第三方平台是否已经接收到运动数据，因此可能推送重复的运动数据） |
| 服务健康检查API | 用于检查第三方平台接口正常提供服务（GET请求，校验http status code是否为200） |

1. 推送步骤：
   1) COROS服务器每5分钟（时间可调整）查询到每个第三方绑定用户新增和之前推送失败的运动数据
   2) 把运动数据概要信息推送给第三方平台，请求第三方提供的https接口服务完成推送
   3) COROS服务接收到第三方平台的返回结果，如果推送失败，会重试两次。
   4) 如果成功，就标记本次推送的运动数据推送成功
   5) 如果重试之后仍然失败，就标记推送状态为失败
   6) 推送失败的运动数据，在下次推送时，和新增运动数据一起推送给第三方，

超过24小时推送失败的运动数据，就不再推送给第三方平台，并提醒第三方核查服务是否正常

推送过程见上方步骤。重复推送的处理建议见[常见问题](#faq)。


<a id="section-5-3"></a>

### 5.3 签名校验

第三方开发者接入COROS开放平台时，在提交资料配置由COROS主动请求（例如运动推送）的接口时，同时也配置一个签名校验使用的token（允许16-32长度，由数字、大小写字母总计62个字符随机组成）。

在COROS推送数据给第三方的接口时，使用sha1自然序生成签名的策略供对方接口服务端进行校验。

 总体流程如下：

 1：第三方开发者向COROS开放平台提交相应的反向调用接口url、签名材料token

 2：当第三方提供的反向调用接口收到请求时，计算签名，并与请求中传入的签名进行比对

 例如，第三方提交url为：

 POST https://${callback_url}?signature=xxx&timestamp=xxx&nonce=xxx

  验签算法:

1. 将token、timestamp、nonce三个参数进行字典序排序 (即字符串天然排序顺序)
2. 将三个参数字符串拼接成一个字符串，然后进行sha1运算得到用于验证的签名值
3. 使用上一步计算的签名值与入参的signature字段对比，标识该请求来源于COROS开放平台

  开发者的反向调用接口中鉴权demo:

```java
// Java代码
import org.apache.commons.codec.digest.DigestUtils;

...

    // sign_token即为开发者向COROS开放平台提交的签名材料token，开发者的服务中需要使用此数据用于签名验证
    private static final String SIGN_TOKEN = "{sign_token}";

    public boolean checkSignature(String signature, String timestamp, String nonce) {

        // 将入参timestamp, nonce和双方约定的sign_token，3个字符串按自然顺序排序
        List<String> li = Arrays.asList(nonce, timestamp, SIGN_TOKEN);
        Collections.sort(li);
        // 然后对排序后的字符串进行sha1Hex处理
        String sigCalc = DigestUtils.sha1Hex(li.get(0) + li.get(1) + li.get(2));

        // 与入参传入的signature比较是否一致
        return Objects.equals(sigCalc, signature);
    }
...
```

```php
// PHP代码
private function checkSignature()
{
    $signature = $_GET["signature"];        //示例: {signature}
    $timestamp = $_GET["timestamp"];        //示例: {timestamp}
    $nonce = $_GET["nonce"];        //示例: {nonce}

    // 与 Java 示例使用同一个签名令牌，接入时替换为双方约定的值。
    $token = "{sign_token}";
    $tmpArr = array($token, $timestamp, $nonce);
    sort($tmpArr, SORT_STRING);
    $tmpStr = implode( $tmpArr );
    $tmpStr = sha1( $tmpStr );

    if( $tmpStr == $signature ){
        return true;
    }else{
        return false;
    }
}
```


<a id="section-5-4"></a>

### 5.4 推送运动数据概要信息

**业务描述**

1. COROS服务把运动数据概要信息推送到第三方平台
2. 把签名信息（signature、nonce、timestamp）放在推送接口请求的url里面
3. 第三方接收到推送请求要校验client和secret
4. 第三方接收到推送请求要校验签名，signature、nonce、timestamp

**第三方需要提供的信息**

| 信息 | 说明 |
| :--------------- | :---------------------------------------- |
| 运动数据接收地址 | 使用https的post服务接口 |
| client | 第三方标识，用于校验有效请求（COROS分配） |
| secret | 第三方密钥，用于校验有效请求（COROS分配） |

**接口调用说明**

**调用方式**

http请求方式: POST

**输入参数说明**

| **参数名** | **必须?** | **参数类型** | **参数描述** | **参数说明** |
| :------------------- | ------- | :--------: | :----------------------- | :------------------- |
| signature | 是 | String | 签名验证 | 参数放在url里面 |
| nonce | 是 | String | 签名验证随机数 | 参数放在url里面 |
| timestamp | 是 | String | 签名验证时间戳 | 参数放在url里面 |
| client | 是 | String | 第三方标识 | 参数放在Head里面 |
| secret | 是 | String | 第三方密钥 | 参数放在Head里面 |
| sportDataList | 是 | 列表 | 运动概要数据 |  |
| - planWorkoutId | 否 | Signed 64 int type | 运动计划的ID |  |
| - openId | 是 | String | 用户openID |  |
| - labelId | 是 | String | 运动ID |  |
| - mode | 是 | 32位 int | 运动类型 | 查看[运动类型说明](#sport-types) |
| - subMode | 是 | 32位 int | 运动子类型 | 查询[运动类型说明](#sport-types) |
| - deviceName | 否 | String | 设备名称 |  |
| - distance | 是 | 浮点数 | 距离 | 单位：米 |
| - calorie | 是 | 浮点数 | 卡路里 | 单位：小卡 |
| - avgSpeed | 是 | 32位 int | 平均配速 | 单位：秒/千米 |
| - avgFrequency | 是 | 32位 int | 平均步频 | 单位：步/分钟 |
| - step | 否 | 32位 int | 总步数 |  |
| - elevGain | 否 | 32位 int | 累计爬升 | 单位：米 |
| - startTime | 是 | 32位 int | 运动开始时间戳 | 精确到秒 |
| - endTime | 是 | 32位 int | 运动结束时间戳 | 精确到秒 |
| - startTimezone | 是 | 32位 int | 开始时间时区 | 15分钟时区制，32表示东八区 |
| - endTimezone | 是 | 32位 int | 结束时间时区 | 15分钟时区制，32表示东八区 |
| - fitUrl | 是 | String | fit格式运动数据文件下载地址；完整下载地址长度不超过500个字符（含签名及其他参数） | 铁三的fit文件下载在运动子项列表中 |
| - triathlonItemList | 否 | 列表 | 铁人三项数据 |  |
| - - mode | 是 | 32位 int | 铁人三项单项运动类型 |  |
| - - subMode | 是 | 32位int | 铁人三项单项运动子类型 |  |
| - - distance | 是 | 浮点数 | 铁人三项单项运动距离 | 单位：米 |
| - - calorie | 是 | 浮点数 | 铁人三项单项卡路里 | 单位：小卡 |
| - - duration | 是 | 32位 int | 铁人三项单项运动时长 | 单位：秒 |
| - - step | 否 | 32位 int | 铁人三项单项步数 |  |
| - - fitUrl | 是 | String | 铁人三项单项运动fit格式运动数据文件下载地址；完整下载地址长度不超过500个字符（含签名及其他参数） |  |


传入参数格式如下：

1. client、secret参数https头域传递
2. json参数如下：放在body中传递

```json
{
    "sportDataList":[
        {
            "avgFrequency":0,
            "avgSpeed":0,
            "calorie":220000,
            "deviceName":"COROS PACE",
            "distance":22000,
            "duration":2200,
            "endTime":1532576178,
            "endTimezone":32,
            "labelId":"{label_id_3}",
            "mode":13,
            "subMode":1,
            "openId":"{open_id_4}",
            "startTime":1522576178,
            "startTimezone":32,
            "step":2200,
            "triathlonItemList":[
                {
                    "calorie":117738,
                    "distance":748.07,
                    "duration":901,
                    "fitUrl":"https://example.com/fit/example-workout_0.fit",
                    "mode":10,
                    "subMode":1
                },
                {
                    "calorie":391220,
                    "distance":19911.3,
                    "duration":2761,
                    "fitUrl":"https://example.com/fit/example-workout_1.fit",
                    "mode":9,
                    "subMode":1
                },
                {
                    "calorie":248338,
                    "distance":5142.43,
                    "duration":1592,
                    "fitUrl":"https://example.com/fit/example-workout_2.fit",
                    "mode":8,
                    "subMode":1
                }
            ]
        },
        {
            "avgFrequency":0,
            "avgSpeed":368,
            "calorie":30000,
            "deviceName":"COROS PACE",
            "distance":40000,
            "duration":300,
            "endTime":1532576159,
            "endTimezone":32,
            "fitUrl":"https://example.com/fit/example-workout.fit",
            "labelId":"{label_id_4}",
            "mode":8,
            "openId":"{open_id_5}",
            "startTime":1522576159,
            "startTimezone":32,
            "step":300,
            "subMode":1
        },
        {
            "avgFrequency":0,
            "avgSpeed":0,
            "calorie":20000,
            "deviceName":"COROS PACE",
            "distance":20000,
            "duration":200,
            "endTime":1532576158,
            "endTimezone":32,
            "fitUrl":"https://example.com/fit/example-workout.fit",
            "labelId":"{label_id_5}",
            "mode":9,
            "openId":"{open_id_6}",
            "startTime":1522576158,
            "startTimezone":32,
            "step":200,
            "subMode":1
        },
        {
            "avgFrequency":199,
            "avgSpeed":368,
            "calorie":10000,
            "deviceName":"COROS PACE",
            "distance":10000,
            "duration":100,
            "endTime":1532576157,
            "endTimezone":32,
            "fitUrl":"https://example.com/fit/example-workout.fit",
            "labelId":"{label_id_6}",
            "mode":8,
            "openId":"{open_id_6}",
            "startTime":1522576157,
            "startTimezone":32,
            "step":30166,
            "subMode":2
        }
    ]
}
```

**第三方返回结果说明**

| **参数名** | **参数类型** | **参数描述** | **说明** |
| :------- | :-------- | :-------- | :------------------------------------------------------------- |
| result | String | 结果编码 | 0000表示成功; <br/>其他表示失败（当返回非0000时，COROS视为该批次推送失败，将进行该批次数据的重新推送） |
| message | String | 信息说明 | 信息说明，根据不同的错误返回不同的错误信息 |

正常调用，返回的结果如：

```json
{
    "message":"ok",
    "result":"0000"
}
```


<a id="section-5-5"></a>

### 5.5 日常数据推送

**业务描述**

1. COROS服务把日常数据推送到第三方平台
2. 把签名信息（signature、nonce、timestamp）放在推送接口请求的url里面
3. 第三方接收到推送请求要校验client和secret
4. 第三方接收到推送请求要校验签名，signature、nonce、timestamp
5. 推送用户最近三天的日常数据

**第三方需要提供的信息**

| 信息 | 说明 |
| :--------------- | :---------------------------------------- |
| 运动数据接收地址 | 使用https的post服务接口 |
| client | 第三方标识，用于校验有效请求（COROS分配） |
| secret | 第三方密钥，用于校验有效请求（COROS分配） |

**接口调用说明**

**调用方式**

http请求方式: POST

**输入参数说明**

| **参数名** | **参数类型** | **参数描述** | **说明** |
| :---------------- | :----------: | :----------------------------------------- | :----------------------------------- |
| signature | String | 签名验证 | 签名验证 |
| nonce | String | 签名验证随机数 | 签名验证随机数 |
| timestamp | String | 签名验证时间戳 | 签名验证时间戳 |
| client | String | 第三方标识 | 第三方标识 |
| secret | String | 第三方密钥 | 第三方密钥 |
| batchDailyList | 列表 | 日常数据，一次性批量推送多个用户的日常数据 |  |
| - openId | String | 用户openID |  |
| - dailyList | 列表 | 日常数据列表 |  |
| -- happenDay | 32位 int | 数据天 | 格式：yyyyMMdd |
| -- sleepStartTime | String | 睡觉开始时间 | 格式：yyyy-MM-dd HH:mm:ss |
| -- sleepEndTime | String | 睡眠结束时间 | 格式：yyyy-MM-dd HH:mm:ss |
| -- calorie | 浮点数 | 卡路里 | 单位：小卡 |
| -- step | 32位 int | 总步数 |  |
| -- rhr | 32位int | 用户静息心率 | 如果用户没有静息心率，就不返回该信息 |
| -- hrvList | 列表 | 用户带当天hrv原始值 |  |
| --- hrv | 32位int | hrv原始值 |  |
| --- timestamp | 32位int | 产生hrv的时间戳 |  |
| --- hr | 32位int | 平均心率 |  |
| -- ppgHrv | 32位int | hrv评估平均值 | Overnight HRV |
| -- sleepAvgHr | 32位int | 睡眠平均心率 |  |

返回结果示例：

```json

{
  "batchDailyList": [
    {
      "openId": "{open_id_2}",
      "dailyList": [
        {
          "happenDay": 20200615,
          "sleepStartTime": "2020-06-14 22:00:01",
          "sleepEndTime": "2020-06-15 08:00:01",
          "calorie": 9553,
          "step": 52,
          "rhr": 56,
          "hrvList": [
            {
              "hrv": 25,
              "hr": 60,
              "timestamp": 1592098222
            },
            {
              "hrv": 30,
              "timestamp": 1592101822
            }
          ],
          "ppgHrv": 50,
          "sleepAvgHr": 70
        },
        {
          "happenDay": 20200616,
          "sleepStartTime": "2020-06-15 22:00:01",
          "sleepEndTime": "2020-06-18 08:00:01",
          "calorie": 9553,
          "step": 52,
          "rhr": 56,
          "hrvList": [
            {
              "hrv": 21,
              "hr": 68,
              "timestamp": 1592247630
            }
          ],
          "ppgHrv": 50,
          "sleepAvgHr": 76
        }
      ]
    },
    {
      "openId": "{open_id_3}",
      "dailyList": [
        {
          "happenDay": 20200616,
          "sleepStartTime": "2020-06-15 22:00:01",
          "sleepEndTime": "2020-06-18 08:00:01",
          "calorie": 9553,
          "step": 52,
          "rhr": 56,
          "hrvList": [
            {
              "hrv": 21,
              "hr": 68,
              "timestamp": 1592247630
            }
          ],
          "ppgHrv": 50,
          "sleepAvgHr": 76
        }
      ]
    }
  ]
}
```

**第三方返回结果说明**

| **参数名** | **参数类型** | **参数描述** | **说明** |
| :--------- | :----------- | :----------- | :----------------------------------------------------------- |
| result | String | 结果编码 | 0000表示成功; <br/>其他表示失败（当返回非0000时，COROS视为该批次推送失败，将进行该批次数据的重新推送） |
| message | String | 信息说明 | 信息说明，根据不同的错误返回不同的错误信息 |

正常调用，返回的结果如：

```json
{
    "message":"ok",
    "result":"0000"
}
```


<a id="section-5-6"></a>

### 5.6 获取fit格式的运动数据详情

**业务描述**

1. 第三方平台根据收到的运动数据概要信息后，通过运动数据ID查询运动数据详情

2. COROS服务会返回运动数据详情.fit格式的文件下载地址

   **使用说明**

3. 如果调用4.2章节接口获取到的运动数据无fitUrl信息，可以通过本接口查询

4. 如果5.4章节，我方推送的运动数据中无fitUrl信息，可以通过本接口查询

**请求URL示例：**

请求方式： GET

接口地址：https://open.coros.com/v2/coros/sport/detail/fit

示例：`https://open.coros.com/v2/coros/sport/detail/fit?mode=8&subMode=1&labelId={label_id_7}&openId={open_id}&token={access_token}`

#### 请求参数说明

| **参数名** | **必须?** | **参数类型** | **长度** | **参数描述** | **说明** |
| :-------: | :-------: | :--------------: | :------: | :---------------: | :---------: |
| token | 是 | String | 64 | 授权凭证accessToken | 参数放在url里面 |
| openId | 是 | String | 32 | COROS用户唯一标识 | 参数放在url里面 |
| labelId | 是 | 64位 int |  | 运动数据ID | 参数放在url里面 |
| mode | 是 | signed 32位 int |  | 运动类型[运动类型说明](#sport-types) | 参数放在url里面 |
| subMode | 是 | signed 32位 int |  | 运动子类型[运动类型说明](#sport-types) | 参数放在url里面 |


**返回参数说明**

| **参数名** | **参数类型** | **参数描述** | **说明** |
| :------------------- | :-------- | :----------------------- | :------------------ |
| result | String | 返回编码 |  |
| message | String | 返回信息 |  |
| data | 数据 |  |  |
| - labelId | String | 运动ID |  |
| - mode | 32位 int | 运动类型 |  |
| - subMode | 32位 int | 运动子类型 |  |
| - deviceName | String | 设备名称 |  |
| - distance | 浮点数 | 距离 | 单位：米 |
| - calorie | 浮点数 | 卡路里 | 单位：小卡 |
| - avgSpeed | 32位 int | 平均配速 | 单位：秒/千米 |
| - avgFrequency | 32位 int | 平均步频 | 单位：步/分钟 |
| - step | 32位 int | 总步数 |  |
| - elevGain | 32位 int | 累计爬升 | 单位：米 |
| - startTime | 32位 int | 运动开始时间戳 | 精确到秒 |
| - endTime | 32位 int | 运动结束时间戳 | 精确到秒 |
| - startTimezone | 32位 int | 开始时间时区 | 15分钟时区制，32表示东八区 |
| - endTimezone | 32位 int | 结束时间时区 | 15分钟时区制，32表示东八区 |
| - fitUrl | String | fit格式运动数据文件下载地址；完整下载地址长度不超过500个字符（含签名及其他参数） | 铁三的fit文件下载在运动子项列表中 |
| - triathlonItemList | 列表 | 铁人三项数据 |  |
| - - mode | 32位 int | 铁人三项单项运动类型 | 查看[运动类型说明](#sport-types) |
| - - subMode | 32位 int | 铁人三项单项运动子类型 | 查看[运动类型说明](#sport-types) |
| - - distance | 浮点数 | 铁人三项单项运动距离 | 单位：米 |
| - - calorie | 浮点数 | 铁人三项单项卡路里 | 单位：小卡 |
| - - duration | 32位 int | 铁人三项单项运动时长 | 单位：秒 |
| - - step | 32位 int | 铁人三项单项步数 |  |
| - - fitUrl | String | 铁人三项单项运动fit格式运动数据文件下载地址；完整下载地址长度不超过500个字符（含签名及其他参数） |  |

正常调用返回的结果如：

非铁三数据详情返回结果如下：

```json
{
    "data":{
        "avgFrequency":199,
        "avgSpeed":368,
        "calorie":310000,
        "deviceName":"COROS PACE",
        "distance":31000,
        "duration":3100,
        "endTime":1546394402,
        "endTimezone":32,
        "fitUrl":"https://example.com/fit/example-workout.fit",
        "labelId":"{label_id_7}",
        "mode":8,
        "startTime":1546394401,
        "startTimezone":32,
        "step":30166,
        "subMode":2
    },
    "message":"OK",
    "result":"0000"
}
```

铁三数据详情返回结果如下：

```json
{
    "data":{
        "avgFrequency":0,
        "avgSpeed":0,
        "calorie":80000,
        "deviceName":"COROS PACE",
        "distance":100000,
        "duration":800,
        "endTime":1532576164,
        "endTimezone":32,
        "labelId":"{label_id_8}",
        "mode":13,
        "subMode":1,
        "startTime":1522576164,
        "startTimezone":32,
        "step":800,
        "triathlonItemList":[
            {
                "calorie":117738,
                "distance":748.07,
                "duration":901,
                "fitUrl":"https://example.com/fit/example-workout_0.fit",
                "mode":10,
                "step":0,
                "subMode":1
            },
            {
                "calorie":391220,
                "distance":19911.29,
                "duration":2761,
                "fitUrl":"https://example.com/fit/example-workout_1.fit",
                "mode":9,
                "step":0,
                "subMode":1
            },
            {
                "calorie":248338,
                "distance":5142.43,
                "duration":1592,
                "fitUrl":"https://example.com/fit/example-workout_2.fit",
                "mode":8,
                "step":4627,
                "subMode":1
            }
        ]
    },
    "message":"OK",
    "result":"0000"
}
```

错误时，可能返回结果如下：

```json
{
    "result":"5016",
    "message":"未找到运动数据详情"
}
```

<a id="section-6"></a>

## 6 训练计划接口

<a id="section-6-1"></a>

### 6.1 接收第三方平台的训练日程

**业务描述**

第三方平台向COROS推送已经绑定过COROS平台，并授权的用户创建的训练日程。COROS接收该用户未来1年的训练日程（含今天），COROS平台收到数据后会返回StartDate，EndDate告知第三方平台在[StartDate，EndDate]日期范围内的数据被有效接收。

**请求URL示例**

请求方式： POST

示例：[https://open.coros.com/coros/tp/list/push](https://open.coros.com/coros/tp/list/push)

```shell
curl --location --request POST 'https://open.coros.com/coros/tp/list/push' \
--data-urlencode 'token={access_token}' \
--data-urlencode 'openId={open_id}' \
--data-urlencode 'data={"AthleteId":"1001","StartDate":"2023-06-16","EndDate":"2024-06-09","Workouts":[{"Description":"Description","LastModifiedDate":"2022-07-26T07:26:33","Title":"10KW09-2E10+5ST","TotalTime":"2660","Id":"10001","WorkoutDay":"2023-06-16","WorkoutType":"run","StartTime":"2023-06-16T06:00:00","Structure":[{"IntensityClass":"Active","Name":"name1","Ftp":236,"ThresholdHr":170,"ThresholdSpeed":2.5,"Length":{"Unit":"Meter","Value":1200},"IntensityTarget":{"Unit":"PercentOfThresholdSpeed","MinValue":90,"MaxValue":100},"Description":"","Type":"Step"}]}]}'
```

**请求参数说明**

body参数说明

| **参数名** | **必须？** | **参数类型** | **单位** | **说明** |
| :---------------------------------- | :-------: | :----------------------------- | :---------------------------------------: | :--------------------------------- |
| token | 是 | String | 64 | 授权凭证accessToken |
| openId | 是 | String | 32 | COROS用户唯一标识 |
| data | 是 |  |  |  |
| - AthleteId | 是 | int |  | 用户编号 |
| - StartDate | 是 | date | 日期，如2020-09-20 | 第一个训练计划所属日期,日期大于等于今天 |
| - EndDate | 是 | date | 日期，如2020-09-20 | 最后一个训练计划所属日期，与StartDate组合，跨度最多是1年 |
| - Workouts | 是 | array | 参考下面Workout结构说明 | 训练方案集合(该集合单次不能超过30个)， |

Workout结构说明

| **参数名** | **必须？** | **参数类型** | **单位** | **说明** |
| :--------------------------------- | :------- | :----------------------------- | :----------------------------------------- | :--------------------------------- |
| Description | 否 | string |  | 描述 |
| LastModifiedDate | 是 | datetime |  | 最后编辑时间,如"2020-09-14T21:05:00.307" |
| Title | 是 | string |  | 标题 |
| TotalTime | 否 | float | 单位：秒 | 总时间 |
| Id | 是 | int |  | 训练日程唯一标识id |
| WorkoutDay | 是 | datetime | 日期 | 所属日期，如"2020-09-10" |
| WorkoutType | 是 | string |  | 训练类型：Swim, Bike, Run, Strength, trailRun |
| StartTime | 否 | datetime | local timezone | 开始时间,如"2020-09-16T00:00:00" |
| Structure | 是 | array | 参考下面Structure结构说明 | 训练动作集合 |

Structure结构说明(json示例)

```jsonc
{
  "Structure": [
      {
         "IntensityClass": "WarmUp",//动作类型
         "Name": "Warm up",//动作名称
         "Description":"Description", //动作描述或简介
         "Ftp": 0,   //阈值功率，IntensityTarget设置为PercentOfFtp时必须传值
         "ThresholdHr": 0, //阈值心率，IntensityTarget设置为PercentOfThresholdHr时必须传值
         "ThresholdSpeed": 0.83,//阈值配速、速度，多少米/秒，IntensityTarget设置为PercentOfThresholdSpeed时必须传值
         "Length": {
             "Unit": "Second",
             "Value": 600
         },
        "Rest": {
             "Unit": "Second",
             "Value": 600
         },
         "Type": "Step",
         "IntensityTarget": {
             "Unit": "PercentOfThresholdHr",
             "MinValue":70,
             "MaxValue":80
         }
       },
       {
          "IntensityClass": "Active",
          "Name": "Active",
             "Length": {
             "Unit": "Second",
             "Value": 2095
          },
          "Type": "Step",
          "IntensityTarget": {
             "Unit": "PercentOfThresholdHr",
             "MinValue":80,
             "MaxValue":100
          }
       },
       {
          "Type": "Repetition",
          "Length": {
            "Unit": "Repetition",
            "Value": 3
          },
          "Steps": [
            {
              "IntensityClass": "Active",
              "Name": "Hard",
              "Length": {
                "Unit": "Second",
                "Value": 60
              },
              "Type": "Step",
              "IntensityTarget": {
                "Unit": "PercentOfThresholdHr",
                "Value": 115
              }
            },
            {
              "IntensityClass": "Rest",
              "Name": "Easy",
              "Length": {
                "Unit": "Second",
                "Value": 60
              },
              "Type": "Step",
              "IntensityTarget": {
                "Unit": "ValueOfEquipmentWeight",
                "Value": 70
              }
            }
          ]
       },
       {
          "IntensityClass": "CoolDown",
          "Name": "Cool Down",
          "Length": {
              "Unit": "Second",
              "Value": 600
           },
           "Type": "Step",
           "IntensityTarget": {
               "Unit": "ValueOfEquipmentWeight",
               "Value": 75
           }
        }
    ]
}
```

1. 枚举值说明：

   - **IntensityClass:**
     - WarmUp：   热身
     - CoolDown：  放松
     - Active：   训练
     - Rest：    休息
   - **Length/Unit:目标类型**
     - Meter：距离，  单位：米， WorkoutType in ["swim", "bike", "run", "trailRun"]
     - Second：时间，单位：秒， WorkoutType in ["swim", "bike", "run", "strength", "trailRun"]
     - Reps：次数，   单位：次， WorkoutType in ["strength"]
     - EndManually : 手动结束
   - **Rest/Unit:休息设置** (可不传,不传则代表不作休息)
     - Second：时间，单位：秒， WorkoutType in ["swim", "bike", "run", "strength", "trailRun"]
     - Bpm：心率， 心率恢复低于xx bpm结束休息， WorkoutType in ["run", "trailRun"]
     - EndManually : 手动结束休息  WorkoutType in ["swim", "bike", "run", "strength", "trailRun"]，当休息类型为手动结束时，Value可不传
   - **Type:**
     - Repetition：支持多个动作重复的分组
     - Step：单个动作
   - **IntensityTarget/Unit: 强度单位** （不传则代表未设定强度目标）
     - PercentOfFtp： 功率百分比, WorkoutType in ["bike", "run", "trailRun"],支持固定值和区间范围
     - PercentOfThresholdHr：心率百分比, WorkoutType in [ "bike", "run", "trailRun"],支持固定值和区间范围
     - PercentOfThresholdSpeed：速度百分比, WorkoutType in ["bike", "run", "trailRun"],支持固定值和区间范围
     - RangeOfFtp： 功率区间, WorkoutType in ["bike", "run", "trailRun"],支持区间范围
     - RangeOfCandence： 步频区间, WorkoutType in ["run", "trailRun"],支持区间范围
     - RangeOfThresholdHr：心率区间, WorkoutType in [ "bike", "run", "trailRun"],支持区间范围
     - RangeOfThresholdSpeed：速度区间, WorkoutType in ["bike", "run", "trailRun"],支持区间范围
     - ValueOfEquipmentWeight：负重值(指器材)，WorkoutType in ["strength"],仅支持固定值
     - ValueOfWeight：体重值(指用户体重)，WorkoutType in ["strength"],仅支持固定值
     - ValueOfStroke: 泳姿，当IntensityTarget/Unit=ValueOfStroke时，Value表示具体的泳姿类型，0=未定义，1=自由泳，2=蛙泳，3=仰泳，4=蝶泳，255=混合类型

2. Units

   * Steps中的Length单位：米、秒、次
   * 心率单位是 次/每分钟
   * 步频单位是 步/每分钟
   * 功率单位是 瓦
   * 速度单位是 米/秒
   * 重量单位是G

3. Notes

   * Length :长度值是整数

   * Steps : 代表多组动作的集合

   * Percentages :百分比是整数

   * IntensityTarget :目标强度值可以是固定值或者区间范围

   * 当需要描述目标强度值为区间范围时,则设置为:

     "IntensityTarget": {

     ​    "Unit": "RangeOfThresholdSpeed",

     ​    "MinValue":70, //左区间值

     ​    "MaxValue":80 //右区间值

     }

   * 当需要描述目标强度值为固定值时,则设置为:

     "IntensityTarget": {

     ​    "Unit": "ValueOfWeight",

     ​    "Value":80

     }

**推送数据示例**

```json
{
    "AthleteId":"3",
    "StartDate":"2020-11-10",
    "EndDate":"2020-11-10",
    "Workouts":[
        {
            "Description":"Description",
            "LastModifiedDate":"2020-11-10T07:26:33",
            "Title":"10KW09-2E10+5ST",
            "TotalTime":"2660",
            "Id":"10002",
            "WorkoutDay":"2020-11-10",
            "WorkoutType":"run",
            "StartTime":"2020-11-10T06:00:00",
            "Structure":[
                {
                    "IntensityClass":"Active",
                    "Name":"name1",
                    "Ftp":0,
                    "ThresholdHr":0,
                    "ThresholdSpeed":0,
                    "Length":{
                        "Unit":"Second",
                        "Value":120
                    },
                    "IntensityTarget":[

                    ],
                    "Description":"",
                    "Type":"Step"
                },
                {
                    "Type":"Repetition",
                    "Length":{
                        "Unit":"Repetition",
                        "Value":"2"
                    },
                    "Steps":[
                        {
                            "IntensityClass":"CoolDown",
                            "Name":"name2",
                            "Ftp":0,
                            "ThresholdHr":0,
                            "ThresholdSpeed":0,
                            "Length":{
                                "Unit":"Second",
                                "Value":600
                            },
                            "Type":"Step",
                            "IntensityTarget":{
                                "Unit":"RangeOfThresholdHr",
                                "MinValue":135,
                                "MaxValue":151
                            },
                            "Description":"heartrate 135 ~ 151 bpm"
                        },
                        {
                            "IntensityClass":"Active",
                            "Name":"name3",
                            "Ftp":0,
                            "ThresholdHr":0,
                            "ThresholdSpeed":0,
                            "Length":{
                                "Unit":"Second",
                                "Value":300
                            },
                            "Type":"Step",
                            "IntensityTarget":[

                            ],
                            "Description":""
                        }
                    ]
                }
            ]
        }
    ]
}
```

返回结果说明：

| 参数名 | 参数类型 | 参数描述 | 说明 |
| ----------- | -------- | --------------------------- | ------------------------------------------------------------ |
| result | String | 结果编码 | 0000表示成功;<br>5001表示缺少必须的参数或参数不合法<br>5006表示授权无效<br/>5028表示数据推送失败<br/>30009表示无此权限<br>其他表示失败 |
| message | String | 信息说明 | 信息说明，根据不同的错误<br>返回不同的错误信息 |
| data | Object | 数据 |  |
| - startDate | 32位int | coros平台接收数据的开始日期 |  |
| - endDate | 32位int | coros平台接收数据的结束日期 |  |
| - deletedIds | 整数列表 | 在coros平台已被删除的id列表 |  |

返回结果示例：

```jsonc
{
  "message": "OK",
  "result": "0000",
  "data": {
    "startDate": 20200917,    //coros平台接收数据的开始日期
    "endDate": 20200922,    //coros平台接收数据的结束日期
    "deletedIds": [10003] //在coros平台已被删除的id
  }
}
```

<a id="section-6-2"></a>

### 6.2 第三方平台根据workoutId删除训练课程

**业务描述**

通过6.1章节接口创建的训练课程，第三方平台可通过此接口删除，只支持删除今天及未来日期中未执行的训练。

**请求URL示例**

请求方式： POST

示例：[https://open.coros.com/coros/tp/workout/deleteById](https://open.coros.com/coros/tp/workout/deleteById)
```shell
curl --location 'https://open.coros.com/coros/tp/workout/deleteById' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'token={access_token}' \
--data-urlencode 'openId={open_id}' \
--data-urlencode 'workoutIds=[10001,10002]'
```

**请求参数说明**

body参数说明

| **参数名** | **必须？** | **参数类型** | **单位** | **说明** |
| :---------------------------------- | :-------: | :----------------------------- | :---------------------------------------: | :--------------------------------- |
| token | 是 | String | 64 | 授权凭证accessToken |
| openId | 是 | String | 32 | COROS用户唯一标识 |
| workoutIds | 是 | String |  | 训练日程唯一标识id列表-6.1提交的Workouts[i].id，如[10001,10002]，注：间隔逗号用半角"," |


返回结果说明：

| 参数名 | 参数类型 | 参数描述 | 说明 |
| ----------- | ------ | ---------------- | ----------------------------------------------------------------------------------------- |
| result | String | 结果编码 | 0000表示成功;<br>5001表示缺少必须的参数或参数不合法<br>5006表示授权无效<br/>30009表示无此权限<br>其他表示失败 |
| message | String | 信息说明 | 信息说明，根据不同的错误<br>返回不同的错误信息 |
| data | Object | 数据 |  |
| - successIdList | 64位int列表 | 删除成功的id |  |
| - failIdList | 64位int列表 | 删除失败的id |  |

返回结果示例：

```json
{
    "data": {
        "failIdList": [
            10001
        ],
        "successIdList": [
            10002
        ]
    },
    "message": "OK",
    "result": "0000"
}
```

<a id="section-6-3"></a>

### 6.3 接收第三方平台的训练课程

**业务描述**

第三方平台向COROS推送训练课程。该接口适用于创建或更新训练课程，请求参数使用 `data` 字符串，`data` 内传入单个训练课程的 Workout JSON。

**请求URL示例**

请求方式： POST

示例：[https://open.coros.com/coros/tp/workout/push](https://open.coros.com/coros/tp/workout/push)

```shell
curl --location --request POST 'https://open.coros.com/coros/tp/workout/push' \
--data-urlencode 'token={access_token}' \
--data-urlencode 'openId={open_id}' \
--data-urlencode 'data={"Description":"Description","LastModifiedDate":"2022-07-26T07:26:33","Title":"10KW09-2E10+5ST","TotalTime":"2660","Id":"10001","WorkoutType":"run","Structure":[{"IntensityClass":"Active","Name":"name1","Ftp":236,"ThresholdHr":170,"ThresholdSpeed":2.5,"Length":{"Unit":"Meter","Value":1200},"IntensityTarget":{"Unit":"PercentOfThresholdSpeed","MinValue":90,"MaxValue":100},"Description":"","Type":"Step"}]}'
```

**请求参数说明**

body参数说明

| **参数名** | **必须？** | **参数类型** | **单位** | **说明** |
| :--- | :---: | :--- | :---: | :--- |
| token | 是 | String | 64 | 授权凭证accessToken |
| openId | 是 | String | 32 | COROS用户唯一标识 |
| data | 是 | String | 参考下面Workout结构说明 | 单个训练课程的 Workout JSON 字符串 |

Workout结构说明

参考[6.1 训练日程](#section-6-1)中的 Workout 和 Structure 结构说明。

**推送数据示例**

```json
{
  "Description": "Description",
  "LastModifiedDate": "2020-11-10T07:26:33",
  "Title": "10KW09-2E10+5ST",
  "TotalTime": "2660",
  "Id": "10002",
  "WorkoutType": "run",
  "Structure": [
    {
      "IntensityClass": "Active",
      "Name": "name1",
      "Ftp": 0,
      "ThresholdHr": 0,
      "ThresholdSpeed": 0,
      "Length": {
        "Unit": "Second",
        "Value": 120
      },
      "IntensityTarget": [],
      "Description": "",
      "Type": "Step"
    }
  ]
}
```

返回结果说明：

| 参数名 | 参数类型 | 参数描述 | 说明 |
| --- | --- | --- | --- |
| result | String | 结果编码 | 0000表示成功;<br>5001表示缺少必须的参数或参数不合法<br>5006表示授权无效<br/>5028表示数据推送失败<br/>30009表示无此权限<br>其他表示失败 |
| message | String | 信息说明 | 信息说明，根据不同的错误返回不同的错误信息 |

返回结果示例：

```json
{
  "message": "OK",
  "result": "0000"
}
```

<a id="section-7"></a>

## 7 路线功能接口

<a id="section-7-1"></a>

### 7.1 接收第三方平台的路线数据

<a id="section-7-1-1"></a>

#### 7.1.1 业务描述

第三方平台向COROS推送路线数据。首先第三方平台要绑定COROS用户，然后把用户的路线数据推送到COROS。

<a id="section-7-1-2"></a>

#### 7.1.2 接口调用说明

##### 调用方式

HTTP请求方式: POST

Content-Type: multipart/form-data;

URL：https://open.coros.com/coros/route/push

测试环境地址：https://opentest.coros.com/coros/route/push

##### HTTP Header参数

| 参数名 | 必须? | 参数类型 | 参数描述 | 参数说明 |
| ----- | --- | ------ | --------------- | ------------- |
| token | 是 | String | 授权凭证accessToken | 参数放在http请求的头域 |

##### HTTP Body参数

| 参数名 | 必须? | 参数类型 | 参数描述 | 参数说明 |
| ------------- | --- | ------- | ----------- | ------------------------- |
| openId | 是 | String | COROS用户唯一标识 |  |
| openUserId | 是 | String | 第三方平台用户ID |  |
| routeId | 是 | 64位int | 路线ID |  |
| routeFile | 是 | 文件 | 路线文件 | 文件大小不能超过50MB |
| routeFileType | 是 | 32位int | 路线文件类型 | 0: GPX<br>1: KML |
| type | 是 | 32位int | 运动类型 | 1：骑行<br>2：跑步 |
| name | 是 | String | 路线名称 | 名称长度100个字符 |
| distance | 是 | 浮点数 | 路线距离 | 单位：米，保留两位小数 |
| timestamp | 是 | 64位 int | 路线创建时间戳 | 精确到秒 |
| language | 是 | String | 路线语言类型 | 比如：zh-CN、en-US |
| duration | 否 | 32位int | 路线时长 |  |
| elevationGain | 否 | 浮点数 | 累计爬升距离 | 单位：米，保留两位小数 |
| imageUrl | 否 | String | 路线缩略图 |  |

##### 输入参数说明

传入参数格式如下：

Content-Type:multipart/form-data;

```shell
curl --location --request POST 'https://opentest.coros.com/coros/route/push' \
--header 'token: {access_token}' \
--form 'openId="{open_id}"' \
--form 'openUserId="{user_id}"' \
--form 'routeId="1001"' \
--form 'routeFile=@"/route/route.gpx"' \
--form 'routeFileType="0"' \
--form 'type="2"' \
--form 'name="路书名称"' \
--form 'distance="1000.56"' \
--form 'timestamp="1641439666"' \
--form 'language="zh-CN"' \
--form 'duration="600"' \
--form 'elevationGain="10.80"' \
--form 'imageUrl="https://example.com/route.jpg"'
```

<a id="section-7-1-3"></a>

#### 7.1.3 返回结果说明

| 参数名 | 参数类型 | 参数描述 | 说明 |
| ------- | ------ | ---- | ------------------------------------------------------------------------------------------------------------------------- |
| result | String | 结果编码 | 0000表示成功;<br>1031表示参数错误<br>1008表示路书文件大小超过限制<br>5006表示授权无效;<br>5010表示openId无效;<br>13001表示路书文件重复上传<br>30009表示无此权限<br>其他表示失败 |
| message | String | 信息说明 | 信息说明，根据不同的错误返回不同的错误信息 |

正常调用，返回的结果如：

```json
{
  "message":"ok",
  "result":"0000"
}
```

<a id="section-8"></a>

## 8 第三方上传运动数据

<a id="section-8-1"></a>

### 8.1 第三方平台同步上传运动数据

<a id="section-8-1-1"></a>

#### 8.1.1 业务描述

第三方平台向COROS上传运动数据。首先第三方平台要绑定COROS用户，然后把用户的运动数据数据上传到COROS。

目前只支持fit文件上传

<a id="section-8-1-2"></a>

#### 8.1.2 接口调用说明

##### 调用方式

HTTP请求方式: POST

Content-Type: multipart/form-data;

URL：https://open.coros.com/coros/file/synchronous

测试环境地址：https://opentest.coros.com/coros/file/synchronous

##### HTTP Header参数

| 参数名 | 必须? | 参数类型 | 参数描述 | 参数说明 |
| ----- | --- | ------ | --------------- | ------------- |
| token | 是 | String | 授权凭证accessToken | 参数放在http请求的头域 |

##### HTTP Body参数

| 参数名 | 必须? | 参数类型 | 参数描述 | 参数说明 |
| --------- | --- | ------ | --------------- | --------------------------------------------------------------------------------- |
| openId | 是 | String | COROS用户唯一标识 |  |
| fileType | 是 | 32位int | 文件类型，目前只支持fit文件 | 4：fit文件 |
| sportFile | 是 | 文件 | 运动数据文件 |  |
| name | 否 | String | 运动数据名称 | 长度不能超过200个字母或字符 |
| language | 否 | String | 用户使用的语言信息 | 例如：<br>en-US：美国英语<br>zh-CN：中国中文<br/>ja-JP：日语<br/>de-DE：德语<br/>fr-FR：法语<br/>或者其他语言 |

##### 输入参数说明

传入参数格式如下：

Content-Type:multipart/form-data;

```shell
curl --location --request POST 'https://opentest.coros.com/coros/file/synchronous' \
--header 'token: {access_token}' \
--form 'sportFile=@"/fitFile/abc.fit"' \
--form 'fileType="4"' \
--form 'openId="{open_id}"'
```

<a id="section-8-1-3"></a>

#### 8.1.3 返回结果说明

| 参数名 | 参数类型 | 参数描述 | 说明 |
| --------- | ------ | ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| result | String | 结果编码 | 0000表示成功;<br>1031表示参数错误<br>1008表示路书文件大小超过限制<br>5006表示授权无效;<br>5010表示openId无效;<br>5082表示运动数据已经上传到COROS平台，不能重复上传<br>5096表示COROS平台不支持该运动数据<br/>5098表示运动数据上传失败<br/>30009表示无此权限<br>其他表示失败 |
| message | String | 信息说明 | 信息说明，根据不同的错误返回不同的错误信息 |
| data | 列表 | 返回数据 |  |
| - labelId | 64位int | 保存到COROS平台的运动数据ID |  |

正常调用，返回的结果如：

```json
{
    "data": [
        {
            "labelId": 10001
        }
    ],
    "message": "OK",
    "result": "0000"
}
```

<a id="section-8-2"></a>

### 8.2 第三方平台异步上传运动数据

<a id="section-8-2-1"></a>

#### 8.2.1 业务描述

第三方平台向COROS上传运动数据。首先第三方平台要绑定COROS用户，然后把用户的运动数据数据上传到COROS。

目前只支持fit文件上传

<a id="section-8-2-2"></a>

#### 8.2.2 接口调用说明

##### 调用方式

HTTP请求方式: POST

Content-Type: multipart/form-data;

URL：https://open.coros.com/coros/file/upload

测试环境地址：https://opentest.coros.com/coros/file/upload

##### HTTP Header参数

| 参数名 | 必须? | 参数类型 | 参数描述 | 参数说明 |
| ----- | --- | ------ | --------------- | ------------- |
| token | 是 | String | 授权凭证accessToken | 参数放在http请求的头域 |

##### HTTP Body参数

| 参数名 | 必须? | 参数类型 | 参数描述 | 参数说明 |
| --------- | --- | ------ | --------------- | --------------------------------------------------------------------------------- |
| openId | 是 | String | COROS用户唯一标识 |  |
| fileType | 是 | 32位int | 文件类型，目前只支持fit文件 | 4：fit文件，文件扩展名：.fit |
| sportFile | 是 | 文件 | 运动数据文件 |  |
| name | 否 | String | 运动数据名称 | 长度不能超过200个字母或字符 |
| language | 否 | String | 用户使用的语言信息 | 例如：<br>en-US：美国英语<br>zh-CN：中国中文<br/>ja-JP：日语<br/>de-DE：德语<br/>fr-FR：法语<br/>或者其他语言 |

##### 输入参数说明

传入参数格式如下：

Content-Type:multipart/form-data;

```shell
curl --location --request POST 'https://opentest.coros.com/coros/file/upload' \
--header 'token: {access_token}' \
--form 'sportFile=@"/fitFile/abc.fit"' \
--form 'fileType="4"' \
--form 'openId="{open_id}"'
```

<a id="section-8-2-3"></a>

#### 8.2.3 返回结果说明

| 参数名 | 参数类型 | 参数描述 | 说明 |
| ---------- | ------ | ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| result | String | 结果编码 | 0000表示成功;<br>1031表示参数错误<br>1008表示路书文件大小超过限制<br>5006表示授权无效;<br>5010表示openId无效;<br>5082表示运动数据已经上传到COROS平台，不能重复上传<br>5096表示COROS平台不支持该运动数据<br/>5098表示运动数据上传失败<br/>30009表示无此权限<br>其他表示失败 |
| message | String | 信息说明 | 信息说明，根据不同的错误返回不同的错误信息 |
| data | 列表 | 返回数据 |  |
| - uploadId | 64位int | 保存到COROS平台的运动上传ID |  |

正常调用，返回的结果如：

```json
{
    "data": [
        {
            "uploadId": 20001
        }
    ],
    "message": "OK",
    "result": "0000"
}
```

<a id="section-8-3"></a>

### 8.3 查询第三方平台异步上传运动数据状态

<a id="section-8-3-1"></a>

#### 8.3.1 业务描述

第三方平台调用异步上传运动数据接口（https://open.coros.com/coros/file/upload）上传运动数据，COROS平台会在后台按顺序执行运动数据上传任务。第三方平台可以通过该接口，查询异步上传运动数据的状态。

<a id="section-8-3-2"></a>

#### 8.3.2 接口调用说明

##### 调用方式

HTTP请求方式: GET


URL：https://open.coros.com/coros/file/upload/get

测试环境地址：https://opentest.coros.com/coros/file/upload/get

##### HTTP Header参数

| 参数名 | 必须? | 参数类型 | 参数描述 | 参数说明 |
| ----- | --- | ------ | --------------- | ------------- |
| token | 是 | String | 授权凭证accessToken | 参数放在http请求的头域 |

##### URL 查询参数

| 参数名 | 必须? | 参数类型 | 参数描述 | 参数说明 |
| -------- | --- | ------ | --------------- | ---- |
| openId | 是 | String | COROS用户唯一标识 |  |
| uploadId | 是 | 64位int | 异步上传返回的uploadId |  |

##### 输入参数说明

传入参数格式如下：


```shell
curl --location --request GET 'https://open.coros.com/coros/file/upload/get?openId={open_id}&uploadId={upload_id}' \
--header 'token: {access_token}'
```

<a id="section-8-3-3"></a>

#### 8.3.3 返回结果说明

| 参数名 | 参数类型 | 参数描述 | 说明 |
| ---------- | ------ | ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| result | String | 结果编码 | 0000表示成功;<br>1031表示参数错误<br>1008表示路书文件大小超过限制<br>5006表示授权无效;<br>5010表示openId无效;<br>5082表示运动数据已经上传到COROS平台，不能重复上传<br>5096表示COROS平台不支持该运动数据<br/>30009表示无此权限<br>其他表示失败 |
| message | String | 信息说明 | 信息说明，根据不同的错误返回不同的错误信息 |
| data | 列表 | 返回数据 |  |
| - uploadId | 64位int | 保存到COROS平台的运动上传ID |  |
| - status | 32位int | 上传状态 | -1：执行失败<br>1：任务执行中<br>2：任务执行完成 |

正常调用，返回的结果如：

```json
{
    "data": {
        "status": 2,
        "uploadId": 20001
    },
    "message": "OK",
    "result": "0000"
}
```

<a id="appendix"></a>

## 附录

<a id="appendix-1"></a>

### 1. 请求频率限制

1. 默认每个clientId每分钟请求频率1000，超出后返回HTTP状态码429, 下一分钟后恢复正常
2. 请求频率超过1000，可申请提高上限

<a id="appendix-2"></a>

### 2. HTTP响应码


HTTP 状态与响应中的 result 是两层结果，应分别检查。下面汇总各接口已列出的常见错误，具体适用范围以对应接口为准。

| 来源 | 值 | 含义 | 建议处理 |
| --- | --- | --- | --- |
| HTTP 状态 | 429 | 请求超过频率限制 | 暂停请求，下一分钟再试；持续超限时降低频率 |
| result | 0000 | 成功 | 按接口读取结果；异步上传仍需查询最终状态 |
| result | 1031、5001 | 参数错误 | 核对必填项、类型、文件格式及提交方式 |
| result | 5006 | 授权无效 | 检查凭证与绑定状态，必要时重新授权 |
| result | 5010 | openId 无效 | 核对授权返回的用户标识 |
| result | 30009 | 无此权限 | 联系 COROS 对接人确认接口权限 |
| result | 1008 | 文件大小超过限制 | 核对对应上传接口的大小要求；路线文件上限为 50MB |
| result | 13001 | 路线文件重复上传 | 避免反复上传同一文件 |
| result | 5082 | 运动数据重复上传 | 不要再次创建相同运动 |
| result | 5096 | 运动数据不受支持 | 核对上传数据是否符合该接口要求 |
| result | 5028、5098 | 推送或上传失败 | 结合 message 排查，持续失败时联系 COROS 对接人 |


<a id="sport-types"></a>

<a id="appendix-3"></a>

### 3. 运动类型说明

| 运动类型 | 运动子类型 | 运动类型说明 |
| :------- | :--------- | :----------------- |
| 8 | 1 | 室外跑步 |
| 8 | 2 | 室内跑步 |
| 9 | 1 | 室外骑行 |
| 9 | 2 | 室内骑行 |
| 9 | 3 | 公路骑行ebike |
| 9 | 4 | 山地骑行 |
| 9 | 5 | 山地骑行ebike |
| 9 | 6 | 砾石路骑行 |
| 10 | 1 | 室外游泳 |
| 10 | 2 | 室内游泳 |
| 13 | 1 | 铁人三项 |
| 13 | 2 | 组合运动 |
| 13 | 3 | 登山滑雪 |
| 13 | 4 | 户外攀登 |
| 14 | 1 | 登山 |
| 15 | 1 | 越野跑 |
| 16 | 1 | 徒步 |
| 18 | 1 | 室外有氧运动 |
| 18 | 2 | 室内有氧运动 |
| 19 | 1 | 越野滑雪 |
| 20 | 1 | 操场跑圈 |
| 21 | 1 | 双板滑雪 |
| 21 | 2 | 单板滑雪 |
| 22 | 1 | 战斗机模式 |
| 23 | 2 | 力量训练 |
| 24 | 1 | 赛艇 |
| 24 | 2 | 划船机 |
| 25 | 1 | 皮划艇 |
| 26 | 1 | 桨板 |
| 27 | 1 | 帆板 |
| 28 | 1 | 竞速帆板 |
| 29 | 1 | 登山滑雪 |
| 30 | 1 | 多段攀登 |
| 31 | 1 | 健走 |
| 33 | 2 | 单段攀登，室内攀岩 |
| 33 | 3 | 抱石攀登 |
| 34 | 2 | 跳绳 |
| 35 | 2 | 爬楼 |
| 32 | 1 | 路亚钓鱼 |
| 32 | 2 | 徒步钓鱼 |
| 32 | 4 | 卡亚克路亚 |
| 32 | 5 | 近海钓鱼 |
| 32 | 6 | 远海钓鱼 |
| 32 | 7 | 飞蝇船钓 |
| 32 | 8 | 飞蝇岸钓 |
| 32 | 9 | 海滩钓鱼 |
| 36 | 2 | 羽毛球（室内） |
| 37 | 2 | 乒乓球（室内） |
| 38 | 1 | 篮球（户外） |
| 39 | 1 | 足球（户外） |
| 40 | 1 | 匹克球 |
| 41 | 2 | 椭圆机（室内） |
| 42 | 2 | 瑜伽（室内） |
| 43 | 2 | 普拉提（室内） |
| 44 | 2 | 拳击（室内） |
| 45 | 1 | 飞盘（户外） |
| 46 | 1 | 滑板（户外） |
| 47 | 1 | 网球（户外） |
| 48 | 1 | 板式网球 |
| 49 | 1 | 户外攀登（户外） |
| 50 | 2 | 综合体能（室内） |
| 98 | 1 | 自定义运动-户外 |
| 99 | 2 | 自定义运动-室内 |
