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
# 在顺序表 list 的第 i 个位置上插入元素 x
这种题从后往前做
从后往前，每一个向后移动，直到遇到希望插入的
```c
#include <stdio.h>

int main(){
    int n;
    scanf("%d", &n);
    int nums[n + 1];
    for(int i = 0; i < n; i++){
        scanf("%d", &nums[i]);
    }
    int index, x;
    scanf("%d%d", &index, &x);
    int insert = 0;
    if(n >= 10000){
        printf("错误：表满不能插入。\n");
    }else if(index > n + 1 || index < 1){
        printf("错误：插入位置不合法。\n");
    }else{
        for(int i = n; i > index - 1; i--){
            nums[i] = nums[i - 1];
        }
        nums[index - 1] = x;
        insert = 1;
    }
    int length = insert? n + 1: n;
    for(int i = 0; i < length; i++){
        printf("%d ", nums[i]);
    }
    printf("\n");
}
```
# 从顺序表 list 中删除第 i 个元素
相应的这种题从前往后做
```c
#include <stdio.h>
int main(){
    int n;
    scanf("%d", &n);
    int nums[n];
    for(int i = 0; i < n; i++){
        scanf("%d", &nums[i]);
    }
    int index;
    scanf("%d", &index);
    int delete = 0;
    if(index < 1 || index > n){
        printf("错误：不存在这个元素。\n");
    }else{
        for(int i = index; i < n; i++){
            nums[i - 1] = nums[i];
        }
        delete = 1;
    }
    int length = delete? n - 1 : n;
    for(int i = 0; i < length; i++){
        printf("%d ", nums[i]);
    }
    
}
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEzOTk5NjY4MDNdfQ==
-->