# 项目介绍

luckybot 是使用 [Golang](https://golang.org/) 语言开发的通用 [Telegram](https://telegram.org/) 加密货币红包机器人框架。封装了充值、提现、发红包、抢红包、操作纪录等诸多功能，开发者只需简单修改配置文件即可运行工作。脚本系统使用 Lua 语言编写，目的是帮助用户快速定制开发自己的红包机器人，无需重新修改、编译程序。开发者也不必关心红包机器人的实现细节，只需对接加密货币的充值提现即可实现自己的红包机器人了。

### 功能特色
* 支持[Telegram](https://telegram.org/)、[币用](https://www.biyong.sg/index)、[币聊](http://www.coinchat.global/)...
* 支持随机红包和固定红包
* 红包可以发给多个群组或个人

# 开发环境
* Golang 1.8+
* Python
* glide

# 快速开始
### 1. 拉取代码
```bash
git clone https://luckybot.git
```

### 2. 编译程序
```bash
go build
```

### 3. 初始配置
```bash
python init_config.py
```
Telegram 机器人必须开启 [Inline mode](https://core.telegram.org/bots/inline) ，再将 server.yml 配置文件中 **token** 字段的值填写为你 Telegram 机器人 Token。 

### 4. 运行服务

**Linux**

```bash
./luckybot
```

**Windows**

```bash
luckybot.exe
```

# 配置文件

luckybot 服务的配置文件模板位于：[server.yml.example](server.yml.example)，详情参见注释。语言包配置文件位于 [lang/zh_cn.lang](lang/zh_cn.lang)，目前只支持简体中文。

# 充值接口

luckybot 提供了一个接收充值通知信息的 HTTP 接口，地址：`http://<host>:<port>/deposit`。当用户发生红包账户充值事件时，可以发起一个 HTTP POST 请求来告知红包机器人进行处理。此请求的 Body 必须时一个 JSON 字符串，并且遵守以下规则：

| 字段 | 类型 | 说明 |
| ------ | ------ | ------ |
| txid | string | 交易ID |
| heigth | uint64 | 区块高度 |
| from | string | 来源地址 |
| to | string | 目标地址 |
| asset | string | 资产符号 |
| amount | string | 金额 |
| memo | string | 备注信息 |

```json
{
    "txid": "96d2453af92d1943140c16e94db22e8e99fef716",
    "heigth": "82001",
    "from": "from",
    "to": "to",
    "asset": "BTS",
    "amount": "1.0",
    "memo": "hello"
}
```


# 脚本系统

脚本系统提供了 `http` 和 `json` 模块用于和外部通信，另外还定义了一系列事件通知函数用于自定义逻辑处理。

## 事件函数

### on_tick
```lua
function on_tick(delaytime : number)
```
此函数将会定期调用，参数 `delaytime` 表示距离上次调用的时间。

### valid_address
```lua
function valid_address(address : string) -> bool
```
此函数用于验证提现地址是否合法，返回 `bool` 类型。

### deposit_address
```lua
function deposit_address(userid : string) -> string, string
```
此函数用于查询指定用户的充值地址，需要返回两个参数。参数一为充值地址，参数二为 `memo` 信息，如若没有则返回 `nil`。

### on_withdraw
```lua
function on_withdraw(to : string, symbol : string, amount : string, future : Future)
```
此函数用于执行提现逻辑的处理，处理完成之后必须调用 `set_result(txid, error)` 函数。如果无法及时获取结果，应该保存 `future`，然后在 `on_tick` 函数中定期检查结果。

### valid_transaction
```lua
function valid_transaction(txid : string, from : string, to : string, symbo : stringl, amount : string, memo : string) -> bool
```
此函数用于验证交易是否合法，当接收到用户充值后通知将被调用进行验证。

## http 模块

http 模块提供了两种简单的 HTTP 请求方法，目前脚本系统里只能使用 http 模块与外界通信。

### get
```lua
function get(url : string) -> table, string
```

### post
```lua
function post(url : string, content_ype : string, body : string) -> table, string
```

这两个函数都有两个返回值，第一个返回值表示响应结果，其中 `header` 表示头信息，`status_code` 表示状态码，`content_length` 表示内容长度，`body` 表示内容数据。第二个返回值表示错误信息。

## json 模块

json 模块提供了解析字符串为 `table`，以及序列化 `table` 为字符串的函数。

### dump
```lua
function dump(table : table) -> string, string
```
此函数用于将 `table` 序列化为 JSON 字符串。返回值一为 JSON 字符串，返回值二为错误信息。

### parse
```lua
function parse(json : string) -> table, string
```
此函数用于将JSON 字符串解析为 `table`。返回值一为解析成功的 `table`，返回值二为错误信息。

# 管理后台

[luckybot-management](https://luckybot-management) 使用 [Vue.js](https://cn.vuejs.org/) 编写的单页面网站。通过 `luckybot` 服务提供的 RESTful API 查询和管理用户信息。`luckybot-management` 默认使用 http://127.0.0.1:18127
 作为端点，如需修改请自行搜索编辑。


**快速启动：**
```
cd admin
python -m SimpleHTTPServer 8080
```