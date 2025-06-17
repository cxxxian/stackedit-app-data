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
# substr
```cpp
int main()
{
	string s="sfsa";
	string a=s.substr(0,3);
	string b=s.substr();
	string c=s.substr(2,3);
	return 0;
}
```
`a = sfs, b = sfsa, c = sa`
返回值： `string`，包含`s`中从`pos`开始的`len`个字符的拷贝（`pos`的默认值是`0`，`len`的默认值是s.size() - pos，即不加参数会默认拷贝整个s）
异常 ：若pos的值超过了string的大小，则substr函数会抛出一个out_of_range异常；若pos+n的值超过了string的大小，则substr会调整n的值，只拷贝到string的末尾
<!--stackedit_data:
eyJoaXN0b3J5IjpbNDI4ODExNzUzLDQ4NTkxMDAxLC0xMzgyMD
k3MTg1XX0=
-->