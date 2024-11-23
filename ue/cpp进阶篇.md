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

## 解决echo死亡后敌人仍然攻击
在BaseCharacter.cpp中，在Die函数中添加Tag，并在Attack中利用Tag进行判断
```
void ABaseCharacter::Attack()
{
	if (CombatTarget && CombatTarget->ActorHasTag(FName("Dead"))) {
		CombatTarget = nullptr;
	}
}

void ABaseCharacter::Die()
{
	Tags.Add(FName("Dead"));
	PlayDeathMontage();
}
```
然后在Enemy.cpp中
如果CombatTarget被设为nullptr即代表死亡，直接return
```
void AEnemy::Attack()
{	
	Super::Attack();
	if (CombatTarget == nullptr)return;

	EnemyState = EEnemyState::EES_Engaged;
	PlayAttackMontage();
}
```
# 灵魂球
制作一个以Item为父类的c++类，叫Soul
## 创建一个接口用来处理Pickup的东西
此事情的目的是为了避免先前在Item中使用SlashCharacter类，如下：
```
void AItem::OnSphereOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
	ASlashCharacter* SlashCharacter = Cast<ASlashCharacter>(OtherActor);
	if (SlashCharacter)
	{
		SlashCharacter->SetOverlappingItem(this);
	}
}
```
在PickupInterface.h中声明一个虚函数并在cpp实现，但是不补充任何内容，因为目的只是为了做一个接口
```
virtual void SetOverlappingItem(class AItem* Item);
```
我们删除先前在SlashCharacter.h中定义的内联函数，使用新声明的接口
```
FORCEINLINE void SetOverlappingItem(AItem* Item) { OverlappingItem = Item; }
```
```
#include "Interfaces/PickupInterface.h"
class SLASH_API ASlashCharacter : public ABaseCharacter, public IPickupInterface
{
	GENERATED_BODY()
	...
virtual void SetOverlappingItem(AItem* Item) override;
	...
}
```
在SlashCharacter.cpp中实现方法
```
void ASlashCharacter::SetOverlappingItem(AItem* Item)
{
	OverlappingItem = Item;
}
```
最后我们可以在Item.cpp中修改成IPickupInterface，就可以完美避免之间使用SlashCharacter，而且也更加灵活了
```
void AItem::OnSphereOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
	IPickupInterface* PickupInterface = Cast<IPickupInterface>(OtherActor);
	if (PickupInterface)
	{
		PickupInterface->SetOverlappingItem(this);
	}
}
```
继续利用这个接口，声明AddSouls
```
virtual void AddSouls(class ASoul* Soul);
```
并实现但不写任何东西，去到SlashCharacter中override
```
void ASlashCharacter::AddSouls(ASoul* Soul)
{
	UE_LOG(LogTemp, Warning, TEXT("ASlashCharacter::AddSouls"));
}
```
在Soul.cpp中重写OnSphereOverlap方法，此时和Soul重叠就会触发AddSouls函数。而且此时PickupInterface是主角调用的，也侧面展示了接口灵活性。
捡起Soul后Destroy消灭
```
#include "Interfaces//PickupInterface.h"

void ASoul::OnSphereOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{

	IPickupInterface* PickupInterface = Cast<IPickupInterface>(OtherActor);
	if (PickupInterface)
	{
		PickupInterface->AddSouls(this);
	}
	Destroy();
}
```
## 捡拾灵魂球效果
在Item.h中声明一个PickupEffect
```
private:
	UPROPERTY(EditAnywhere)
	class UNiagaraSystem* PickupEffect;
```
在Item.h中声明一个virtual的函数用来spawn捡起的效果
```
virtual void SpawnPickupSystem();
```
然后我们就能进入Item.cpp中，导入`NiagaraFunctionLibrary.h`头文件，使用`SpawnSystemAtLocation`函数
```
void AItem::SpawnPickupSystem()
{
	if (PickupEffect) {
		UNiagaraFunctionLibrary::SpawnSystemAtLocation(this, PickupEffect, GetActorLocation());
	}
}
```
然后到Soul.cpp中的重叠球的方法中调用此方法
```
void ASoul::OnSphereOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{

	IPickupInterface* PickupInterface = Cast<IPickupInterface>(OtherActor);
	if (PickupInterface)
	{
		PickupInterface->AddSouls(this);
		SpawnPickupSystem();
		Destroy();
	}
	
}
```
最后去蓝图中选择此Effect

