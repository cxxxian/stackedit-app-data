前者是
```
UE_LOG(LogTemp, Warning, TEXT("begin play called"));
if (GEngine) {
		GEngine->AddOnScreenDebugMessage(1, 60.f, FColor::Cyan, FString("Item OnScreen Message"));
	}
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTU3MjQxODg3N119
-->