## 玩家的生命值受伤
在SlashCharrcter.h中进行受伤方法的继承，
```
virtual float ASlashCharacter::TakeDamage(float DamageAmount, FDamageEvent const& DamageEvent, AController* EventInstigator, AActor* DamageCauser) overrid;
```
```
float ASlashCharacter::TakeDamage(float DamageAmount, FDamageEvent const& DamageEvent, AController* EventInstigator, AActor* DamageCauser)
{
	HandleDamage(DamageAmount);
	return DamageAmount;
}
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE1NzQ3NDc4MDVdfQ==
-->