![输入图片说明](/imgs/2024-11-02/iJDK3MqFUJuOku6a.png)

我们还希望加入捡拾的音效，在Item.h中制作一个变量以及函数用来播放声音
```
UPROPERTY(EditAnywhere)
USoundBase* PickupSound;

virtual void SpawnPickupSound();
```
```
#include "Kismet/GameplayStatics.h"
void AItem::SpawnPickupSound()
{
	if (PickupSound) {
		UGameplayStatics::SpawnSoundAtLocation(this, PickupSound, GetActorLocation());
	}
}
```
同样回去Soul.cpp中调用
```
void ASoul::OnSphereOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
	...
	SpawnPickupSystem();
	SpawnPickupSound();
	Destroy();
}
```
# 制作Gold和Souls数量计算
到AttributeComponent.h中进行变量声明，此component专门用来声明角色身上的属性
```
private:
	UPROPERTY(EditAnywhere, Category = "Actor Attributes")
	int32 Gold;

	UPROPERTY(EditAnywhere, Category = "Actor Attributes")
	int32 Souls;
public:
	void AddSouls(int32 NumberOfSouls);
	void AddGold(int32 NumberOfGold);
	FORCEINLINE int32 GetGold() const { return Gold; }
	FORCEINLINE int32 GetSouls() const { return Souls; }
```
然后去到AttributeComponent.cpp中实现：
```
void UAttributeComponent::AddSouls(int32 NumberOfSouls)
{
	Souls += NumberOfSouls;
}
void UAttributeComponent::AddGold(int32 NumberOfGold)
{
	Gold += NumberOfGold;
}
```
翻回我们之前在SlashCharacter.cpp中做的方法，我们发现这里的参数是`ASoul* Soul`，这说明我们可以到Soul中进行价值的声明，这里传进来可以是一个或者十个或者一百个，
这说明在面对不同的敌人可以对他们的Soul值进行不同的声明！！！
```
void ASlashCharacter::AddSouls(ASoul* Soul)
{
	UE_LOG(LogTemp, Warning, TEXT("ASlashCharacter::AddSouls"));
}
```
所以我们回到Soul.h中，声明一个Souls，这里的Souls和上面AttributeComponent的Souls可不一样，前者是Soul类中的Souls，用来增加不同敌人不同Souls的复用性，而AttributeComponent中的Souls是用来计算玩家身上拥有多少个Souls数量
```
private:
	UPROPERTY(EditAnywhere, Category = "Soul Properties")
	int32 Souls;
public:
	FORCEINLINE int32 GetSouls() const { return Souls; }
```
然后，我们终于可以用上了，去SlashCharacter.cpp中，使用刚刚做的函数
```
#include "I/Soul.h"
#include "I/Treasure.h"
void ASlashCharacter::AddSouls(ASoul* Soul)
{
	if (Attributes) {
		Attributes->AddSouls(Soul->GetSouls());
	}
}
```
## 补齐Gold的逻辑
在PickupInterface.h中依照AddSouls做一个AddGold
```
public:
	virtual void SetOverlappingItem(class AItem* Item);
	virtual void AddSouls(class ASoul* Soul);
	virtual void AddGold(class ATreasure* Treasure);
```
在SlashCharacter.h中也按照AddSouls override 一个AddGold
```
virtual void AddSouls(ASoul* Soul) override;
virtual void AddGold(ATreasure* Treasure) override;
```
并且去SlashCharacter.cpp中实现
```
void ASlashCharacter::AddGold(ATreasure* Treasure)
{
	if (Attributes) {
		Attributes->AddGold(Treasure->GetGold());
	}
}
```
最后，可以去蓝图中点击灵魂球直接给它设置Souls的价值！！！

