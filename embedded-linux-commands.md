# 嵌入式 Linux 常用命令速查

按**用途分类**，同类内大致按日常使用频率从前到后排列（不标注频率星级）。偏板端 shell / 调试，不绑定某一具体 SDK。

每条给出**用途 + 最小用法示例**；`[]` 表示可选项，`<名>` 表示需替换的参数。

---

## 1. 目录与文件

| 命令 | 用途 | 用法示例 |
|------|------|----------|
| `ls` | 列出文件 | `ls`　`ls -l`　`ls -la /etc` |
| `cd` | 切换目录 | `cd /var/log`　`cd ..`　`cd -`（回上一目录） |
| `pwd` | 当前路径 | `pwd` |
| `cat` | 查看/拼接文本 | `cat a.txt`　`cat a.txt b.txt > c.txt` |
| `less` / `more` | 分页阅读 | `less big.log`（`q` 退出，`/` 搜索） |
| `head` / `tail` | 看头/尾 | `head -n 20 f`　`tail -n 50 f`　`tail -f f`（跟日志） |
| `cp` | 复制 | `cp a.txt /tmp/`　`cp -r dir1 dir2` |
| `mv` | 移动/重命名 | `mv old.txt new.txt`　`mv f /tmp/` |
| `rm` | 删除 | `rm f`　`rm -r dir`（慎用 `rm -rf`） |
| `mkdir` / `rmdir` | 建/删空目录 | `mkdir -p a/b/c`　`rmdir empty_dir` |
| `touch` | 建空文件/刷新时间 | `touch new.txt` |
| `find` | 查找文件 | `find / -name "*.ko"`　`find . -type f -mtime -1` |
| `tree` | 树状目录 | `tree -L 2 /etc`（需安装） |
| `du` | 目录占用 | `du -sh *`　`du -h --max-depth=1` |
| `df` | 分区空间 | `df -h` |
| `ln -s` | 软链接 | `ln -s /real/path link_name` |
| `readlink` | 解析链接 | `readlink link`　`readlink -f link` |
| `file` | 文件类型/架构 | `file ./app`　`file /bin/ls` |
| `stat` | 文件元信息 | `stat /etc/passwd` |
| `chmod` | 改权限 | `chmod 755 run.sh`　`chmod +x run.sh` |
| `chown` | 改所有者 | `chown root:root f`　`chown -R user:group dir` |
| `sync` | 刷写缓存到盘 | `sync` |

---

## 2. 文本检索与编辑

| 命令 | 用途 | 用法示例 |
|------|------|----------|
| `grep` | 搜字符串 | `grep "error" log`　`grep -rn "TODO" .`　`grep -i foo f` |
| `grep -E` / `egrep` | 扩展正则 | `grep -E "err\|fail" log` |
| `sed` | 替换/过滤行 | `sed -n '10,20p' f`　`sed 's/old/new/g' f` |
| `awk` | 按列处理 | `awk '{print $1,$3}' f`　`awk -F: '{print $1}' /etc/passwd` |
| `wc` | 计数 | `wc -l f`　`wc -c f` |
| `diff` | 比差异 | `diff a b`　`diff -u a b` |
| `sort` / `uniq` | 排序去重 | `sort f \| uniq`　`sort -n f` |
| `cut` / `tr` | 切列/换字符 | `cut -d: -f1 /etc/passwd`　`tr 'a-z' 'A-Z' < f` |
| `vi` / `vim` | 编辑器 | `vi f`（`i` 编辑，`Esc`，`:wq` 保存退出，`:q!` 弃改） |
| `nano` | 简易编辑 | `nano f`（`Ctrl+O` 存，`Ctrl+X` 退出） |
| `hexdump` / `xxd` | 十六进制查看 | `hexdump -C bin \| head`　`xxd bin \| head` |

---

## 3. 进程与系统状态

| 命令 | 用途 | 用法示例 |
|------|------|----------|
| `ps` | 进程列表 | `ps`　`ps aux`　`ps -ef \| grep ssh` |
| `top` / `htop` | 实时资源 | `top`（`q` 退出）　`htop` |
| `kill` | 结束进程 | `kill 1234`　`kill -9 1234` |
| `killall` | 按名结束 | `killall myapp` |
| `pidof` / `pgrep` | 查 PID | `pidof myapp`　`pgrep -a ssh` |
| `nice` / `renice` | 优先级 | `nice -n 10 ./job`　`renice -n 5 -p 1234` |
| `uptime` | 运行时间/负载 | `uptime` |
| `free` | 内存 | `free -h` |
| `uname` | 内核/机器 | `uname -a` |
| `/proc` | 内核导出信息 | `cat /proc/cpuinfo`　`cat /proc/meminfo`　`cat /proc/version` |
| `dmesg` | 内核日志 | `dmesg \| tail`　`dmesg -w`　`dmesg \| grep usb` |
| `lsof` | 占用者 | `lsof /path`　`lsof -i :80`（需安装） |
| `strace` | 跟踪系统调用 | `strace -e openat ./app`　`strace -p 1234` |
| `time` | 测耗时 | `time ./app` |

