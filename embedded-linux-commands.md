# 嵌入式 Linux 常用命令速查

按**用途分类**，同类内大致按日常使用频率从前到后排列（不标注频率星级）。偏板端 shell / 调试，不绑定某一具体 SDK。

---

## 1. 目录与文件

| 命令 | 用途 |
|------|------|
| `ls` / `ls -l` / `ls -la` | 列出文件；`-l` 权限大小，`-a` 含隐藏 |
| `cd` | 切换目录 |
| `pwd` | 显示当前路径 |
| `cat` | 查看/拼接文本 |
| `less` / `more` | 分页看大文件（`less` 可上下翻） |
| `head` / `tail` / `tail -f` | 看头/尾；`-f` 跟日志 |
| `cp` / `mv` / `rm` | 复制 / 移动重命名 / 删除 |
| `mkdir` / `rmdir` / `rm -r` | 建目录 / 删空目录 / 递归删 |
| `touch` | 建空文件或更新时间戳 |
| `find` | 按名/类型/时间查找 |
| `tree` | 树状看目录（若已安装） |
| `du -sh` | 看目录占用空间 |
| `df -h` | 看分区剩余空间 |
| `ln -s` | 建软链接 |
| `readlink` / `readlink -f` | 看链接指向；`-f` 解析绝对路径 |
| `file` | 判断文件类型/CPU 架构（交叉编译必用） |
| `stat` | 看 inode、时间、权限详情 |
| `chmod` / `chown` | 改权限 / 改所有者 |
| `sync` | 把缓存写回存储（掉电前常用） |

---

## 2. 文本检索与编辑

| 命令 | 用途 |
|------|------|
| `grep` / `grep -r` / `grep -n` | 搜字符串；递归、带行号 |
| `egrep` / `grep -E` | 扩展正则 |
| `sed` | 流编辑、替换打印 |
| `awk` | 按列处理文本 |
| `wc` | 行数/字数统计 |
| `diff` / `diff -u` | 比较文件差异 |
| `sort` / `uniq` | 排序 / 去重 |
| `cut` / `tr` | 切列 / 字符替换 |
| `vi` / `vim` / `nano` | 终端编辑器 |
| `hexdump` / `xxd` | 十六进制看二进制 |

---

## 3. 进程与系统状态

| 命令 | 用途 |
|------|------|
| `ps` / `ps aux` / `ps -ef` | 看进程 |
| `top` / `htop` | 动态看 CPU/内存（`htop` 需安装） |
| `kill` / `kill -9` / `killall` | 发信号结束进程 |
| `pidof` / `pgrep` | 按名找 PID |
| `nice` / `renice` | 调进程优先级 |
| `uptime` | 运行时长、负载 |
| `free -h` | 内存/交换分区 |
| `uname -a` | 内核与机器信息 |
| `cat /proc/cpuinfo` | CPU 信息 |
| `cat /proc/meminfo` | 内存详情 |
| `cat /proc/version` | 内核版本字符串 |
| `dmesg` / `dmesg -w` | 内核环缓日志；跟随打印 |
| `lsof` | 谁占用了文件/端口（若有） |
| `strace` | 跟踪系统调用（排权限/打开失败） |
| `time` | 测命令耗时 |

---

## 4. 设备、驱动与硬件节点

| 命令 | 用途 |
|------|------|
| `ls /dev` | 看设备节点 |
| `lsmod` | 已加载内核模块 |
| `insmod` / `rmmod` | 加载/卸载 `.ko`（需路径） |
| `modprobe` / `modprobe -r` | 按模块名加载/卸载（解析依赖） |
| `lsusb` | USB 设备树（若有 usbutils） |
| `lspci` | PCI 设备（若有） |
| `i2cdetect` / `i2cget` / `i2cset` | I2C 总线探测与读写（i2c-tools） |
| `gpiodetect` / `gpioinfo` / `gpioset` | 现代 GPIO 工具（libgpiod） |
| `cat /sys/class/...` | sysfs 读状态（LED、pwm、net 等） |
| `echo ... > /sys/...` | sysfs 写控制（注意权限与路径） |
| `udevadm` | udev 规则与事件（发行版差异大） |
| `hwclock` | 硬件 RTC 时间 |

---

## 5. 网络

