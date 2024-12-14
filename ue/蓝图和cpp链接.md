比如我现在有一个方法叫`Die`，我希望蓝图可以独立于`cpp`
```cpp
UFUNCTION(BlueprintNativeEvent)
void Die();
```
实现方法的时候加个`_Implementation`
```cpp
void ABaseCharacter::Die_Implementation()
{
	Tags.Add(FName("Dead"));
	PlayDeathMontage();
}
```
`SlashCharacter`和`Enemy`要同步进行`override`的修改
然后我们就可以到蓝图中使用这个`Die`事件，后面那个父类是`cpp`设计的`Die`，我们就能做到在蓝图设计自己想要的然后调用父类的`Die`函数
![输入图片说明](/imgs/2024-12-14/a1zepHtPTQPzbu5i.png)
<!--stackedit_data:
eyJoaXN0b3J5IjpbODY0Mjg5MjgxLDc2MDc2NzM3OSwtMTY4Nj
g0MDgwNSwtMTg4MTAzOTE5N119
-->