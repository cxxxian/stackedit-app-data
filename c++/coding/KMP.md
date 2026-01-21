构建`next`数组方法：
```cpp
void getNext(vector<int>& next, const string& s){
	int j = 0;
	next[0] = 0;
	for(int i = 1; i < s.length(); i++){
		while(j > 0 && s[i] != s[j]){
			j = next[j - 1];
		}

		if(s[i] == s[j]){
			j++;
		}
		next[i] = j;
	}
}
```
# 实现 strStr()
其实`getNext`和主函数`strStr`很像，就是注意一下`getNext`的`i`是从`1`开始循环的，而`strStr`是从`0`
```cpp
class Solution {
public:
	void getNext(vector<int>& next, const string& s){
		int j = 0;
		next[0] = 0;
		for(int i = 1; i < s.length(); i++){
			while(j > 0 && s[i] != s[j]){
				j = next[j - 1];
			}

			if(s[i] == s[j]){
				j++;
			}
			next[i] = j;
		}
	}
	int strStr(string haystack, string needle) {
		if(needle.length() == 0){
			return 0;
		}
		vector<int> next(needle.length(), 0);
		getNext(next, needle);
		
		int j = 0;

		for(int i = 0; i < haystack.length(); i++){
			while(j > 0 && haystack[i] != needle[j]){
				j = next[j - 1];
			}
			if(haystack[i] == needle[j]){
				j++;
			}

			if(j == needle.length())
				return (i - needle.length() + 1);
			}

		}
	return -1;
	}
};
```

## eg2
```cpp
class Solution {
public:
    void getNext (vector<int>& next, const string& s){
        next[0] = 0;
        int j = 0;
        for(int i = 1;i < s.size(); i++){
            while(j > 0 && s[i] != s[j]) {
                j = next[j - 1];
            }
            if(s[i] == s[j]) {
                j++;
            }
            next[i] = j;
        }
    }
    bool repeatedSubstringPattern (string s) {
        if (s.size() == 0) {
            return false;
        }
        int next[s.size()];
        getNext(next, s);
        int len = s.size();
        if (next[len - 1] != 0 && len % (len - (next[len - 1] )) == 0) {
            return true;
        }
        return false;
    }
};

```

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTMzNjY1ODg3MSwtMTgyMjc5NTc2MSwtMj
A3NjA5ODYxM119
-->