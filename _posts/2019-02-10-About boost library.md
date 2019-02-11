---
layout: post
title: Notes about C++ boost
category: 
- C++
tags:
- C++
---



## Notes about C++ boost 

### 1. Build a full path string safely from separated strings

```c++
#include <iostream>
#include <boost/filesystem.hpp>

namespace fs = boost::filesystem;

int main ()
{
    fs::path dir ("/tmp");
    fs::path file ("foo.txt");
    fs::path full_path = dir / file; // '/' is overloaded for class path as path concatenation symbol
    std::cout << full_path << std::endl;
    return 0;
}
```



### 2. Convert boost path to string

```c++
#include <iostream>
#include <boost/filesystem.hpp>

namespace fs = boost::filesystem;

int main ()
{
    fs::path dir ("/tmp");
    std::string dir_str = dir.string(); // call .string() method of a path object to get its string representation 
}
```



### 3. Declaring a thread without initialization

Using C++ 11, you can declare a thread without initialization and initialize it later.

```c++
boost::thread t;
// initialize t like this by assigning another thread to it
t = boost::thread(&some_func);
// or like this by using swap()
// boost::thread(&some_func).swap(t);
```



### 4. Create a thread using function with arguments

```c++
void workerFunc(const char* msg, unsigned delaySecs); 
//...
boost::thread workerThread(workerFunc, "Hello, boost!", 3);
// The first arg 'workerFunc' is the name of function used to create this thread
// The following args are the args passed to the function 'workerFunc'.
```



### 5. Create a thread using non-static member function by standard c++ 11 or boost

(1) You need to provide an object for you to call a non-static member function, just as you can't call `method()` on its own. To provide that object, pass it to `std::thread`'s constructor after the argument where you put the function.

```c++
class Test {
    void func(int x) {}
};


int main() {
    Test obj;
    std::thread t(&Test::func, &obj, 42);
    t.join();
}
```

Notice that I've passed `&x`. This is because non-static class functions accepts a pointer to the object where it is being called from, and this pointer is the `this` pointer. The rest, which is `42`, is the arguments that corresponds to the method's parameter declaration with `42` coinciding with `int x` in the example.

(2) By boost thread, it is the same case:

```c++
class Test {
    void func(int x) {}
};


int main() {
    Test obj;
    boost::thread t(&Test::func, &obj, 42);
    t.join();
}
```

(3) If the created thread and the function it's based on are in the same class, we can pass `this` as the object pointer argument:

```c++
class Test {
    void func(int x) {}
    void func2()
    {
        boost::thread t(&Test::func, this, 42); 
        // func and t are both in class Test, so we use 'this' as object pointer here.
    }
};
```



