## Direct and indirect calls

Direct and indirect calls refer to different types of function calls.

### Direct call

Direct calls are calls to functions whose target is known at compile-time. 

In meta-tracing, the tracing system can easily handle direct calls because it knows exactly where the control flow will go. This allows for efficient tracing and optimization strategies. 

### Indirect Calls

Indirect calls are calls to functions whose target is determined dynamically at runtime, typically through function pointers or dynamic dispatch mechanisms (such as virtual function calls in object-oriented languages). 

In meta-tracing, handling indirect calls is more challenging because the target of the call may not be known statically. 
The tracing system needs to employ techniques to determine the possible targets of the call dynamically and handle them appropriately during tracing.



### setjjmp + longjump¦

`setjmp` is a function in C that is used for implementing non-local jumps. 

It saves the current execution state, including the program counter, stack pointer, and other relevant registers, into a `jmp_buf` object, which can later be used to return to this state via a call to `longjmp`. 

This allows for a form of exception handling or control flow that can jump out of deeply nested function calls.

Example:

```c
#include <stdio.h>
#include <setjmp.h>

jmp_buf env;

void foo() {
    printf("foo\n");
    longjmp(env, 1);
}

int main() {
    if (setjmp(env) == 0) {
        printf("setjmp called\n");
        foo();
    } else {
        printf("longjmp called\n");
    }
    return 0;
}
```

### goto
TODO

### computed goto
TODO