---
title: 'Elixir Notes (1) -- Basic Issues'
date: 2018-09-13
permalink: /posts/2018/09/blog-1-Elixir_Notes_1_Basic_Issues/
tags:
  - Elixir
  - Distributed Systems
---

Basic notes for learning Elixir.

### 1. Returning values 

**(1) Function**

Elixir has no keyword "return". It regards value of the last expression in a function as its returning value. 

```elixir
def addOne(arg) do
	arg + 1
end
```

**(2) Block**

In Elixir, a block (do .. end) also has returning value. It is the same case as a function, which means the returning value of a block is value of the last expression in this block.

So this code is correct, you will assign value of var "ans" correctly.

**Note:** This is a common way to assign or change values of existing variables according to different cases, though it seems strange. We will explain it later in this article.

```elixir
ans = 
	if list == [nil] do
		IO.puts XXXX
		ans # the last expression in the first case
	else
		IO.puts XXXX
		ans ++ list # the last expression in the second case
	end  

```

But this code is wrong because the last expression is not the value to be assigned to "ans". This will cause "ans" to become nil.

```elixir
# returning value of this block would be the value of "IO.puts XXXX" instead of what we want.
ans = 
	if list == [nil] do
		ans
		IO.puts XXXX 
	else
		ans ++ list
		IO.puts XXXX
	end  
```



### 2. Rebinding of variables

(1) Elixir will always rebind a variable when using assignment expression.

For example, 

```elixir
x = [1]
x = x ++ [2]
```

x would be [1, 2] finally. That is obvious. But the left-hand-side 'x' in the second line is not the one in the first line actually.  The left-hand-side 'x' in the second line is a rebound variable, that is, we did not change the original 'x' but rebound the name 'x' to another list [1, 2].

This will be a disaster when you want to change existing variables in a sub-block.

For example,

```elixir
x = [1]
if 2 > 1 do
	x = x ++ [2]
else
	x = x -- [1]
end  
```

After this block, x is still [1]. It is because in the block, x will be rebound. So the x inside is not the same one as the x outside. So after that block, x is unchanged.

In short, in Elixir, **variables inside a block will not overwrite variables outside with same names.** 

(2) Then what if we want to change an existing variable inside a block? Let's recall that blocks in Elixir have their returning values, which means we can let the existing variable receive the returning value like below:

```
x = [1]
x = 
	if 2 > 1 do
		x ++ [2]
	else
		x -- [1]
	end
```

Then x would be [1, 2] finally.



### 3. Module attributes

Module attributes (the variables starting with "@") are more like constants, which would be initialized during compilation. So they cannot be changed in runtime. That means you may not use them as state variables to store some intermediate results.



### 4. State memorization

For Elixir and most other functional programming languages, the common way to store states is to pass current states as arguments into next function call instead of using global state variables.

For example, suppose you want to compare the difference between current process list and previous list 5 seconds ago periodically, you can do like below:

```elixir
def compare(previous_list) do
	cur_list = Process.list()
	if cur_list != previous_list do
		IO.puts "Processes changed!"
	end
	:time.sleep(5000)
	compare(cur_list) # pass current state as argument to next function call
end
```



### 5. Loop

**Elixir has no "while" keyword**. So you cannot write loop like many other languages.

There are normally two ways to do iteration.

(1) Use "for" keyword.

Fortunately, Elixir has "for" keyword to iterate over a list, just like Python and C#/C/C++.

```elixir
list = Enum.to_list(1..10) 
# list contains all integers from 1 to 10, including 1 and 10.
for i <- list do # iterate over list
	IO.puts i
end
```

But do not abuse it. It is **not recommended**.

(2) **Recursion** (Recommended)

Functional languages normally prefer to use recursion to implement loop. Let's rewrite the example above using recursion:

```elixir
def loop(i) do
	if i < 10 do
		IO.puts i
		loop(i+1)
	end
end

loop(1)
```



### 6. Daemon

Elixir will not set the main process as daemon process by default. For example, 

```elixir
# this is our main code file "project.exs"
p = spawn(some_module, :loop_and_do_something, [])
```

If we run "project.exs", it would **NOT** **act as a daemon process**, that is, when project.exs run to its end, it finishes, and **kills all processes spawned inside it**. That is weird, but true. 

So we have to set our own daemon process in the end of project.exs to check the status of our program and to determine when to end everything.

Actually, our own "daemon process" is not necessary to be a real process which is created by spawn(). It can be a recursive function which loops until we decide to end our program, like below:

```elixir
# this is our main code file "project.exs"
p = spawn(some_module, :loop_and_do_something, [])
checkStatus(...) 
# checkStatus() will check status periodically until some conditions are satisfied.
```



### 7. Input arguments from command line

Only need one line to do this.

```elixir
[arg1, arg2, ..., argn] = System.argv()
```

There is one point you need to care about. All args read by System.argv() are strings. So you may have to convert them to the types you want.



### 8. Anonymous recursive function

We can use recursion in anonymous functions, even if they do not have names to be called.

```elixir
print = fn f, i -> # the first parameter stands for this function itself
	if i < 10 do
		IO.puts i
    	f.(f) # call self like this, do not forget the "."
	end
end

print.(print, 1) 
# call print with "." since it is defined as an anonymous function
# the first parameter is print itself
```



### 9. Identify a process by name

If you want to find a process, you can use its name or PID. 

(1) PID:

There are many ways to get a process's PID, like self(), or returning values of spawn(), ...

(2) Process name:

Before using name to find a process, you have to register its name like below:

```elixir
Process.register(pid, name)
```

This will create mapping between the specific process's PID and name such that you can use name to identify the process.

**Note:** It is not necessary to always register a process's name. To avoid name conflict, you'd better only use PID to identify a process, especially when you need to spawn multi processes from one module.
