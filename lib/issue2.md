# 玩家参与9月8日的幻灵小队活动后，出现掉线

**排查流程**：

1. 查看线上日志，发现有报错；
![avatar](/res/TIM截图20190911162818.jpg)

**错误代码**：

```elixir
defp make_guid() do
  System.unique_integer([:positive, :monotonic]) <<< 40 ||| Utils.timestamp()
end
```

**代码分析**：
System.unique_integer([:positive, :monotonic])是程序启动后的递增值，当该值变大后会出现生成的guid在存储mongo时越界；需改变生成guid方式；

**修改后代码**：

```elixir
defp make_guid() do
  DbAgent.next_id("team_checkin")
end
```

**总结**：

1. 以后涉及位移数值的情况，要考虑被移位数值递增后是否会导致数值剧增的问题，也要考虑mongo和redis数值精度的问题；最好被移位数值是固定的，这样可以保证移位后的数值范围可控；

---
