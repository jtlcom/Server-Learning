# 内网常用命令

## 238服务器

### realm服

```bash
# 启动
docker start develop
docker start release

# 重启
docker restart develop
docker restart release

# 进入console，退出依次按 ctrl+p + ctrl+q
docker attach develop
docker attach release
```

### mongo

```bash
# 启动(for a long time)
docker start mongo

# 重启
docker restart mongo
```

### couch_diff

```bash
# 启动
forever start /home/projects/couch_diff/server.js
```

### 刷表工具

```bash
# 启动
forever start /home/projects/config-sync/src/main.js
```

---
