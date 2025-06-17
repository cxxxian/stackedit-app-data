# reverse
`reverse`用法：
```cpp
string t = "aabbcc"
reverse(result.begin(), result.end());`
```

# std::stoi(s)
可以将字符串转成`int`
```cpp
std::string s = "12345";
int num = std::stoi(s);
std::cout << num;  // 输出 12345
```
# to_string
把数字转化成字符串
```cpp
std::string to_string(int value);
```

形式 ： s.substr(pos, len)
返回值： string，包含s中从pos开始的len个字符的拷贝（pos的默认值是0，len的默认值是s.size() - pos，即不加参数会默认拷贝整个s）
异常 ：若pos的值超过了string的大小，则substr函数会抛出一个out_of_range异常；若pos+n的值超过了string的大小，则substr会调整n的值，只拷贝到string的末尾
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTYwOTQ4NTExNiw0ODU5MTAwMSwtMTM4Mj
A5NzE4NV19
-->