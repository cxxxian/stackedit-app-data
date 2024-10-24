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
在SlashOverlay.cpp中完善方法，将百分比和数字赋值
```
#include "Components/ProgressBar.h"
#include "Components/TextBlock.h"

void USlashOverlay::SetHealthBarPercent(float Percent)
{
	if (HealthProgressBar) {
		HealthProgressBar->SetPercent(Percent);
	}
}

void USlashOverlay::SetStaminaBarPercent(float Percent)
{
	if (StaminaProgressBar) {
		StaminaProgressBar->SetPercent(Percent);
	}
}

void USlashOverlay::SetGold(int32 Gold)
{
	if (GoldText) {
		const FString String = FString::Printf(TEXT("%d"), Gold);//转string
		const FText Text = FText::FromString(String);//转text
		GoldText->SetText(Text);
	}
}

void USlashOverlay::SetSoul(int32 Soul)
{
	if (SoulText) {
		const FString String = FString::Printf(TEXT("%d"), Soul);
		const FText Text = FText::FromString(String);
		SoulText->SetText(Text);
	}
}

```
回到SlashOverlay的蓝图点击图表选择类设置，将父类修改为自己的c++类
![输入图片说明](/imgs/2024-10-09/sikQRNU2Y9AtH82J.png)
### 蓝图做法
在关卡蓝图中添加到视口即可在游戏画面中看见该widget，但是此处只针对当前关卡地图。
![输入图片说明](/imgs/2024-10-09/7zMMl7Ft7BMm4Ime.png)
通过创建一个HUD类
![输入图片说明](/imgs/2024-10-09/vMO6ms1DkP4jB5pX.png)
在该蓝图中的beginPlay将widget添加上去
![输入图片说明](/imgs/2024-10-09/QTN7aJgFjmgxCPEl.png)
最后在世界场景中，游戏模式的HUD将其改为我们自己的HUD
![输入图片说明](/imgs/2024-10-09/cS1ydo3nhtFSiMWW.png)
### cpp做法
新建一个c++的HUD类，取名为SlashHUD
![输入图片说明](/imgs/2024-10-09/chnWzsq4LpNQh3pI.png)
在SlashHUD，声明一个SlashOverlayClass，TSubclassOf用来限制class USlashOverlay类型
```
private:
	UPROPERTY(EditDefaultsOnly, Category = Slash)
	TSubclassOf<class USlashOverlay> SlashOverlayClass;
```
将原本蓝图的HUD修改为以此为父类
即可以在细节面板中看到此设置，将其改为我们的蓝图widget
![输入图片说明](/imgs/2024-10-09/C3PwuQviTc5CNScN.png)
我们希望通过cpp来将widget添加到视口而不是用蓝图
在SlashHUD.h，继承BeginPlay函数
```
protected:
	virtual void BeginPlay() override;

private:
	UPROPERTY(EditDefaultsOnly, Category = Slash)
	TSubclassOf<USlashOverlay> SlashOverlayClass;

	USlashOverlay* SlashOverlay;
public:
	FORCEINLINE USlashOverlay* GetSlashOverlay() const { return SlashOverlay; }
```
在SlashHUD.cpp实现，
```
void ASlashHUD::BeginPlay()
{
	Super::BeginPlay();

	UWorld* World = GetWorld();
	if (World) {
		APlayerController* Controller = World->GetFirstPlayerController();
		if (Controller && SlashOverlayClass) {
			SlashOverlay = CreateWidget<USlashOverlay>(Controller, SlashOverlayClass);
			SlashOverlay->AddToViewport();
		}
	}
}
```
### 让SlashCharacter可以访问到我们的SlashOverlay
在SlashCharacter.cpp的beginPlay中，将SlashOverlay和character联系起来。
通过cast投射，默认的APlayerController和ASlashHUD是访问不到我们自己设置的东西。
在SlashCharacter.h中声明一个USlashOverlay* SlashOverlay；
然后再SlashCharacter.cpp如下实现：
```
void ASlashCharacter::BeginPlay()
{
	Super::BeginPlay();
	
	Tags.Add(FName("EngageableTarget"));
	APlayerController* PlayerController = Cast<APlayerController>(GetController());
	if (PlayerController) {
		ASlashHUD* SlashHUD = Cast<ASlashHUD>(PlayerController->GetHUD());
		if (SlashHUD) {
			SlashOverlay = SlashHUD->GetSlashOverlay();
			if (SlashOverlay && Attributes) {
				SlashOverlay->SetHealthBarPercent(Attributes->GetHealthPercent());
				SlashOverlay->SetStaminaBarPercent(1.f);
				SlashOverlay->SetGold(0);
				SlashOverlay->SetSoul(0);
			}
		}
	}
}
```
# Echo死亡
把敌人死亡逻辑换到BaseCharacter里面，给Echo添加死亡状态Type，继承即可
此处通过这个linked anim graph，把cpp的状态连接到动画蓝图上
![输入图片说明](/imgs/2024-10-24/ZfziwwnL7hpwiwf1.png)

<!--stackedit_data:
eyJoaXN0b3J5IjpbMjM5Mzc1NTM2LDEwMDQ4OTY0NDcsOTkwMj
gwNDEzLC0xNzM4NjEyNTg4LDEzNDMzOTM2NDEsMTU5NzkzNDU3
MiwxNTM0OTE5MDA3LC04NDMxNjU2NjQsMTYxNDUzMjA5LC0zND
c3MDYxNTksMTI0NDQ3OTk2NCwyNTMzNTkxMDYsMTgyMjAyODQy
MiwxNzgwMjAwOTI0XX0=
-->