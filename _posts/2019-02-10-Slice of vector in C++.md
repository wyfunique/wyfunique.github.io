---
layout: post
title: How to get sub-vector (slice) of a given vector
category: 
- C++
---



## How to get sub-vector (slice) of a given vector

1. Create a new vector using starting iterator and ending iterator of the slice:

   ```c++
   vector<T>::const_iterator first = myVec.begin() + 100000;
   vector<T>::const_iterator last = myVec.begin() + 101000;
   vector<T> newVec(first, last);
   ```

   

2. Create a new vector using starting address and ending address of the slice: 

   ```c++
   std::vector<int> data();
   std::vector<int> sub(&data[100000],&data[101000]);
   ```

   