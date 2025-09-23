```cpp
#include <iostream>
#include <cstring>

class MyString {
private:
    char* data;
    size_t length;

public:
    // 构造函数
    MyString(const char* str = "") {
        length = std::strlen(str);
        data = new char[length + 1];
        std::memcpy(data, str, length + 1); // 包含 '\0'
    }

    // 拷贝构造函数 (深拷贝)
    MyString(const MyString& other) {
        length = other.length;
        data = new char[length + 1];
        std::memcpy(data, other.data, length + 1); // 包含 '\0'
    }

    // 析构函数
    ~MyString() {
        delete[] data;
    }

    // 插入函数
    void insert(size_t pos, const char* str) {
        if (pos > length) pos = length; // 防止越界

        size_t addLen = std::strlen(str);
        size_t newLen = length + addLen;

        char* newData = new char[newLen + 1];

        // 复制前半部分
        std::memcpy(newData, data, pos);

        // 插入新字符串
        std::memcpy(newData + pos, str, addLen);

        // 复制剩余部分
        std::memcpy(newData + pos + addLen, data + pos, length - pos);

        newData[newLen] = '\0';

        delete[] data;
        data = newData;
        length = newLen;
    }

    // 打印字符串
    void print() const {
        std::cout << data << std::endl;
    }
};

// 测试
int main() {
    MyString s1("Hello");
    s1.print(); // Hello

    MyString s2 = s1;  // 拷贝构造
    s2.print(); // Hello

    s1.insert(5, " World"); // 插入
    s1.print(); // Hello World

    s2.insert(0, "Hi, ");
    s2.print(); // Hi, Hello

    return 0;
}

```
<!--stackedit_data:
eyJoaXN0b3J5IjpbNDE4MjQ1Njg0XX0=
-->