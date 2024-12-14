
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
<!--stackedit_data:
eyJoaXN0b3J5IjpbNzYwNzY3Mzc5LC0xNjg2ODQwODA1LC0xOD
gxMDM5MTk3XX0=
-->