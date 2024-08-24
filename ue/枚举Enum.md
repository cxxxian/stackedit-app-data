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

`TEnumAsByte` 是 Unreal Engine 中的一个模板类，它用于将枚举类型的存储空间缩小为一个字节（`uint8`），以节省内存。这在处理大量枚举类型数据时特别有用，尤其是当你知道枚举的值不会超过 256 个（即 0 到 255 ）。

### 作用与用法

通常，枚举类型在 C++ 中的底层存储类型是 `int`，这意味着即使你的枚举值范围很小，仍然会占用 4 个字节（32 位）。`TEnumAsByte` 将其存储空间缩小到 1 个字节（8 位），使枚举占用的内存减少。

### 示例代码

```cpp
TEnumAsByte<EDeathPose> DeathPose;
```

在这个例子中：

- **`EDeathPose`**：这是一个枚举类型，假设它的声明类似于 `enum EDeathPose { Dead, Dying, Reviving };`。
- **`TEnumAsByte<EDeathPose>`**：这个模板类将 `EDeathPose` 枚举的存储空间压缩到 1 个字节。换句话说，`DeathPose` 这个变量只会占用 1 个字节的内存，而不是通常的 4 个字节。

## TEnumAsByte
### 适用场景

`TEnumAsByte` 常用于对内存敏感的场景，如网络数据包、存储大量对象的容器等。在这些场景下，节省的内存空间可以显著提高效率。

### 注意事项

- **枚举值范围**：使用 `TEnumAsByte` 时，要确保枚举值不会超过 `uint8` 的范围，即 0 到 255。如果枚举值超出这个范围，可能会导致不可预期的行为。
- **内存优化**：虽然可以节省内存，但在某些情况下，这种节省可能并不明显。因此，使用 `TEnumAsByte` 时需要权衡实际需求。

总体来说，`TEnumAsByte` 是 Unreal Engine 提供的一个实用工具，帮助开发者优化枚举类型的内存占用。
eg、
```
UPROPERTY(BlueprintReadOnly)
TEnumAsByte<EDeathPose> DeathPose;
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTk5NTE1MTI5M119
-->