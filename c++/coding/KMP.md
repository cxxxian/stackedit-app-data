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

# 
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTYwMzYxNTMxOCwtMjA3NjA5ODYxM119
-->