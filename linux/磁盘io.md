## 磁盘io

###  iostat
```bash
root@Thinkpad-T470p:~# iostat
Linux 5.15.133.1-microsoft-standard-WSL2 (Thinkpad-T470p)       11/23/23        _x86_64_        (4 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           3.79    0.01    4.10    0.57    0.00   91.53

Device             tps    kB_read/s    kB_wrtn/s    kB_dscd/s    kB_read    kB_wrtn    kB_dscd
sda               0.01         0.52         0.00         0.00      74681          0          0
sdb              20.17        15.88       290.76       122.42    2297501   42060792   17708772
sdc               0.11         5.63         2.07         1.80     814064     299572     259988
sdd               0.00         0.02         0.00         0.00       2989        348         24
```

- 当你运行 `iostat` 命令时，它会显示关于磁盘IO情况的详细信息。下面是 `iostat` 命令的响应解释：

1. `Device`: 列出了系统中的每个设备或分区的名称。

2. `tps` (Transactions Per Second): 表示每秒完成的读写操作的数量。较高的值表示更多的IO活动。

3. `kB_read/s` 和 `kB_wrtn/s` (Kilobytes Read/Write per Second): 表示每秒读取和写入的数据量，以千字节为单位。

4. `kB_read` 和 `kB_wrtn` (Kilobytes Read/Write): 表示自系统启动以来读取和写入的总数据量，以千字节为单位。

5. `await` (Average I/O Wait Time): 表示每个IO操作的平均等待时间，以毫秒为单位。较高的值可能表示IO延迟。

6. `svctm` (Average Service Time): 表示每个IO操作的平均服务时间，以毫秒为单位。较高的值可能表示IO延迟。

7. `%util` (Device Utilization Percentage): 表示设备在特定时间内的利用率百分比。较高的值表示设备正处于高负载状态。

以上是 `iostat` 命令常见的响应字段及其解释。你可以根据这些信息来评估系统磁盘IO的性能和负载情况。

### iotop
```bash
Total DISK READ:         0.00 B/s | Total DISK WRITE:        37.26 K/s
Current DISK READ:       0.00 B/s | Current DISK WRITE:      59.62 K/s
    TID  PRIO  USER     DISK READ  DISK WRITE  SWAPIN     IO>    COMMAND
   1218 be/4 root        0.00 B/s   18.63 K/s  ?unavailable?  k8s-dqlite --storage-dir=/var/snap/microk8s/610~ok8s/6102/var/kubernetes/backend/kine.sock:12379
   1220 be/4 root        0.00 B/s   18.63 K/s  ?unavailable?  k8s-dqlite --storage-dir=/var/snap/microk8s/610~ok8s/6102/var/kubernetes/backend/kine.sock:12379
      1 be/4 root        0.00 B/s    0.00 B/s  ?unavailable?  init

```

- 当你运行 `iotop` 命令时，它会显示关于磁盘IO情况的详细信息，包括每个进程的磁盘IO使用情况。下面是 `iotop` 命令的响应解释：

1. `Total DISK READ` 和 `Total DISK WRITE`: 分别表示自 `iotop` 启动以来读取和写入的总数据量，以千字节为单位。

2. `Actual DISK READ` 和 `Actual DISK WRITE`: 分别表示自上次刷新以来读取和写入的数据量，以千字节为单位。

3. `Process`: 列出了所有正在进行磁盘IO操作的进程的名称和进程ID。

4. `PID`: 进程ID。

5. `User`: 进程所属的用户。

6. `DISK READ` 和 `DISK WRITE`: 分别表示自上次刷新以来进程读取和写入的数据量，以千字节为单位。

7. `SWAPIN` 和 `IO`: 分别表示自上次刷新以来进程从磁盘交换区域读取到内存中的数据量和进程发起的IO操作数量。

以上是 `iotop` 命令常见的响应字段及其解释。你可以根据这些信息来查看系统中哪些进程正在进行磁盘IO操作，以及它们的IO使用情况。

