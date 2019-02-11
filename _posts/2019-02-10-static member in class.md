---
layout: post
title: Notes about C++ static members of class
category: 
- C++
---



## Notes about C++ static members of class

### 1. We cannot initialize a static member variable in-class

Such case is not allowed:

```c++
// An example of singleton pattern
class Logger{
public:
   static Logger* Instance();
   bool openLogFile(std::string logFile);
   void writeToLogFile();
   bool closeLogFile();
private:
   Logger(){};  // Private so that it can  not be called
   Logger(Logger const&){};             // copy constructor is private
   Logger& operator=(Logger const&){};  // assignment operator is private
   
   // The initialization of static member here is not allowed
   // This will cause "In-class initialization" error
   static Logger* m_pInstance = NULL; 
};
```

We have to initialize static members **outside of the class definition**.

```c++
// Correct example
// logger.h

class Logger{
public:
   static Logger* Instance();
   bool openLogFile(std::string logFile);
   void writeToLogFile();
   bool closeLogFile();
private:
   Logger(){};  // Private so that it can  not be called
   Logger(Logger const&){};             // copy constructor is private
   Logger& operator=(Logger const&){};  // assignment operator is private
   
   // Only declare and do not initialize here
   static Logger* m_pInstance; 
};


// In implementation, initialize the static member
// logger.cpp

Logger* Logger::m_pInstance = NULL; // initialization of static member 

Logger* Logger::Instance()
{
   if (!m_pInstance)   // Only allow one instance of class to be generated.
      m_pInstance = new Logger;
   return m_pInstance;
}

bool Logger::openLogFile(std::string _logFile)
{
   ...
   ...
   ...
}

...
...

```

