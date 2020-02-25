# 服务器端日常活动整体架构

---

<img src="/res/fbmy.png" align="right">

## 目录

* [数据结构](#数据结构)
* [数据存取](#数据存取)
* [IDIP](#idip)
* [数据初始化](#数据初始化)
* [数据重置](#数据重置)
* [相关模块](#相关模块)
* [部分函数](#部分函数)

## [数据结构](#目录)

1. 和周期运营活动的存取模式不太一样，日常活动的数据，目前不需要采用**结构体**的方式进行业务操作

2. 日常活动的游戏数据，一般只有玩家数据，而没有服务器端的数据

3. 对于玩家活动数据，主要存放在玩家数据的**activity_data**中

```elixir
data = %{
  level: 1,
  # 日常活动数据
  activity_data: %{
    act_id_1 => %{}
    # 对于大部分日常活动而言，基本都是副本刷怪类型
    act_id_2 => %{
      entered: entered, # 当日进入次数
      times: times, # 每日免费次数
      bought: bought, # 当日购买次数
      other_parameter: other_parameter
    }
  }
}

# act_id = 1 or 3位活动ID :: integer
```

## [数据存取](#目录)

1. 因为对目前而言，日常活动数据还没有采用结构体的形式进行业务操作，所以直接从data中获取后，进行操作即可

2. 存放活动数据时，应尽可能小心，检查会不会对之前的活动数据造成影响

```elixir
use Stages, [act_id: @act_id, instance_id: nil]

def challenge(target_level, {id, %{activity_data: activity_data}}) do
  with some_judgements do
    try do
      # 此处调用了 Stages 模块中的increment_entered
      # 但Stages只是个转运层，最终实现是在 Activity.Role 模块
      increment_entered(id, activity_data)
      start_instance(id, data, map)
    rescue
      _ -> :ok
    end
  else
    :ok
  end
end
```

## [IDIP](#目录)

### [简介](#idip)

1. 和周期运营活动类似，日常活动也采用了IDIP功能控制。

2. 参见[腾讯游戏接入平台](https://tea.qq.com/)。

3. 此功能为游戏预留的后门功能，能在不影响整体游戏运行时，**强行关闭部分游戏功能**。

4. 该功能可能用在游戏上线后突现漏洞、违反部分法规或者策划设计关闭部分游戏功能等。

5. 要在服务器端实现该功能也很简单，只需要在进行相关活动操作时，判断IDIP状态有没有被关闭即可。

6. 在配置表中，有一个**IDIP功能列表**，在**状态**那个属性里面，可配1, 2, 3这三个值，对于服务器端而言，只要状态值不为 1，都表示活动无法正常进行。

<img src="/res/TIM截图20191122153854.jpg">

### [服务器端代码](#idip)

```elixir
@func_id 1201

def function(some_parameter, {id, %{activity_data: activity_data}}) do
  with Idip.is_func_open?(@func_id),
       some_judgements do
    do_something
  else
    :ok
  end
end
```

## [数据初始化](#目录)

1. 日常活动的游戏数据，最开始依然是在**repo**模块中声明的
2. 在repo中声明时，是没有任何数据的，初始化数据是在**Activity.Role**模块的**init_acts**函数中

```elixir
defmodule Activity.Role do
  # init activity_data
  def init_acts(acts) do
    Cfgs.get_all()
    |> Map.keys()
    |> Enum.reduce(acts, fn (act_id, acc) ->
      if not Map.has_key?(acc, act_id) do
        %{times: times} = Cfgs.get(act_id)
        can_back = Cfgs.can_res_back?(act_id)
        init = init_act(act_id, times, can_back)
        Map.put(acc, act_id, init)
      else
        acc
      end
    end)
  end
end
```

## [数据重置](#目录)

1. 日常活动的数据大部分会在每日零点进行重置，重置操作是在**Activity.Role**模块的**refresh_acts**函数中完成的
2. 零点一般会重置当日的活动进入次数、购买次数以及资源找回次数

```elixir
defmodule Activity.Role do
  def refresh_acts(data, _) do
    # 零点重置
    # 购买次数重置
    # 资源找回重置
  end
end
```

## [相关模块](#目录)

```elixir
# 活动管理上行协议处理
Activities

# Server活动开启管理，Supervisor进程监视器模块
Activity.Mgr.Server

# 活动开启公共配置
Activity.Cfgs

# 活动实例模块映射管理器，管理活动状态
Activity.Modules

# 活动数据处理模块
Activity.Role
```

---
