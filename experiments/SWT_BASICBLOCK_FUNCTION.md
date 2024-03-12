# YK/SWT Tracing function overhead

We know that Software Tracer has a significant overhead when compared to Hardware Tracer - Approx 30x times difference.

In this experiment we want to see what is the impact of the tracing function that is called for each LLVM basic block when compared to a NOOP function that we expect to be optimised by the hardware branch prediction.

Essentially the idea is to create two versions of the YK/SWT, one with the full tracing functionality and the other with a dummy patched version whe the tracing function just returns.

Note: we don't want to compare the tracing itself - so we need to set the YK threshold high enough in-order to not trigger the JIT.

Example:
```c
    yk_mt_hot_threshold_set(g->yk_mt, 9999999999);
```


## YK normal build

```shell
YKB_TRACER=swt cargo build
```

### YkLua build

```shell
export YK_BUILD_TYPE="debug"
export PATH="/home/pd/ykjit/2024-03-04/yk-fork/bin/:${PATH}"
export YKB_TRACER=swt
make clean && make -j $(nproc)
```

### YkLua run

```shell
YKD_SERIALISE_COMPILATION=1 hyperfine -m 50 './src/lua ./tests/closure.lua'
Benchmark 1: ./src/lua ./tests/closure.lua
  Time (mean ± σ):     268.5 ms ±   5.1 ms    [User: 372.5 ms, System: 35.5 ms]
  Range (min … max):   252.3 ms … 288.4 ms    50 runs
```



## YK noop build

```diff
$ git diff
diff --git a/ykcapi/src/lib.rs b/ykcapi/src/lib.rs
index 68eedfb..6b758f8 100644
--- a/ykcapi/src/lib.rs
+++ b/ykcapi/src/lib.rs
@@ -101,5 +101,5 @@ pub extern "C" fn yk_location_drop(loc: Location) {
 #[cfg(tracer_swt)]
 #[no_mangle]
 pub extern "C" fn yk_trace_basicblock(function_index: usize, block_index: usize) {
-    trace_basicblock(function_index, block_index);
+    // trace_basicblock(function_index, block_index);
 }
```

```shell
YKB_TRACER=swt cargo build
```

### YkLua build

```shell
export YK_BUILD_TYPE="debug"
export PATH="/home/pd/ykjit/2024-03-04/yk-fork/bin/:${PATH}"
export YKB_TRACER=swt
make clean && make -j $(nproc)
```

### YkLua run

```shell
YKD_SERIALISE_COMPILATION=1 hyperfine -m 50 './src/lua ./tests/closure.lua'
Benchmark 1: ./src/lua ./tests/closure.lua
  Time (mean ± σ):      57.9 ms ±   3.6 ms    [User: 63.3 ms, System: 21.3 ms]
  Range (min … max):    43.5 ms …  64.2 ms    50 runs
```
