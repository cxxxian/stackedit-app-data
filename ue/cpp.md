## DeBug
### UE_LOG
在输出日志中输出，
```
UE_LOG(LogTemp, Warning, TEXT("begin play called"));
```
### GEngine->AddOnScreenDebugMessage
直接输出在屏幕上。
此处GEngine是个指针可能为空，空指针会导致程序崩溃，所以加上if判断。
```
if (GEngine) {
		GEngine->AddOnScreenDebugMessage(1, 60.f, FColor::Cyan, FString("Item OnScreen Message"));
	}
```
### FString
想要格式化Name字符串，必须在前面加上*，C style
![输入图片说明](/imgs/2024-07-31/jm6yKw7MBWMWw55T.jpeg)
```
FString Name = GetName();
FString Message = FString::Printf(TEXT("Item Name: %s"), *Name);
```
### DrawDebugSphere
在actor的位置处绘制一个球形Debug体
```
UWorld* World = GetWorld();
	if (World) {
		FVector Location = GetActorLocation();
		DrawDebugSphere(World, Location, 25.f, 24, FColor::Red, false, 30.f);
	}
```
### DrawDebugLine
```
FVector Location = GetActorLocation();
	if (World) {
		FVector Forward = GetActorForwardVector();
		DrawDebugLine(World, Location, Location + Forward * 100.f, FColor::Red, true, -1.f, 0, 1.f);
	}
```
## Actor
### SetActorLocation
设置actor的位置
```
SetActorLocation(FVector(0.f, 0.f, 50.f));
```
### SetActorRotation
设置actor的旋转
```
SetActorRotation(FRotator(0.f, 45.f, 0.f));
```
### AddActorWorldOffset
actor移动增量:
- 每帧（frame）向前移动1.f（即一厘米）
```
AddActorWorldOffset(FVector(1.f, 0.f, 0.f));
```
- 每秒移动50.f
MovementRate（cm/s） * DeltaTime（s/frame）= （cm/frame）
```
float MovementRate = 50.f;
AddActorWorldOffset(FVector(MovementRate * DeltaTime, 0.f, 0.f));
```
### AddActorWorldRotation
float RotationRate = 45.f;
- 每秒转45.f
```
AddActorWorldRotation(FRotator(0.f, RotationRate * DeltaTime, 0.f));
```
### Sin
Amplitude用来控制振幅
TimeConstant用来控制时间
RunningTime随着每帧运行不断增加
```
RunningTime += DeltaTime;

	float DeltaZ = Amplitude * FMath::Sin(RunningTime * TimeConstant);
```

### UPROPERTY
EditDefaultsOnly将变量暴露给默认蓝图
EditInstanceOnly将变量暴露给蓝图示例
EditAnywhere同时将变量暴露给默认蓝图和蓝图示例
```
UPROPERTY(EditDefaultsOnly)
float Amplitude = 0.25f;

UPROPERTY(EditInstanceOnly)
float TimeConstant = 5.f;

UPROPERTY(EditAnywhere)
UPROPERTY(VisibleDefaultsOnly)
UPROPERTY(VisibleInstanceOnly)
UPROPERTY(VisibleAnywhere)
```
在蓝图中获取变量（**前提是变量在.h文件中需在protected而不能在private中**）
BlueprintReadOnly只能get到变量
BlueprintReadWrite即可get也可set
```
UPROPERTY(EditAnywhere, BlueprintReadOnly);
float Amplitude = 0.25f;

UPROPERTY(EditAnywhere, BlueprintReadWrite);
```
如需要在private下定义的情况下，使用meta = (AllowPrivateAccess = "true")，同样也可以达到在蓝图中获取变量的效果
```
private:
	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, meta = (AllowPrivateAccess = "true"))
	float RunningTime;
```
对于变量进行分类
Category = "Sine Parameters"，将Amplitude和TimeConstant两个变量分类在Sine Parameters下
```
UPROPERTY(EditAnywhere, BlueprintReadWrite,Category = "Sine Parameters");
float Amplitude = 0.25f;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Sine Parameters")
float TimeConstant = 5.f;
```
### UFUNCTION
将函数暴露给蓝图
```
UFUNCTION(BlueprintCallable)
float TransformedSin(float value);
```
以纯函数的形式暴露给蓝图
```
UFUNCTION(BlueprintPure)
float TransformedSin(float value);
```

### template
模板函数的使用
```
template<typename T>
T Avg(T First, T Second);
```
内联函数构造，类似于宏的替换，替换上述定义的方法
```
template<typename T>
inline T AItem::Avg(T First, T Second)
{
	return (First + Second) / 2;
}
```
函数的使用（ue中的int为int32）
灵活替换模板中的数据类型
```
int32 AvgInt = Avg<int32>(1, 3);
UE_LOG(LogTemp, Warning, TEXT("Avg of 1 and 3: %d"), AvgInt);

float AvgFloat = Avg<float>(3.45f, 7.86f);
	UE_LOG(LogTemp, Warning, TEXT("Avg of 3.45 and 7.86: %f"), AvgFloat);
```
### Component
在.cpp文件中，创建默认子对象，子对象的类型为静态网格体组件（UStaticMeshComponent），使用文本宏给该组件添加内部名字TEXT("ItemMeshComponent")。
```
CreateDefaultSubobject<UStaticMeshComponent>(TEXT("ItemMeshComponent"));
```
以上该函数返回一个指向新创建对象的指针，赋值给在.h中声明的ItemMesh。
在.h中声明的ItemMesh初始状态下只是一个空指针，需要创建新组建并将地址存储在这个指针中。被指针指向的对象，引擎会识别出有指针指向的对象仍然在被使用，而不会被当作垃圾被收集删除。
```
UPROPERTY(VisibleAnywhere)
UStaticMeshComponent* ItemMesh;
```
RootComponent = ItemMesh此句话将根组件指针变量存储的默认场景根改为ItemMesh，此时默认场景根没有被指针所引用，所以默认场景根会被自动删除。
```
ItemMesh = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("ItemMeshComponent"));
RootComponent = ItemMesh;
```
所以综上，目前根组件指针指向新创建的项目网格子对象即ItemMesh。
编译后ue5生成如下：
ItemMesh是声明的指针，ItemMeshComponent是文本宏创建的名字
![输入图片说明](/imgs/2024-08-01/HWf3S6b64VphAP2F.png)

