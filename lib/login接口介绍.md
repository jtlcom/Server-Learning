# login接口介绍

## 目录

* [1. 检查资源版本号接口](#检查资源版本号接口)
* [2.	登录验证](#登录验证)
* [3.	获取服务器列表](#获取服务器列表)

## 客户端登录时会访问以下接口

## [检查资源版本号接口](#目录)

| &nbsp; | &nbsp; |
| :-----: | :------ |
| 请求地址 | <https://fbmy2-login.kingsoft.com/check> |
| 请求方式 | HTTP POST |
| 输入参数 | version(**string**, 当前客户端版本号) |
| 输出参数 | cdn_url(**string**, cdn地址) |
| &nbsp; | version(**string**, 当前服务器资源版本号) |
| 请求事例 | &nbsp; |

```json
输入： https://fbmy2-login.kingsoft.com/check?version=1.0.0_1.0.0_android 
输出：
{
    "cdn_url": "http://fbmy2.kingsoft.com/android/",
    "version": "1.0.0"
}
```

## [登录验证](#目录)

| &nbsp; | &nbsp; |
| :-----: | :------ |
| 请求地址 | <https://fbmy2-login.kingsoft.com/auth> |
| 请求方式 | HTTP POST |
| 输入参数 | platform(**string**, 渠道类型，xg表示西瓜sdk) |
| &nbsp; | openid(**string**, 账号id) |
| &nbsp; | accessToken(**string**, 账号token) |
| &nbsp; | version(**string**, 当前客户端版本号) |
| 输出参数 | cdn_url(**string**, cdn地址) |
| &nbsp; | code(**string**, 返回码”0”表示成功) |
| &nbsp; | list_url(**string**, 获取服务器列表url) |
| &nbsp; | pop(**int**, 用于弹窗使用，暂未用，默认为0) |
| &nbsp; | ret_body(**JSONObject**, sdk返回信息) |
| &nbsp; | server(**JSONObject**, 推荐区服信息) |
| &nbsp; | &emsp;&emsp;host(**string**, ip或域名) |
| &nbsp; | &emsp;&emsp;id(**int**, 区服id) |
| &nbsp; | &emsp;&emsp;name(**string**, 名称) |
| &nbsp; | &emsp;&emsp;port(**int**, 端口) |
| &nbsp; | &emsp;&emsp;status(**string**, 状态”ok”表示正常) |
| &nbsp; | token(**string**, login生成的登录token) |
| &nbsp; | version(**string**, 当前服务器资源版本号) |
| 请求事例 | &nbsp; |

```json
输入：	
https://fbmy2-login.kingsoft.com/auth?platform=xg&openid=jinshan__599e79732adef2db557871__EXP_.&accessToken=eyJhdXRoVG9rZW4iOiIyZGZiMjY1MDY5YTU2YTc0IiwiY2hhbm5lbElkIjoibXV5b3UiLCJkZXZpY2VJZCI6IjZiZDgwZDgxNzA1NWQwMmIiLCJleHREYXRhIjoie1widmVyc2lvblwiOlwiNC4wLjdcIn0iLCJuYW1lIjoiNTk5ZTc5NzMyYWRlZjJkYjU1Nzg3MSIsInBsYW5JZCI6IjgyMzAiLCJ0cyI6IjIwMjAwMjEyMTEyOTM3IiwidWlkIjoiNTk5ZTc5NzMyYWRlZjJkYjU1Nzg3MV9fRVhQXy4iLCJ4Z0FwcElkIjoiMTExMTExNjI0Iiwic2lnbiI6ImI3NTY1Y2QwNWI0NTU5ZThjZTE5NDg5OGIxNDc3YjU0MTU1MDZmMGEifQ==&version=1.0.0_1.0.0_android
输出：
{
    "cdn_url": "http://fbmy2.kingsoft.com/android/",
    "code": "0",
    "list_url": "https://fbmy2-login.kingsoft.com/list?platform=xg&openid=jinshan__599e79732adef2db557871__EXP_.&version=1.0.0_1.0.0_android",
    "pop": 0,
    "ret_body": {
        "certification": "0",
        "channelId": "muyou",
        "deviceId": "6bd80d817055d02b",
        "planId": "8230",
        "uid": "jinshan__599e79732adef2db557871__EXP_.",
        "userName": "599e79732adef2db557871"
    },
    "server": {
        "host": "fbmy2-game.kingsoft.com",
        "id": 8004,
        "name": "s8004",
        "port": 8004,
        "status": "ok"
    },
    "token": "1db+roUGwGgmue48HWHVzzE1ODE0NzgzNjQ=",
    "version": "1.0.0"
}
```

## [获取服务器列表](#目录)

| &nbsp; | &nbsp; |
| :-----: | :------ |
| 请求地址 | <https://fbmy2-login.kingsoft.com/list> |
| 请求方式 | HTTP POST |
| 输入参数 | platform(**string**, 渠道类型，xg表示西瓜sdk) |
| &nbsp; | openid(**string**, 账号id) |
| &nbsp; | version(**string**, 当前客户端版本号) |
| 输出参数 | area(**JSONObject**, 区服分组信息) |
| &nbsp; | &emsp;&emsp;group(**int**, 单个分组区服数量) |
| &nbsp; | &emsp;&emsp;name(**string**, 分组名称) |
| &nbsp; | &emsp;&emsp;start(**int**, 起始区服id) |
| &nbsp; | roles(**JSONObject**, 账号用有的角色列表) |
| &nbsp; | &emsp;&emsp;bp(**int**, 战力) |
| &nbsp; | &emsp;&emsp;gene(**int**, 职业) |
| &nbsp; | &emsp;&emsp;id(**int**, 角色id) |
| &nbsp; | &emsp;&emsp;level(**int**, 等级) |
| &nbsp; | &emsp;&emsp;sex(**int**, 性别) |
| &nbsp; | &emsp;&emsp;sid(**int**, 服务器id) |
| &nbsp; | &emsp;&emsp;vip_level(**int**, vip等级) |
| &nbsp; | &emsp;&emsp;name(**string**, 角色名称) |
| &nbsp; | servers(**JSONObject**, 区服信息) |
| &nbsp; | &emsp;&emsp;host(**string**, ip或域名) |
| &nbsp; | &emsp;&emsp;id(**int**, 区服id) |
| &nbsp; | &emsp;&emsp;name(**string**, 名称) |
| &nbsp; | &emsp;&emsp;port(**int**, 端口) |
| &nbsp; | &emsp;&emsp;status(**string**, 状态”ok”表示正常) |
| 请求事例 | &nbsp; |

```json
输入：					
https://fbmy2-login.kingsoft.com/list?platform=zw&openid=96&version=1.0.0_1.0.0_android 
输出：
{
    "area": {
        "group": 20,
        "name": "风暴魔域2 ",
        "start": 8001
    },
    "roles": [
        {
            "bp": 5,
            "gene": 11,
            "id": 134284836866,
            "level": 1,
            "name": "圣殿欧格登",
            "sex": 1,
            "sid": 8004,
            "vip_level": 0
        }
    ],
    "servers": [
        {
            "host": "fbmy2-game.kingsoft.com",
            "id": 8004,
            "name": "s8004",
            "port": 8004,
            "status": "ok"
        }
    ]
}
```

---
