---
layout: post
title: How to check if a string is set in C++
category: 
- C++
---



## How to check if a string is set in C++

1. Unlike `char*` or `char[]` , variable name of `string` in C++ is a reference instead of a pointer, so it cannot be set to `NULL` or `nullptr`. But a `string` can be **empty**.

2. When a `string ` object is declared but not initialized explicitly, it will be initialized as an empty string.

   ```c++
   // The creating results are the same, a, b and c are all string "".
   string a; // a is an empty string "".
   string b = "";
   string c("");
   ```

3.  So we can check **if a string is set** by checking **if it is empty**:

   ```c++
   string s;
   
   if (s.empty())
       // s is not set
   ```

   