## Pawn
### Capsule
头文件
```
#include "Components/CapsuleComponent.h"
```
构建胶囊体
在.h中声明如下
```
private:
	UPROPERTY(VisibleAnywhere)
	UCapsuleComponent* Capsule;
```
在.cpp中声明如下
SetRootComponent(Capsule);等同于RootComponent = Capsule;
都能改变默认根组件.
中间两句话用来设置胶囊体的高和半径
```
Capsule = CreateDefaultSubobject<UCapsuleComponent>(TEXT("Capsule"));
Capsule->SetCapsuleHalfHeight(20.f);
Capsule->SetCapsuleRadius(15.f);
SetRootComponent(Capsule);
```
如果使用此形式声明，在前面加上class，即可以不使用头文件导入
#include "Components/CapsuleComponent.h"
```
private:
	UPROPERTY(VisibleAnywhere)
	class UCapsuleComponent* Capsule;
```
也可在程序开头处写**class UCapsuleComponent**，则之后遇到UCapsuleComponent* Capsule;即可自动识别（两种方法都可）
```
private:
	UPROPERTY(VisibleAnywhere)
	UCapsuleComponent* Capsule;
```
### SkeletalMesh
头文件
```
#include "Components/SkeletalMeshComponent.h"
```
在.h中声明如下
```
UPROPERTY(VisibleAnywhere)
USkeletalMeshComponent* BirdMesh;
```
在.cpp中声明如下，SetupAttachment用来将BirdMesh附加到根组件下面，此处用GetRootComponent()，RootComponent，或者Capsule都是可以的，都是根组件的引用。
```
BirdMesh = CreateDefaultSubobject<USkeletalMeshComponent>(TEXT("BirdMesh"));
BirdMesh->SetupAttachment(GetRootComponent());
```
设置控制器
```
AutoPossessPlayer = EAutoReceiveInput::Player0;
```
### Input
在.h中先声明一个函数，并在对应的.cpp中进行构造
```
protected:
void MoveForward(float Value);
```
在.cpp中：
```
void ABird::MoveForward(float Value)
{
}
```
在此方法中将我们定义的MoveForward进行轴映射绑定
- 方法中的第一个参数传入字符串（可以是直接"MoveForward"，或者通过文字宏命名TEXT("MoveForward"),或者使用构造方法FName("MoveForward")），此字符串将与项目设置中的输入映射进行文字上的对应
![输入图片说明](/imgs/2024-08-02/dG7R3fmfJBPHYPHE.png)
- 第二个参数传入this指针，指的是控制器所控制的的对象为目前的对象，即Bird
- 第三个参数是传入对应函数，直接传入函数的地址即&ABird::MoveForward，需要将类限定ABird::一起传入
```
void ABird::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	Super::SetupPlayerInputComponent(PlayerInputComponent);

	PlayerInputComponent->BindAxis(FName("MoveForward"), this, &ABird::MoveForward);
}
```
完善MoveForward方法，已知按下W键即前进的时候，Value等于1，满足if条件，得到前进向量，调用AddMovementInput(Forward, Value)，此时value为1，会向着Forward的方向运动，如果是-1则对向量取反（对应S键的值为-1），则会倒退
```
void ABird::MoveForward(float Value)
{
	if ((Controller != nullptr) && (Value != 0.f)) {
		FVector Forward = GetActorForwardVector();
		AddMovementInput(Forward, Value);
	}
}
```
需要手动添加运动组件**FloatingPawnMovement**，即可进行前进后退的运动。
### Camera && Spring Arm
在. h中声明ViewCamera和SpringArm，在程序开头需要声明（满足向前声明）
```
class USpringArmComponent
class UCameraComponent
```
```
UPROPERTY(VisibleAnywhere)
USpringArmComponent* SpringArm;

UPROPERTY(VisibleAnywhere)
UCameraComponent* ViewCamera;
```
在.cpp中，创建SpringArm并且将其添加到根组件上
```
#include "GameFramework/SpringArmComponent.h"
#include "Camera/CameraComponent.h"
SpringArm = CreateDefaultSubobject<USpringArmComponent>(TEXT("SpringArm"));
SpringArm->SetupAttachment(GetRootComponent());
```
创建ViewCamera并且将其添加到SpringArm上
```
#include "Camera/CameraComponent.h"
ViewCamera = CreateDefaultSubobject<UCameraComponent>(TEXT("ViewCamera"));
ViewCamera->SetupAttachment(SpringArm);
```
### 左右及上下转向
在.h中声明方法
```
void Turn(float Value);
void LookUp(float Value);
```
在.cpp中进行方法的构造
```
void ABird::Turn(float Value)
{
    AddControllerYawInput(Value);
}

void ABird::LookUp(float Value)
{
    AddControllerPitchInput(Value);
}
```
在SetupPlayerInputComponent方法中进行周映射绑定
```
PlayerInputComponent->BindAxis(FName("Turn"), this, &ABird::Turn);
PlayerInputComponent->BindAxis(FName("LookUp"), this, &ABird::LookUp);
```
**注意**：记得要在蓝图中将**使用控制器旋转Pitch/Yaw**勾选上才能正确进行旋转
![输入图片说明](/imgs/2024-08-06/tgWzvJyAl1jnLy1i.png)
## Character
与bird类一样进行相机弹簧臂的绑定，以及前进后退的实现
在构造函数中，对以下三个controller旋转的初始化
```
bUseControllerRotationPitch = false;
bUseControllerRotationYaw = false;
bUseControllerRotationRoll = false;
```
优化原来只能沿着向前向量移动的问题，以下方法：
> 1.通过GetControlRotation()得到控制器旋转的向量，用ControlRotation存储起来
> 2.通过YawRotation(0.f, ControlRotation.Yaw, 0.f)构造器得到ControlRotation的Yaw方向的向量
> 3.FRotationMatrix(YawRotation).GetUnitAxis(EAxis::X)，先得到旋转值的旋转矩阵表示方法，然后得到X方向上的单位向量
> 4.最后将Direction作为向量传入AddMovementInput方法中
```
void ASlashCharacter::MoveForward(float Value)
{
	if (Controller && (Value != 0.f))
	{
		const FRotator ControlRotation = GetControlRotation();
		const FRotator YawRotation(0.f, ControlRotation.Yaw, 0.f);

		const FVector Direction = FRotationMatrix(YawRotation).GetUnitAxis(EAxis::X);
		AddMovementInput(Direction, Value);
	}
}
void ASlashCharacter::MoveRight(float Value)
{
	if (Controller && (Value != 0.f))
	{
		const FRotator ControlRotation = GetControlRotation();
		const FRotator YawRotation(0.f, ControlRotation.Yaw, 0.f);

		const FVector Direction = FRotationMatrix(YawRotation).GetUnitAxis(EAxis::Y);
		AddMovementInput(Direction, Value);
	}
}
```
设置运动方向朝向旋转，且调节旋转速度
```
GetCharacterMovement()->bOrientRotationToMovement = true;
GetCharacterMovement()->RotationRate = FRotator(0.f, 400.f, 0.f);
```
### 头发和眉毛的设置
需要在build.cs内手动添加对应的模块"HairStrandsCore"和"Niagara"
```
PublicDependencyModuleNames.AddRange(new string[] { "Core", "CoreUObject", "Engine", "InputCore", "HairStrandsCore", "Niagara" });
```
在.h中声明对应的变量Hair和Eyebrows
```
class UGroomComponent;
UPROPERTY(VisibleAnywhere, Category = Hair)
UGroomComponent* Hair;

UPROPERTY(VisibleAnywhere, Category = Hair)
UGroomComponent* Eyebrows;
```
在.cpp中的构造函数中进行赋值和添加到对应的mesh上
```
Hair = CreateDefaultSubobject<UGroomComponent>(TEXT("Hair"));
Hair->SetupAttachment(GetMesh());
Hair->AttachmentName = FString("head");

Eyebrows = CreateDefaultSubobject<UGroomComponent>(TEXT("Eyebrows"));
Eyebrows->SetupAttachment(GetMesh());
Eyebrows->AttachmentName = FString("head");
```
![输入图片说明](/imgs/2024-08-07/tM0qzH0fPaw5mcBn.png)
### 动画
![输入图片说明](/imgs/2024-08-10/n4SfnX3mOEJzNTRx.png)
![输入图片说明](/imgs/2024-08-08/Zife25LjEgqAihOI.png)
对应蓝图，在.h中声明这两个方法，声明SlashCharacter，SlashCharacterMovement（移动组件），GroundSpeed（地面速度）
```
public:
	virtual void NativeInitializeAnimation() override;
	virtual void NativeUpdateAnimation(float DeltTime) override;

	UPROPERTY(BlueprintReadOnly)
	class ASlashCharacter* SlashCharacter;

	UPROPERTY(BlueprintReadOnly, Category = Movement)
	class UCharacterMovementComponent* SlashCharacterMovement;

	UPROPERTY(BlueprintReadOnly, Category = Movement)
	float GroundSpeed;
```
在.cpp中引用以下头文件
```
#include "Characters/SlashAnimInstance.h"
#include "Characters/SlashCharacter.h"
#include "GameFramework/CharacterMovementComponent.h"
#include "Kismet/KismetMathLibrary.h"
```
并实现方法
```
void USlashAnimInstance::NativeInitializeAnimation()
{
	Super::NativeInitializeAnimation();//继承父类

	SlashCharacter = Cast<ASlashCharacter>(TryGetPawnOwner());//这句话用来将TryGetPawnOwner()得到的对象转化成ASlashCharacter，对应蓝图中的cast to方法
	if (SlashCharacter)//如果成功转化了SlashCharacter
	{
		SlashCharacterMovement = SlashCharacter->GetCharacterMovement();//赋值SlashCharacterMovement
	}
}
```
VSizeXY用来计算矢量的长度，SlashCharacterMovement->Velocity得到该移动组件的速度矢量
```

void USlashAnimInstance::NativeUpdateAnimation(float DeltTime)
{
	Super::NativeUpdateAnimation(DeltTime);//继承父类

	if (SlashCharacterMovement)//如果得到了SlashCharacterMovement
	{
		GroundSpeed = UKismetMathLibrary::VSizeXY(SlashCharacterMovement->Velocity);//计算速度矢量在xy面的长度
	}
}
```
![输入图片说明](/imgs/2024-08-10/gEPKReLtXYM58RQG.png)
最后在动画蓝图的类设置中更改父类
### Jump
在操作映射中添加跳跃
![输入图片说明](/imgs/2024-08-08/CVr0sAXab2FBwoMp.png)
此处用的是操作映射而不是轴映射
第一个参数对应名字，第二个参数为一个枚举类，选择IE_Pressed意为按下后生效
```
PlayerInputComponent->BindAction(FName("Jump"), IE_Pressed, this, &ACharacter::Jump);
```
![输入图片说明](/imgs/2024-08-08/TSHFGoJpVtTVxO9v.png)
通过缓存姿势可以做到如上效果，然后在其他状态机内即可使用Ground Locomoton
## 委托
### 重叠事件
在.h中声明如下
```
class USphereComponent;
protected:
UFUNCTION()
void OnSphereOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult);
private:
UPROPERTY(VisibleAnywhere)
USphereComponent* Sphere;
```
在beginplay中绑定该函数，在函数中实现重叠时输出重叠角色的名字
```
void AItem::BeginPlay()
{
	Super::BeginPlay();
	Sphere->OnComponentBeginOverlap.AddDynamic(this, &AItem::OnSphereOverlap);
}
void AItem::OnSphereOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
	const FString OtherActorName = OtherActor->GetName();
	if (GEngine) {
		GEngine->AddOnScreenDebugMessage(1, 30.f, FColor::Red, OtherActorName);
	}
}
```
## Weapon
由于weapon和和先前定义item有共同之处
![输入图片说明](/imgs/2024-08-09/bjdJt3ILYNcQVzD1.png)
直接通过item创建子类，将原本在item中的重叠方法改成virtual，在weapon中进行override覆盖
```
virtual void OnSphereOverlap
virtual void OnSphereEndOverlap
```
在weapon.h中写方法并override，此时不可以写FUNCTION否则会导致编译错误，因为本质上是继承关系
```
virtual void OnSphereOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult) override;

virtual void OnSphereEndOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex) override;
```
### 将weapon添加到手上（自动拾取）
在.cpp中引入头文件使用人物，完善重叠方法
```
#include "Characters/SlashCharacter.h"
void AWeapon::OnSphereOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
	Super::OnSphereOverlap(OverlappedComponent, OtherActor, OtherComp, OtherBodyIndex, bFromSweep, SweepResult);

	ASlashCharacter* SlashCharacter = Cast<ASlashCharacter>(OtherActor);//将OtherActor转化成ASlashCharacter，如果失败SlashCharacter则会是null
	if (SlashCharacter)//检测是否cast成功
	{
		FAttachmentTransformRules TransformRules(EAttachmentRule::SnapToTarget, true);//
		ItemMesh->AttachToComponent(SlashCharacter->GetMesh(), TransformRules, FName("RightHandSocket"));
	}
}
```
### 将weapon添加到手上（按键E拾取）
在SlashCharacter.h中声明方法，并之后在.cpp中实现，声明了一个内联函数，FORCEINLINE意为强制内联。
```
void EKeyPressed();
public:
	FORCEINLINE void SetOverlappingItem(AItem* Item) { OverlappingItem = Item; }
```
在SlashCharacter.cpp中将事件绑定
```
PlayerInputComponent->BindAction(FName("Equip"), IE_Pressed, this, ASlashCharacter::EKeyPressed);
```
在item.cpp中修改两个重叠函数，重叠的时候将item作为参数传入方法中，结束重叠的时候传入nullptr，意为重叠结束
```
void AItem::OnSphereOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
	ASlashCharacter* SlashCharacter = Cast<ASlashCharacter>(OtherActor);
	if (SlashCharacter)
	{
		SlashCharacter->SetOverlappingItem(this);
	}
}

void AItem::OnSphereEndOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex)
{
	ASlashCharacter* SlashCharacter = Cast<ASlashCharacter>(OtherActor);
	if (SlashCharacter)
	{
		SlashCharacter->SetOverlappingItem(nullptr);
	}
}
```
在weapon.h中声明并在.cpp中定义此函数
```
void AWeapon::Equip(USceneComponent* Inparent, FName InSocketName)
{
	FAttachmentTransformRules TransformRules(EAttachmentRule::SnapToTarget, true);
	ItemMesh->AttachToComponent(Inparent, TransformRules, InSocketName);
}
```
最后在SlashCharacter.cpp中完善EKeyPressed方法
```
void ASlashCharacter::EKeyPressed()
{
	AWeapon* OverlappingWeapon = Cast<AWeapon>(OverlappingItem);//将重叠的Item进行AWeapon的转化
	if(OverlappingWeapon)
	{
		OverlappingWeapon->Equip(GetMesh(), FName("RightHandSocket"));//转化成功则运行Equip方法
	}
}
```
## 角色状态
新建一个头文件CharacterTypes.h声明枚举类
```
UENUM(BlueprintType)//加上BlueprintType即成了可以在蓝图中使用的类型
enum class ECharacterState : uint8//虚幻约定在枚举类前加上E，uint8意为只能是八位正整数
{
	ECS_Unequiped UMETA(DisplayName = "Unequipped"),//ECS为虚幻约定(是ECharacterState的缩写)，为了优化用户的蓝图使用在后面加上UMETA(DisplayName = "Unequipped")可以自己更改在蓝图中显示的名字
	ECS_EquippedOneHandedWeapon UMETA(DisplayName = "Equipped One-Handed Weapon"),
	ECS_EquippedTwoHandedWeapon UMETA(DisplayName = "Equipped Two-Handed Weapon")
};
```
在character.h初始化State为Unequiped
```
private:
	ECharacterState CharacterState = ECharacterState::ECS_Unequiped;
```
在character.cpp中，完善方法将状态修改成ECS_EquippedOneHandedWeapon
```
void ASlashCharacter::EKeyPressed()
{
	AWeapon* OverlappingWeapon = Cast<AWeapon>(OverlappingItem);
	if(OverlappingWeapon)
	{
		OverlappingWeapon->Equip(GetMesh(), FName("RightHandSocket"));
		CharacterState = ECharacterState::ECS_EquippedOneHandedWeapon;
	}
}
```
由于我们在SlashCharacter.h中对于
ECharacterState CharacterState = ECharacterState::ECS_Unequiped;
的声明是private的，所以我们在public中构造一个公开的get函数供其他类使用
```
public:
FORCEINLINE ECharacterState GetCharacterState() const { return CharacterState; }
```
在SlashAnimInstance.cpp中使用此get方法
```
void USlashAnimInstance::NativeUpdateAnimation(float DeltTime)
{
	Super::NativeUpdateAnimation(DeltTime);

	if (SlashCharacterMovement)
	{
		GroundSpeed = UKismetMathLibrary::VSizeXY(SlashCharacterMovement->Velocity);
		IsFalling = SlashCharacterMovement->IsFalling();
		CharacterState = SlashCharacter->GetCharacterState();
	}
}
```
在SlashAnimInstance.h中声明CharacterState
```
UPROPERTY(BlueprintReadOnly, CateGory = "Movement | Character State")
ECharacterState CharacterState;
```
最后在动画蓝图中进行姿势混合，可以右键添加先前创造的各种状态，然后进行姿势之间的切换
![输入图片说明](/imgs/2024-08-10/kvH7G5CLNH6up044.png)
![输入图片说明](/imgs/2024-08-10/7Z07JVf3PS2nvpBN.png)

