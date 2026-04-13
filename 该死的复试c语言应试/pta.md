# 最大公约数
辗转相除
```c
#include <stdio.h>
int gcd(int a, int b){
    while(b > 0){
        int temp = a % b;
        a = b;
        b = temp;
    }
    return a;
}
int main(){
    int a, b;
    scanf("%d %d", &a, &b);
    printf("%d", gcd(a, b));
}
```
# 在顺序表 list 中查找元素 x
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTc3NDkyNTAzNF19
-->