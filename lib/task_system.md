# 任务系统

## 目录
* [涉及到的玩家数据](#涉及到的玩家数据)
* [流程图](#流程图)
* [各函数功能](#各函数功能)
* [协议及其含义](#协议及其含义)
* [涉及的消息](#涉及的消息)

## 涉及到的玩家数据

```elixir
%{
  can: %{   #存放可以接受的任务的id
    301010047 => 1,
    ......
  },
  #存放每日任务的数据 [当前任务id、当前任务等级, 任务完成数, bit_flag, reset_tick]
  daily: [0, 0, 0, 0, 1570982400],  
  end: %{301010046 => 1},   #存放已完成的任务
  escort: [0, 0, 0, 1570982400],    #护送任务数据
  #第一次每日任务的标识，第一次的20个任务按照特定的顺序进行，获得的奖励的计算方式也不同
  first_daily_task: true,   
  god_daily: [0, 0, 0, 0, 1570982400],  #神之日常任务数据
  ring: [0, 0, 0, 0, 1570982400],   #军团任务数据
  run: %{      #正在进行中的任务
    302010002 => %{
      0 => 0,
      :god_level => 0,  #接受任务时的等级, 用于奖励计算
      :lv => 40,    #接受任务时的等级
      :over => false,   #任务是否完成
      :prog => 256  #条件完成进度(progress, 客户端可不关注) 总条件数 左移8位 与 当前完成条件数
    }
  },
  teamring: [0, 0, 0, 0, 1570982400]    #组队任务数据
}
```

## 流程图

![avatar](/res/task_system.png)

## 各函数功能

TaskMsg.accept(task_id, astate) :调用TaskSystem中的accept函数
TaskMsg.submit(task_id, astate) :调用TaskSystem中的submit函数

TaskSystem.accept(task_id, astate) :先调用can_accept(id, task_map, config)判断是否满足接受任务的条件，再调用mod对应模块的on_accept，再调用TaskSystem.do_accept函数

xxx.on_accept(task_id, config, task_map, astate) :先判断接受条件，在调用do_accept

xxx.do_accept(alv, task_id, config, task_map, astate) :调用update_ex_data(aid, task_id, task_map)跟新数据，利用update_ex_data向客户端发送消息

xxx.update_ex_data(aid, task_id, task_map) :更新模块特性数据(当前数量, 领取标记)

TaskSystem.call(module, action, args) :通过apply调用module模块的action函数

TaskSystem.do_accept(lv, task_id, config, task_map, adata) :更新task_map中的信息，并返回 :notify


TaskSystem.submit(task_id, astate) :通过task_info.over和函数SceneApi.near_npc?(aid, adata, npc)判断是否可以完成该任务。再通过call调用mod对应模块的on_submit函数，再调用TaskSystem.do_submit

xxx.on_submit(task_id, _config, task_map, astate) :通过调用get_next_tasks获取下一个任务的id，再返回

TaskSystem.do_submit(task_id, config, next_tasks, task_map, astate) :先获取信息，再通过call调用mod对应模块的on_get_rewards函数获取任务奖励，再组装{:resolve, ......}

xxx.on_get_rewards(task_id, config, gene, task_map, astate) :通过计算得出获得的奖励（不是直接从配置文件取出）


## 协议及其含义
协议在于告知客户端用户数据的变化。

接受任务：
  1. `[:del_acceptable, aid, task_id]` 在can中删除task_id
  2. `[:update_task, aid, [task_id, run[task_id]]]` 更新run中的信息（将task_id对应的 task_info 添加到了run中，并将这个信息发送给客户端）

完成任务：
  1. `[:del_accepted, aid, task_id, true]` 删除run中 task_id 对应的信息，true表示任务完成
  2. `[:add_acceptable, aid, next_task]` 将下一任务添加到can中
  3. `[:gain_exp, aid, %{exp: xxx}]` 活得多少经验
  4. `[prop_changed, aid, %{xxx}]` 属性的改变
  5. `[{:achievements, :update}, aid, %{xxx}]` 成就

## 涉及的消息

接受任务：
  1. `{:notify, {:task_daily_data, aid, ex_data}}` xxx.update_ex_data/3 函数中利用Router.route 发送了该消息，ex_data是上述数据中 daily: 对应的信息
  2. `{:notify, events, changed}` 所有逻辑处理完之后发送

完成任务：
  1. `{:resolve, %{action: action}, effects}` 在任务配置中有submitBuff 的情况下会发送此消息，获得一个buff
  2. 每日任务、军团任务等可能会在on_get_rewards/5中发送额外的消息
  3. `{:resolve, %{action: {:task, mod}, events: events, changed: changed}, effects}` 正常逻辑处理完成之后发送