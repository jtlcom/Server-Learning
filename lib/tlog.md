# Tlog整体架构

## [目的](#目录)

在游戏运行时，Realm服主要有三种输出方式：
1. 通过**IO**所输出的控制台消息，这部分的内容只会出现在控制台上
2. 通过**Logger**所输出的日志，如info、debug、warn，这部分的内容不光会输出到控制台，还会保存在在文件**realm.log**中
3. 通过**Tlog**输出的玩家操作记录，这部分的内容会保存在tlog的目录中

Tlog的主要目的是记录玩家的操作记录

## [目录](#tlog)

* [设置选项](#设置选项)
* [执行流程](#执行流程)
* [表](#表)
* [数据](#数据)
* [bulk](#bulk)
* [数据上报](#数据上报)

## [设置选项](#目录)

在realm服的**config/config.exs**文件中，有一部分是Tlog的配置信息：

```elixir
config :my_server, Tlog,
  is_debug: false,  # 是否输出debug信息
  is_use_tlog: false, # 是否开启tlog记录
  is_udp_send: false, # 是否使用udp传输
  is_write_log: false,  # 是否写入文件
  log_path: "./", # 文件路径
  udp_host: 'shanghai.nanhui8.tglog.datacenter.db', # udp地址
  udp_port: 45460,  # udp端口
  separator: "|", # 分隔符
  pool_size: 5
```

如果需要开启tlog记录，把**is_use_tlog**设置为true即可，在控制台上，可以输入Tlog.config()来查看realm服的Tlog的配置。

## [执行流程](#目录)

![avatar](/res/TIM截图20200224170932.jpg)

```elixir
# 1: 功能模块调用Tlog功能
def entity_die(some_parameter, {id, data}) do
  do_somethings

  Tlog.Flow.bosskillflow(killer_id, %{actid: @act_id, bossid: temp_id, drops: share_drops})

  do_somethings
end
```

```elixir
# 2: Tlog模块
defmacro structs(name, do: quoted) do
  def unquote(func)(args) do
    mod = __MODULE__
    func = unquote(func)

    if Tlog.config(:is_use_tlog) do
      Tlog.Pool.process({mod, func, args, CommonAPI.timestamp()})
    end
  end

  def unquote(func)(id, args) do
    mod = __MODULE__
    func = unquote(func)

    if Tlog.config(:is_use_tlog) do
      Tlog.Pool.process({mod, func, id, args, CommonAPI.timestamp()})
    end
  end

  def unquote(func)(id, args, time) do
    mod = __MODULE__
    func = unquote(func)

    if Tlog.config(:is_use_tlog) do
      Tlog.Pool.process({mod, func, id, args, time})
    end
  end
end
```

```elixir
# 3: Tlog.Pool通过GenServer发给Tlog.Server
defmodule Tlog.Pool do
  @server Tlog.Pool

  def process(msg) do
    cast(msg)
  end

  def cast(request) do
    :poolboy.transaction(@server, fn
      pid -> GenServer.cast(pid, request)
    end)
  end
end
```

```elixir
# 4: Tlog.Server通过apply调用具体的Tlog文件
def handle_cast({mod, func, id, data, args, timestamp}, %{socket: socket} = state) do
  do_something

  extra =
    try do
      apply(mod, :transform, [func, id, data, args])
    rescue
      err ->
        Logger.warn(
          "tlog: call transform failed, context: \n #{
            inspect(%{avatar_id: id, mod: mod, tag: func, args: args}, pretty: true)
          } \n error: #{inspect(err, pretty: true)}, stacktrace is #{
            inspect(:erlang.get_stacktrace(), pretty: true)
          }"
        )

        :missed_transform
    end
  do_something
end
```

## [表](#目录)

Tlog的流水表、状态表主要存放在**lib/tlog**目录的文件里(如**eco、flow、character**等)。

```elixir
defmodule Tlog.Props do
  use Tlog  #文件开头写上 use Tlog  方便调用Tlog的一些转换函数

  @version 1
  @annotation "人物等级流水表"
  @desc "人物等级流水表"
  structs :PlayerExpFlow do
    %{name: :GameSvrId,             type: "string",   size: 25,                          desc: "(必填)登录的游戏服务器编号"}
    . # other_maps
    %{name: :HexRoleId,             type: "string",   size: 64,   defaultvalue: "NULL",  desc: "(必填)十六进制角色ID"}
  end

  # player_exp_flow为表名PlayerExpFlow的小驼峰
  def transform(:player_exp_flow, _id, _data, args) do
    do_something
  end

  use Tlog.Specs  # 文件结尾写上 use Tlog.Specs 主要用于生成xml
end
```

## [数据](#目录)

Tlog中的数据主要分为三类：

1. 元数据 (如：服务器ID、平台等)
2. 玩家基本信息 (如：账号ID、等级、VIP等级、设备、ip地址等)
3. 该表所需的特定数据 (如：坐骑表的 坐骑ID)

元数据和玩家基本信息是在模块**Tlog.Server**中生成的

```elixir
  def meta(_platform \\ nil) do
    realm_id = Realm.id()
    {app_id, platform, area} = realm_id |> realm_info()

    %{
      :GameSvrId => realm_id,
      . # other_data
      :area => area
    }
  end

  def base_info(%{id: id, name: name, gene: gene} = data) do
    account_id = Map.get(data, :account_id, 0)
    level = Map.get(data, :level, 0)
    . # other_data
    xssLevel = Map.get(data, :xss, %{}) |> Map.get(:level, 0)

    Map.merge(data, %{
      openid: account_id,
      . # other_data
      vClientIP: client_ip
    })
  end
```

而特定数据则是在功能模块中，以参数的形式发给**transform**函数

```elixir
# 功能模块
  Tlog.Props.vip_level_flow(id, %{prev: prev_level, current: level, exp: vip_exp})

# transform函数
  def transform(:vip_level_flow, _id, _data, %{prev: prev_level, current: level, exp: vip_exp}) do
    %{
      iBeforeVipLevel: prev_level,
      iAfterVipLevel: level,
      VipExp: vip_exp,
    }
  end
```

## [bulk](#目录)

对于部分的表（如：符印、坐骑、宝石、背包等），可能会一次性生成多条记录

```elixir
# 功能模块
  Tlog.Character.runeinfo(id, rune_equip, time)

# transform函数
  def transform(:runeinfo, _id, _data, rune_equip) do
    bulk = rune_equip |> Enum.reject(&(elem(&1, 1) == nil))  |> Enum.map(fn
      {index, %{id: rune_id} = rune} ->
        %{
          index: index,
          rune_id: rune_id,
          rune_lv: Map.get(rune, :level, 1)
        }

      {index, _} ->
        %{
          index: index,
          rune_id: 0,
          rune_lv: 0
        }
    end)

    %{
      bulk_output: true,
      bulk: bulk
    }
  end
```

而在**Tlog.Server**模块中，则会判断bulk是否为true

```elixir
  case extra do
    e when is_map(e) ->
      data = Map.merge(base, e) |> Map.put(:dtEventTime, time_from_ts(timestamp))
      log("tlog data", data)

      case e do
        %{bulk_output: true, bulk: bulk} when is_list(bulk) and bulk != [] ->
          Enum.each(bulk, fn kv ->
            data = Map.merge(data, kv)
            output(name, func, entries, data, socket, timestamp)
          end)

        _ ->
          output(name, func, entries, data, socket, timestamp)
      end

    _ ->
      :ok
  end
```

## [数据上报](#目录)

```elixir
  # Tlog.Server

  def send_data(:ok, _, _), do: :ok

  def send_data(output, socket, timestamp) do
    Tlog.config(:is_write_log) && TlogWriter.cast({:write, output, timestamp})

    if Tlog.config(:is_udp_send) do
      :gen_udp.send(
        socket,
        Tlog.config(:udp_host),
        Tlog.config(:udp_port),
        output
      )
    end
  end
```

---
