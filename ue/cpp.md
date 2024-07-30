前者是在输出日志中输出，后者是直接输出在屏幕上。
此处GEngineshi
```
UE_LOG(LogTemp, Warning, TEXT("begin play called"));
if (GEngine) {
		GEngine->AddOnScreenDebugMessage(1, 60.f, FColor::Cyan, FString("Item OnScreen Message"));
	}
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTcyMzg2MjQ0MV19
-->