## 攻击
在.h中声明攻击方法和动画蒙太奇变量
```
class UAnimMontage;
protected:
void Attack();
private:
//Animation Montages
UPROPERTY(EditDefaultsOnly, Category = Montages)
UAnimMontage* AttackMontage;
```
在.cpp中，绑定完attack的按键之后，实现attack方法
```
void ASlashCharacter::Attack()
{
	UAnimInstance* AnimInstance = GetMesh()->GetAnimInstance();//得到角色动画实例
	if(AnimInstance && AttackMontage)
	{
		AnimInstance->Montage_Play(AttackMontage);//AttackMontage是在.h中声明的默认蓝图可编辑对象，可在默认蓝图为其赋值
		int32 Selection = FMath::RandRange(0, 1);
		FName SectionName = FName();
		switch (Selection)
		{
		case 0:
			SectionName = FName("Attack1");
			break;
		case 1:
			SectionName = FName("Attack2");
			break;
		default:
			break;
		}
		AnimInstance->Montage_JumpToSection(SectionName, AttackMontage);
	}
}
```
### 完善
在CharacterTypes.h新建一个枚举类，用来表示在攻击或不在攻击中的状态
```
UENUM(BlueprintType)
enum class EActionState : uint8
{
	EAS_Unoccupied UMETA(DisplayName = "Unoccupied"),
	EAS_Attacking UMETA(DisplayName = "Attacking")
};
```
在SlashCharacter.h中初始化ActionState
```
UPROPERTY(BlueprintReadWrite, meta = (AllowPrivateAccess = "true"))
EActionState ActionState = EActionState::EAS_Unoccupied;
```
利用封装的思想，将攻击蒙太奇封装成一个函数
```
void ASlashCharacter::Attack()
{
	if (ActionState == EActionState::EAS_Unoccupied) {
		PlayAttackMontage();
		ActionState = EActionState::EAS_Attacking;
	}
	
}

void ASlashCharacter::PlayAttackMontage()
{
	UAnimInstance* AnimInstance = GetMesh()->GetAnimInstance();
	if (AnimInstance && AttackMontage)
	{
		AnimInstance->Montage_Play(AttackMontage);
		const int32 Selection = FMath::RandRange(0, 1);
		FName SectionName = FName();
		switch (Selection)
		{
		case 0:
			SectionName = FName("Attack1");
			break;
		case 1:
			SectionName = FName("Attack2");
			break;
		default:
			break;
		}
		AnimInstance->Montage_JumpToSection(SectionName, AttackMontage);
	}
}
```
### 在蒙太奇中建立动画通知
蓝图做法![输入图片说明](/imgs/2024-08-11/U4dz4JD3l9SEAsIc.png)
c++做法
在SlashCharacter.h中声明一个可以在蓝图中被调用的函数
```
UFUNCTION(BlueprintCallable)
void AttackEnd();
```
在SlashCharacter.cpp完善此函数，将状态改回未攻击状态
```
void ASlashCharacter::AttackEnd()
{
	ActionState = EActionState::EAS_Unoccupied;
}
```
![输入图片说明](/imgs/2024-08-11/lCP0dUuJpA9PTyIR.png)
### 完善武器未拾取时做sin运动，拾取后静止
在item.h中创建枚举类用来区分item是否被装备，并创造一个初始值为未装备的变量
```
enum class EItemState : uint8
{
	EIS_Hovering,
	EIS_Equipped,
};
EItemState ItemState = EItemState::EIS_Hovering;
```
在item.cpp中，如果满足if条件则进行正弦运动
```
void AItem::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

	RunningTime += DeltaTime;

	if (ItemState == EItemState::EIS_Hovering) {
		AddActorWorldOffset(FVector(0.f, 0.f, TransformedSin()));
	}
	
}
```
在weapon.cpp中的equip方法中，修改itemstate状态
```
void AWeapon::Equip(USceneComponent* Inparent, FName InSocketName)
{
	FAttachmentTransformRules TransformRules(EAttachmentRule::SnapToTarget, true);
	ItemMesh->AttachToComponent(Inparent, TransformRules, InSocketName);
	ItemState = EItemState::EIS_Equipped;
}
```
### 完善攻击时还能移动的bug
在SlashCharacter.cpp中的moveForward和moveRight的方法中加入这句话，检查此时是否在攻击状态中，如果是则return，即无法运动
```
if (ActionState == EActionState::EAS_Attacking) { return; }
```
## 装备
### 动画准备
先准备好装备武器以及解除装备武器的动画，并将其制作为蒙太奇
### 代码部分
在SlashCharacter.h中声明装备蒙太奇
```
public:
void PlayEquipMontage(FName SectionName);
bool CanDisarm();
bool CanArm();
private:
UPROPERTY(EditDefaultsOnly, Category = Montages)
UAnimMontage* EquipMontage;

UPROPERTY(VisibleAnywhere, Category = Weapon)
AWeapon* EquippedWeapon;
```
在SlashCharacter.cpp中，完善canDisarm和canArm方法，以及播放装备蒙太奇的方法。
```
bool ASlashCharacter::CanDisarm()
{
	return ActionState == EActionState::EAS_Unoccupied && CharacterState != ECharacterState::ECS_Unequiped;
}

bool ASlashCharacter::CanArm()
{
	return ActionState == EActionState::EAS_Unoccupied && CharacterState == ECharacterState::ECS_Unequiped && EquippedWeapon;
}
void ASlashCharacter::PlayEquipMontage(FName SectionName)
{
	UAnimInstance* AnimInstance = GetMesh()->GetAnimInstance();
	if (AnimInstance && EquipMontage) {
		AnimInstance->Montage_Play(EquipMontage);
		AnimInstance->Montage_JumpToSection(SectionName, EquipMontage);
	}
}
```
为EKeyPressed添加else情况，使得重叠时可以捡起武器，非重叠时检查自身是否有武器，如有的话此时E键用来装备和解除装备
```
void ASlashCharacter::EKeyPressed()
{
	AWeapon* OverlappingWeapon = Cast<AWeapon>(OverlappingItem);
	if(OverlappingWeapon)
	{
		OverlappingWeapon->Equip(GetMesh(), FName("RightHandSocket"));
		CharacterState = ECharacterState::ECS_EquippedOneHandedWeapon;
		EquippedWeapon = OverlappingWeapon;
		OverlappingItem = nullptr;
	}
	else {
		if (CanDisarm()) {
			PlayEquipMontage(FName("Unequip"));
			CharacterState = ECharacterState::ECS_Unequiped;
		}
		if (CanArm()) {
			PlayEquipMontage(FName("Equip"));
			CharacterState = ECharacterState::ECS_EquippedOneHandedWeapon;
		}
	}
}
```
在背部添加一个插槽（SpineSocket）用来放剑，在蒙太奇轨道中建立动画通知，创建arm和disarm的通知
在SlashCharacter.h创建两个函数
```
UFUNCTION(BlueprintCallable)
void Disarm();
UFUNCTION(BlueprintCallable)
void Arm();
```
在SlashCharacter.cpp中实现，用来切换插槽（手与背之间）
```
void ASlashCharacter::Disarm()
{
	if (EquippedWeapon) {
		EquippedWeapon->AttachMeshToSocket(GetMesh(), FName("SpineSocket"));
	}
}

void ASlashCharacter::Arm()
{
	if (EquippedWeapon) {
		EquippedWeapon->AttachMeshToSocket(GetMesh(), FName("RightHandSocket"));
	}
}
```
上面运用的方法时在Weapon.cpp中提取成函数的两句代码，增强可复用性
```
void AWeapon::AttachMeshToSocket(USceneComponent* Inparent, const FName& InSocketName)
{
	FAttachmentTransformRules TransformRules(EAttachmentRule::SnapToTarget, true);
	ItemMesh->AttachToComponent(Inparent, TransformRules, InSocketName);
}
```
完善枚举类的代码，增加一个装备武器的状态，可以便于做出更多判断，比如在装备武器时禁止移动
```
enum class EActionState : uint8
{
	EAS_Unoccupied UMETA(DisplayName = "Unoccupied"),
	EAS_Attacking UMETA(DisplayName = "Attacking"),
	EAS_EquippingWeapon UMETA(DisplayName = "EquippingWeapon")
};
```
在装备结束时的动画添加一个通知，并在SlashCharacter里面声明函数，将状态从装备中切换回未被占用
```
void ASlashCharacter::FinshEquipping()
{
	ActionState = EActionState::EAS_Unoccupied;
}
```
### 装备时添加音效
Weapon.h中声明EquipSound
```
class UBaseSound;
private:

UPROPERTY(EditAnywhere, Category = "Weapon Properties")
USoundBase* EquipSound;
```
在Weapon.cpp中，将声音播放置于装备的方法中，并对于Sphere，先将它从item.h中的private改成protected，然后使它再捡起后的碰撞变为无碰撞，这样就不会有捡起武器后按E仍然会有shink的声音了
```
void AWeapon::Equip(USceneComponent* Inparent, FName InSocketName)
{
	AttachMeshToSocket(Inparent, InSocketName);
	ItemState = EItemState::EIS_Equipped;
	if (EquipSound) {
		UGameplayStatics::PlaySoundAtLocation(this, EquipSound, GetActorLocation());
	}
	if (Sphere) {
		Sphere->SetCollisionEnabled(ECollisionEnabled::NoCollision);
	}
}
```
## Tracing（武器打击检测）
### 1.蓝图做法，使用box tracing by channel（按通道进行盒体追踪）
想要有一个组件，没有网格或其他任何东西，只有位置信息，可以使用Scene，制作一个Start和一个End，例如剑的话一个放在剑头一个剑尾
![输入图片说明](/imgs/2024-08-13/iqbISCwzI8aYhdg8.png)
**pay attention**：对于想要得到重叠响应的目标，必须勾选生成重叠事件！！！然后根据情况设置碰撞预设
![输入图片说明](/imgs/2024-08-13/AjX9Fbzl5Lhk44mm.png)

