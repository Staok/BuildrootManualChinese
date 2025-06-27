## 18.5. The `SNNfoo` start script

Packages that provide a system daemon usually need to be started somehow at boot. Buildroot comes with support for several init systems, some are considered tier one (see [Section 6.3, “init system”](https://buildroot.org/downloads/manual/manual.html#init-system)), while others are also available but do not have the same level of integration. Ideally, all packages providing a system daemon should provide a start script for BusyBox/SysV init and a systemd unit file.

For consistency, the start script must follow the style and composition as shown in the reference: `package/busybox/S01syslogd`. An annotated example of this style is shown below. There is no specific coding style for systemd unit files, but if a package comes with its own unit file, that is preferred over a buildroot specific one, if it is compatible with buildroot.

The name of the start script is composed of the `SNN` and the daemon name. The `NN` is the start order number which needs to be carefully chosen. For example, a program that requires networking to be up should not start before `S40network`. The scripts are started in alphabetical order, so `S01syslogd` starts before `S01watchdogd`, and `S02sysctl` starts thereafter.

```
#!/bin/sh

DAEMON="syslogd"
PIDFILE="/var/run/$DAEMON.pid"

SYSLOGD_ARGS=""

# shellcheck source=/dev/null
[ -r "/etc/default/$DAEMON" ] && . "/etc/default/$DAEMON"

# BusyBox' syslogd does not create a pidfile, so pass "-n" in the command line
# and use "--make-pidfile" to instruct start-stop-daemon to create one.
start() {
        printf 'Starting %s: ' "$DAEMON"
        # shellcheck disable=SC2086 # we need the word splitting
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
                # Restart, since there is no true "reload" feature.
                restart;;
        *)
                echo "Usage: $0 {start|stop|restart|reload}"
                exit 1
esac
```

Scripts should use long form options where possible for clarity.

### 18.5.1. Start script configuration

Both start scripts and unit files can source command line arguments from `/etc/default/foo`, where `foo` is the daemon name as set in the `DAEMON` variable. In general, if such a file does not exist it should not block the start of the daemon, unless there is some site specific command line argument the daemon requires to start. For start scripts `FOO_ARGS="-s -o -m -e -args"` can be defined to a default value in the script, and the user can override this from `/etc/default/foo`.

### 18.5.2. Handling the PID file

A PID file is needed to keep track of what the main process of a service is. How to handle it depends on whether the service creates its own PID file, and if it deletes it on shutdown.

- If your service doesn’t create its own PID file, invoke the daemon in foreground mode, and use `start-stop-daemon --make-pidfile --background` to let `start-stop-daemon` create the PID file. See `S01syslogd` for example:

  ```
  start-stop-daemon --start --background --make-pidfile \
          --pidfile "$PIDFILE" --exec "/sbin/$DAEMON" \
          -- -n $SYSLOGD_ARGS
  ```

- If your service creates its own PID file, pass the `--pidfile` option to both `start-stop-daemon` ***\*and the daemon itself\**** (or set it appropriately in a configuration file, depending on what the daemon supports) so they agree on where the PID file is. See `S45NetworkManager` for example:

  ```
  start-stop-daemon --start --pidfile "$PIDFILE" \
          --exec "/usr/sbin/$DAEMON" \
          -- --pid-file="$PIDFILE" $NETWORKMANAGER_ARGS
  ```

- If your service removes its PID file on shutdown, use a loop testing that the PID file has disappeared on stop, see `S45NetworkManager` for example:

  ```
  while [ -f "$PIDFILE" ]; do
          sleep 0.1
  done
  ```

- If your service doesn’t remove its PID file on shutdown, use a loop with `start-stop-daemon` checking if the service is still running, and delete the PID file after the process is gone. See `S01syslogd` for example:

  ```
  while start-stop-daemon --stop --test --quiet --pidfile "$PIDFILE" \
          --exec "/sbin/$DAEMON"; do
          sleep 0.1
  done
  rm -f "$PIDFILE"
  ```

  Note the `--test` flag, which tells `start-stop-daemon` to not actually stop the service, but test if it would be possible to, which fails if the service is not running.

### 18.5.3. Stopping the service

The stop function should check that the daemon process is actually gone before returning, otherwise restart might fail because the new instance is started before the old one has actually stopped. How to do that depends on how the PID file for the service is handled (see above). It is recommended to always append `--exec "/sbin/$DAEMON"` to all `start-stop-daemon` commands to ensure signals are sent to a PID that matches `$DAEMON`.

### 18.5.4. Reloading service configuration

Programs that support reloading their configuration in some fashion (e.g. `SIGHUP`) should provide a `reload()` function similar to `stop()`. The `start-stop-daemon` command supports `--stop --signal HUP` for this. When sending signals this way, whether SIGHUP or others, make sure to use the symbolic names and not signal numbers. Signal numbers can vary between CPU architectures, and names are also easier to read.

### 18.5.5. Return codes

The action functions of the start script should return a success (or failure) code, usually the return code of the relevant start-stop-daemon action. The last one of those should be the return code of the start script as a whole, to allow automatically checking for success, e.g. when calling the start script from other scripts. Note that without an explicit `return` the return code of the last command in a script or function becomes its return code, so an explicit return is not always necessary.

### 18.5.6. Logging

When a service forks to the background, or `start-stop-daemon --background` does that, stdout and stderr are generally closed, and anything log messages the service may write there get lost. If possible, configure your service to log to syslog (preferably), or a dedicated log file.