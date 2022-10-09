---
title: 'Why you must implement a template class in the header file?'
date: 2019-02-12
permalink: /posts/2019/02/blog-1-Implement_template_class_in_Cpp/
tags:
  - C++
---

```
The only portable way of using templates at the moment is to implement them in header files by using inline functions.
```

For a template class, you cannot implement it separately in a source code file (i.e., .cpp) and only place its declaration in the header file. Instead, you have to **implement it in the header file where it is declared.**

To understand this problem, we need to talk about compilation and linkage of C++ first.



### 1. C++ compilation and linkage

1. Only source code files will be compiled. **Header files will not be compiled directly**.

   ```
   1. Before compilation, C++ compiler will preprocess the source code, including replace "#include<XXX.h>" with its content in source code files. 
   2. Then in compilation, only the preprocessed source code files will be compiled to object files. Header files will be ignored.
   ```

    

2. **Separated compilation**

   (1) C++ compiler will do "separated compilation", which means it will compile each source file separately. When compiling one source file, the compiler does not know about any other source file. 

   (2) What if there are unsolved external symbols in a source file, like functions or classes imported from other source files? Compiler will not care about it. It just mark them and continue compiling. 

   (3) After compilation, linker will link all object files into an executable. So linker will know the whole picture. **In each object file, there is a symbol table** which records symbols implemented in this object file. So when linker meets a unsolved external symbol, it will try to find the symbol in other object files' symbol tables. If the symbol cannot be found in any object file, linker will throw an error "Undefined reference to XXX" where "XXX" is the symbol. 

   (4) In short, **separated compilation** means that **compiler only knows about current file** that is **being compiled** while it is the **linker who knows the whole picture**.



3. Summary

   ```
   1. Preprocessor replaces "#include<XXX.h>" with text content of the included header file in source code file.
   2. Compiler compiles source code files to object files one by one separately 
   3. Linker searches symbols in symbol tables and links all object files to an executable. 
   ```



### 2. Template class in compilation 

Template is only a pattern for your code. Compiler will not generate actual machine code for it in object files. Compiler will only generate machine code for instances of a template. 

Foe example, if you only has definition of a template class without explicit instantiation, like below:

```c++
// example.cpp
template <typename T>
class temp
{
    ......
}
```

Then you compile `example.cpp` to `example.o`. But there is **no information** about class `temp` in `example.o` actually as compiler does not generate machine code for a template class prototype. 

Only if you instantiate the template class, compiler will generate machine code for the instances:

```c++
// example.cpp
template <typename T>
class temp
{
    ......
}

// main.cpp
......
temp<int> test;
```

Compiler will generate machine code for `test` and linker can find information of `test` in `main.o`.



### 3. How compiler works for instances of a template class?

```c++
// example.cpp
template <typename T>
class temp
{
    public:
    	std::vector<T> storage;
    	void func(T input) {} // implementation
}

// main.cpp
......
temp<int> test;
```

When compiler goes to statement  `temp<int> test;`, it will first internally generate an instance of template class `temp` where `T` is replaced with the instantiated data type `int` :

```c++
temp<int> test
{
    public:
    	std::vector<int> storage;
    	void func(int input) {}
}
```

Then compiler compiles this instance, generates machine code, inserts its info into symbol table, etc. After that, the linker can find `test` in `main.o`.



### 4. What if you separate declaration and implementation of template in different files?

Suppose you declare a template class in header file while implement it in source file:

```c++
// temp.h
template <typename T>
class temp
{
    public:
    	std::vector<T> storage;
    	void func(T input); // Only declaration, no implmentation 
}

// temp.cpp
template <typename T>
void temp::func(T input) 
{
    //...
}
```

Then in `main.cpp` you include the header file and instantiate `temp`:

```c++
#include "temp.h"
int main()
{
    temp<int> test;
    return 0;
}
```

In this case, compilation will still succeed but linker will throw error "Undefined reference to temp\<int\>", as linker cannot determine what "temp\<int>\" is.

It is because implementation of class `temp` is not in `temp.h` , so even though you include it in `main.cpp`, you only import the interfaces of `temp` instead of all information about the class. So compiler will consider `temp<int>` as an external symbol and leave it to linker. But as compiler will not generate symbol table info for content in `temp.cpp`, linker cannot find information of `temp<int>`, so it throws error of undefined reference. 



### 5. Two ways to solve the problem

1. Combine declaration and implementation of a template into one header file

   ```c++
   // temp.h
   template <typename T>
   class temp
   {
       public:
       	std::vector<T> storage;
       	void func(T input); // Only declaration, no implmentation 
   }
   
   // Also in temp.h
   template <typename T>
   void temp::func(T input) 
   {
       //...
   }
   ```

   Then in `main.cpp`,

   ```c++
   #include "temp.h"
   int main()
   {
       temp<int> test;
       return 0;
   }
   ```

   At this time `main.cpp` still includes `temp.h`, but both declaration and implementation of `temp` are imported now. So compiler will consider it as internal (local) symbol and generate an instance of `temp` with type `int` when compiling. Then machine code and symbol table info of `temp<int>` will be generated to object file `main.o`. Later linker can find it in `main.o`.



2. Separate declaration and implementation, and explicitly instantiate all the template instances you need

   ```c++
   // temp.h
   template <typename T>
   class temp
   {
       public:
       	std::vector<T> storage;
       	void func(T input); // Only declaration, no implmentation 
   }
   
   // temp.cpp
   template <typename T>
   void temp::func(T input) 
   {
       //...
   }
   // explicitly instantiate template instances
   template class temp<int>; 
   template class temp<float>;
   ```

   As we said, compiler will not compile template but only its instances. So you can explicitly instantiate it. In the example above, when compiler goes to the last two lines, it will compile these two instances into `temp.o`, then linker can find them. 

   **NOTE:** In this solution, you have to instantiate every type of instance you need. For example, you only instantiate `temp<int>` and `temp<float>`, then you cannot use other types like `temp<string>`. You can only use types of the instances you instantiate in advance. So if you do not know what type you will need, you may have to choose the first solution. That is why somebody says that implementing template class in header file is *the **only** portable way of using templates*.

