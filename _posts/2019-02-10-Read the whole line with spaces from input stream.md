---
layout: post
title: How to read the whole line with spaces from input stream
category: 
- C++
tags:
- C++
---



## How to read the whole line with spaces from input stream

1. If you use `in >> var` to read a line with spaces, the line will be split by space automatically.

   ```c++
   std::string s;
   std::cin >> s;
   
   // if you input "I am XXX", s will be "I". Because "I am XXX" will be split by space to "I", "am" and "XXX". So s will only be set to the first value "I".
   ```

   

2. For reading a line without split, you can use `getline(stream, var)`. This function will read the raw input line without preprocessing it.

   ```c++
   std::string s;
   getline(std::cin, s);
   
   // When you input "I am XXX", s will be also "I am XXX".
   ```

   



