* **问题一**
  
```C
#include <stdio.h>
#include<stdlib.h>
#include<unistd.h>

int main(int argc,char* argv[]){
    int rc = fork();
    int x =100;
    if(rc<0){
       fprintf(stderr,"fork failed\n");
       exit(1);
    }else if(rc == 0){
            x = 200;
            printf("child x: %d\n",x);
    }else {
            printf("parent x:%d\n",x);
    }
    return 0;
}
```
* **问题一**
  
```C
#include <stdio.h>
#include<stdlib.h>
#include<unistd.h>

int main(int argc,char* argv[]){
    int rc = fork();
    int x =100;
    if(rc<0){
       fprintf(stderr,"fork failed\n");
       exit(1);
    }else if(rc == 0){
            x = 200;
            printf("child x: %d\n",x);
    }else {
            printf("parent x:%d\n",x);
    }
    return 0;
}
```

运行结果:

```
(base) jktian@DESKTOP-8RITHV0:~/projects/CS$ ./a.out
parent x:100
child x: 200
```

解释一下代码：
首先调用fork(),创建一个新的进程，fork()返回两次，在父进程中返回子进程的PID，在子进程中返回0。

定义一个整型x为100，父进程和子进程都有自己独立的x副本

if（rc == 0），表示当前在子进程中，那么设置x = 200，打印；

if（rc >0），表示当前在父进程中，rc的值就是子进程的pid，打印出来自己的x，输出为100

* **问题二**
  
```C
#include <stdio.h>  //输入输出文件，用于printf
#include <stdlib.h> //库头文件，用于exit
#include <unistd.h> //Unix标准头文件，用于fork函数
#include <sys/types.h> //系统类型头文件，用于定义数据类型
#include <sys/stat.h>  //文件状态头文件，用于文件打开模式等
#include <fcntl.h>  //文件控制头文件，用于open函数

int main(int argc,char* argv[]){
        int rc = fork(); //创建一个子进程

        int fd = open("./test.txt",O_RDWR);  //打开一个test.txt的文件，由于打开文件在fork之后，所以父子进程各自打开一次文件

        if(rc < 0){
                fprintf(stderr,"fork failed\n");
                exit(1);
        } else if(rc == 0){
                printf("child fd:%d\n",fd);  //子进程打印自己的文件描述符fd

                char buffer1[] = "chilid:hello,ostep!\n"; 
                write(fd,buffer1,sizeof(buffer1)); //将buffer1写入文件
        } else {
                printf("parent fd: %d\n",fd); //父进程打印自己的文件描述符fd

                char buffer2[] = "parent:hello,ostep!\n"; //父进程准备另一段字符串
                write(fd,buffer2,sizeof(buffer2));  //写入文件
        }
        return 0;
}
```

```
(base) jktian@DESKTOP-8RITHV0:~/projects/CS$ ./a.out
parent fd: 3
child fd:3
(base) jktian@DESKTOP-8RITHV0:~/projects/CS$ cat test.txt
chilid:hello,ostep!
```
运行结果如上。

* **问题三**