### 1. `enum class EActionState : uint8`

**`enum class`的特点**：

- **强类型（Scoped Enum）**：`enum class` 是一种强类型枚举，这意味着枚举的枚举成员必须通过作用域来访问。例如，如果有一个枚举成员 `Idle`，你需要通过 `EActionState::Idle` 来访问它。
- **命名空间隔离**：枚举成员不会自动引入到外层作用域中，这减少了命名冲突的风险。
- **底层类型指定**：在这里，`: uint8` 指定了枚举的底层存储类型为 `uint8`，也就是无符号 8 位整数。这样可以明确控制枚举所占用的内存大小，可能对内存优化有帮助。

**示例**：

```cpp
enum class EActionState : uint8
{
    Idle,
    Walking,
    Running,
    Jumping
};

// 使用时
EActionState CurrentState = EActionState::Idle;
```

### 2. `enum EDeathPose`

**传统`enum`的特点**：

- **弱类型（Unscoped Enum）**：传统的 `enum` 是弱类型的，枚举成员直接暴露在外层作用域中。你可以直接使用成员名称，而不需要通过枚举名访问它们。例如，如果有一个枚举成员 `Dead`，你可以直接使用 `Dead`，不需要 `EDeathPose::Dead`。
- **未指定底层类型**：默认情况下，传统枚举的底层类型通常是 `int`（可以根据编译器不同有所变化）。你不能显式指定它的底层类型，除非通过编译器特定的扩展。
- **潜在的命名冲突**：由于枚举成员直接暴露在外层作用域中，如果不同的枚举类型有相同名称的成员，会导致命名冲突。

**示例**：

```cpp
enum EDeathPose
{
    Dead,
    Dying,
    Reviving
};

// 使用时
EDeathPose CurrentPose = Dead;
```

### 主要区别总结

- **类型安全**：`enum class` 更加类型安全，需要使用作用域来访问成员，减少命名冲突的风险。而传统的 `enum` 成员会直接暴露在外层作用域中，可能导致命名冲突。
- **底层类型**：`enum class` 可以显式指定底层类型（如 `uint8`），有助于内存优化；传统的 `enum` 则由编译器决定其底层类型，通常是 `int`。
- **使用场景**：`enum class` 通常用于需要更强类型检查的场合，而传统 `enum` 常用于简单的枚举情况。

在实际使用中，`enum class` 是更现代的用法，推荐用于新代码中以避免潜在的错误。
<!--stackedit_data:
eyJoaXN0b3J5IjpbMjAyOTc4NTM5MF19
-->