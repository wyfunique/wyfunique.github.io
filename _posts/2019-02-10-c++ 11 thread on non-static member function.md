---
layout: post
title: How to create a thread using non-static member function in c++ 11
category: 
- C++
---



## How to create a thread using non-static member function in c++ 11

You need to provide an object for you to call a non-static member function, just as you can't call `method()` on its own. To provide that object, pass it to `std::thread`'s constructor after the argument where you put the function.

```c++
struct Test {
    void func(int x) {}
};


int main() {
    Test obj;
    std::thread t(&Test::func, &obj, 42);
    t.join();
}
```

Notice that I've passed `&x`. This is because non-static class functions accepts a pointer to the object where it is being called from, and this pointer is the `this` pointer. The rest, which is `42`, is the arguments that corresponds to the method's parameter declaration with `42` coinciding with `int x` in the example.