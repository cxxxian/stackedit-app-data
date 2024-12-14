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
![输入图片说明](/imgs/2024-12-14/a1zepHtPTQPzbu5i.png)
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE2NDU5MjQyOTRdfQ==
-->