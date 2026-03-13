# SLS 日志服务操作指南

SLS（Simple Log Service）是 ROA 风格的 API，是阿里云 CLI 最常用的操作场景之一。

## 目录

- [写入日志（PutLogs）](#写入日志putlogs)
- [查询日志（GetLogs）](#查询日志getlogs)
- [常用操作速查](#常用操作速查)
- [时间范围计算](#时间范围计算)

---

## 写入日志（PutLogs）

当用户提供 JSON 数据要写入 SLS 时，需要将其转换为 SLS 日志格式。

### 转换规则

将 JSON 对象的每个字段转为 `{"Key": "字段名", "Value": "字段值"}` 的格式，所有值都转为字符串。嵌套对象和数组序列化为 JSON 字符串。

### 示例

用户提供的原始数据：
```json
{"id": 10234, "name": "张伟", "tags": ["dev", "ops"], "addr": {"city": "杭州"}}
```

转换后的命令：
```bash
aliyun sls PutLogs \
  --logstore <logstore名> \
  --project <project名> \
  --body '[{
    "Time": <当前Unix时间戳>,
    "Contents": [
      {"Key": "id", "Value": "10234"},
      {"Key": "name", "Value": "张伟"},
      {"Key": "tags", "Value": "[\"dev\", \"ops\"]"},
      {"Key": "addr", "Value": "{\"city\": \"杭州\"}"}
    ]
  }]'
```

**获取当前时间戳**：执行 `date +%s` 获取当前 Unix 时间戳，用于 Time 字段。

### 数据转换细节

| 原始类型 | 转换方式 | 示例 |
|---------|---------|------|
| 字符串 | 直接使用 | `"张伟"` → `"张伟"` |
| 数字 | 转为字符串 | `28` → `"28"` |
| 布尔值 | 转为字符串 | `true` → `"true"` |
| null | 转为字符串 | `null` → `"null"` |
| 数组 | JSON 序列化 | `[1,2]` → `"[1, 2]"` |
| 对象 | JSON 序列化 | `{"a":1}` → `"{\"a\": 1}"` |

### 批量写入

PutLogs 支持一次写入多条日志，将多个日志对象放入数组即可：

```bash
aliyun sls PutLogs \
  --logstore my-logstore \
  --project my-project \
  --body '[
    {"Time": 1234567890, "Contents": [{"Key": "msg", "Value": "log1"}]},
    {"Time": 1234567891, "Contents": [{"Key": "msg", "Value": "log2"}]}
  ]'
```

---

## 查询日志（GetLogs）

```bash
aliyun sls GetLogs \
  --logstore <logstore名> \
  --project <project名> \
  --from <起始Unix时间戳> \
  --to <结束Unix时间戳> \
  --query '<查询语句>'
```

### 查询语句示例

```bash
# 简单关键词搜索
--query 'error'

# 字段匹配
--query 'level: ERROR'

# 组合条件
--query 'level: ERROR AND service: payment'

# 统计分析
--query '* | SELECT count(*) as cnt, level GROUP BY level'
```

---

## 常用操作速查

| 操作 | 命令 |
|------|------|
| 列出所有 Project | `aliyun sls ListProject` |
| 列出 Logstore | `aliyun sls ListLogStores --project <name>` |
| 创建 Logstore | `aliyun sls CreateLogStore --project <name> --body '{"logstoreName":"<name>","ttl":30,"shardCount":2}'` |
| 删除 Logstore | `aliyun sls DeleteLogStore --logstore <name> --project <name>` |
| 查看 Shard 列表 | `aliyun sls ListShards --logstore <name> --project <name>` |
| 获取索引配置 | `aliyun sls GetIndex --logstore <name> --project <name>` |

---

## 时间范围计算

```bash
# macOS
from=$(date -v-1H +%s)      # 1小时前
from=$(date -v-1d +%s)      # 1天前
from=$(date -v-7d +%s)      # 7天前

# Linux
from=$(date -d '1 hour ago' +%s)
from=$(date -d '1 day ago' +%s)
from=$(date -d '7 days ago' +%s)

# 当前时间
to=$(date +%s)
```

### 完整查询示例

查询最近 1 小时内的错误日志：

```bash
# macOS
aliyun sls GetLogs \
  --logstore my-logstore \
  --project my-project \
  --from $(date -v-1H +%s) \
  --to $(date +%s) \
  --query 'level: ERROR'

# Linux
aliyun sls GetLogs \
  --logstore my-logstore \
  --project my-project \
  --from $(date -d '1 hour ago' +%s) \
  --to $(date +%s) \
  --query 'level: ERROR'
```
