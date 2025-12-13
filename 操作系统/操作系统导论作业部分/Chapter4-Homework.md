* **问题一**
  [问题代码链接](https://github.com/remzi-arpacidusseau/ostep-homework/blob/master/cpu-intro/process-run.py)

```
./process-run.py -l 5:100,5:100
```

代码的含义是，构建两个进程，都是100%使用CPU，不会进行IO

```
./process-run.py -l 5:100,5:100 -c
```

```
Time        PID: 0        PID: 1           CPU           IOs
  1        RUN:cpu         READY             1
  2        RUN:cpu         READY             1
  3        RUN:cpu         READY             1
  4        RUN:cpu         READY             1
  5        RUN:cpu         READY             1
  6           DONE       RUN:cpu             1
  7           DONE       RUN:cpu             1
  8           DONE       RUN:cpu             1
  9           DONE       RUN:cpu             1
 10           DONE       RUN:cpu             1
```

查看结果后，发现cpu按照顺序执行，PID0先执行，PID1先ready，等PID0执行完后再执行。

* **问题二**
```
./process-run.py -l 4:100,1:0
```

命令的意思是：
运行两个进程
    - 第一个进程有4个指令，是100%使用CPU的，不会进行IO
    - 第二个进程有1个指令，是100%进行IO的
```

Time        PID: 0        PID: 1           CPU           IOs
  1        RUN:cpu         READY             1
  2        RUN:cpu         READY             1
  3        RUN:cpu         READY             1
  4        RUN:cpu         READY             1
  5           DONE        RUN:io             1
  6           DONE       BLOCKED                           1
  7           DONE       BLOCKED                           1
  8           DONE       BLOCKED                           1
  9           DONE       BLOCKED                           1
 10           DONE       BLOCKED                           1
 11*          DONE   RUN:io_done             1
 ```

先运行pid0的4个指令
然后在Time == 5时运行pid1的io指令，发起IO（每次io的默认占用5个单位时间）
经过5个单位时间的WAITING状态后，PID1的状态变为io_done

* **第三题**

```
./process-run.py -l 1:0,4:100
```
 
 交换进程的顺序

 ```
Time        PID: 0        PID: 1           CPU           IOs
  1         RUN:io         READY             1
  2        BLOCKED       RUN:cpu             1             1
  3        BLOCKED       RUN:cpu             1             1
  4        BLOCKED       RUN:cpu             1             1
  5        BLOCKED       RUN:cpu             1             1
  6        BLOCKED          DONE                           1
  7*   RUN:io_done          DONE             1
 ```

 运行结果显示，先执行PID0的指令，发起IO
 然后PID0进入BLOCKED状态，同时CPU切换到PID1，执行PID1的cpu指令。
 Time=5的时候PID1执行完成
 Time=7的时候IO完全结束，状态变为IO_done

* **第四题**
  
```
./process-run.py -l 1:0,4:100 -S SWITCH_ON_END
```

在第3题基础上加上 -s swithc_on_end
代表进程之间切换模式修改了，现在cpu等IO结束才会结束进程。

```
Time        PID: 0        PID: 1           CPU           IOs
  1         RUN:io         READY             1
  2        BLOCKED         READY                           1
  3        BLOCKED         READY                           1
  4        BLOCKED         READY                           1
  5        BLOCKED         READY                           1
  6        BLOCKED         READY                           1
  7*   RUN:io_done         READY             1
  8           DONE       RUN:cpu             1
  9           DONE       RUN:cpu             1
 10           DONE       RUN:cpu             1
 11           DONE       RUN:cpu             1
 ```

 即使在BOLOCKED状态，也不会切换进程，等到5个BLOCKED之后，RUN:I0_done，然后进行四个RUN:cpu。

 * **第五题**
  
```
./process-run.py -l 1:0,4:100 -S SWITCH_ON_IO
```

```
Time        PID: 0        PID: 1           CPU           IOs
  1         RUN:io         READY             1
  2        BLOCKED       RUN:cpu             1             1
  3        BLOCKED       RUN:cpu             1             1
  4        BLOCKED       RUN:cpu             1             1
  5        BLOCKED       RUN:cpu             1             1
  6        BLOCKED          DONE                           1
  7*   RUN:io_done          DONE             1
  ```

在第三题基础上加上swith_on_io,代表BLOCKED状态时CPU会切换进程。

* **第六题**

```
process-run.py -l 3:0,5:100,5:100,5:100 -S SWITCH_ON_IO -I IO_RUN_LATER -c -p
```

运行4个进程：
 - 第1个进程：3个指令，均为io指令
 - 第2-4个进程：5个指令，均为cpu指令
 - 命令中的-I表示指定io结束后的行为模式
    - IO_RUN_IMMEDIATE: io结束后马上切换到发出io的进程
    - IO_RUN_LATER: io结束后自然的切换到发出io的进程（默认）
 - 命令中的-c表示打印答案
 - 命令中的-p表示打印状态信息

```
Time        PID: 0        PID: 1        PID: 2        PID: 3           CPU           IOs
  1         RUN:io         READY         READY         READY             1
  2        BLOCKED       RUN:cpu         READY         READY             1             1
  3        BLOCKED       RUN:cpu         READY         READY             1             1
  4        BLOCKED       RUN:cpu         READY         READY             1             1
  5        BLOCKED       RUN:cpu         READY         READY             1             1
  6        BLOCKED       RUN:cpu         READY         READY             1             1
  7*         READY          DONE       RUN:cpu         READY             1
  8          READY          DONE       RUN:cpu         READY             1
  9          READY          DONE       RUN:cpu         READY             1
 10          READY          DONE       RUN:cpu         READY             1
 11          READY          DONE       RUN:cpu         READY             1
 12          READY          DONE          DONE       RUN:cpu             1
 13          READY          DONE          DONE       RUN:cpu             1
 14          READY          DONE          DONE       RUN:cpu             1
 15          READY          DONE          DONE       RUN:cpu             1
 16          READY          DONE          DONE       RUN:cpu             1
 17    RUN:io_done          DONE          DONE          DONE             1
 18         RUN:io          DONE          DONE          DONE             1
 19        BLOCKED          DONE          DONE          DONE                           1
 20        BLOCKED          DONE          DONE          DONE                           1
 21        BLOCKED          DONE          DONE          DONE                           1
 22        BLOCKED          DONE          DONE          DONE                           1
 23        BLOCKED          DONE          DONE          DONE                           1
 24*   RUN:io_done          DONE          DONE          DONE             1
 25         RUN:io          DONE          DONE          DONE             1
 26        BLOCKED          DONE          DONE          DONE                           1
 27        BLOCKED          DONE          DONE          DONE                           1
 28        BLOCKED          DONE          DONE          DONE                           1
 29        BLOCKED          DONE          DONE          DONE                           1
 30        BLOCKED          DONE          DONE          DONE                           1
 31*   RUN:io_done          DONE          DONE          DONE             1

Stats: Total Time 31
Stats: CPU Busy 21 (67.74%)
Stats: IO Busy  15 (48.39%)
```
  
解释：
因为是IO_RUN_LATER + SWITCH_ON_IO模式

所以按顺序：
先运行PID0的io指令，在PID0为BLOCKED的同时，运行PID1的CPU指令，然后PIDO变回READY，
但是由于处于IO_RUN_LATER模式，会先运行PID2和PID3的cpu指令，最后切换为PIO0（这是导致CPU没有被充分使用的最主要的原因）。

* **第七题**

```
process-run.py -l 3:0,5:100,5:100,5:100 -S SWITCH_ON_IO -I IO_RUN_IMMEDIATE-c -p
```

采用IO_RUN_IMMEDIATA模式

```
Time        PID: 0        PID: 1        PID: 2        PID: 3           CPU           IOs
  1         RUN:io         READY         READY         READY             1
  2        BLOCKED       RUN:cpu         READY         READY             1             1
  3        BLOCKED       RUN:cpu         READY         READY             1             1
  4        BLOCKED       RUN:cpu         READY         READY             1             1
  5        BLOCKED       RUN:cpu         READY         READY             1             1
  6        BLOCKED       RUN:cpu         READY         READY             1             1
  7*   RUN:io_done          DONE         READY         READY             1
  8         RUN:io          DONE         READY         READY             1
  9        BLOCKED          DONE       RUN:cpu         READY             1             1
 10        BLOCKED          DONE       RUN:cpu         READY             1             1
 11        BLOCKED          DONE       RUN:cpu         READY             1             1
 12        BLOCKED          DONE       RUN:cpu         READY             1             1
 13        BLOCKED          DONE       RUN:cpu         READY             1             1
 14*   RUN:io_done          DONE          DONE         READY             1
 15         RUN:io          DONE          DONE         READY             1
 16        BLOCKED          DONE          DONE       RUN:cpu             1             1
 17        BLOCKED          DONE          DONE       RUN:cpu             1             1
 18        BLOCKED          DONE          DONE       RUN:cpu             1             1
 19        BLOCKED          DONE          DONE       RUN:cpu             1             1
 20        BLOCKED          DONE          DONE       RUN:cpu             1             1
 21*   RUN:io_done          DONE          DONE          DONE             1

Stats: Total Time 21
Stats: CPU Busy 21 (100.00%)
Stats: IO Busy  15 (71.43%)
```

更换模式后，IO结束后直接切换到发出IO的进程，这时候，由于BLOCKED,可以等待其他进程完成，减少了total time，同时把cpu利用率提高到了100%。

* **第八题**

开放作业

25年12月13日 完成