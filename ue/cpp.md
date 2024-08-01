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
创建默认子对象，子对象的类型为静态网格体组件（UStaticMeshComponent），使用文本宏给该组件添加内部名字TEXT("ItemMeshComponent")。
```
CreateDefaultSubobject<UStaticMeshComponent>(TEXT("ItemMeshComponent"));
```
以上该函数返回一个指向新创建对象的指针，赋值给在.h中声明的ItemMesh。
在.h中声明的ItemMesh初始状态下只是一个空指针，需要创建新组建并将地址存储在这个指针中。
```
UPROPERTY(VisibleAnywhere)
UStaticMeshComponent* ItemMesh;
```
```
ItemMesh = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("ItemMeshComponent"));
RootComponent = ItemMesh;
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTk1MDY0MjQ1NiwxMjg0MDMyNDI0LDE0Nz
k3NDI4MzksNTAxNDgxMjg2LDI1NzMwNzc5Miw1NTEzMTk1NjMs
LTExOTk0NjYyNDIsNzI0MDcxNDE1LC0yMTI0NDczMjY1LC01Nj
U4ODM2NjAsMTI4MTg2MDcyNCwtMTAxMjUwNjQ0NywtMTQwMjg2
OTY2Myw2MTA3NjIwMTYsLTg5NzMzNDk5OCw2NjY5OTg5MjIsMT
IxNDg0MzI2LC0xMzQ3MjI5OTUzLC05Mzk1MTk5NzUsLTE1ODg0
OTg0MDVdfQ==
-->