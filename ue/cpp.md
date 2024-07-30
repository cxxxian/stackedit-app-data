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
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEzNDcyMjk5NTMsLTkzOTUxOTk3NSwtMT
U4ODQ5ODQwNV19
-->