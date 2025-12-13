* **问题一**
  [问题代码链接](https://github.com/remzi-arpacidusseau/ostep-homework/blob/master/cpu-intro/process-run.py)

```
./process-run.py -l 5:100,5:100
```

代码的含义是，构建两个进程，都是100%使用CPU，不会进行IO

```
Produce a trace of what would happen when you run these processes:
Process 0
  cpu
  cpu
  cpu
  cpu
  cpu

Process 1
  cpu
  cpu
  cpu
  cpu
  cpu
```

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
 然后