---

## 4. 设备、驱动与硬件节点

| 命令 | 用途 | 用法示例 |
|------|------|----------|
| `ls /dev` | 设备节点 | `ls /dev`　`ls /dev/tty*`　`ls /dev/video*` |
| `lsmod` | 已加载模块 | `lsmod` |
| `insmod` / `rmmod` | 按路径装/卸 `.ko` | `insmod ./hello.ko`　`rmmod hello` |
| `modprobe` | 按名装/卸（含依赖） | `modprobe xxx`　`modprobe -r xxx` |
| `lsusb` | USB 列表 | `lsusb`　`lsusb -v`（需 usbutils） |
| `lspci` | PCI 列表 | `lspci`（需 pciutils） |
| `i2cdetect` | 扫 I2C | `i2cdetect -y 0`（总线号按板子） |
| `i2cget` / `i2cset` | I2C 读写 | `i2cget -y 0 0x50 0x00`　`i2cset -y 0 0x50 0x00 0x12` |
| `gpiodetect` 等 | GPIO（libgpiod） | `gpiodetect`　`gpioinfo gpiochip0`　`gpioset gpiochip0 12=1` |
| sysfs 读 | 看类设备状态 | `ls /sys/class/leds`　`cat /sys/class/net/eth0/address` |
| sysfs 写 | 控制（需权限） | `echo 1 > /sys/class/leds/xxx/brightness` |
| `udevadm` | udev | `udevadm info -a -n /dev/sda`　`udevadm monitor` |
| `hwclock` | RTC | `hwclock -r`　`hwclock -w`（系统时间写入 RTC） |

---

## 5. 网络

| 命令 | 用途 | 用法示例 |
|------|------|----------|
| `ip` | 现代网络配置 | `ip addr`　`ip link set eth0 up`　`ip route` |
| `ifconfig` / `route` | 旧式（BusyBox 常见） | `ifconfig eth0`　`ifconfig eth0 192.168.1.10`　`route -n` |
| `ping` | 连通性 | `ping -c 4 8.8.8.8`　`ping gateway` |
| `ping6` | IPv6 | `ping6 -c 4 fe80::1%eth0` |
| `ss` / `netstat` | 端口连接 | `ss -tlnp`　`netstat -an` |
| `wget` / `curl` | 下载/HTTP | `wget URL`　`curl -I URL`　`curl -O URL` |
| `scp` | 远程拷文件 | `scp file root@192.168.1.10:/tmp/` |
| `ssh` | 远程登录 | `ssh root@192.168.1.10` |
| `nc` | 端口/传数据 | `nc -zv 192.168.1.10 22`　`nc -l -p 1234` |
| `tcpdump` | 抓包 | `tcpdump -i eth0 -n`　`tcpdump port 80 -w a.pcap` |
| `iperf3` | 吞吐 | 一端 `iperf3 -s`，另一端 `iperf3 -c <ip>` |
| `arp` / `arping` | ARP | `arp -n`　`arping -I eth0 192.168.1.1` |
| `hostname` | 主机名 | `hostname`　`hostname newname` |
| `iptables` / `nft` | 防火墙 | `iptables -L -n`　`nft list ruleset` |

配置地址示例（`ip`）：

```bash
ip addr add 192.168.1.10/24 dev eth0
ip link set eth0 up
ip route add default via 192.168.1.1
```

---

## 6. 存储、分区与文件系统

| 命令 | 用途 | 用法示例 |
|------|------|----------|
| `mount` / `umount` | 挂载/卸载 | `mount /dev/sda1 /mnt`　`umount /mnt` |
| `lsblk` | 块设备树 | `lsblk`　`lsblk -f` |
| `fdisk` / `parted` | 分区（慎用） | `fdisk -l`　`fdisk /dev/sdX` |
| `mkfs.ext4` 等 | 格式化（慎用） | `mkfs.ext4 /dev/sdX1`　`mkfs.vfat /dev/sdX1` |
| `fsck` | 检查修复 | `fsck /dev/sdX1`（先卸载） |
| `dd` | 块拷贝（极危险） | `dd if=image.of of=/dev/sdX bs=4M status=progress` |
| `blkid` | UUID/类型 | `blkid`　`blkid /dev/sda1` |
| `sync` | 刷盘 | `sync` |
| `losetup` | loop 设备 | `losetup -fP img`　`losetup -d /dev/loop0` |

挂载示例：

```bash
mkdir -p /mnt/usb
mount /dev/sda1 /mnt/usb
# 用完
umount /mnt/usb
```

---

## 7. 用户、权限与服务

