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
![输入图片说明](/imgs/2024-08-30/GQeYmjPXhzgCqKxg.png)
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTgyMjAyODQyMiwxNzgwMjAwOTI0XX0=
-->