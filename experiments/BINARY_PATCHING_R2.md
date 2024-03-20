# Binary Patching

The main purpose of this experiment is to understand how can we patch the binary at runtime.
Based on [previous experiment](./SWT_BASICBLOCK_FUNCTION.md) we demonstrated that noop tracing function gives significant performance improvement. If we could switch between noop and tracing function at runtime we can gain that improvement.

## Offline binary patch

C code:
```c
#include <stdio.h>

void yk_trace_basicblock(size_t function_index, size_t block_index) {
    fprintf(stdout, "[yk_trace_basicblock] %ld, %ld\n", function_index, block_index);
}

int main(int argc, char *argv[]) {
  fprintf(stdout, "main start\n");
  yk_trace_basicblock(0,0);
  yk_trace_basicblock(1,0);
  yk_trace_basicblock(1,1);
  yk_trace_basicblock(2,0);
  fprintf(stdout, "main end\n");
  return 0;
}
```

## Compile with gcc

Compile:
```shell
$ gcc ./trace.c -o trace
```
Run:
```shell
$ ./trace
[yk_trace_basicblock] 0, 0
[yk_trace_basicblock] 1, 0
[yk_trace_basicblock] 1, 1
[yk_trace_basicblock] 2, 0
```
```shell
$ r2 -w ./trace

[0x00001060]> s sym.yk_trace_basicblock
[0x00001060]> aaa
[0x00001169]> pdf
            ;-- rip:
            ; CALL XREFS from main @ 0x11e3(x), 0x11f2(x), 0x1201(x), 0x1210(x)
┌ 58: sym.yk_trace_basicblock (long int arg1, long int arg2);
│           ; arg long int arg1 @ rdi
│           ; arg long int arg2 @ rsi
│           ; var long int var_8h @ rbp-0x8
│           ; var long int var_10h @ rbp-0x10
│           0x00001169      f30f1efa       endbr64
│           0x0000116d      55             push rbp
│           0x0000116e      4889e5         mov rbp, rsp
│           0x00001171      4883ec10       sub rsp, 0x10
│           0x00001175      48897df8       mov qword [var_8h], rdi     ; arg1
│           0x00001179      488975f0       mov qword [var_10h], rsi    ; arg2
│           0x0000117d      488b058c2e..   mov rax, qword [obj.stdout] ; obj.__TMC_END__
│                                                                      ; [0x4010:8]=0
│           0x00001184      488b4df0       mov rcx, qword [var_10h]
│           0x00001188      488b55f8       mov rdx, qword [var_8h]
│           0x0000118c      488d35750e..   lea rsi, str._yk_trace_basicblock___ld___ld_n ; 0x2008 ; "[yk_trace_basicblock] %ld, %ld\n" ; const char *format
│           0x00001193      4889c7         mov rdi, rax                ; FILE *stream
│           0x00001196      b800000000     mov eax, 0
│           0x0000119b      e8c0feffff     call sym.imp.fprintf        ; int fprintf(FILE *stream, const char *format,   ...)
│           0x000011a0      90             nop
│           0x000011a1      c9             leave
└           0x000011a2      c3             ret
...
[0x00001169]> px
- offset -  696A 6B6C 6D6E 6F70 7172 7374 7576 7778  9ABCDEF012345678
0x00001169  f30f 1efa 5548 89e5 4883 ec10 4889 7df8  ....UH..H...H.}.
0x00001179  4889 75f0 488b 058c 2e00 0048 8b4d f048  H.u.H......H.M.H
...
[0x00001169]> wa ret 0 # OR: wx C3
[0x00001169]> pdf
            ; CALL XREFS from main @ 0x11e3(x), 0x11f2(x), 0x1201(x), 0x1210(x)
┌ 58: sym.yk_trace_basicblock (long int arg1, long int arg2);
│           ; arg long int arg1 @ rdi
│           ; arg long int arg2 @ rsi
│           ; var long int var_8h @ rbp-0x8
│           ; var long int var_10h @ rbp-0x10
│           0x00001169      c20000         ret 0
..
│           0x0000116d      55             push rbp
│           0x0000116e      4889e5         mov rbp, rsp
│           0x00001171      4883ec10       sub rsp, 0x10
│           0x00001175      48897df8       mov qword [var_8h], rdi     ; arg1
│           0x00001179      488975f0       mov qword [var_10h], rsi    ; arg2
│           0x0000117d      488b058c2e..   mov rax, qword [obj.stdout] ; obj.__TMC_END__
│                                                                      ; [0x4010:8]=0
│           0x00001184      488b4df0       mov rcx, qword [var_10h]
│           0x00001188      488b55f8       mov rdx, qword [var_8h]
│           0x0000118c      488d35750e..   lea rsi, str._yk_trace_basicblock___ld___ld_n ; 0x2008 ; "[yk_trace_basicblock] %ld, %ld\n" ; const char *format
│           0x00001193      4889c7         mov rdi, rax                ; FILE *stream
│           0x00001196      b800000000     mov eax, 0
│           0x0000119b      e8c0feffff     call sym.imp.fprintf        ; int fprintf(FILE *stream, const char *format,   ...)
│           0x000011a0      90             nop
│           0x000011a1      c9             leave
└           0x000011a2      c3             ret
[0x00001169]> px
- offset -  696A 6B6C 6D6E 6F70 7172 7374 7576 7778  9ABCDEF012345678
0x00001169  c200 00fa 5548 89e5 4883 ec10 4889 7df8  ....UH..H...H.}.
...
```