| 命令 | 用途 | 用法示例 |
|------|------|----------|
| `whoami` / `id` | 当前身份 | `whoami`　`id` |
| `su` / `sudo` | 切换/提权 | `su -`　`sudo command` |
| `passwd` | 改密码 | `passwd`　`passwd username` |
| `useradd` 等 | 用户管理 | `useradd -m u`　`usermod -aG sudo u`　`userdel -r u` |
| `systemctl` | systemd 服务 | `systemctl status ssh`　`systemctl start\|stop\|restart\|enable xxx` |
| `service` | SysV/BusyBox | `service xxx start`　`service xxx status` |
| `reboot` 等 | 重启关机 | `reboot`　`poweroff`　`halt` |

---

## 8. 编译、开发与调试

| 命令 | 用途 | 用法示例 |
|------|------|----------|
| `gcc` / 交叉 `*-gcc` | 编译 | `gcc -o app app.c`　`aarch64-none-linux-gnu-gcc -o app app.c` |
| `make` | 构建 | `make`　`make -j4`　`make clean`　`make install` |
| `cmake` | 生成工程 | `cmake -S . -B build && cmake --build build` |
| `gdb` | 调试 | `gdb ./app`　`gdb-multiarch ./app`（板端 ELF 在主机上） |
| `objdump` | 反汇编 | `objdump -d app \| less` |
| `readelf` | 看 ELF | `readelf -h app`　`readelf -d app` |
| `nm` | 符号表 | `nm app` |
| `ldd` | 动态库依赖 | `ldd ./app`（须与目标架构匹配的 ldd） |
| `strip` | 去符号 | `strip app` |
| `ar` | 静态库 | `ar rcs libx.a a.o b.o` |
| `pkg-config` | 编译参数 | `pkg-config --cflags --libs libfoo` |

`gdb` 内常用：`break main`　`run`　`next`　`step`　`print x`　`backtrace`　`quit`

---

## 9. 打包、压缩与传输

| 命令 | 用途 | 用法示例 |
|------|------|----------|
| `tar` | 打包解包 | `tar czf a.tar.gz dir`　`tar xzf a.tar.gz`　`tar tzf a.tar.gz` |
| `gzip` / `xz` | 压缩 | `gzip f`　`gunzip f.gz`　`xz -k f` |
| `zip` / `unzip` | zip | `zip -r a.zip dir`　`unzip a.zip` |
| `md5sum` / `sha256sum` | 校验 | `md5sum f`　`sha256sum f`　`sha256sum -c f.sha256` |
| `rsync` | 同步目录 | `rsync -avP src/ user@host:dst/` |

---

## 10. Shell 与脚本常用

| 命令/语法 | 用途 | 用法示例 |
|-----------|------|----------|
| `echo` / `printf` | 打印 | `echo hello`　`printf "%s\n" "$var"` |
| `export` / `env` | 环境变量 | `export PATH=$PATH:/opt/bin`　`env`　`printenv PATH` |
| `which` / `type` | 命令路径 | `which gcc`　`type cd` |
| `alias` | 别名 | `alias ll='ls -l'` |
| `history` | 历史 | `history`　`!123`（执行历史编号） |
| `date` | 时间 | `date`　`date +%Y-%m-%d` |
| `sleep` | 延时 | `sleep 1`　`sleep 0.5` |
| `xargs` | 参数化管道 | `find . -name "*.c" \| xargs grep foo` |
| `tee` | 显示并写入 | `make 2>&1 \| tee build.log` |
| 后台任务 | 后台跑 | `./app &`　`nohup ./app &`　`jobs`　`fg`　`bg` |
| `source` / `.` | 当前 shell 执行 | `source env.sh`　`. ./env.sh` |
| `basename` / `dirname` | 路径拆分 | `basename /a/b/c.txt`　`dirname /a/b/c.txt` |
| `test` / `[ ]` | 条件 | `[ -f f ] && echo yes`　`[ -d d ]`　`[ "$a" = "$b" ]` |

管道与重定向：

```bash
cmd1 | cmd2          # 管道
cmd > out.txt         # 覆盖写 stdout
cmd >> out.txt        # 追加
cmd 2> err.txt        # 只重定向 stderr
cmd > all.txt 2>&1    # stdout+stderr
```

---

## 11. BusyBox 提示

很多嵌入式 rootfs 是 **BusyBox**：

```bash
busybox             # 看帮助
busybox --list      # 支持哪些命令（若有）
busybox ls --help   # 某 applet 选项（常比完整 GNU 少）
```

同一命令名可能是精简实现，以板子实际帮助为准。

---

## 12. 安全习惯

- `rm -rf`、`dd`、`mkfs`、写裸盘：先 `pwd`，再核对路径
- 写 `/sys`、`/dev`：确认节点含义
- 优先 `ssh`，少用明文 `telnet`；改掉默认密码

---

## 快速记忆

```text
文件: ls cd pwd cat less grep find cp mv rm
进程: ps top kill dmesg free
网络: ip addr ping ss scp ssh
存储: df mount sync
驱动: lsmod insmod modprobe file chmod
```

不会用时：`命令 --help` 或 `man 命令`（BusyBox 上 `man` 可能没有，多用 `--help`）。