Start和End分别传入刚刚制作的Scene组件，HalfSize是制作出的盒子的边长。
![输入图片说明](/imgs/2024-08-13/HYO8vWCUuB5VsMD3.png)
将OutHit节点break（中断命中结果），会得到很多位置信息，我们将impactPoint绘制球体，即可以得到以命中点为中心绘制的球体
![输入图片说明](/imgs/2024-08-13/ChKsX0Lv5a2r2tDz.png)
### 2.代码做法
在Weapon.cpp的AWeapon构造函数中，对于WeaponBox进行碰撞的处理，先将所有碰撞频道设为重叠，然后单独将Pawn轨道设为忽略
```
AWeapon::AWeapon() {
	WeaponBox = CreateDefaultSubobject<UBoxComponent>(TEXT("Weapon Box"));
	WeaponBox->SetupAttachment(GetRootComponent());
	WeaponBox->SetCollisionEnabled(ECollisionEnabled::QueryOnly);
	WeaponBox->SetCollisionResponseToAllChannels(ECollisionResponse::ECR_Overlap);
	WeaponBox->SetCollisionResponseToChannel(ECollisionChannel::ECC_Pawn, ECollisionResponse::ECR_Ignore);
}
```
在Weapon.h中声明override的BeginPlay()和OnBoxOverlap
```
protected:
	virtual void BeginPlay() override;
	UFUNCTION()
	void OnBoxOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult);
```
在Weapon.cpp中在BeginPlay中绑定重叠事件
```
void AWeapon::BeginPlay()
{
	Super::BeginPlay();

	WeaponBox->OnComponentBeginOverlap.AddDynamic(this, &AWeapon::OnBoxOverlap);
}
```
并完善重叠函数
```
void AWeapon::OnBoxOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
	const FVector Start = BoxTraceStart->GetComponentLocation();
	const FVector End = BoxTraceEnd->GetComponentLocation();

	TArray<AActor*> ActorsToIgnore;
	ActorsToIgnore.Add(this);
	FHitResult BoxHit;

	UKismetSystemLibrary::BoxTraceSingle(
		this, Start, End,
		FVector(5.f, 5.f, 5.f),
		BoxTraceStart->GetComponentRotation(),
		ETraceTypeQuery::TraceTypeQuery1,
		false,
		ActorsToIgnore,
		EDrawDebugTrace::ForDuration,
		BoxHit, true);
}

```
### 3.完善武器未攻击时也能重叠的bug


将先前在Weapon.cpp的AWeapon构造函数中的QueryOnly改为NoCollision
```
WeaponBox->SetCollisionEnabled(ECollisionEnabled::NoCollision);
```
在Weapon.h中，制作此方法用来返回WeaponBox
```
public:
	FORCEINLINE UBoxComponent* GetWeaponBox() const { return WeaponBox; }
```
在SlashCharacter.h中定义一个用来开启武器碰撞的方法
```
UFUNCTION(BlueprintCallable)
	void SetWeaponCollisionEnable(ECollisionEnabled::Type CollisionEnabled);
```
完善方法，此句运用的方法EquippedWeapon->GetWeaponBox()是刚刚在Weapon.h中定义的使其可以取得WeaponBox，
```
void ASlashCharacter::SetWeaponCollisionEnable(ECollisionEnabled::Type CollisionEnabled)
{
	if (EquippedWeapon) {
		EquippedWeapon->GetWeaponBox()->SetCollisionEnabled(CollisionEnabled);
	}
}
```

## 接口（Unreal Interface）
在HitInterface.h中声明一个纯虚函数GetHit()，通过=0来表示它是一个纯虚函数，即无需在HitInterface.cpp中声明该函数的实现
```
class SLASH_API IHitInterface
{
	GENERATED_BODY()
public:
	virtual void GetHit(const FVector& ImpactPoint) = 0;
};
```
在Enemy.cpp中，先完善AEnemy()构造器，
1.首先不能将敌人设置为pawn，因为我们的武器设置了忽略pawn，所以将其设置为WorldDynamic
2.使用的**Visibility**通道，所以将mesh设为Visibility的block
3.不希望它与玩家摄像机产生碰撞，所以将Camera设为ignore
4.将SetGenerateOverlapEvents（生成重叠事件）设置为true，才能有后续对于重叠判断的展开
```
AEnemy::AEnemy()
{
 	// Set this character to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;

	GetMesh()->SetCollisionObjectType(ECollisionChannel::ECC_WorldDynamic);
	GetMesh()->SetCollisionResponseToChannel(ECollisionChannel::ECC_Visibility, ECollisionResponse::ECR_Block);
	GetMesh()->SetCollisionResponseToChannel(ECollisionChannel::ECC_Camera, ECollisionResponse::ECR_Ignore);
	GetCapsuleComponent()->SetCollisionResponseToChannel(ECollisionChannel::ECC_Camera, ECollisionResponse::ECR_Ignore);
	GetMesh()->SetGenerateOverlapEvents(true);
}
```
### 多重继承
在Enemy.h中，同时继承自ACharacter和IHitInterface
```
#include "Interfaces/HitInterface"
class SLASH_API AEnemy : public ACharacter, public IHitInterface
```
并且override函数 GetHit
```
virtual void GetHit(const FVector& ImpactPoint) override;
```
在Enemy.cpp中实现方法GetHit，根据传入的ImpactPoint绘制debug球体，即可做到在受击点的位置绘制球
```
void AEnemy::GetHit(const FVector& ImpactPoint)
{
	DRAW_SPHERE(ImpactPoint);
}
```
在Weapon.cpp中，在先前OnBoxOverlap的基础上，利用BoxHit来进行操作。
1.if (BoxHit.GetActor())，如果击中了对象返回为true后
2.将BoxHit.GetActor()转化为IHitInterface，此时能转化的原因是我们击中的敌人为enemy类型，而enemy先前定义的时候既继承了ACharacter类又继承了IHitInterface，所以可以转化成IHitInterface，转化失败则会返回为空
```
IHitInterface* HitInterface = Cast<IHitInterface>(BoxHit.GetActor());
```
3.if (HitInterface) {HitInterface->GetHit(BoxHit.ImpactPoint);}
判断HitInterface是否为空，如果非空的话调用GetHit方法。
```
#include "Interfaces/HitInterface.h"
void AWeapon::OnBoxOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
	const FVector Start = BoxTraceStart->GetComponentLocation();
	const FVector End = BoxTraceEnd->GetComponentLocation();

	TArray<AActor*> ActorsToIgnore;
	ActorsToIgnore.Add(this);
	FHitResult BoxHit;

	UKismetSystemLibrary::BoxTraceSingle(
		this, Start, End,
		FVector(5.f, 5.f, 5.f),
		BoxTraceStart->GetComponentRotation(),
		ETraceTypeQuery::TraceTypeQuery1,
		false,
		ActorsToIgnore,
		EDrawDebugTrace::ForDuration,
		BoxHit, true);
	if (BoxHit.GetActor()) {
		IHitInterface* HitInterface = Cast<IHitInterface>(BoxHit.GetActor());
		if (HitInterface) {
			HitInterface->GetHit(BoxHit.ImpactPoint);
		}
	}
}
```
### 修改DEBUG
新建一条宏定义，新加变量Color可以自己更改颜色，画出的球体5秒后消失
```
#define DRAW_SPHERE_COLOR(Location, Color) DrawDebugSphere(GetWorld(), Location, 8.f, 12, Color, false, 5.f);
```
修改GetHit方法，使用新的宏定义，使用橙色
```
void AEnemy::GetHit(const FVector& ImpactPoint)
{
	DRAW_SPHERE_COLOR(ImpactPoint, FColor::Orange);
}
```
## 受击动画
在Enemy.h中声明变量，HitReactMontage在Enemy默认蓝图位置进行赋值
```
private:
	/**
	*  Animation Montages
	**/
	UPROPERTY(EditDefaultsOnly, Category = Montages)
	UAnimMontage* HitReactMontage;

protected:
	/**
	* Play montage function
	**/
	void PlayHitReactMontage(const FName& SectionName);
```
在Enemy.cpp中实现该方法，声明动画实例对象AnimInstance，播放对应蒙太奇中名字相符合的片段
```
void AEnemy::PlayHitReactMontage(const FName& SectionName)
{
	UAnimInstance* AnimInstance = GetMesh()->GetAnimInstance();
	if (AnimInstance && HitReactMontage) {
		AnimInstance->Montage_Play(HitReactMontage);
		AnimInstance->Montage_JumpToSection(SectionName, HitReactMontage);
	}
}
```
## 计算受击角度
在Enemy.h中声明DirectionHitReact方法且在cpp中实现
 1. 首先得到敌人向前的向量，以及受击点减去敌人位置的向量，将会得到一个从敌人位置指向受击点位置的向量，GetSafeNormal()将其单位化。
 2. ImpactLowered是将受击点改为到与敌人同一Z轴高度，使得与向前向量在同一Z轴高度更加方便观察
 3. CosTheta得到点乘的值，再利用Acos得到反余弦的值，最终得到Theta的值
 4. FMath::RadiansToDegrees(Theta)将弧度制转化为度数
 5. 计算叉乘的向量结果
 6. 根据得到的角度来进行if分支的判断，初始默认为FromBack，如果有其他情况则改成对应情况，没有的话则是FromBack，将对应的名字传入蒙太奇进行播放。
 7. UKismetSystemLibrary::DrawDebugArrow用来绘制箭头，根据传入的向量来绘制

