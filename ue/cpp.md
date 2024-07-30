前者是在输出日志中输出，后者是直接输出在屏幕上。
此处GEngine是个指针可能为空，空指针会导致程序崩溃，所以加上if判断。
```
UE_LOG(LogTemp, Warning, TEXT("begin play called"));
if (GEngine) {
		GEngine->AddOnScreenDebugMessage(1, 60.f, FColor::Cyan, FString("Item OnScreen Message"));
	}
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE1ODg0OTg0MDVdfQ==
-->