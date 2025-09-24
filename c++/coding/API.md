# reverse
`reverse`用法：
```cpp
#include<algorithm>
string t = "aabbcc"
reverse(result.begin(), result.end());`
```

# std::stoi(s)
可以将字符串转成`int`
```cpp
#include<string>
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
返回值： `string`，包含`s`中从`pos`开始的`len`个字符的拷贝（`pos`的默认值是`0`，`len`的默认值是`s.size() - pos`，即不加参数会默认拷贝整个`s`）
异常 ：若`pos`的值超过了`string`的大小，则`substr`函数会抛出一个`out_of_range`异常；若`pos+n`的值超过了`string`的大小，则`substr`会调整`n`的值，只拷贝到`string`的末尾


# string的insert
在字符串`s`的某个位置加入一个`char`字符
```cpp
s.insert(s.begin() + i + 1, '.');
```
字符串增加字符的方法：

![输入图片说明](/imgs/2025-06-17/gGFaRv7baVROCfZV.png)

# string的声明
长度为`n`，内部都是`'.'`
然后`vector<string> chess`同理，`n`个元素，每个元素都是`str`
```cpp
string str(n, '.');
vector<string> chess(n, str);
backTracing(n, 0, chess);
return result;
```
# getline
整行输入
```cpp
string word;
getline(cin, word);
```
如果如以下写法，会造成例如输入`ABSIB T`，只会输入`ABSIB`
```cpp
string word;
cin >> word;
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTM4ODE2MTY4NiwtMTI4ODA0MTU5LC02MD
AzMTkzNywxNjM0NzU3ODAzLDQ4NTkxMDAxLC0xMzgyMDk3MTg1
XX0=
-->