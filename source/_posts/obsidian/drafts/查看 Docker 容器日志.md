### **Docker logs - 查看 Docker 容器日志**

**简介**

Docker logs 命令允许您查看 Docker 容器的日志输出。这对于故障排除、监视和分析容器行为非常有用。

**命令格式**

```
$ docker logs [OPTIONS] CONTAINER
```

**选项**

| 选项                 | 描述                                                         |
| -------------------- | ------------------------------------------------------------ |
| `--details`          | 显示更多信息                                                 |
| `-f`, `--follow`     | 跟踪实时日志                                                 |
| `--since string`     | 显示自某个时间戳之后的日志，或相对时间，如 42m（即 42 分钟） |
| `--tail string`      | 从日志末尾显示多少行日志，默认是 all                         |
| `-t`, `--timestamps` | 显示时间戳                                                   |
| `--until string`     | 显示自某个时间戳之前的日志，或相对时间，如 42m（即 42 分钟） |

**示例**

**查看指定时间后的日志，只显示最后 100 行：**

```
$ docker logs -f -t --since="2018-02-08" --tail=100 CONTAINER_ID
```

**查看最近 30 分钟的日志：**

```
$ docker logs --since 30m CONTAINER_ID
```

**查看某时间之后的日志：**

```
$ docker logs -t --since="2018-02-08T13:23:37" CONTAINER_ID
```

**查看某时间段日志：**

```
$ docker logs -t --since="2018-02-08T13:23:37" --until "2018-02-09T12:23:37" CONTAINER_ID
```

**高级用法**

**过滤日志输出**

您可以使用 `--filter` 选项过滤日志输出。例如，要仅显示包含特定字符串的日志行，您可以使用以下命令：

```
$ docker logs --filter "string=my-string" CONTAINER_ID
```

**重定向日志输出**

您可以使用 `--output` 选项将日志输出重定向到文件或其他命令。例如，要将日志输出重定向到文件，您可以使用以下命令：

```
$ docker logs --output=file.txt CONTAINER_ID
```

**故障排除**

**无法查看日志输出**

如果您无法查看日志输出，请确保您具有查看容器日志的权限。您还可以尝试使用 `--details` 选项查看更多信息。

**日志输出太长**

如果您需要查看大量日志输出，可以使用 `--tail` 选项指定要显示的日志行数。