| 命令 | 用途 |
|------|------|
| `ip addr` / `ip link` / `ip route` | 地址、网卡、路由（推荐） |
| `ifconfig` / `route` | 旧式网卡/路由（BusyBox 常有） |
| `ping` | 连通性 |
| `ping6` | IPv6 连通 |
| `netstat` / `ss` | 端口与连接 |
| `wget` / `curl` | 下载 / HTTP 调试 |
| `scp` / `sftp` | 远程拷文件 |
| `ssh` | 远程登录 |
| `nc` / `netcat` | 端口探测、简易传数据 |
| `tcpdump` | 抓包（若已装） |
| `iperf` / `iperf3` | 吞吐测试（若已装） |
| `arp` / `arping` | ARP 相关 |
| `hostname` | 主机名 |
| `nft` / `iptables` | 防火墙（视系统而定） |

---

## 6. 存储、分区与文件系统

| 命令 | 用途 |
|------|------|
| `mount` / `umount` | 挂载/卸载 |
| `lsblk` | 块设备树（若有） |
| `fdisk` / `parted` | 分区（慎用） |
| `mkfs.ext4` / `mkfs.vfat` | 格式化 |
| `fsck` | 检查修复文件系统 |
| `dd` | 镜像读写、拷贝块设备（极危险，核对 if/of） |
| `blkid` | 查看 UUID/类型 |
| `sync` | 刷缓存到盘 |
| `losetup` | loop 设备（挂镜像时） |

---

## 7. 用户、权限与服务

| 命令 | 用途 |
|------|------|
| `whoami` / `id` | 当前用户与组 |
| `su` / `sudo` | 切换/提权 |
| `passwd` | 改密码 |
| `useradd` / `usermod` / `userdel` | 用户管理 |
| `systemctl` | systemd 服务启停（若用 systemd） |
| `service` | SysV 风格服务（BusyBox/旧系统） |
| `reboot` / `poweroff` / `halt` | 重启/关机 |
| `login` / `getty` | 登录相关（一般由 init 拉起） |

---

## 8. 编译、开发与调试（主机或板端）

| 命令 | 用途 |
|------|------|
| `gcc` / `g++` / 交叉前缀 `*-gcc` | 编译 |
| `make` / `make clean` | 构建 |
| `cmake` | 生成构建文件 |
| `gdb` / `gdb-multiarch` | 调试；多架构 |
| `objdump` / `readelf` / `nm` | 反汇编、看 ELF、看符号 |
| `ldd` | 看动态库依赖（目标架构对应工具链） |
| `strip` | 去掉符号减小体积 |
| `ar` / `ranlib` | 静态库 |
| `pkg-config` | 查编译链接参数 |

---

## 9. 打包、压缩与传输

| 命令 | 用途 |
|------|------|
| `tar` / `tar czf` / `tar xzf` | 打包/解包 |
| `gzip` / `gunzip` / `xz` | 压缩 |
| `zip` / `unzip` | zip 格式 |
| `md5sum` / `sha256sum` | 校验完整性 |
| `rsync` | 高效同步目录 |

---

## 10. Shell 与脚本常用

| 命令/语法 | 用途 |
|-----------|------|
| `echo` / `printf` | 打印 |
| `export` / `env` / `printenv` | 环境变量 |
| `which` / `type` / `command -v` | 查命令路径 |
| `alias` | 别名 |
| `history` | 历史命令 |
| `date` | 日期时间 |
| `sleep` | 延时 |
| `xargs` | 把标准输入转参数 |
| `tee` | 一边显示一边写入文件 |
| `nohup` / `&` / `jobs` / `fg` / `bg` | 后台任务 |
| `source` / `.` | 执行脚本到当前 shell |
| `basename` / `dirname` | 取文件名/目录名 |
| `test` / `[ ]` / `[[ ]]` | 条件判断 |

---

## 11. BusyBox 提示

很多嵌入式 rootfs 里命令是 **BusyBox 多合一**：

- 同一名字可能是精简实现（选项更少）
- 用 `busybox` 或不带路径的命令，行为以板子为准
- `busybox --list` 可看当前支持哪些 applet（若提供）

---

## 12. 安全习惯（简短）

- `rm -rf`、`dd`、`mkfs`、写 `/dev/sdX`：先 `pwd`、再确认路径
- 改 sysfs/设备节点：确认节点含义，避免写错脚或分区
- 生产环境少开 `telnet`，优先 `ssh`；注意默认密码

---

## 快速记忆：每天最高频的一串

```text
ls / cd / pwd / cat / grep / find
ps / top / kill / dmesg
ip addr / ping / mount / df
insmod / lsmod / file / chmod
```

按项目再叠加：`i2c*`、`gpio*`、`tcpdump`、交叉 `*-gcc`、`gdb` 等。
