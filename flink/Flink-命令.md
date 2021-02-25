# Flink-命令

## 启动

```cmd
/baerwang/flink-1.11.3/bin/start-cluster.sh
```

## 暂停

```cmd
/baerwang/flink-1.11.3/bin/stop-cluster.sh
```

## 提交任务

```cmd
/baerwang/flink-1.11.3/bin/flink run -c <入口类> -p <并行度> <jar包路径> <启动参数>
```

## 查看任务-查看已提交的任务

```cmd
/baerwang/flink-1.11.3/bin/flink list
```

## 查看任务-全部

```cmd
/baerwang/flink-1.11.3/bin/flink list -a
```

## 取消任务

```cmd
/baerwang/flink-1.11.3/bin/flink cancel JobId
```

