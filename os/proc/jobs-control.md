
## Bash Jobs

串行执行：先执行 `a` ，等待其返回 `0` 再执行 `b` 。注意，
* `a` 可能启动后台进程，`&&` 无法保证 `a` 的后台进程和 `b` 严格串行。
* `a` 如果返回非 `0`，不会再执行 `b` 

```bash
a && b
```

### background jobs

```bash
command &
pid=$!

jobs -l
wait "$pid"
wait
```

```bash
Ctrl-Z      # 暂停前台任务
bg %1       # 后台继续
fg %1       # 切回前台
kill %1     # 结束 job
```

### find a process

```bash
ps -ef
pstree -p

pgrep -af pattern
pkill -TERM -f pattern

kill -TERM "$pid"
kill -KILL "$pid"   
```

优先 `SIGTERM`，少用 `kill -9`。

### cleanup signals

```bash
pids=()

cleanup() {
    kill "${pids[@]}" 2>/dev/null
    wait "${pids[@]}" 2>/dev/null
}

trap cleanup EXIT INT TERM

worker &
pids+=("$!")
```

## Parallel

```bash
command1 &
command2 &
wait
```

```bash
printf '%s\0' *.jpg | xargs -0 -n1 -P8 convert
```

复杂并行任务使用 GNU Parallel：

```bash
parallel -j8 'convert {} {.}.png' ::: *.jpg
```
