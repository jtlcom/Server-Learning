# 线上运维常用命令

## 服务器执行命令

```bash
# 查询玩家数据
remote_exec_cmd 25001 "/data/realm-KEY/tools/my3d_realm_ctrl.sh rpc 'Debug.avatar_info(id, :mitama)'"
remote_exec_cmd 25001 "/data/realm-KEY/tools/my3d_realm_ctrl.sh rpc 'DbAgentPlayer.load_player(id, vsn) |> get_in([:mitama])'"

# 给玩家发放奖励
remote_exec_cmd 25001 "/data/realm-KEY/tools/my3d_realm_ctrl.sh rpc 'Router.route(id, {:resolve, %{action: :gm}, Effect.from_rewards({:coin, 500})})'"
remote_exec_cmd 25001 "/data/realm-KEY/tools/my3d_realm_ctrl.sh rpc 'Router.route(id, {:resolve, %{}, AsyncEvts.add_effect_evt(Effect.from_rewards({:coin, 500}), id, :gm)})'"

# world,global服执行操作
remote_exec_cmd global "/data/tools/my3d_global_ctrl.sh rpc 'Timex.local'"
```

## 查询TLOG数据

```bash
remote_exec_cmd 25001 "cat /data/log/tlog/serverKEY/tlog_20210310*.log | grep id | grep MoneyFlow"
```

## 服务器热更代码、配置

部分情况下，服务器代码或配置的热更需要由我方完成。基本流程如下：

1. 数值给出需要热更的表名
2. 从打包服(63)出热更包
3. 完成热更包的测试
4. 在打包服中，找到包的路径，解压
5. 解压后，进入beam文件所在目录
6. 上传每个需要热更的beam文件至目标服务器
7. 服务器热更beam文件

```bash
# 解压热更包
tar -zxf realm_hotfix_20210309_md5.tar.gz

# 解压热更包(展示细节)
tar -zxvf realm_hotfix_20210309_md5.tar.gz

# 上传热更包
remote_exec_cmd 25001 "scp Elixir.Periods.WorldGroupBattle.beam"
remote_exec_cmd an "scp Elixir.Periods.WorldGroupBattle.beam"
remote_exec_cmd zs "scp Elixir.Periods.WorldGroupBattle.beam"
remote_exec_cmd douyin "scp Elixir.Periods.WorldGroupBattle.beam"

# 热更模块
remote_exec_cmd 25001 "/data/realm-KEY/tools/update_mods_realm.sh EudemonSoul"
remote_exec_cmd global "/data/tools/update_mods_global.sh Periods.WorldGroupBattle"
remote_exec_cmd world "/data/tools/update_mods_world.sh Debug"
```

---