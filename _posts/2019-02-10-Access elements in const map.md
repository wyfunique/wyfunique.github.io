---
layout: post
title: How to access elements in a const map in C++?
category: 
- C++
---



# How to access elements in a const map in C++?

If you try to access to elements in a const map using operator[], it will throw a compilation error.

```c++
using namespace std;
int main()
{
	const map<string, int> m = {{"A", 0}, {"B", 1}, {"C", 2}};
	cout << m["A"] << enel;
}
```

This will throw a compilation error like below:

```c++
main.cc: In function ‘int64_t count(const std::map<std::basic_string<char>, long int>&)’:
main.cc:7:19: error: passing ‘const std::map<std::basic_string<char>, long int>’ as ‘this’ argument of ‘std::map<_Key, _Tp, _Compare, _Alloc>::mapped_type& std::map<_Key, _Tp, _Compare, _Alloc>::operator[](std::map<_Key, _Tp, _Compare, _Alloc>::key_type&&) [with _Key = std::basic_string<char>; _Tp = long int; _Compare = std::less<std::basic_string<char> >; _Alloc = std::allocator<std::pair<const std::basic_string<char>, long int> >; std::map<_Key, _Tp, _Compare, _Alloc>::mapped_type = long int; std::map<_Key, _Tp, _Compare, _Alloc>::key_type = std::basic_string<char>]’ discards qualifiers [-fpermissive]
   return map["one"] + map["two"];
                   ^
main.cc:7:32: error: passing ‘const std::map<std::basic_string<char>, long int>’ as ‘this’ argument of ‘std::map<_Key, _Tp, _Compare, _Alloc>::mapped_type& std::map<_Key, _Tp, _Compare, _Alloc>::operator[](std::map<_Key, _Tp, _Compare, _Alloc>::key_type&&) [with _Key = std::basic_string<char>; _Tp = long int; _Compare = std::less<std::basic_string<char> >; _Alloc = std::allocator<std::pair<const std::basic_string<char>, long int> >; std::map<_Key, _Tp, _Compare, _Alloc>::mapped_type = long int; std::map<_Key, _Tp, _Compare, _Alloc>::key_type = std::basic_string<char>]’ discards qualifiers [-fpermissive]
   return map["one"] + map["two"];
                                ^
```



**You can use `.at()` method to access elements in const map.**

The reason why `operator[]` can not be used for a const map is because that, if the key from `operator[]` does not exist, a pair with that key is created using default values and then returned, which will actually change the map.

Note that `at()` may throw `std::out_of_range` exception if no such data is present.

