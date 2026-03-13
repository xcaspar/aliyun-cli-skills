---
name: aliyun-cli-skills
description: 通过自然语言自主操作阿里云 CLI，将用户的自然语言指令转换为阿里云CLI命令并执行。支持 ECS、SLS、RDS、OSS、SLB 等 100+ 云产品的增删改查操作。当用户提到"阿里云"、"aliyun"、"云服务器"、"日志服务"、"SLS"、"ECS"、"OSS"、"写入日志"、"查询日志"、"创建实例"、"管理云资源"等与阿里云相关的操作时触发。即使用户没有明确说"阿里云CLI"，只要涉及阿里云产品的命令行操作，都应使用此 skill。当用户询问"如何安装阿里云CLI"、"怎么更新aliyun命令"、"卸载CLI"等安装相关问题时也应触发。
---

# 阿里云 CLI 自主操作

接收用户的自然语言指令，将其转换为正确的阿里云 CLI 命令并执行。核心目标是让用户无需记忆命令语法，用日常语言描述需求即可完成云资源操作。

## 工作流程

### 第一步：理解用户意图

从用户的自然语言中提取关键信息：
- **目标产品**：用户操作的云产品（如 SLS、ECS、RDS、OSS 等）
- **操作类型**：增（创建/写入）、删（删除）、改（更新/修改）、查（查询/列出）
- **资源标识**：项目名、实例名、Logstore 名、Bucket 名等
- **操作数据**：要写入或修改的具体数据内容
- **附加条件**：地域、时间范围、过滤条件等

### 第二步：探索 API

如果不确定该使用哪个 API 或参数格式，通过 `--help` 逐层探索：

```bash
# 查看产品支持的所有 API
aliyun <ProductCode> --help

# 查看具体 API 的参数详情
aliyun <ProductCode> <APIName> --help
```

这一步很重要，因为不同产品的 API 风格（RPC 或 ROA）和参数要求各不相同。先查帮助再执行，可以避免参数错误。

### 第三步：构建并执行命令

根据探索到的 API 信息，构建正确的命令并执行。

## 命令结构

阿里云 CLI 有两种 API 风格，产品内通常统一使用一种风格：

### RPC 风格（大多数产品，如 ECS、RDS）

```bash
aliyun <ProductCode> <APIName> --参数名 '参数值'
```

**示例：**
```bash
aliyun ecs DescribeInstances --RegionId cn-hangzhou
aliyun ecs DescribeInstanceAttribute --InstanceId 'i-uf6f5trc95ug8t33****'
```

### ROA 风格（如 SLS、CS）

```bash
aliyun <ProductCode> <Method> <PathPattern> --body '...'
# 或使用语法糖：
aliyun <ProductCode> <APIName> --参数名 值 --body '...'
```

**示例：**
```bash
# 使用 APIName 语法糖（推荐，更简洁）
aliyun sls PutLogs --logstore my-logstore --project my-project --body '<JSON数据>'

# 使用原始 Method + PathPattern
aliyun sls POST /logstores/my-logstore/shards/lb --project my-project --body '<JSON数据>'
```

## 参数格式（macOS/Linux 环境）

参数名严格区分大小写。

- **普通字符串**：无特殊字符可直接传入，有特殊字符用单引号包裹
- **JSON 参数**：用单引号包裹整个 JSON，内部用双引号
- **JSON 数组**：`'["value1","value2"]'`
- **JSON 对象列表**：`'[{"key":"value"},{"key":"value"}]'`
- **日期时间**：ISO8601 格式 `YYYY-MM-DDThh:mm:ssZ`
- **特殊字符问题**：若引号包含后仍报错，尝试 `--参数名=值` 格式

## 常用产品操作速查

### ECS 云服务器（RPC 风格）

```bash
# 查询实例列表
aliyun ecs DescribeInstances --RegionId cn-hangzhou

# 启动实例
aliyun ecs StartInstance --InstanceId 'i-xxxxx'

# 停止实例
aliyun ecs StopInstance --InstanceId 'i-xxxxx'

# 查询地域列表
aliyun ecs DescribeRegions
```

### OSS 对象存储

阿里云 CLI 内置了 OSS 专用命令（不同于 API 调用）：

```bash
# 列出 Bucket
aliyun oss ls

# 上传文件
aliyun oss cp <local-file> oss://<bucket>/<path>

# 下载文件
aliyun oss cp oss://<bucket>/<path> <local-file>

# 列出对象
aliyun oss ls oss://<bucket>/<prefix>
```

### RDS 云数据库（RPC 风格）

```bash
# 查询实例列表
aliyun rds DescribeDBInstances --RegionId cn-hangzhou

# 查询数据库列表
aliyun rds DescribeDBInstanceAttribute --DBInstanceId 'rm-xxxxx'
```

## 常用命令行选项

| 选项 | 说明 |
|------|------|
| `--region` | 指定地域，如 `cn-hangzhou`、`cn-beijing` |
| `--profile` | 指定凭证配置名称 |
| `--output cols=<字段>` | 表格化输出 |
| `--dryrun` | 模拟执行，不实际操作 |
| `--force` | 强制调用（绕过元数据检查） |
| `--body-file <path>` | 从文件读取请求体（ROA风格） |

## 执行原则

1. **先探索后执行**：对不熟悉的 API，先用 `--help` 查看参数要求
2. **危险操作确认**：执行删除、停止、修改类操作前，先告知用户操作内容和影响范围，获得确认后再执行
3. **使用 dryrun 验证**：对复杂或高风险操作，先用 `--dryrun` 预览请求内容
4. **错误处理**：命令执行失败时，分析错误信息并给出修复建议。常见问题包括参数格式不正确、权限不足、资源不存在等
5. **批量操作**：处理多条数据时，考虑合并为一次 API 调用（如 PutLogs 支持一次写入多条日志），提升效率
6. **结果解读**：执行完成后，用自然语言向用户解释返回结果的含义

## 参考文档

- [安装、更新、卸载](references/installation-guide.md) - 安装、更新、卸载 aliyun cli
- [SLS 日志服务操作指南](references/sls-guide.md) - 写入日志、查询日志、数据转换规则