```
void AEnemy::DirectionHitReact(const FVector& ImpactPoint)
{
	const FVector Forward = GetActorForwardVector();
	const FVector ImpactLowered(ImpactPoint.X, ImpactPoint.Y, GetActorLocation().Z);
	const FVector ToHit = (ImpactLowered - GetActorLocation()).GetSafeNormal();

	const double CosTheta = FVector::DotProduct(Forward, ToHit);
	double Theta = FMath::Acos(CosTheta);
	Theta = FMath::RadiansToDegrees(Theta);

	const FVector CrossProduct = FVector::CrossProduct(Forward, ToHit);

	if (CrossProduct.Z < 0) {

		Theta *= -1.f;
	}

	FName Section("FromBack");
	if (Theta >= -45.f && Theta < 45.f) { Section = FName("FromFront"); }
	else if (Theta >= -135.f && Theta < -45.f) { Section = FName("FromLeft"); }
	else if (Theta >= 45.f && Theta < 135.f) { Section = FName("FromRight"); }

	PlayHitReactMontage(Section);

	UKismetSystemLibrary::DrawDebugArrow(this, GetActorLocation(), GetActorLocation() + CrossProduct * 100.f, 5.f, FColor::Blue, 5.f);

	if (GEngine) {
		GEngine->AddOnScreenDebugMessage(1, 5.f, FColor::Green, FString::Printf(TEXT("Theta: %f"), Theta));
	}
	UKismetSystemLibrary::DrawDebugArrow(this, GetActorLocation(), GetActorLocation() + Forward * 60.f, 5.f, FColor::Red, 5.f);
	UKismetSystemLibrary::DrawDebugArrow(this, GetActorLocation(), GetActorLocation() + ToHit * 60.f, 5.f, FColor::Green, 5.f);
}
```
### 解决攻击一次可能会触发多个触发盒子导致攻击变向问题
在Weapon.h中声明一个数组
```
public:
TArray<AActor*> IgnoreActors;
```
在Weapon.cpp中添加一个循环，遍历IgnoreActors，将重叠的，未添加过的ActorsToIgnore.AddUnique(Actor);添加进去。
```
void AWeapon::OnBoxOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
	const FVector Start = BoxTraceStart->GetComponentLocation();
	const FVector End = BoxTraceEnd->GetComponentLocation();

	TArray<AActor*> ActorsToIgnore;
	ActorsToIgnore.Add(this);

	for (AActor* Actor : IgnoreActors) {
		ActorsToIgnore.AddUnique(Actor);
	}
...(省略)
```
由于IgnoreActors是每次判断的依据，所以在武器每次开启，结束碰撞时，需要在结束时清零，以便于下次继续计算。
所以我们可以在SlashCharacter.cpp中，利用先前在动画蓝图里面定义的结束碰撞的轨道，在结束碰撞后将其清零
![输入图片说明](/imgs/2024-08-15/9AYdt4I18lp6ZNL2.png)
```
void ASlashCharacter::SetWeaponCollisionEnable(ECollisionEnabled::Type CollisionEnabled)
{
	if (EquippedWeapon && EquippedWeapon->GetWeaponBox()) {
		EquippedWeapon->GetWeaponBox()->SetCollisionEnabled(CollisionEnabled);
		EquippedWeapon->IgnoreActors.Empty();

	}
}
```
## 受击时在受击点释放音效/溅血效果
在Enemy.h中，声明HitSound，制作MetaSound音效，在Enemy蓝图中赋值使用，在蓝图中选择使用导入的粒子特效赋值给HitParticles
```
private:
UPROPERTY(EditAnywhere, Category = Sounds)
USoundBase* HitSound;
UPROPERTY(EditAnywhere, Category = VisualEffects)
UParticleSystem* HitParticles;
```
在Enemy.cpp中，完善GetHit方法，制作释放音效/溅血的功能
```
#include "Kismet/GameplayStatics.h"
void AEnemy::GetHit(const FVector& ImpactPoint)
{
	DRAW_SPHERE_COLOR(ImpactPoint, FColor::Orange);
	

	DirectionHitReact(ImpactPoint);

	if (HitSound) {
		UGameplayStatics::PlaySoundAtLocation(
			this,
			HitSound,
			ImpactPoint);
	}
	if (HitParticles && GetWorld()) {
		UGameplayStatics::SpawnEmitterAtLocation(
			GetWorld(), HitParticles, ImpactPoint);
	}
}
```
## 制作武器打碎瓦罐效果
此处采用cpp声明蓝图实现
在weapon蓝图中添加FieldSystem组件以及相应需要的组件
![输入图片说明](/imgs/2024-08-16/kym2NRzXKTwowHzQ.png)
在Weapon.h中声明函数
```
UFUNCTION(BlueprintImplementableEvent)
void CreateFields(const FVector& FieldLocation);
```
由于此处我们是想在蓝图中实现，所以在Weapon.cpp中调用函数并传入BoxHit.ImpactPoint
```
void AWeapon::OnBoxOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
...省略
		CreateFields(BoxHit.ImpactPoint);
}
```
最后在weapon蓝图中使用该事件并完善其功能，参考**击碎网格体**
![输入图片说明](/imgs/2024-08-16/T2reRwbX0FDJ5Jee.png)
## 制作一个Actor适用于所有可被打碎的物品
创建BreakableActor类。
在BreakableActor.h中声明变量
```
private:
	UPROPERTY(VisibleAnywhere)
	UGeometryCollectionComponent* GeometryCollection;
```
在BreakableActor.cpp中的构造函数进行初始化actor步骤
```
#include "GeometryCollection/GeometryCollectionComponent.h"


ABreakableActor::ABreakableActor()
{
	PrimaryActorTick.bCanEverTick = false;

	GeometryCollection = CreateDefaultSubobject<UGeometryCollectionComponent>(TEXT("GeometryCollection"));
	SetRootComponent(GeometryCollection);
	GeometryCollection->SetGenerateOverlapEvents(true);
}
```

 - 此处为了能成功编译需要在Build.cs中添加"GeometryCollectionEngine"模块
