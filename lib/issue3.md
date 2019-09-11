# 部分玩家无法登陆

**排查流程**：

1. 查看线上日志，发现有报错；
![avatar](/res/TIM截图20190911163634.jpg)

**错误代码**：

```elixir
equip_ids =
  equipments
  |> Map.values
  |> Enum.reduce([], fn args, acc ->  
      cond do
        args != %{} ->
          [Map.get(args, :id) | acc]
        true ->
          acc
      end
    end)
```

**代码分析**：
Map.get(args, :id)未考虑args没有id字段的情况

**修改后代码**：

```elixir
equip_ids =
  equipments
  |> Map.values()
  |> Enum.reduce([], fn
    %{id: id}, acc -> [id | acc]
    _, acc -> acc
  end)
```

**总结**：

1. 这类情况多使用指定匹配+通用匹配；
2. 在读取其他模块数据时，多考虑异常情况，例如没有指定字段、和想要类型不一致等情况；

---
