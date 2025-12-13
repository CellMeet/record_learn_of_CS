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