### 制作同时可在cpp中实现也可以在蓝图实现的函数
先前在HitInterface.h中定义的GetHit，将其修改为
```
public:
	UFUNCTION(BlueprintNativeEvent)
	void GetHit(const FVector& ImpactPoint);
```
表示为可以在蓝图中进行实现，为了在cpp中也进行实现，需改写之前继承自HitInterface接口的类
例如Enemy.h中的
```
virtual void GetHit(const FVector& ImpactPoint) override;
```
需改写成（虚幻定义的），则一共两版本，GetHit为蓝图版本，GetHit_Implementation为cpp版本
```
virtual void GetHit_Implementation(const FVector& ImpactPoint) override;
```
对应的Enemy.cpp中也需改名。
而在Weapon.cpp调用此方法GetHit，则需要采用这种类型Execute_GetHit，需要传入第一个为执行事件的对象，第二个是方法定义时的参数
```
void AWeapon::OnBoxOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
···省略
		if (HitInterface) {
			HitInterface->Execute_GetHit(BoxHit.GetActor(), BoxHit.ImpactPoint);
		}
}
```
Breakable的蓝图中，调用事件GetHit，右键可以调用cpp中的GetHit函数，即GetHit_Implementation
![输入图片说明](/imgs/2024-08-16/C7ulCpzSFnw6v2Xz.png)
![输入图片说明](/imgs/2024-08-16/V32MzKth3BfZj7av.png)
制作瓦罐破碎音效以及设置生命周期（在破碎的三秒后销毁碎片），最后执行cpp实现的部分
![输入图片说明](/imgs/2024-08-17/SZhMktgyyuFROpkF.png)
## 制作宝藏
以item为父类创建子类cpp文件：Treasure
在Treasure.h中，重写父类的重叠事件以及声明蓝图可编辑的声音
```
protected:
	virtual void OnSphereOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult) override;
	
private:
	UPROPERTY(EditAnywhere, Category = Sounds)
	USoundBase* PickupSound;
```
在Treasure.cpp中，触发重叠时，播放声音以及销毁掉宝藏资源
```
void ATreasure::OnSphereOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
	ASlashCharacter* SlashCharacter = Cast<ASlashCharacter>(OtherActor);
	if (SlashCharacter) {
		if (PickupSound) {
			UGameplayStatics::PlaySoundAtLocation(
				this,
				PickupSound,
				GetActorLocation()
				);
		}
		Destroy();
	}
}
```
## 制作瓦罐破碎后生成宝藏的效果
### 蓝图做法
![输入图片说明](/imgs/2024-08-17/JBWEmDxlf2V0Wd2u.png)
### 代码做法
使用SpawnActor函数
在BreakableActor.h中声明一个TreasureClass，是为了在瓦罐的蓝图中指定宝藏蓝图
```
private:
	UPROPERTY(EditAnywhere)
	UClass* TreasureClass;
```
在BreakableActor.cpp中完善GetHit方法，在BreakableActor.h中声明一个TreasureClass主要是因为如果不用此方法的话，SpawnActor无法根据蓝图类进行生成，而我们的Treasure蓝图是更加完善并最终要使用的。此方法乐意指定该Class为蓝图类或者C++类都行
> 如果需要基于c++类生成的话，即使用ATreasure::StaticClass()即可

```
void ABreakableActor::GetHit_Implementation(const FVector& ImpactPoint)
{
	UWorld* World = GetWorld();
	if (World && TreasureClass) {
		FVector Location = GetActorLocation();
		Location.Z += 75.f;
		World->SpawnActor<ATreasure>(TreasureClass, Location, GetActorRotation());
	}
}
```
**改进**：先前可以在蓝图中随意选取蓝图，而我们只想要关于Treasure类的，可以用TSubclassOf来加以限制
```
UPROPERTY(EditAnywhere)
TSubclassOf<class ATreasure> TreasureClass;
```
![输入图片说明](/imgs/2024-08-17/hPnKDrxIVvcNiQjp.png)
### 改进瓦罐破碎后飞溅碎片与角色的碰撞问题
可以将整个瓦罐改为ignore pawn，但是此时未破碎时角色可以直接穿过瓦罐
此时在BreakableActor.h声明一个胶囊体
```
protected:
	UPROPERTY(VisibleAnywhere, BlueprintReadWrite)
	class UCapsuleComponent* Capsule;
```
在BreakableActor.cpp的构造函数中对capsule进行初始化，将其改为对所有都不忽略，再单独将其设为对角色pawn忽略，即也可以做到了阻挡的效果
```
Capsule = CreateDefaultSubobject<UCapsuleComponent>(TEXT("Capsule"));
	Capsule->SetupAttachment(GetRootComponent());
	Capsule->SetCollisionResponseToAllChannels(ECollisionResponse::ECR_Ignore);
	Capsule->SetCollisionResponseToChannel(ECollisionChannel::ECC_Pawn, ECollisionResponse::ECR_Block);
```
但是此时破碎时胶囊体仍然存在，因为我们先前设置了破碎后三秒后整个物体才会消失，所以此时即使破碎了但是还是有个胶囊体挡住我们。
我们可以手动在蓝图中进行碰撞响应的改变，在破碎之后将对pawn轨道的碰撞改为忽略
![输入图片说明](/imgs/2024-08-17/mQwVFAPo3BZRq4CG.png)
## 制作随机生成不同的宝藏
制作一个BaseTreasure，以其为基础创造子类蓝图，将子类改为不同的网格体
![输入图片说明](/imgs/2024-08-17/QF9R5jZWSF9PqMyk.png)
将之前在BreakableActor.h声明的TreasureClass改成一个数组
```
private:

	UPROPERTY(EditAnywhere, Category = "Breakable Properties")
	TArray<TSubclassOf<class ATreasure>> TreasureClasses;
```
所以修改BreakableActor.cpp中的方法，

 - 增加判断条件TreasureClasses.Num() > 0，说明此破碎物品会掉落宝藏
 - 使用FMath::RandRange(0, TreasureClasses.Num() - 1)用来进行随机数的选择，得到随机宝藏的效果
```
void ABreakableActor::GetHit_Implementation(const FVector& ImpactPoint)
{
	UWorld* World = GetWorld();
	if (World && TreasureClasses.Num() > 0) {
		FVector Location = GetActorLocation();
		Location.Z += 75.f;
		const int32 Selection = FMath::RandRange(0, TreasureClasses.Num() - 1);
		World->SpawnActor<ATreasure>(TreasureClasses[Selection], Location, GetActorRotation());
	}
}
```
### 由于可破坏的物体增多后，调用GetHit函数次数也增多，次数过多容易发生死循环错误
在BreakableActor.h中声明一个bBroken
```
bool bBroken = false;
```
完善BreakableActor.cpp中的方法，如果bBroken等于true就说明罐子已经破碎了，直接return
```
void ABreakableActor::GetHit_Implementation(const FVector& ImpactPoint)
{
	if (bBroken) { return; }
	bBroken = true;
	UWorld* World = GetWorld();
	if (World && TreasureClasses.Num() > 0) {
		FVector Location = GetActorLocation();
		Location.Z += 75.f;
		const int32 Selection = FMath::RandRange(0, TreasureClasses.Num() - 1);
		World->SpawnActor<ATreasure>(TreasureClasses[Selection], Location, GetActorRotation());
	}
}
```
## Niagara组件
在Item.h中声明
```
UPROPERTY(EditAnywhere)
class UNiagaraComponent* EmbersEffect;
```
在Item.cpp中的构造函数中对EmbersEffect进行初始化，则此时可以在蓝图中对EmbersEffect进行赋值
```
#include "NiagaraComponent.h"
EmbersEffect = CreateDefaultSubobject<UNiagaraComponent>(TEXT("Embers"));
	EmbersEffect->SetupAttachment(GetRootComponent());
```
假设我们在武器上添加Niagara特效，则此时我们捡起武器时特效并不会消失，需要我们手动关停特效
```
#include "NiagaraComponent.h"
void AWeapon::Equip(USceneComponent* Inparent, FName InSocketName)
{
	···省略
	if (EmbersEffect) {
		EmbersEffect->Deactivate();
	}
}
```
## 制作自己的组件（component）
制作自己的Attribute组件
新建c++类基于Actor组件（ActorComponent）
在Attribute.h中声明变量
```
private:
	UPROPERTY(EditAnywhere, Category = "Actor Attributes")
	float Health;
	UPROPERTY(EditAnywhere, Category = "Actor Attributes")
	float MaxHealth;
```
先应用到Enemy身上，在Enemy.h中声明Attributes
```
class UAttributeComponent;
private:
	UPROPERTY(VisibleAnywhere)
	UAttributeComponent* Attributes;
```
在Enemy.cpp中的构造函数进行初始化
```
Attributes = CreateDefaultSubobject<UAttributeComponent>(TEXT("Attributes"));
```
所以编译运行即可看到Enemy蓝图中有Attribute组件，组件中有可编辑的Actor Attributes
![输入图片说明](/imgs/2024-08-18/mRdOsG1rODVVwhog.png)
## 制作血条
在Build.cs中加上OMG模块，以及右键新建c++类以widgetComponent建立HealthBarComponent
在Enemy.h中声明
```
class UHealthBarComponent;
private:
UHealthBarComponent* HealthBarWidget;
```
在Enemy.cpp中的构造方法中初始化HealthBarWidget
```
#include "Components/WidgetComponent.h"
HealthBarWidget = CreateDefaultSubobject<UHealthBarComponent>(TEXT("HealthBar"));
HealthBarWidget->SetupAttachment(GetRootComponent());
```
在HealthBar.h中，声明变量。
**注意此处**：我们希望此变量绑定蓝图控件中的进度条，需要对应**相同**的名字
![输入图片说明](/imgs/2024-08-18/ih3BlHrXyr255rSy.png)
```
public:
	UPROPERTY(meta = (BindWidget))
	class UProgressBar* HealthBar;
```
在我们创建的widget蓝图中，点开细节面板，可以将父类改成我们自己创建的HealthBar
![输入图片说明](/imgs/2024-08-18/A1EPZDj8TXZp4YGY.png)
在HealthBarComponent.h中制作计算方法
```
public:
	void SetHealthPercent(float Percent);
```
在HealthBarComponent.cpp中实现该方法
由于我们可能会反复调用此方法，即会反复使用Cast函数，这可能是一项十分耗能昂贵的操作，因为GetUserWidgetObject()如果转换不成功，它会从父类一直往上一层不断地查找转换。
```
void UHealthBarComponent::SetHealthPercent(float Percent)
{
	UHealthBar* HealthBarWidget = Cast<UHealthBar>(GetUserWidgetObject());
	if (HealthBar) {
	}
}
```
所以为了改进，我们在HealthBarComponent.h中声明一个HealthBar，为了保证它是初始化为**nullptr**，使用UPROPERTY()声明，防止被**垃圾数据初始化**。
```
private:
	UPROPERTY()
	class UHealthBar* HealthBarWidget;
```
HealthBarComponent.cpp中完善方法，此处为了防止上述的重复cast问题，采用了存储变量的方法，将cast成功的值存入HealthBarWidget，以后即直接用HealthBarWidget
```
void UHealthBarComponent::SetHealthPercent(float Percent)
{
	if (HealthBarWidget == nullptr) {
		HealthBarWidget = Cast<UHealthBar>(GetUserWidgetObject());
	}
	if (HealthBarWidget && HealthBarWidget->HealthBar) {
		HealthBarWidget->HealthBar->SetPercent(Percent);
	}
}
```
## 伤害制作
**最重要的两个函数**AActor类下的**TakeDamage**和UGameplayStatics类下的**ApplyDamage**，通过调用ApplyDamage会转而调用TakeDamage
### TakeDamage和ApplyDamage
1. **功能关系**:
- TakeDamage 函数负责处理 Actor 受到伤害时的逻辑,比如扣除生命值、播放受伤动画等。
- ApplyDamage 函数负责调用目标 Actor 的 TakeDamage 函数,从而触发伤害处理逻辑。
2. **参数传递**:
 - ApplyDamage 函数会将伤害量、伤害类型、伤害施加者等信息作为参数传递给目标 Actor 的 TakeDamage 函数。
 - TakeDamage 函数可以根据这些参数信息来决定具体的伤害处理行为。