![输入图片说明](/imgs/2024-11-02/mMUvdP6YnGsVP9rM.png)

# 敌人掉落Soul
去Enemy.h中声明SoulClass，提供更灵活的设计，不同的敌人可以去各自的蓝图中设置Soul的数值
```
UPROPERTY(EditAnywhere, Category = Combat)
TSubclassOf<class ASoul> SoulClass;
```
在Enemy.cpp中做一个SpawnSoul()函数，并在Die()函数的最后调用
```
void AEnemy::SpawnSoul()
{
	UWorld* World = GetWorld();
	if (World && SoulClass) {
		ASoul* SpawnedSoul = World->SpawnActor<ASoul>(SoulClass, GetActorLocation(), GetActorRotation());
	}
}
```
此时我们希望对Soul可以进行属性的赋值
去Soul.h中建一个内联函数，用来设置Soul的数量
```
FORCEINLINE void SetSouls(int32 NumberOfSouls){ Souls = NumberOfSouls; }
```
回到Enemy.cpp做出完善，为生成的Souls进行赋值
```
void AEnemy::SpawnSoul()
{
	UWorld* World = GetWorld();
	if (World && SoulClass && Attributes) {
		ASoul* SpawnedSoul = World->SpawnActor<ASoul>(SoulClass, GetActorLocation(), GetActorRotation());
		if (SpawnedSoul) {
			SpawnedSoul->SetSouls(Attributes->GetSouls());
		}
	}
}
```
最后我们去Enemy蓝图中将BP_Soul添加上去

![输入图片说明](/imgs/2024-11-04/DPIgJWpVFCKiPmhn.png)

并且，我们之前对AttributeComponent中将Gold和Souls设为EditAnywhere，此处就可以很方便的针对每个敌人进行赋值！！！

![输入图片说明](/imgs/2024-11-04/0uI8KRcyx88E9MhY.png)

## 发现bug
在Soul生成的一瞬间我们已经在灵魂球的重叠框中，此时会被捡拾起来但是不会累加值，我们只需要将灵魂球的重叠框做的小一点就好了！
# Dodge
新建操作映射

![输入图片说明](/imgs/2024-11-05/kwSduZ5ZbU2sLPGq.png)

在SlashCharacter.h中声明函数
```
void Dodge();
```
并在cpp实现以及绑定
```
void ASlashCharacter::Dodge(){}
```
```
void ASlashCharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	Super::SetupPlayerInputComponent(PlayerInputComponent);
	...
	PlayerInputComponent->BindAction(FName("Dodge"), IE_Pressed, this, &ASlashCharacter::Dodge);
}
```
我们去到BaseCharacter.h中声明Dodge蒙太奇，以防以后我们敌人也想制作dodge效果，此处dodge只有一个动画不需要随机，声明为void即可
```
virtual void PlayDodgeMontage();
UPROPERTY(EditDefaultsOnly, Category = Combat)
UAnimMontage* DodgeMontage;
```
到BaseCharacter.cpp中完善方法
```
void ABaseCharacter::PlayDodgeMontage()
{
	PlayMontageSection(DodgeMontage, FName("Default"));
}
```
最后我们回到SlashCharacter.h中，调用父类方法即可。
```
void ASlashCharacter::Dodge()
{
	PlayDodgeMontage();
}
```
### Dodge状态修复
因为我们希望在Dodge的过程中不会被打断
在CharacterTypes.h枚举类中添加一个Dodge状态
```
EAS_Dodge UMETA(DisplayName = "Dodge")
```
在Dodge开始的时候将状态设为Dodge，那就意味着我们也需要一个从Dodge设置回Unoccupied的函数
```
void ASlashCharacter::Dodge()
{
	if (ActionState != EActionState::EAS_Unoccupied)return;
	PlayDodgeMontage();
	ActionState = EActionState::EAS_Dodge;
}
```
我们去BaseCharacter.h中做这个事
```
UFUNCTION(BlueprintCallable)
virtual void DodgeEnd();
```
并在cpp中实现，但是我们什么都不补充，而是去SlashCharacter.h中进行override并实现
```
void ASlashCharacter::DodgeEnd()
{
	Super::DodgeEnd();
	ActionState = EActionState::EAS_Unoccupied;
}
```
我们怎么调用这个DodgeEnd呢
动画通知！！！

