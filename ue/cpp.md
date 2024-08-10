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
在.
void EKeyPressed();
将事件绑定
```
PlayerInputComponent->BindAction(FName("Equip"), IE_Pressed, this, ASlashCharacter::EKeyPressed);
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTEwNjk4MzM3OSwxNjkyNjk2NTI4LC02MT
QzMDk2MzIsMTQ0MDAzODU5NiwtODMwNDE0NTU2LDc2Nzc3NzIy
MCwtMTUyNDYxMzE1LC0yMDcyMjM1MDc3LC0yMDk2Mjc0MjYxLC
0xMDUzNTcwNDcwLC0xMzAzODQwNTIzLDM5MDMzMDMwMiw3Mzc3
ODMzMDUsNTQ5NTM3MjE5LDI5MjM1MTEyOCwyMDU2MjQ2NDAxLC
0xNjgyNTI0MzA0LC00NDg4MTU1MjksMzg0NTE5MTIwLDI5OTgz
OTgzOV19
-->