## Compile with clang

```shell
$ clang ./trace.c -o ./trace-clang++
$ ./trace-clang++
main start
[yk_trace_basicblock] 0, 0
[yk_trace_basicblock] 1, 0
[yk_trace_basicblock] 1, 1
[yk_trace_basicblock] 2, 0
main end
```
```shell
$ r2 -w ./trace-clang++
[0x00001050]> aaa # analyze
[0x00001180]> s sym.yk_trace_basicblock # go to function
[0x00001140]> pdf
            ; CALL XREFS from main @ 0x11b5(x), 0x11c3(x), 0x11d0(x), 0x11de(x)
┌ 54: sym.yk_trace_basicblock (long int arg1, long int arg2);
│           ; arg long int arg1 @ rdi
│           ; arg long int arg2 @ rsi
│           ; var long int var_8h @ rbp-0x8
│           ; var long int var_10h @ rbp-0x10
│           0x00001140      55             push rbp
│           0x00001141      4889e5         mov rbp, rsp
│           0x00001144      4883ec10       sub rsp, 0x10
│           0x00001148      48897df8       mov qword [var_8h], rdi     ; arg1
│           0x0000114c      488975f0       mov qword [var_10h], rsi    ; arg2
│           0x00001150      488b05892e..   mov rax, qword [reloc.stdout] ; [0x3fe0:8]=0
│           0x00001157      488b38         mov rdi, qword [rax]        ; FILE *stream
│           0x0000115a      488b55f8       mov rdx, qword [var_8h]
│           0x0000115e      488b4df0       mov rcx, qword [var_10h]
│           0x00001162      488d359b0e..   lea rsi, str._yk_trace_basicblock___ld___ld_n ; 0x2004 ; "[yk_trace_basicblock] %ld, %ld\n" ; const char *format
│           0x00001169      b000           mov al, 0
│           0x0000116b      e8c0feffff     call sym.imp.fprintf        ; int fprintf(FILE *stream, const char *format,   ...)
│           0x00001170      4883c410       add rsp, 0x10
│           0x00001174      5d             pop rbp
└           0x00001175      c3             ret
[0x00001140]> wx C3 # patch first instruction as routine return
[0x00001140]> pdf
            ; CALL XREFS from main @ 0x11b5(x), 0x11c3(x), 0x11d0(x), 0x11de(x)
┌ 54: sym.yk_trace_basicblock (long int arg1, long int arg2);
│           ; arg long int arg1 @ rdi
│           ; arg long int arg2 @ rsi
│           ; var long int var_8h @ rbp-0x8
│           ; var long int var_10h @ rbp-0x10
│           0x00001140      c3             ret
│           0x00001141      4889e5         mov rbp, rsp
│           0x00001144      4883ec10       sub rsp, 0x10
│           0x00001148      48897df8       mov qword [var_8h], rdi     ; arg1
│           0x0000114c      488975f0       mov qword [var_10h], rsi    ; arg2
│           0x00001150      488b05892e..   mov rax, qword [reloc.stdout] ; [0x3fe0:8]=0
│           0x00001157      488b38         mov rdi, qword [rax]        ; FILE *stream
│           0x0000115a      488b55f8       mov rdx, qword [var_8h]
│           0x0000115e      488b4df0       mov rcx, qword [var_10h]
│           0x00001162      488d359b0e..   lea rsi, str._yk_trace_basicblock___ld___ld_n ; 0x2004 ; "[yk_trace_basicblock] %ld, %ld\n" ; const char *format
│           0x00001169      b000           mov al, 0
│           0x0000116b      e8c0feffff     call sym.imp.fprintf        ; int fprintf(FILE *stream, const char *format,   ...)
│           0x00001170      4883c410       add rsp, 0x10
│           0x00001174      5d             pop rbp
└           0x00001175      c3             ret
$ ./trace-clang++ # run it
main start
main end
```


