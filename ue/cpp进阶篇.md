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
我们建立好蓝图WBP_SlashOverlay的界面后，希望通过cpp来控制数值
![输入图片说明](/imgs/2024-08-30/3eozFlssKP8dw0ej.png)
所以新建c++类来作为蓝图的父类，新建类的名字为SlashOverlay
![输入图片说明](/imgs/2024-08-30/GQeYmjPXhzgCqKxg.png)
在SlashOverlay.h中，将需要用的的变量全部声明，名字对应蓝图中的变量方便于用meta = (BindWidget)绑定
```
public:
	void SetHealthBarPercent(float Percent);
	void SetStaminaBarPercent(float Percent);
	void SetGold(int32 Gold);
	void SetSoul(int32 Soul);

private:
	UPROPERTY(meta = (BindWidget))
	class UProgressBar* HealthProgressBar;

	UPROPERTY(meta = (BindWidget))
	class UProgressBar* StaminaProgressBar;

	UPROPERTY(meta = (BindWidget))
	class UTextBlock* GoldText;

	UPROPERTY(meta = (BindWidget))
	class UTextBlock* SoulText;
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbODUxMjU1NzU2LDI1MzM1OTEwNiwxODIyMD
I4NDIyLDE3ODAyMDA5MjRdfQ==
-->