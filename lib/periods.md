# 服务器端周期运营活动整体架构

---

<img src="res/fbmy.png" align="right">

## 目录

* [数据结构](#数据结构)
* [数据存取](#数据存取)
* [活动状态](#活动状态)
* [活动中控](#活动中控)
* [数据初始化](#数据初始化)
* [数据重置](#数据重置)
* [消息协议](#消息协议)
* [依赖库](#依赖库)

## [数据结构](#目录)

1. 采用**结构体**的方式对活动数据进行业务操作

2. 采用**map**的形式存放玩家数据和全服活动数据

3. 对于玩家 数据存放在玩家数据的periods字段，以活动ID为键

4. 对于服务器 数据可采用Repo和ETS共同存储 服务器重启时，从Repo取出放在ETS 每次调用时，从ETS读取数据 保存时，Repo和ETS都要进行保存

以兑换活动为例

```elixir
%{
  level: 1,
  periods: %{
    act_id => %{
      last_update_time: 0,
      # 个人可兑换次数
      limit: %{}
    }
  }
} = player_data

%{
  last_update_time: 0,
  # 世界等级
  world_level: 0,
  # 全服可兑换次数
  limit: %{}
} = server_data <- ETS.load(act_id)

# act_id = 4位活动ID :: integer
```

## [数据存取](#目录)

数据存储主要采用磁盘和内存**相结合**的方式进行

1. 基于磁盘 **调用DB.RepoAPI**，使用磁盘的目的主要是防止数据丢失，服务器可能会在停服更新、宕机或其他因素下被关闭，如果不实时备份磁盘，则可能导致数据异常丢失
2. 基于内存 **采用ETS**，主要是为了读取数据时速度更快
3. 对于异常处理的情况 当服务器关闭重启时，周期运营活动会调用封装库**PeriodsAPI.Behavior**(周期运营活动依赖模块)中*on_restart/2*回调函数(callbacks),在该函数中，将Repo中的数据传入ETS即可。
4. 关于数据清除 数据存储时，是以周期运营活动的4位活动ID为键，该ID唯一，且后续不会再使用，所以可以在活动正常终止时，清除服务器端的数据，但对于玩家的活动数据可以暂时保留。
5. 对活动数据进行业务操作时，对象都是结构体，所以在取用时，需要先将map转换为struct，而在存储时，则需要将struct转化为map。

数据保存手段：

```elixir
defmodule Periods.Exchange.Repo do
  # 调用 DB.RepoAPI (DB泛型API模块)
  use DB.RepoAPI, name: :periods_exchange
end

defp save(act_id, new_act_data) do
  new_act_data = Map.from_struct(new_act_data) # convert struct to map
  Ets.save(act_id, new_act_data)
  Repo.save({act_id, new_act_data}) # 分别存储于内存和磁盘中
end
```

## [活动状态](#目录)

在封装库**PeriodsAPI.Behavior**(周期运营活动依赖模块)中，定义了几个周期运营活动关于状态的回调函数，详细内容可参考[依赖库](#依赖库)：

```elixir
# 周期运营活动开始的回调事件
on_begin(id, data)

# 周期运营活动停止的回调事件
on_close(id, data)

# 玩家登录时检查玩家周期运营活动数据的回调函数
on_login_reset(id, now_time, {})

# 重启服务器时，周期运营活动重新开始的回调事件
on_restart(id, data)

# 周期运营活动阶段开始的回调事件
on_run(id, data)

# (id :: integer(), data :: :ok | period_struct()) :: map()

# 活动开启
def on_begin(act_id, _act_data) do
  init_act_world_info(act_id, Utils.timestamp()) # 初始化服务器端的活动数据
  Realm.broadcast({{:periods, :reset}, [act_id]}) # 活动数据重置公告
  %{}
end
```

## [活动中控](#目录)

### [简介](#活动中控)

<img src="res/TIM截图20191122153854.jpg" align="right">

1. 活动中控功能也被 [腾讯游戏接入平台](https://tea.qq.com/)，称作IDIP功能。

2. 此功能为游戏预留的后门功能，能在不影响整体游戏运行时，**强行关闭部分游戏功能**。

3. 该功能可能用在游戏上线后突现漏洞、违反部分法规或者策划设计关闭部分游戏功能等。

4. 要在服务器端实现该功能也很简单，只需要在进行相关活动操作时，判断IDIP状态有没有被关闭即可。

5. 在配置表中，有一个**IDIP功能列表**，在**状态**那个属性里面，可配1, 2, 3这三个值，对于服务器端而言，只要状态值不为 1，都表示活动无法正常进行。

### [服务器端代码](#活动中控)

```elixir
def function(act_id, other_parameter, {id, %{periods: periods} = data}) do
  # 在 with 语句中，判断IDIP即可
  with {true, _} <- Periods.opened_with_idip?(act_id, {id, data}) || :idip_closed,
       some_judgement do
    do_something
    {:resolve, context, effect}

  else
    {:notify, events}
  end
end
```

## [数据初始化](#目录)

1. 可以采用封装库**PeriodsAPI.Behavior**中的*init_avatar_act_data/3*，进行数据的初始化，也可以自定义一个函数，在*on_begin*和*on_run*的时候调用。
2. 玩家和服务器的初始数据都是从表中读取，读取时应考虑兼容问题。
3. 如果服务器启动时，活动已经开始了，则需要从Repo中获取，并保存在ETS中。
4. ETS需要进行初始化，初始化函数的调用可放在Main或Periods模块中。

```elixir
defmodule Periods.Some_periods.Ets do
  @ets_name :some_periods

  def init_ets() do
    :ets.info(@ets_name) == :undefined && :ets.new(@ets_name, [:set, :public, :named_table])
    :ets.delete_all_objects(@ets_name)
  end
end

defmodule Periods.Some_periods do
  # 数据初始化
  def init_avatar_act_data(act_id, now_time) do
    parameter =
      do_something

    %AvatarStruct{%AvatarStruct{} | parameter: parameter}
  end

  def avatar_act_data(act_id, periods) do
    case periods |> Map.get(act_id) do
      # init for nil
      nil ->
        init_avatar_act_data(act_id, Utils.timestamp())

      # 业务操作，采用结构体
      act_data ->
        Map.merge(%AvatarStruct{}, act_data)
    end
  end
end

```

## [数据重置](#目录)

数据重置主要分为三种，零点重置、登录重置、在线重置，他们都是活动状态里面的回调函数。

1. 对于**整点重置**而言，目标可以是服务器端数据和玩家数据。
2. 发起重置请求是在**Scheduler**模块中的*am0/0*中进行的，请求在**Avatar**模块中得到处理，并调用了封装库**PeriodsAPI**中的*do_avatar_am0_reset*回调函数。
3. 对于**登录重置**而言，目标则一般针对于玩家数据。
4. 对于**在线重置**而言，一般会配合**功能开启**的回调函数一起使用，目标为玩家数据。
5. 重置时，可通过**Periods.opened?/2**判断活动有没有开启，如果活动已经关闭，则可以清除玩家的活动数据。

```elixir
  # 凌晨在线重置
  def on_avatar_am0_reset(act_id, now_time, {id, %{periods: periods} = data}) do
    case Periods.opened?(act_id, {id, data}) do
      {true, _} ->
        do_something

      _ ->
        {[], %{}}
    end
  end

  # 对在线玩家重置活动数据
  def on_avatar_reset(act_id, {id, %{periods: periods} = data}) do
    {events, changed} =
      case Periods.opened?(act_id, {id, data}) do
        {false, _} ->
          {[], %{}}

        {true, _} ->
          do_something
          {events, changed}
      end

    {:notify, events, changed}
  end

  # 登陆重置
  def on_login_reset(act_id, now_time, {id, %{periods: periods} = data}) do
    case Periods.opened?(act_id, {id, data}) do
      {false, _} ->
        do_something

      {true, _} ->
        do_something
    end
  end

  # 功能开启
  def on_function_unlock(act_id, _now, state) do
    {_, events, changed} = on_avatar_reset(act_id, state)
    {events, changed}
  end
```

## [消息协议](#目录)

对于周期运营活动的socket协议，在**avatar**模块中，有对周期运营活动进行特殊处理。

```elixir
# 客户端请求: ["periods:函数名", act_id, 参数1, 参数2 ...]
# act_id 必须放在参数的首位
defmodule Avatar do
  def handle_cast({{:periods, action}, [act_id | args]}, {id, _session, data} = state)
      when action != :dispatch do
    Periods.dispatch(act_id, action, args, {id, data}) |> handle_result(state)
  end
end

# 在Periods模块中，会通过依赖库 PeriodsAPI 中dispatch函数，来调度到指定的函数进行业务操作
defmodule Periods do
  use PeriodsAPI

  def dispatch(id, action, args \\ [], avatar_state)

  def dispatch(id, action, args, {aid, data}) do
    PeriodsAPI.dispatch(id, action, List.wrap(args) ++ [{aid, data}], :ok)
  end

  def dispatch(id, action, args, _) do
    Logger.warn("unknown dispatch, id:#{id}, action: #{inspect(action)}, args:#{inspect(args)}")
  end
end

# PeriodsAPI
  dispatch(id, action, args, default \\ nil)
  dispatch(
    id :: integer(),
    action :: atom() | String.t(),
    args :: list(),
    default :: nil | any()
  ) :: any()
```

## [依赖库](#目录)

在周期运营活动中，主要会调用如下的封装依赖库：

* DB.RepoAPI (DB泛型API模块)
* PeriodsAPI (周期运营活动模块)
* PeriodsAPI.Behavior (周期运营活动依赖模块)

### [DB.RepoAPI](#依赖库)

```elixir
use DB.RepoAPI, name: 数据表名, format: 存储的字段名, 默认为{:_id, :data}

# 初始化
init() :: :ok

# 关闭
close() :: :ok

# 根据id读取数据
load(id :: integer) :: any

# 根据id读取数据
load(id :: integer, realm_id :: integer) :: any

# 插入数据
insert(rec :: any) :: any

# 插入数据
insert(rec :: any, realm_id :: integer) :: any

# 保存数据
save({id :: integer | String.t(), data :: any}) :: any

# 保存数据
save({id :: integer | String.t(), data :: any}, realm_id :: integer) :: any

# 保存多条数据
save(records :: list) :: any

# 读取全部数据
records() :: list

# 删除单条数据
delete(id :: integer) :: any

# 删除全部数据
clear() :: any

# 执行db函数
perform(type :: nil | :call | :cast, func :: atom, args :: BSON.document()) :: any

# tuple转map
transform(rec :: tuple) :: map

# map转tuple
transform(rec :: map) :: tuple
```

### [PeriodsAPI](#依赖库)

```elixir
# 添加周期运营活动
add_period(id, type, mod, func_id, scope, time, is_has_process?)

# 获取所有周期运营活动id
all_ids()

# 清除所有之前修改的周期运营活动时间信息
clear_local()

# 删除周期运营活动
del_period(id)

# 判断时间戳是否和当前周期运营活动开始结束时间段不同，若活动不存在则返回true
different_act?(id, timestamp)

# 判断时间戳是否和当前周期运营活动开启时间段不同，若活动未开启则返回true
different_stage?(id, timestamp)

# 执行周期运营活动函数
dispatch(id, action, args, default \\ nil)

# 执行周期运营活动函数
dispatch(id, action, args, arg4, default)

# 执行周期运营活动0点的回调函数
do_am0_reset()

# 执行0点重置玩家周期运营活动数据的回调函数
do_avatar_am0_reset(arg)

# 执行功能解锁重置玩家周期运营活动数据的回调函数
do_function_unlock(func_ids, arg)

# 执行玩家登录时检查玩家周期运营活动数据的回调函数
do_login_reset(arg)

# 计算活动开始时间距离时间戳的天数，若活动不存在则返回0
duration_days(id, timestamp \\ CommonAPI.timestamp())

# 根据id获取周期运营活动数据
get_period(id)

# 根据状态获取周期运营活动id
ids_of_status(status)

# 根据类型获取周期运营活动id
ids_of_type(type)

# 判断是否在周期运营活动时间范围内
in_time_scope?(id, time)

# 修改周期运营活动时间，服务器会保存该时间信息
modify_time(id, scope, time)

# 根据id获取周期运营活动是否开启
opened?(id)

# 获取已开启的周期运营活动数据
opened_periods()

# 获取所有周期运营活动数据
periods()

# 根据id删除之前修改的周期运营活动时间信息
remove_local(id)
```

### [PeriodsAPI.Behavior](#依赖库)

```elixir
# 根据活动id获取玩家周期运营活动数据函数
avatar_act_data(id, avatar_periods, default, args)

# 生成同步玩家周期运营活动数据事件函数
avatar_prop_events(id, avatar_act_data, args)

# 全服广播同步周期运营活动信息的回调函数
do_broadcast(type, config)

# 重置玩家周期运营活动数据函数
init_avatar_act_data(id, now_time, args)

# 周期运营活动0点的回调函数
on_am0_reset(id, now_time)

# 0点重置玩家周期运营活动数据的回调函数
on_avatar_am0_reset(id, now_time, {})

# 重置玩家的周期运营活动数据的回调函数
on_avatar_reset(id, {})

# 周期运营活动开始的回调事件
on_begin(id, data)

# 周期运营活动停止的回调事件
on_close(id, data)

# 有单独进程的周期运营活动创建的回调事件
on_create(id, data)

# 功能解锁重置玩家周期运营活动数据的回调函数
on_function_unlock(id, now_time, {})

# 玩家登录时检查玩家周期运营活动数据的回调函数
on_login_reset(id, now_time, {})

# 周期运营活动阶段停止的回调事件
on_pause(id, data)

# 重启服务器时，周期运营活动重新开始的回调事件
on_restart(id, data)

# 周期运营活动阶段开始的回调事件
on_run(id, data)

# 周期运营活动开始显示的回调事件
on_show(id, data)

# 根据活动id获取玩家周期运营活动数据函数
update_avatar_periods(avatar_periods, id, avatar_act_data, args)
```

更多内容，可详见[原网站](http://192.168.1.63/my_library/api-reference.html)。

---