![输入图片说明](/imgs/2024-11-05/tQhDewPK9nCrsMtM.png)

然后去动画蓝图通过动画通知调用函数。
perfect，以上就完成了Dodge功能
![输入图片说明](/imgs/2024-11-05/Sscaj8nNiiqifYMu.png)

# 耐力消耗
在`attributeComponent.h`中模仿`health`建立有关`stamina`的变量和方法
```
private:
UPROPERTY(EditAnywhere, Category = "Actor Attributes")
float Stamina;
UPROPERTY(EditAnywhere, Category = "Actor Attributes")
float MaxStamina;
public:
void UseStamina(float StaminaCost);
float GetStaminaPercent();
```
在`attributeComponent.cpp`中实现
```
void UAttributeComponent::UseStamina(float StaminaCost)
{
	Stamina = FMath::Clamp(Stamina - StaminaCost, 0.f, MaxStamina);
}
float UAttributeComponent::GetStaminaPercent()
{
	return Stamina / MaxStamina;
}
```
在`SlashCharacter.cpp`中完善之前的闪避，没有耐力无法闪避
```
void ASlashCharacter::Dodge()
{
	if (ActionState != EActionState::EAS_Unoccupied)return;
	if (Attributes && Attributes->GetStaminaPercent() > 0.f) {
		PlayDodgeMontage();
		ActionState = EActionState::EAS_Dodge;
	}
}
```
在`attributeComponent.h`声明`DodgeCost`，并且我们需要一个public的方法来获取`cost`和`Stamina`
```
private:
	UPROPERTY(EditAnywhere, Category = "Actor Attributes")
	float DodgeCost = 14.f;
public:
	FORCEINLINE int32 GetDodgeCost() const { return DodgeCost; }
	FORCEINLINE int32 GetStamina() const { return Stamina; }
```
我们就可以回去`SlashCharacter.cpp`中完善方法，`Attributes->GetStamina() > Attributes->GetDodgeCost()`才能翻滚
```
bool ASlashCharacter::HasEnoughStamina()
{
	return Attributes && Attributes->GetStamina() > Attributes->GetDodgeCost();
}
```
以及在dodge函数中进行耐力的消耗，以及在对应UI进行衰减。
`IsOccupied() || !HasEnoughStamina()`注意这里应该用`or`而不是`and`
```
void ASlashCharacter::Dodge()
{
	if (IsOccupied() || !HasEnoughStamina())return;

	PlayDodgeMontage();
	ActionState = EActionState::EAS_Dodge;
	if (Attributes && SlashOverlay) {
		Attributes->UseStamina(Attributes->GetDodgeCost());
		SlashOverlay->SetStaminaBarPercent(Attributes->GetStaminaPercent());
	}
}
```
当然耐力既然消耗那也就需要回复
在`attributeComponent.h`中声明一个回复的数值和方法
```
private:
	UPROPERTY(EditAnywhere, Category = "Actor Attributes")
	float StaminaRegenRate = 8.f;
public:
	void RegenStamina(float DeltaTime);
```
在`attributeComponent.cpp`实现
```
void UAttributeComponent::RegenStamina(float DeltaTime)
{
	Stamina = FMath::Clamp(Stamina + StaminaRegenRate * DeltaTime, 0.f, MaxStamina);
}
```
我们去到`SlashCharacter.h`中声明Tick函数（原本应该有的但是被我们删掉了）
```
virtual void Tick(float DeltaTime) override;
```
`SlashCharacter.cpp`实现：
```
void ASlashCharacter::Tick(float DeltaTime)
{
	if (Attributes && SlashOverlay) {
		Attributes->RegenStamina(DeltaTime);
		SlashOverlay->SetStaminaBarPercent(Attributes->GetStaminaPercent());
	}
}
```
要使用Tick函数还有这个重要前提，在构造函数中把`bCanEverTick`改成`true`
```
ASlashCharacter::ASlashCharacter()
{
	PrimaryActorTick.bCanEverTick = true;
	...
}
```
# 模板
我们会有各式各样的敌人，我们用模板就可以复用动画蓝图的功能
右键->动画->动画蓝图。
选择模板创建