3. **调用关系**:
 - 当需要对某个 Actor 造成伤害时,通常会调用 ApplyDamage 函数。
- ApplyDamage 函数内部会寻找目标 Actor,并调用其 TakeDamage 函数。

在weapon.h中声明一个变量Damage用来设置武器伤害
```
UPROPERTY(EditAnywhere, Category = "Weapon Properties")
float Damage = 20.f;
```
在Enemy.h中override来自AActor的方法（因为Enemy继承自AActor）
```
virtual float TakeDamage(float DamageAmount, struct FDamageEvent const& DamageEvent, class AController* EventInstigator, AActor* DamageCauser) override;
```
由于在ApplyDamage我们需要得到使用该武器的controller
所以完善Equip方法，在Weapon.h添加两个参数NewOwner，NewInstigator
```
void Equip(USceneComponent* Inparent, FName InSocketName, AActor* NewOwner, APawn* NewInstigator);
```
在Weapon.cpp方法实现部分，得到NewOwner和NewInstigator
```
void AWeapon::Equip(USceneComponent* Inparent, FName InSocketName, AActor* NewOwner, APawn* NewInstigator)
{
	SetOwner(NewOwner);
	SetInstigator(NewInstigator);
。。。省略
}
```
由于此处在Equip方法处添加了两个变量，所以在SlashCharacter对应的地方也要修改方法
```
void ASlashCharacter::EKeyPressed()
{
。。。省略
		OverlappingWeapon->Equip(GetMesh(), FName("RightHandSocket"), this, this);
	。。。省略
}
```
回到Weapon.cpp中处理造成伤害事件，此处applyDamage在GetHit之前是因为，GetHit方法中有判断敌人受击方向的方法，如果先执行了GetHit，则当敌人生命为0时还需要再打一下才能判断死亡
```
void AWeapon::OnBoxOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
。。。省略
UGameplayStatics::ApplyDamage(
		BoxHit.GetActor(),
		Damage,
		GetInstigator()->GetController(),
		this,
		UDamageType::StaticClass()
		);
IHitInterface* HitInterface = Cast<IHitInterface>(BoxHit.GetActor());
		if (HitInterface) {
			HitInterface->Execute_GetHit(BoxHit.GetActor(), BoxHit.ImpactPoint);
		}
		IgnoreActors.AddUnique(BoxHit.GetActor());

		CreateFields(BoxHit.ImpactPoint);

		
	}
}
```
由于在Enemy.cpp中的TakeDamage需要伤害的接收，所以
5. 在AttributeComponent.h中声明一个用来计算生命值的方法，因为我们在AttributeComponent.h中声明的Health和MaxHealth都是private的，我们不想把他们改成public，即建立函数来进行计算
6. GetHealthPercent()用来计算血条百分比
```
public:
	void ReceiveDamage(float Damage);
	float GetHealthPercent();
```
在AttributeComponent.cpp中实现方法，clamp夹值函数，最小不低于0，最大不高于100
```
void UAttributeComponent::ReceiveDamage(float Damage)
{
	Health = FMath::Clamp(Health - Damage, 0.f, MaxHealth);
}
float UAttributeComponent::GetHealthPercent()
{
	return Health / MaxHealth;
}
```
最终回到Enemy.cpp完善最后的函数，应用伤害以及设置百分比血条
```
float AEnemy::TakeDamage(float DamageAmount, FDamageEvent const& DamageEvent, AController* EventInstigator, AActor* DamageCauser)
{
	if (Attributes && HealthBarWidget) {
		Attributes->ReceiveDamage(DamageAmount);

		HealthBarWidget->SetHealthPercent(Attributes->GetHealthPercent());

	}
	return DamageAmount;
}
```
## 制作敌人死亡
由于此时不仅是受击且要判断死亡，所以需要修改之前的GetHit函数
在Enemy.h中声明一个DeathMontage，在蓝图中给其赋值，Die函数用来播放蒙太奇
```
private:
	UPROPERTY(EditDefaultsOnly, Category = Montages)
	UAnimMontage* DeathMontage;

protected:
	void Die();
```
在Enemy.cpp中，将其分为受击或死亡两种情况，通过IsAlive()函数来判断
```
void AEnemy::GetHit_Implementation(const FVector& ImpactPoint)
{
	//DRAW_SPHERE_COLOR(ImpactPoint, FColor::Orange);
	
	if (Attributes && Attributes->IsAlive()) {
		DirectionHitReact(ImpactPoint);
	}
	else {
		Die();
	}
。。。省略
}
```
对于Die函数的实现，通过其名字来对应蒙太奇片段并播放
```
void AEnemy::Die()
{
	UAnimInstance* AnimInstance = GetMesh()->GetAnimInstance();
	if (AnimInstance && DeathMontage) {
		AnimInstance->Montage_Play(DeathMontage);

		const int32 Selection = FMath::RandRange(0, 5);
		FName SectionName = FName();
		switch (Selection) {
		case0:
			SectionName = FName("Death1");
			break;
		case1:
			SectionName = FName("Death2");
			break;
		case2:
			SectionName = FName("Death3");
			break;
		case3:
			SectionName = FName("Death4");
			break;
		case4:
			SectionName = FName("Death5");
			break;
		case5:
			SectionName = FName("Death6");
			break;
		default:
			break;
		}
		AnimInstance->Montage_JumpToSection(SectionName, HitReactMontage);
	}
}
```
此时我们会发现死亡后会经历死亡动画之后回到Idle状态，这是由于我们的状态机只设定了Idle状态，所以
1. 建立状态机Main States
2. 在Main States中实现Idle和Dead的切换
![输入图片说明](/imgs/2024-08-19/D5EfPysUSv6wZeHu.png)
![输入图片说明](/imgs/2024-08-19/CfS2O6U5AHf4qQYA.png)
此时引入新问题，如果播放对应的六种Dead动画，引入新的枚举类
### 死亡枚举类
在CharacterTypes.h中新建一个EDeathPose用来枚举存活和死亡姿势
```
UENUM(BlueprintType)
enum class EDeathPose : uint8
{
	EDP_Alive UMETA(DisplayName = "Alive"),
	EDP_Death1 UMETA(DisplayName = "Death1"),
	EDP_Death2 UMETA(DisplayName = "Death2"),
	EDP_Death3 UMETA(DisplayName = "Death3"),
	EDP_Death4 UMETA(DisplayName = "Death4"),
	EDP_Death5 UMETA(DisplayName = "Death5"),
	EDP_Death6 UMETA(DisplayName = "Death6")
};
```
在Enemy.h中声明一个DeathPose初始值为Alive
```
#include "Characters/CharacterTypes.h"
protected:
	UPROPERTY(BlueprintReadOnly)
	EDeathPose DeathPose = EDeathPose::EDP_Alive;
```
在Enemy.cpp中，完善之前的Die函数，利用Switch把状态分别设置
```
void AEnemy::Die()
{
	UAnimInstance* AnimInstance = GetMesh()->GetAnimInstance();
	if (AnimInstance && DeathMontage) {
		AnimInstance->Montage_Play(DeathMontage);

		const int32 Selection = FMath::RandRange(0, 5);
		FName SectionName = FName();
		switch (Selection) {
		case 0:
			SectionName = FName("Death1");
			DeathPose = EDeathPose::EDP_Death1;
			break;
		case 1:
			SectionName = FName("Death2");
			DeathPose = EDeathPose::EDP_Death2;
			break;
		case 2:
			SectionName = FName("Death3");
			DeathPose = EDeathPose::EDP_Death3;
			break;
		case 3:
			SectionName = FName("Death4");
			DeathPose = EDeathPose::EDP_Death4;
			break;
		case 4:
			SectionName = FName("Death5");
			DeathPose = EDeathPose::EDP_Death5;
			break;
		case 5:
			SectionName = FName("Death6");
			DeathPose = EDeathPose::EDP_Death6;
			break;
		default:
			break;
		}
		AnimInstance->Montage_JumpToSection(SectionName, DeathMontage);
	}
}
```
## 怎么样在敌人死亡后播放随机动画
我们在代码中已经处理好了随机播放死亡动画的效果，通过Enemy中声明的DeathPose可以得到此时是哪一种死亡姿势
### 接下来我们在动画蓝图中处理播放动画的效果
通过**事件蓝图初始化动画**，我们将Enemy提升为变量，此时即可访问我们在c++中Enemy类里面所声明的DeathPose枚举
![输入图片说明](/imgs/2024-08-20/SHOLAgRpRKYRMPtp.png)
### 方法1
利用事件蓝图更新动画（此事件为每帧调用），将DeathPose提升为变量，然后根据c++中的DeathPose来Set刚刚提升为变量的蓝图DeathPose
![输入图片说明](/imgs/2024-08-20/gUMd3sgfqvoTphVr.png)
### 方法2
通过**蓝图线程安全更新动画**函数（同样是每帧调用）
此行为通过多线程技术，需要遵守线程安全，可以避免其他地方同时修改了此状态导致同时发生了变化
![输入图片说明](/imgs/2024-08-20/U2MkL5yvH0aWYJHm.png)
找到Enemy，以及Enemy属性中的DeathPose，用它来设置动画蓝图中的DeathPose
![输入图片说明](/imgs/2024-08-20/s650jvYQOHVCUdfg.png)
![输入图片说明](/imgs/2024-08-20/KOkBSYIVGdzjja96.png)
![输入图片说明](/imgs/2024-08-20/jdx6sMQroTKVj9WM.png)
最后判断条件，判断什么时候应该从Idle转化成Dead
![输入图片说明](/imgs/2024-08-20/uf7FybSu9ldCJ0vB.png)
![输入图片说明](/imgs/2024-08-20/oFSAD3Ob8pK8aAoV.png)
在Dead状态下使用混合姿势，在最后Active Enum Value使用我们的DeathPose枚举值，即可根据枚举值来进行姿势的选择输出
![输入图片说明](/imgs/2024-08-20/S5deB9qAmQ1gAE6J.png)

