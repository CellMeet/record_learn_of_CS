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

  