![输入图片说明](/imgs/2024-11-20/fFlyFhXwsWKJucJA.png)

所以我们现在重新理一遍paladin的动画蓝图逻辑并且移植过来
1. 第一步就是重载函数蓝图线程安全更新动画

![输入图片说明](/imgs/2024-11-20/1OvGa26pLFRmscpS.png)

2. 声明变量

![输入图片说明](/imgs/2024-11-20/LXx0MI8iqgmPHrUt.png)

3. 初始化基础变量包括（角色、移动）

![输入图片说明](/imgs/2024-11-20/XhUqgbHHjH6fiBGe.png)

4. 剩余变量利用线程更新赋值

![输入图片说明](/imgs/2024-11-20/JNoEWcWVj9XJHi8f.png)

5. 动画状态机按照老样子设计即可

![输入图片说明](/imgs/2024-11-20/bpnViejbHAZwNnmn.png)

6. 在死亡动画，由于我们希望的是多个敌人可以灵活使用的模板
我们选择sequence player

![输入图片说明](/imgs/2024-11-20/Fjy9AkYixApNAJlv.png)

在右侧面板中我们可以将其sequence公开为引脚

![输入图片说明](/imgs/2024-11-20/FHfPCI5FFIUIF74P.png)

把引脚公开为变量

![输入图片说明](/imgs/2024-11-20/AVp2huIkFeATCHIX.png)

把这些变量修改名字一一对应

![输入图片说明](/imgs/2024-11-20/0491XrDWxcyVg9te.png)

7. 对于blend space， 我们用blend space player。
总的来说反正就是避免直接用专属动画，都用这种播放器来代替。
![输入图片说明](/imgs/2024-11-20/7WSOjs7ae3qEbLoU.png)

这里同样可以把blend space像刚刚那样暴露出来，但我们可以尝试另一种做法，等会介绍

![输入图片说明](/imgs/2024-11-20/XXtMUfUC73vBXl5M.png)
## 创建子类
用我们刚刚的模板创建子类动画蓝图，骨骼就用paladin

![输入图片说明](/imgs/2024-11-20/DVwrXhilnkob3t03.png)

进来后我们就可以看得到我们刚刚做的变量，很不错

![输入图片说明](/imgs/2024-11-20/YDlZQsXTKie9mFD9.png)

补充刚刚的坑，在这里我们进行blender space的赋值。

![输入图片说明](/imgs/2024-11-20/ArZXGLQBJbD9Hc4Z.png)

小提一嘴，为什么我们刚刚deadpose不用这样
其实这里也是可以赋值的，但是我们看不到我们自己取的名字，无法知道是1还是6

![输入图片说明](/imgs/2024-11-20/1F63yIruvqABgEHF.png)

# 新敌人Raptor
用我们做的模板进行动画蓝图创建，为其赋上死亡和blend运动

![输入图片说明](/imgs/2024-11-23/PEKRv9tZVWvxqIrG.png)
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTMxODY5OTg3OSwtMjM1NjAwMTIsLTIwMT
E3NjM2MCwtMTc2Mzg2MTg5MCwyMTEyODM3MzU3LDQyMDYxNDE0
OSwtNDUzMzQ1Mzk5LDE3MzcwNDQzODksMTEzOTYxNDMxLC0yNz
U3NTk1NTAsLTE5MTMzMTEwNzIsMTcyMTA5NzU0NCwtNDQ5NTQ5
MTcxLC0xMDU1NjA3MzIwLDEyMjg3NzE2MTgsLTEzMTE2MjQwMT
MsLTE2NjY1MDQ5MTMsLTI5Mzg3ODM3NCwtMjE0Njk5NDk5OCwt
MTA3OTczMjcyOF19
-->