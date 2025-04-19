+++
date = '2025-04-19T20:14:06+05:30'
draft = false
title = 'Iron Below the Rust'
+++
Having worked on Python for 6+ years, learning Rust took me back to C/C++ days in college. Compile the code before running it. With that it also brought back the nostalgic fear of pointers. Rust put all of the fears to rest. 

There are two types of memory: Stack and Heap. As the name suggests on stack memory things will be put one above other and popped in reverse order. Since you know where you are putting things and removing from, stack memory is extremely fast. 

But with that comes a catch. To push something on stack memory, you must know the size of thing being pushed. We won't be knowing size of all things at compile time, for example the name of user using our app. That would vary person to person. So, we can't allocate this on stack memory. Instead what we do is we allocate it a dynamic Heap memory at runtime and store `pointer` to that memory in our stack memory.

Simple. Now, let's take the discussion further. 

Memory is a limited resource. So, it must be managed in some way so that we don't run out of memory. Depending on how the memory is managed languages can be categorized in 3 classes. 
- C / C++ type languages: These languages give full freedom to user. User can allocate memory and is also responsible for releasing memory when it is not in use. Consider it like a self-service restaurant. You put your plate in basin once you are done eating. There are few problems with this. If you forget to clean memory, it would result in memory leak. If you try to reference a memory location that has been cleaned, it would result in dangling reference. If you try to free an already freed memory that would also be a bug. All in all, with great power comes great responsibility. 

- Java / Python type languages: So, there came other type of languages that have their garbage collection. You allocate memory and forget about deallocating it. Garbage collector will take care of it. This is a periodically running process that deallocates unused memory. Consider it like a restaurant where waiter cleans the plates after you leave. This takes away the headache of freeing memory but introduces garbage collection pauses which are not desirable for performance critical projects. It also needs extra memory overhead to run the collector. 

- Rust type language: Then comes rust which automatically clears a variable once it goes out of scope. This is done at compile time. When you compile code, compiler adds `drop` statement as per rule at appropriate places. So, you don't incur unpredictability of Garbage collector or the bugs in freeing memory like C/C++. Great. 

The concept seems interesting. But how does Rust achieve this ?
Rust has the concept of `Ownership`. A value can be `owned` by only single owner at a time. This is the foundation of Rust. The Iron below the Rust. Let's dive into it. 

When we allocate some value to a variable in rust, it creates an owner. 
```
{
    let s1 = String::from("Hello");
    let s2 = s1;
    println!("{s1}"); // Will result in compilation error
}
```
Here, `s1` owns the `Hello` String. It has a `pointer` to data stored on Heap memory. With second assignment `let s2 = s1` we are creating a second pointer to the same data. When scope of the variable ends at curly braces `}` there will be two owners of the data. Meaning that it would try to clean the same memory twice. This would result in error when we try to clean an already cleaned memory. Hence, Rust allows a single ownership of value. So, when we assign `let s2 = s1` s1 relinquishes its ownership (`moves` in Rust terms). Hence, when we try to print `s1` it throws compilation error as `s1` doesn't own any data. This is quite contrasting to other languages. 

Now suppose, we pass a variable to a function. Who owns the value ? Is it the original variable or is it the function ? With each allocation ownership changes. So, if we pass a variable to a function, it is the function that now owns the value. Consider the following code: 
```
fn main(){
    let s = String::from("Pavan");
    greet_me(s);
    println!("{s}");  // This will throw error
}

fn greet_me(s: String){
    println!("Hello {s}");
}
```
When we pass `s` to `greet_me` function, the function takes its ownership. And when the function execution is done its scope completes. So, variable `s` is dropped. As a result, when we try to access `s` in `main` function it gives us compilation error. A simple solution is to return `s` from `greet_me` function and reallocate it to `s`. But this would become cumbersome over time. Hence, Rust has concept of `references`. 

Instead of passing actual value to the function, we can pass its reference (which is just a pointer). But unlike C++ references, Rust reference guarantees that there won't ever be a dangling pointer (through its single ownership). 

Variables are `immutable` by default in Rust unless you make them mutable using `mut` keyword. Similarly, references `&` are also `immutable` by default unless you make them mutable with `&mut`. 

Now if two mutable references try to access the same data, it can lead to data race condition. Hence, Rust restricts multiple `mutable` references to the same variable though there can be multiple `immutable` references. 

That's all. If we get these concepts the foundation is set to understand Rust. 