## Patch revert

Here we want to revert the patch we did before by patching back the original instruction.

Bench of YKLua wihtout patch:
```shell
$ YKD_SERIALISE_COMPILATION=1 hyperfine -m 50 './src/lua ./tests/closure.lua'
Benchmark 1: ./src/lua ./tests/closure.lua
  Time (mean ± σ):     286.6 ms ±   4.8 ms    [User: 385.5 ms, System: 33.5 ms]
  Range (min … max):   270.7 ms … 295.9 ms    50 runs
```

```shell
$ ./trace-clang++
main start
main end
$ r2 -w ./trace-clang++
[0x00001050]> aaa
[0x00001140]> s sym.yk_trace_basicblock
[0x00001140]>
            ;-- rip:
            ; CALL XREFS from main @ 0x11b5(x), 0x11c3(x), 0x11d0(x), 0x11de(x)
┌ 1: sym.yk_trace_basicblock ();
│ rg: 0 (vars 0, args 0)
│ bp: 0 (vars 0, args 0)
│ sp: 0 (vars 0, args 0)
└           0x00001140      c3             ret
[0x00001140]> wx 55

```

## Patching ykcapi yk_trace_basicblock

```shell
$ YKB_TRACER=swt cargo build
```

```shell
$ nm ./target/debug/libykcapi.so  | grep yk_trace
00000000000e5b40 T yk_trace_basicblock

$ r2 -w ./target/debug/libykcapi.so
[0x000e4e80]> aaa
[0x000e4e80]> s sym.yk_trace_basicblock
[0x000e5b40]> pdf
            ;-- yk_trace_basicblock:
┌ 25: dbg.yk_trace_basicblock (int64_t arg1, int64_t arg2);
│           ; arg int64_t arg1 @ rdi
│           ; arg int64_t arg2 @ rsi
│           ; var usize function_index @ rsp+0x8
│           ; var usize block_index @ rsp+0x10
│           0x000e5b40      4883ec18       sub rsp, 0x18               ; lib.rs:103 ; void yk_trace_basicblock(usize function_index,usize block_index);
│           0x000e5b44      48897c2408     mov qword [function_index], rdi ; arg1
│           0x000e5b49      4889742410     mov qword [block_index], rsi ; arg2
│           0x000e5b4e      ff15cc752100   call qword [obj._ZN4ykrt16trace_basicblock17hd3aa19a08ccde3b2E_got] ; lib.rs:104 ; [0x2fd120:8]=0x1afd00 dbg.trace_basicblock
│           0x000e5b54      4883c418       add rsp, 0x18               ; lib.rs:105
└           0x000e5b58      c3             ret
[0x000e5b40]> wx C3 # patch first instruction as routine return
```

```shell
$ scp ./libykcapi.so  remote:/home/pd/yk-fork/target/debug/libykcapi.so
```

Benchmark:
```shell
$ YKD_SERIALISE_COMPILATION=1 hyperfine -m 50 './src/lua ./tests/closure.lua'
Benchmark 1: ./src/lua ./tests/closure.lua
  Time (mean ± σ):      55.7 ms ±   3.4 ms    [User: 62.8 ms, System: 19.7 ms]
  Range (min … max):    45.6 ms …  66.2 ms    50 runs
```

Reversing the patch:
```shell
[0x000e5b40]> wa sub rsp, 0x18
...
```

Benchmarks:
```shell
$ YKD_SERIALISE_COMPILATION=1 hyperfine -m 50 './src/lua ./tests/closure.lua'
Benchmark 1: ./src/lua ./tests/closure.lua
  Time (mean ± σ):     286.6 ms ±   4.2 ms    [User: 386.8 ms, System: 31.8 ms]
  Range (min … max):   272.7 ms … 293.1 ms    50 runs
```
