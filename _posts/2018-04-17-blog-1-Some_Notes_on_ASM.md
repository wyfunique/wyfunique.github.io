---
title: 'Some Notes on ASM'
date: 2018-04-17
permalink: /posts/2018/04/blog-1/
tags:
  - Compiler
  - Java
---

ASM is an important tool for manipulating java bytecode. Here is a summary of typical problems when I was implementing a compiler using Java and ASM.

### 1. InvocationTargetException

This is usually because of error in description of function. For example, if a function defined like below:

​	void func(long arg) {...}

and your description in visitMethodInsn() is "(I)V", an InvocationTargetException will be raised, since "I" in bytecode stands for integer while "J" is the correct symbol of long integer. So here you should use "(J)V" as description.



### 2. Bad type on operand stack

Literally, this error means that operand's type is not what JVM expected. It will provide details about it like below:

Exception Details:
  Location:
    prog.main([Ljava/lang/String;)V @106: swap
  Reason:
    Type double_2nd (current frame, stack[2]) is not assignable to category1 type
  Current Frame:
    bci: @106
    flags: { }
    locals: { '[Ljava/lang/String;' }
    stack: { integer, double, double_2nd }

The details tell you the type that JVM expected and the type you provided here. In this example, JVM wanted a "category1" type while your program provided a  "double_2nd" type. In addition, the details also tell you status of stack right before the error, which is very useful for debugging.



### 3. About type double

Int, char,  boolean and float are all one or less than one word long, while double and long int are both 2 words long. One unit of stack in JVM is one word long, which means double and long int type will take 2 units in stack. As shown above in section 2, the stack contains 'double' and 'double_2nd' which are the first and second word of double type. 

The error above is caused by using "SWAP" instruction on double type. Suppose you want to swap the two items on top of stack, if there are float and integer items, that is , when stack looks like this:

​	stack: { integer, float }

Then that is OK to swap them directly.

But if there are double and integer items, the "two" items on top of stack are actually **three** items:

​	stack: { integer, double, double_2nd }

So swap instruction will throw an error since JVM thought you wanted to swap the 2 words of a double type item. 



### 4. visitLocalVariable

Many novice are confused about usage of visitLocalVariable() in ASM. This method will declare scope of a local variable, that is, the scope starting from some label and ending at some label, where the two labels are passed as parameters into this method. 

Before you use it, please remember the **main requirement**: 

​	**The labels referenced in the call of visitLocalVariable must be visited (with method visitLabel) before the call.**

If you forget this requirement and call visitLocalVariable with a label that is not visited yet, you will see an error saying "Expecting a stackmap frame in branch XXX" when your program has nested blocks.

Because of this requirement, the safest way is calling visitLocalVariable at the end of your program for all local variables in it, like:

Program begin:

​	localVar_1 .....

​	localVar_2 .....

​	localVar_3 .....

​	.....

​	methodVisitor.visitLocalVariable(... localVar_1 ...);

​	methodVisitor.visitLocalVariable(... localVar_2 ...);	

​	methodVisitor.visitLocalVariable(... localVar_3 ...);

Program end

Actually, that is the strategy Java uses to visit local variables during generating bytecode.



### 5. Some little tips

5.1 Remember to put semi-colon ";" in the end of each description.

5.2 When you see exception like:

​	java.lang.ArrayIndexOutOfBoundsException: 0
	at prog.main(Unknown Source)

This is usually due to missing of command line arguments. 

5.3 If you want to pass null to any method, you need to do like this "**methodVisitor.visitInsn(ACONST_NULL);**".

​	a. You should **NOT** use methodVisitor.visitLdcInsn(ACONST_NULL) because this is the instruction for loading a constant. So it will treat ACONST_NULL as an integer, which is the internal representation of  ACONST_NULL. As a result, this instruction will load a number on top of stack instead of null.

​	b. You should **NOT** use mv.visitInsn(NULL) because this will also load a number on stack instead of null.  
