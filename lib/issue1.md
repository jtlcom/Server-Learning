# 玩家完成神劫活动后，幻兽丢失

**排查流程**：

1. 查看线上日志，并无报错；
2. 查看神劫活动代码，有操作幻兽数据；

**错误代码**：

```elixir
def recover_eudemonds_living(rate, {id, _session, data}) do
  now_time = System.system_time(:millisecond)
  # fixme acc的初始值为%{}，导致之前的幻兽数据丢失
  new_eudemons = Map.get(data, :eudemons, %{}) |> Enum.reduce(%{}, fn {index, eud}, acc ->
    health = get_eud_health(eud)
    hp = Map.get(eud, :hp, 0)
    if is_dead?(eud) && (data.points.hp > 0) do
      acc   # fixme
    else
      hp1 = (hp + health * rate / 100) |> trunc |> min(health)
      new_eud = eud |> Map.put(:hp, hp1) |> Map.put(:last_recover_time, now_time)
      ## notify events
      if Map.has_key?(eud, :slot) do
        Router.route(data.scene, {:eudemon_prop_changed, id, eud.slot, %{hp: hp1}})
      end
      Map.put(acc, index, new_eud)
    end
  end)
  %{data | eudemons: new_eudemons}
end
```

**代码分析**：

代码里reduce的初始值设置为了%{}，而当玩家未死亡且幻兽已死亡时，直接将该幻兽忽略了，未加入到acc里，这才导致了幻兽丢失；只需将reduce的初始值设置为玩家当前幻兽数据即可；

**修改后代码**：

```elixir
def recover_eudemonds_living(rate, {id, _session, data}) do
  now_time = System.system_time(:millisecond)

  prev_eudemons = Map.get(data, :eudemons, %{})
  new_eudemons = prev_eudemons |> Enum.reduce(prev_eudemons, fn {index, eud}, acc ->

    health = get_eud_health(eud)
    hp = Map.get(eud, :hp, 0)
    if is_dead?(eud) && (data.points.hp > 0) do
      acc
    else
      hp1 = (hp + health * rate / 100) |> trunc |> min(health)
      new_eud = eud |> Map.put(:hp, hp1) |> Map.put(:last_recover_time, now_time)
      ## notify events
      if Map.has_key?(eud, :slot) do
        Router.route(data.scene, {:eudemon_prop_changed, id, eud.slot, %{hp: hp1}})
      end
      Map.put(acc, index, new_eud)
    end
  end)
  %{data | eudemons: new_eudemons}
end
```

**总结**：

1. 以后直接操作玩家原有数据时，要多次检查代码逻辑以及确认修改后的值；
2. 特殊处理的需求，要求策划写入功能文档，提redmine需求单，通知测试增加测试用例，确保测试用例能覆盖到；
3. 代码有条件分支的地方，尽量和测试同步信息，确保测试用例能覆盖到；
4. 后续增加功能影响数据检查方式，例如比较并输出前后玩家数据差异；

---