## 敌人死亡后解除胶囊体碰撞以及销毁操作
1. 使用SetCollisionEnabled将敌人的胶囊体设为无碰撞
2. 
```
void AEnemy::Die()
{
	。。。省略
	GetCapsuleComponent()->SetCollisionEnabled(ECollisionEnabled::NoCollision);
	SetLifeSpan(3.f);
}
```
## 关于血条UI的设置
### 初始状态下不显示
在Enemy.cpp的BeginPlay()中，将HealthBarWidget设为不可见
```
void AEnemy::BeginPlay()
{
	Super::BeginPlay();
	if (HealthBarWidget) {
		HealthBarWidget->SetVisibility(false);
	}
	
}
```
### 受击后显示
```
void AEnemy::GetHit_Implementation(const FVector& ImpactPoint)
{
	//DRAW_SPHERE_COLOR(ImpactPoint, FColor::Orange);
	if (HealthBarWidget) {
		HealthBarWidget->SetVisibility(true);
	}
	。。。省略
}
```
### 远离一定距离后再次隐藏血条UI
在Enemy.h中设置攻击对象，UPROPERTY()使其保证为空指针。
CombatRadius为可视距离
```
private:
	UPROPERTY()
	AActor* CombatTarget;
	UPROPERTY(EditAnywhere)
	double CombatRadius = 500.f;
```
在Enemy.cpp中的TakeDamage，由EventInstigator->GetPawn()得到Pawn，赋值给CombatTarget
```
float AEnemy::TakeDamage(float DamageAmount, FDamageEvent const& DamageEvent, AController* EventInstigator, AActor* DamageCauser)
{
	if (Attributes && HealthBarWidget) {
		Attributes->ReceiveDamage(DamageAmount);

		HealthBarWidget->SetHealthPercent(Attributes->GetHealthPercent());

	}
	CombatTarget = EventInstigator->GetPawn();
	return DamageAmount;
}
```
在Tick函数中，用距离大小来判断是否该显示，如果超过距离则将CombatTarget设为空指针，再次击打才会赋值
```
void AEnemy::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

	if (CombatTarget) {
		const double DistanceToTarget = (CombatTarget->GetActorLocation() - GetActorLocation()).Size();
		if (DistanceToTarget > CombatRadius) {
			CombatTarget = nullptr;
			if (HealthBarWidget) {
				HealthBarWidget->SetVisibility(false);
			}

		}
	}
}
```
在Die函数中也添加SetVisibility(false)，否则三秒后空血条才会消失
```
void AEnemy::Die()
{
。。。省略
	if (HealthBarWidget) {
		HealthBarWidget->SetVisibility(false);
	}

	GetCapsuleComponent()->SetCollisionEnabled(ECollisionEnabled::NoCollision);
	SetLifeSpan(3.f);
}
```
## 制作敌人动画追击效果
制作一个BlendSpace1D（混合空间）
并在蓝图中使用
此时我们需要如下一个GroundSpeed变量
![输入图片说明](/imgs/2024-08-20/HiMxPdPR6yKxa9VF.png)
可以到事件图表中获取角色移动并将其提升为变量
![输入图片说明](/imgs/2024-08-20/bJ4zFCLCQaQkDCjW.png)
根据之前的多线程函数原理，在Property Access中获得CharacterMovement中的Velocity（速度），并根据此的搭配向量长度XY，即可用来Set速度，即得到了Ground Speed
![输入图片说明](/imgs/2024-08-20/XUqEl0UcQOJEwqIu.png)
### 敌人ai追击漂移问题
![输入图片说明](/imgs/2024-08-21/C0ZrPsIUZbjaJQoO.png)
取消Yaw轴旋转
![输入图片说明](/imgs/2024-08-21/52rnnQ0tJNtFtSSb.png)
在Enemy.cpp中的构造器中用程序声明以上操作
```
#include "GameFramework/CharacterMovementComponent.h"
AEnemy::AEnemy()
{
。。。省略
	GetCharacterMovement()->bOrientRotationToMovement = true;
	bUseControllerRotationPitch = false;
	bUseControllerRotationYaw = false;
	bUseControllerRotationRoll = false;
}
```
## 制作敌人巡逻系统
在build.cs加入"**AIMoudle**"模块
在Enemy.h中声明变量
1. EnemyController为敌人的控制器
2. PatrolTarget当前准备前往的巡逻点
3. PatrolTargets为一个TArray数组存放着许多巡逻点
```
/**
	* Navigation
	**/
	UPROPERTY()
	class AAIController* EnemyController;

	// Current patrol target
	UPROPERTY(EditInstanceOnly, Category = "AI Navigation")
	AActor* PatrolTarget;

	UPROPERTY(EditInstanceOnly, Category = "AI Navigation")
	TArray<AActor*> PatrolTargets;
```
### 对于Target Point
在Enmey.cpp中，在BeginPlay的方法中制作敌人走向PatrolTarget的功能
1. 将敌人的controller转化为AIController
2. 声明一个FAIMoveRequest MoveRequest
3. 将MoveRequest的目标actor设为PatrolTarget
4. 将MoveRequest的AcceptanceRadius设为15，意为走到距离目标点15的地方结束
5. FNavPathSharedPtr NavPath为共享指针声明的导航路径
6. NavPath->GetPathPoints()返回的为TArray数组，使用debug画出这些点会发现它是寻路中生成的导航点，每两个点中间是一条直线
```
#include "AIController.h"
void AEnemy::BeginPlay()
{
	Super::BeginPlay();
	if (HealthBarWidget) {
		HealthBarWidget->SetVisibility(false);
	}
	
	EnemyController = Cast<AAIController>(GetController());
	if (EnemyController && PatrolTarget) {
		FAIMoveRequest MoveRequest;
		MoveRequest.SetGoalActor(PatrolTarget);
		MoveRequest.SetAcceptanceRadius(15.f);
		FNavPathSharedPtr NavPath;
		EnemyController->MoveTo(MoveRequest, &NavPath);
		TArray<FNavPathPoint>& PathPoints = NavPath->GetPathPoints();
		for (auto& Point : PathPoints) {
			const FVector& Location = Point.Location;
			DrawDebugSphere(GetWorld(), Location, 12.f, 12, FColor::Green, false, 10.f);
		}
	}
}
```
在蓝图中放置一个target point并在实例中进行配置
![输入图片说明](/imgs/2024-08-21/Zoh4rkLYlbIP8BQS.png)
在实例中进行设置，可以分别为不同的敌人设置不同的巡逻点
![输入图片说明](/imgs/2024-08-21/LYlUzrm9QCO80ggQ.png)
运行后：：：
此时我们做到了ai寻路到targetPoint，但仅是该点
![输入图片说明](/imgs/2024-08-21/SrhREBr0fcK7dc5e.png)
### 对于Target Points（数组）
在Enemy.cpp中将判断距离声明成一个方法
```
bool AEnemy::InTargetRange(AActor* Target, double Radius)
{
	const double DistanceToTarget = (Target->GetActorLocation() - GetActorLocation()).Size();
	return DistanceToTarget <= Radius;
}
```
在Tick函数中，实现根据巡逻点随机巡逻的功能
```
void AEnemy::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

	if (CombatTarget) {
		if (!InTargetRange(CombatTarget, CombatRadius)) {
			CombatTarget = nullptr;
			if (HealthBarWidget) {
				HealthBarWidget->SetVisibility(false);
			}

		}
	}
	if (PatrolTarget && EnemyController) {
		if (InTargetRange(PatrolTarget, PatrolRadius)) {

			TArray<AActor*> VaildTargets;//有效巡逻点数组
			for (auto Target : PatrolTargets) {
				if (Target != PatrolTarget) {//将不是当前的巡逻点的剩下全部添加进该数组中
					VaildTargets.AddUnique(Target);//防止随机到同一个巡逻点
				}
			}

			const int32 NumPatrolTargets = VaildTargets.Num();
			if (NumPatrolTargets > 0) {
				const int32 TargetSelection = FMath::RandRange(0, NumPatrolTargets - 1);//随机
				AActor* Target = VaildTargets[TargetSelection];

				PatrolTarget = Target;

				FAIMoveRequest MoveRequest;
				MoveRequest.SetGoalActor(PatrolTarget);
				MoveRequest.SetAcceptanceRadius(15.f);
				EnemyController->MoveTo(MoveRequest);//此时我们不需要声明上面的FNavPathSharedPtr NavPath，
				//是因为我们刚刚目的是为了绘制debug球体进行观察
			}
		}
	}
}
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIwMjk2ODM4MTMsMTAzNTcyNDE5OCwxMT
gxOTUzODg3LC02NTg3MTU2MjQsNzYzNzY0MjkwLDE4MTE4Nzg5
NTEsMTM3ODYwMDc3NSwtMTUwMDAyNTAsLTE2MjE3OTI5ODgsMT
ExOTc3MDEwMiwtMzAzMjY5ODAxLC0xNjY2NTU2NTY0LDQ5Nzgy
MDkyMyw3Nzc4OTEzOTAsNjQzMTc0NjAxLC0xMDc1MTM0NTIxLC
00NjQ4OTQzMjUsLTI1NDIzOTE3MywxODYzNjU4MDYsNDYwNjQz
MTA4XX0=
-->