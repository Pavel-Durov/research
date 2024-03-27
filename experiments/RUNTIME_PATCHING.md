# Runtime patching

## Dynamic patching wth C

```c
#include <stdio.h>
#include <string.h>
#include <sys/mman.h>
#include <unistd.h>
#include <stdint.h>

void yk_trace_basicblock() { printf("@@ Hello from original function\n"); }

unsigned char original_instructions[] = {};
#if defined(__x86_64__)
unsigned char patch_instructions[] = {0xC3};
#else
#error "Unsupported architecture"
#endif

void save_original_instructions(unsigned char instructions[], int num_of_instructions)
{
  // Define a function pointer to the original function
  void (*func_ptr)() = yk_trace_basicblock;

  // Calculate the size of the function's instructions
  uintptr_t func_address = (uintptr_t)func_ptr;
  uintptr_t func_end_address = func_address + 100; // Adjust this value based on your function's size
  size_t size = func_end_address - func_address;

  // Copy the function's instructions into the original_code array
  memcpy(instructions, (unsigned char *)func_ptr, num_of_instructions);
}

void patch_function(unsigned char code[], size_t size)
{
  size_t page_size = sysconf(_SC_PAGESIZE);
  void *func_address = (void *)((size_t)yk_trace_basicblock & ~(page_size - 1));
  size_t page_size_aligned =
      ((size_t)yk_trace_basicblock + sizeof(yk_trace_basicblock)) -
      (size_t)func_address;
  mprotect(func_address, page_size_aligned, PROT_READ | PROT_WRITE | PROT_EXEC);
  memcpy(yk_trace_basicblock, code, size);
  mprotect(func_address, page_size_aligned, PROT_READ | PROT_EXEC);
}

int main()
{
  printf("original call\n");
  yk_trace_basicblock();

  if (sizeof(original_instructions) == 0)
  {
    save_original_instructions(original_instructions, 1);
  }

  patch_function(patch_instructions, 1);
  printf("patch call\n");
  yk_trace_basicblock();

  patch_function(original_instructions, 1);
  printf("restore call\n");
  yk_trace_basicblock();
  return 0;
}
```

```shell
$ clang patch.c -o ./patch && ./patch
original call
@@ Hello from original function
patch call
restore call
@@ Hello from original function
```

Seems to work.

## Dynamic patching with Rust unsafe

```rust
use libc::{mprotect, size_t, sysconf, PROT_EXEC, PROT_READ, PROT_WRITE};
use std::mem;
use std::{ffi::c_void, sync::Once};

static ORIGINAL_INSTRUCTIONS_INIT: Once = Once::new();
static mut ORIGINAL_INSTRUCTIONS: [u8; 100] = [0; 100]; // Assuming that we have
static mut PATCH_INSTRUCTIONS: [u8; 1] = [0xC3]; // Patch instructions

unsafe fn yk_trace_basicblock() {
    println!("Hello 1");
    println!("Hello 2");
    println!("Hello 3");
    println!("Hello 4");
}

unsafe fn save_original_instructions(
    function_ptr: usize,
    instructions: *mut u8,
    num_of_instructions: usize,
) {
    let func_ptr: *const () = function_ptr as *const ();
    std::ptr::copy_nonoverlapping(func_ptr as *const u8, instructions, num_of_instructions);
}

unsafe fn patch_function(function_ptr: usize, code: *const u8, size: size_t) {
    let page_size = sysconf(libc::_SC_PAGESIZE) as usize;

    let func_address = ((function_ptr as usize) & !(page_size - 1)) as *mut c_void;
    let page_size_aligned = (((function_ptr as usize) + mem::size_of_val(&function_ptr))
        - (func_address as usize)) as usize;

    // Set function memory region to be writable
    let result = mprotect(
        func_address,
        page_size_aligned,
        PROT_READ | PROT_WRITE | PROT_EXEC,
    );
    if result != 0 {
        panic!("Failed to change memory protection to be writable");
    }

    // Set function memory region back to be non-writable
    std::ptr::copy_nonoverlapping(code, function_ptr as *mut u8, size);

    let result = mprotect(func_address, page_size_aligned, PROT_READ | PROT_EXEC);
    if result != 0 {
        panic!("Failed to change memory protection to not writable");
    }
}

unsafe fn patch_trace_function() {
    ORIGINAL_INSTRUCTIONS_INIT.call_once(|| {
        save_original_instructions(
            yk_trace_basicblock as usize,
            ORIGINAL_INSTRUCTIONS.as_mut_ptr(),
            1,
        );
    });

    patch_function(yk_trace_basicblock as usize, PATCH_INSTRUCTIONS.as_ptr(), 1);
}

unsafe fn restore_trace_function() {
    ORIGINAL_INSTRUCTIONS_INIT.call_once(|| {
        save_original_instructions(
            yk_trace_basicblock as usize,
            ORIGINAL_INSTRUCTIONS.as_mut_ptr(),
            1,
        );
    });
    patch_function(
        yk_trace_basicblock as usize,
        ORIGINAL_INSTRUCTIONS.as_ptr(),
        1,
    );
}

fn main() {
    unsafe {
        println!("original call");
        yk_trace_basicblock();
        patch_trace_function();
        println!("patched call - start");
        yk_trace_basicblock();
        println!("patched call - end");
        println!("restored call - start");
        restore_trace_function();
        yk_trace_basicblock();
        println!("restored call - end");
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    fn test_function() -> i32 {
        return 42;
    }

    #[test]
    fn test_runtime_patch_return_instruction() {
        unsafe {
            assert_eq!(test_function(), 42);
            patch_function(test_function as usize, PATCH_INSTRUCTIONS.as_ptr(), 1);
            assert_eq!(test_function(), 0);
        }
    }
}
```

```shell
$ cargo run
original call
Hello 1
Hello 2
Hello 3
Hello 4
patched call - start
patched call - end
restored call - start
Hello 2
Hello 1
Hello 3
Hello 4
restored call - end
```
