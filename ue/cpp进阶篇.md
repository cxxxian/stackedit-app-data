## 玩家的生命值受伤
在SlashCharrcter.h中进行受伤方法的继承
```
virtual float ASlashCharacter::TakeDamage(float DamageAmount, FDamageEvent const& DamageEvent, AController* EventInstigator, AActor* DamageCauser) overrid;
```
并在SlashCharrcter.cpp中实现
```
float ASlashCharacter::TakeDamage(float DamageAmount, FDamageEvent const& DamageEvent, AController* EventInstigator, AActor* DamageCauser)
{
	HandleDamage(DamageAmount);
	return DamageAmount;
}
```
我们建立好蓝图的界面后，希望通过cpp来控制数值
所以新建c++类来作为蓝图的父类，
![输入图片说明](/imgs/2024-08-30/GQeYmjPXhzgCqKxg.png)
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTk2NTY4NjM0MSwxODIyMDI4NDIyLDE3OD
AyMDA5MjRdfQ==
-->