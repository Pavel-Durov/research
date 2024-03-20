# Runtime patching

## C sample with dynamic patching

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

## Rust sample

```rust
use libc::{c_void, size_t, sysconf, PROT_EXEC, PROT_READ, PROT_WRITE};
use libc::{memcpy, mprotect, _SC_PAGESIZE};
use std::mem;

#[no_mangle]
pub extern "C" fn yk_trace_basicblock() {
    println!("@@ Hello from original function")
}

fn patch_function() {
    unsafe {
        let page_size: size_t = sysconf(_SC_PAGESIZE) as size_t;
        let aligned_func_address = ((yk_trace_basicblock as usize) & !(page_size - 1)) as *mut c_void;
        let aligned_page_size = ((yk_trace_basicblock as usize) + mem::size_of::<extern "C" fn()>()
            - (yk_trace_basicblock as usize)) as size_t;
        // Change memory protection
        let result = mprotect(
            aligned_func_address,
            aligned_page_size,
            PROT_READ | PROT_WRITE | PROT_EXEC,
        );
        if result != 0 {
            panic!("Error changing memory protection");
        }
        // Patch the function with return instructions
        const PATCHED_CODE: [u8; 1] = [0xC3];
        memcpy(aligned_func_address, PATCHED_CODE.as_ptr() as *const c_void, PATCHED_CODE.len());
    }
}

fn main() {
    println!("original call");
    yk_trace_basicblock();
    patch_function();
    println!("patch call");
    yk_trace_basicblock();
}
```

```shell
$ cargo run
original call
@@ Hello from original function
patch call
@@ Hello from original function
```

## Notes
Looks like in Rust example I am miscalculating the function address. Might be to do with inlining?
