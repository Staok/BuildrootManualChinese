## 18.5. `SNNfoo` 启动脚本

提供系统守护进程（daemon）的软件包通常需要在启动时以某种方式启动。Buildroot 支持多种初始化系统（init system），其中一些被视为一级支持（参见[第 6.3 节，“init system”](https://buildroot.org/downloads/manual/manual.html#init-system)），其他也可用但集成度不高。理想情况下，所有提供系统守护进程的软件包都应为 BusyBox/SysV init 提供启动脚本，并为 systemd 提供 unit 文件。

为保持一致性，启动脚本必须遵循参考样例 `package/busybox/S01syslogd` 的风格和结构。下面给出了这种风格的带注释示例。systemd unit 文件没有特定的编码风格，但如果软件包自带 unit 文件且兼容 buildroot，则优先使用。

启动脚本的命名由 `SNN` 和守护进程名组成。`NN` 是启动顺序号，需要谨慎选择。例如，需要网络服务的程序不能在 `S40network` 之前启动。脚本按字母顺序启动，因此 `S01syslogd` 会在 `S01watchdogd` 之前启动，`S02sysctl` 随后启动。

```sh
#!/bin/sh

DAEMON="syslogd"
PIDFILE="/var/run/$DAEMON.pid"

SYSLOGD_ARGS=""

# shellcheck source=/dev/null
[ -r "/etc/default/$DAEMON" ] && . "/etc/default/$DAEMON"

# BusyBox 的 syslogd 不会创建 pidfile，因此命令行需传递 "-n"，并用 "--make-pidfile" 让 start-stop-daemon 创建 pidfile。
start() {
        printf 'Starting %s: ' "$DAEMON"
        # shellcheck disable=SC2086 # 需要参数分割
        start-stop-daemon --start --background --make-pidfile \
                --pidfile "$PIDFILE" --exec "/sbin/$DAEMON" \
                -- -n $SYSLOGD_ARGS
        status=$?
        if [ "$status" -eq 0 ]; then
                echo "OK"
        else
                echo "FAIL"
        fi
        return "$status"
}

stop() {
        printf 'Stopping %s: ' "$DAEMON"
        start-stop-daemon --stop --pidfile "$PIDFILE" --exec "/sbin/$DAEMON"
        status=$?
        if [ "$status" -eq 0 ]; then
                echo "OK"
        else
                echo "FAIL"
                return "$status"
        fi
        while start-stop-daemon --stop --test --quiet --pidfile "$PIDFILE" \
                --exec "/sbin/$DAEMON"; do
                sleep 0.1
        done
        rm -f "$PIDFILE"
        return "$status"
}

restart() {
        stop
        start
}

case "$1" in
        start|stop|restart)
                "$1";;
        reload)
                # 没有真正的 "reload" 功能，直接重启。
                restart;;
        *)
                echo "Usage: $0 {start|stop|restart|reload}"
                exit 1
esac
```

脚本应尽量使用长选项以增强可读性。

### 18.5.1. 启动脚本配置

启动脚本和 unit 文件都可以从 `/etc/default/foo`（其中 `foo` 为 DAEMON 变量指定的守护进程名）读取命令行参数。一般来说，如果该文件不存在，不应阻止守护进程启动，除非有必须的站点特定参数。对于启动脚本，可以在脚本中为 `FOO_ARGS="-s -o -m -e -args"` 设定默认值，用户可通过 `/etc/default/foo` 覆盖。

### 18.5.2. PID 文件处理

PID 文件用于跟踪服务主进程。如何处理取决于服务是否自己创建 PID 文件，以及是否在关闭时删除。

- 如果服务不会自己创建 PID 文件，请以前台模式启动守护进程，并用 `start-stop-daemon --make-pidfile --background` 让 `start-stop-daemon` 创建 PID 文件。参见 `S01syslogd` 示例：

  ```sh
  start-stop-daemon --start --background --make-pidfile \
          --pidfile "$PIDFILE" --exec "/sbin/$DAEMON" \
          -- -n $SYSLOGD_ARGS
  ```

- 如果服务会自己创建 PID 文件，请将 `--pidfile` 选项同时传递给 `start-stop-daemon` **和守护进程本身**（或在配置文件中设置），确保两者一致。参见 `S45NetworkManager` 示例：

  ```sh
  start-stop-daemon --start --pidfile "$PIDFILE" \
          --exec "/usr/sbin/$DAEMON" \
          -- --pid-file="$PIDFILE" $NETWORKMANAGER_ARGS
  ```

- 如果服务在关闭时会删除 PID 文件，请在停止时用循环检测 PID 文件是否已消失，参见 `S45NetworkManager` 示例：

  ```sh
  while [ -f "$PIDFILE" ]; do
          sleep 0.1
  done
  ```

- 如果服务在关闭时不会删除 PID 文件，请用 `start-stop-daemon` 循环检测服务是否仍在运行，进程结束后再删除 PID 文件。参见 `S01syslogd` 示例：

  ```sh
  while start-stop-daemon --stop --test --quiet --pidfile "$PIDFILE" \
          --exec "/sbin/$DAEMON"; do
          sleep 0.1
  done
  rm -f "$PIDFILE"
  ```

  注意 `--test` 标志，它会让 `start-stop-daemon` 实际上不停止服务，而是测试是否可以停止，如果服务未运行则测试失败。

### 18.5.3. 停止服务

stop 函数在返回前应检查守护进程是否真正退出，否则 restart 可能失败，因为新实例可能在旧实例未完全退出前启动。具体做法取决于服务 PID 文件的处理方式（见上文）。建议所有 `start-stop-daemon` 命令都加上 `--exec "/sbin/$DAEMON"`，确保信号只发给与 `$DAEMON` 匹配的 PID。

### 18.5.4. 重新加载服务配置

支持某种方式重新加载配置（如 `SIGHUP`）的程序应提供类似于 stop() 的 reload() 函数。`start-stop-daemon` 支持 `--stop --signal HUP`。发送信号时请务必使用符号名而非信号编号，因为信号编号在不同 CPU 架构间可能不同，符号名也更易读。

### 18.5.5. 返回码

启动脚本的动作函数应返回成功（或失败）码，通常为相关 `start-stop-daemon` 操作的返回码。最后一个动作的返回码应作为整个启动脚本的返回码，便于自动检测成功与否（如被其他脚本调用时）。注意如果没有显式 `return`，脚本或函数的最后一条命令的返回码就是其返回码，因此并非总需要显式 return。

### 18.5.6. 日志

当服务进程 fork 到后台，或 `start-stop-daemon --background` 这样做时，stdout 和 stderr 通常会被关闭，服务写到这些的日志信息会丢失。建议尽可能配置服务日志到 syslog（优先），或专用日志文件。
