# elixir安装及demo相关资料

---

<img src="/res/elixir-logo.png" align="right">

## 目录

* [1. Elixir安装](#1elixir安装)
* [2. demo流程分析](#2demo流程分析)
  * [2.1 demo代码调试方法](#21demo代码调试方法)
* [3. Elixir的细节](#3elixir的细节)
* [4. Demo框架图](#4demo框架图)
* [5. 消息序列图](#5消息序列图)

## [1.Elixir安装](#目录)

### 根据官网提供的Windows安装方法会有如下问题

1. 按照图示网址下载
<http://elixir-lang.org/elixir.csv>
![avatar](/res/TIM截图20190822112557.jpg)

2. 打开elixir文件下载文件
![avatar](/res/TIM截图20190822113839.jpg)
![avatar](/res/TIM截图20190822113924.jpg)
选择自己下载的版本下载 这是1.8.1
<https://github.com/elixir-lang/elixir/releases/download>

3. 解压
![avatar](/res/TIM截图20190822114057.jpg)
这是下载的文件,解压文件到一个目录
![avatar](/res/TIM截图20190822114157.jpg)

4. 配置环境变量
找到系统变量

![avatar](/res/TIM截图20190822114249.jpg)
![avatar](/res/TIM截图20190822114421.jpg)

把Precompiled下面bin目录的路径添加(**不要覆盖**)到path里面，elixir便安装完成，可在cmd中输入iex进行验证

![avatar](/res/TIM截图20190822114702.jpg)

如果出现以上情况，请把erl的bin目录也添加到Path

## [2.Demo流程分析](#目录)

![avatar](/res/TIM截图20190822114849.jpg)

demo代码中，每个客户端会通过socket连接到服务器的session，session会启动一个对应角色的avatar。客户端的各种操作通过socket给session发消息，session将消息转发给avatar，avatar分析操作类型，然后调用对应的模块完成相关信息的修改，avatar再将需要反馈的结果传给session，session再传给客户端。

### [2.1Demo代码调试方法](#目录)

1. 在demo根目录中打开cmd
2. mix deps.get  #获取依赖
3. iex -S mix   #编译并加载项目到iex中

通过上面3个步骤（若没有error出现），demo（服务器端）已经运行，接下来可以在打开的iex中调用相关代码进行调试

1. {:ok, avatar_id}=Character.create(account_id, name, gene)  #创建一个玩家角色，例如：Character.create(111, "111", 11)，表示account_id是111，玩家的昵称name是“111”，性别和职业gene是11
2. {:ok, avatar_pid} = Realm.start_avatar avatar_id, self() #为刚才创建的角色启动一个avatar，此处默认session为iex所在的进程
3. GenServer.cast(avatar_pid, {:login, []}) #将该角色登录

经过上面三个步骤之后（没有error出现），就可以演示功能了

```elixir
GenServer.cast(avatar_pid, {{:bag,:sell},[0,3]}) #表示把bag中index为0的物品卖出3个，下图中红色框中的物品卖出3个
```

![avatar](/res/TIM截图20190822115159.jpg)

```elixir
GenServer.cast(avatar_pid, {{:bag,:sell},[0,3]})
```

这个GenServer.cast有2个参数，第1个参数是角色对应avatar进程的pid，第2个参数是一个tuple，tuple中第一项是 模块名和对应里面的函数名组成的tuple，第二项是那个函数接收的参数组成的list
**注：在bag模块中的sell实际上有3个参数，前两个是上面所说，第三个由avatar在调用该模块和函数之前填充进去**

有些模块也不用通过GenServer.cast这种方式调用,
如：Repo、Item等中的函数，可以直接通过 模块名.函数名(参数) 的方式进行调用
若想清除角色和物品数据，可以删除项目根目录下的characters、items文件，重新运行iex -S mix即可

## [3.Elixir的细节](#目录)

1. 使用mix deps.get 下载依赖时，可能会因为超时下载不成功，可根据提示配置两个环境变量，如下：
HEX_HTTP_CONCURRENCY     值为1
HEX_HTTP_TIMEOUT  值为120
![avatar](/res/TIM截图20190822115405.jpg)

2. git 版本控制工具的使用
git使用教程：<https://git-scm.com/book/zh/v2/%E8%B5%B7%E6%AD%A5-%E5%85%B3%E4%BA%8E%E7%89%88%E6%9C%AC%E6%8E%A7%E5%88%B6>

最常用：git clone url
例如：git clone <http://192.168.1.170:8081/xujingde/server.git>
注意：url末尾处有.git，还有尽量通过http（容易些），不用ssl

![avatar](/res/TIM截图20190822115511.jpg)

注意页面位置，尽量用1处点击复制url，在通过git clone 进行下载，不推荐通过2处直接下载

## [4.Demo框架图](#目录)

![avatar](/res/demo框架-改.png)

### 消息示例及作用

`{:reply, response}` 将此消息直接发给客户端，服务器不做其余处理

`{:notify, events}` 将此消息直接发给客户端，服务器不做其余处理

`{:notify, events, changed}` 调用`apply_rules({events, changed}, {id, data})`函数处理events和changed，并返回新的events和changed。将events返回给客户端，changed merge到玩家里。返回新的OTP状态

`{:resolve, context, effects}` 与上条消息类似，多一步在effect中的处理。

`response/events => [{{:xxx, :xx}, id, %{msg}}, {}, ...]` :xx：标识；msg：想发送给客户端的信息

`changed => %{bag: new_bag}`

`context => %{action: {}, events: events, changed: changed}`

---

### 各函数功能（以背包卖东西为例）

`handle_cast({{module, action}, args}, {id, _, data} = state)` OTP函数；接收从session/cmder发出的消息，并调用 dispatch |> handle_result

`dispatch(module, action, args)` 对传入的参数进行处理；使用apply调用相应模块的函数，返回值为该模块函数的返回值

`buy(index, count, currencyType\\:bindGold, {_id, %{bag: bag, currencies: currencies}} = all)` 主要的逻辑处理函数，获取背包中index对应的物品信息；获取玩家持有的金钱；判断是否能够购买以及获得物品单价。组成effect消息；使用:resolve返回

`handle_result({:resolve, context, effects}, {id, session, data})` 消息处理函数。调用effect里面的函数。将events返回给客户端，将changed `merge`到玩家数据里。返回新的OTP状态（也就是handle_cast的返回值）

`Effect.esolve({:gain, :item, item}, {id, _context, %{bag: bag}})` effect消息处理函数。更具effect为背包增加某样物品，返回event、changed消息。

## [5.消息序列图](#目录)

![avatar](/res/序列图.png)

---
