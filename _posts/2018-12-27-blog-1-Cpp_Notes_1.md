---
title: 'C++ Notes (1)'
date: 2018-12-27
permalink: /posts/2018/12/blog-1-Cpp_Notes_1/
tags:
  - C++
  - WinAPI
---

Notes on C++ (1).

### 1. Map / Unordered_map inserts reference

When inserting reference object into vector, map, unordered_map, etc., copy constructor of the object will be called to generate a copy, then this copy will be inserted instead of the original object. So if your want to insert the reference object itself, please insert a pointer pointing to it. 



### 2. List vs. vector

In c++, list is implemented using doubly linked list, while vector is implemented using array.

So list only needs O(1) to insert into front while vector needs O(n). And list requires O(n) for random access while vector only needs O(1).



### 3. Get current directory (on Windows)

```c++
#include <windows.h>

char buf[256];
GetCurrentDirectoryA(256, buf);
std::string cur_dir = std::string(buf) + std::string("\\");
```



### 4. Check if a file or directory exists (on Windows)

```c++
DWORD ftyp = GetFileAttributesA(dir.c_str());
// Something is wrong with the path
if (ftyp == INVALID_FILE_ATTRIBUTES)
{
	if (GetLastError() == ERROR_FILE_NOT_FOUND) // No such dir
	{
		_mkdir(dir.c_str()); // Create it
	}
	else
	{
		// Error in dir name
        exit(1);
	}
}
```

NOTE: 

(1) `INVALID_FILE_ATTRIBUTES` means many cases. It can be case of invalid dirname with special chars. It may be also because the dir does not exist.

(2) So we need  `GetLastError()` to get the last error to know what the error actually is, like `ERROR_PATH_NOT_FOUND`, `ERROR_FILE_NOT_FOUND`, `ERROR_INVALID_NAME`, or `ERROR_BAD_NETPATH` .

Reference: https://stackoverflow.com/questions/8233842/how-to-check-if-directory-exist-using-c-and-winapi



### 5. Determine CPU and memory consumption from inside a process

It is possible to determine CPU, virtual memory and physical memory consumption from inside a program. 

Reference: https://stackoverflow.com/questions/63166/how-to-determine-cpu-and-memory-consumption-from-inside-a-process



### 6. Read huge text file line by line in C++ 

**(1) Opening a file will NOT lead it to be loaded into memory**

Opening a file using interface like `fopen()` will just retrieve a file descriptor. And content of the file will be loaded into memory only when you read them explicitly, like using `fread()` or `getline()`. So opening a huge file will not make your memory overflow.

**(2) Speed of different reading methods**

```
(1) IO streams in various combinations: ~10 MB/s. Pure parsing (f >> i1 >> i2 >> d) is faster than a getline into a string followed by a sstringstream parse.
(2) C file operations like fscanf get about 40 MB/s.
(3) getline with no parsing: 180 MB/s.
(4) fread: 500-800 MB/s (depending on whether or not the file was cached by the OS).
```

Reference: https://stackoverflow.com/questions/24851291/read-huge-text-file-line-by-line-in-c-with-buffering


