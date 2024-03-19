## Terminology

`Inlining` - Replaces a function call site with the body of the function as part of optimisation?

`Outlining` - Reverse of inlining? Replaces  body of a function with a function call as part of de-optimisation?

`Entry block` - The first block of a function in LLVM IR

`Mappable Block` - LLVM IR Basicblock that we can see its instructions

`Unmappable Block` - LLVM IR Basicblock that we can't see its instructions

`Trace` - A trace is simply a program path, which may cross function call boundaries.

`Typed trace` - A trace annotated with a type for every variable (including temporaries) on the trace.

`Meta-tracing` - recording and compilation of the code is automatic.

`Tracing` - manual code that records a trace and compiles it.

`Phi nodes` - A phi node is an instruction used to select a value depending on the predecessor of the current block.

`Side exit` - Refer to situations where the execution of the program being traced diverges from the expected trace due to conditions that cannot be efficiently predicted or handled by the tracing system i.e when guard fails.

`Constant folding` - Optimisation technique. Instead of generating code to perform the computation at runtime, the JIT compiler calculates the result of the constant expression during compilation.

`Static Single Assignment (SSA)` - Iintermediate representation used in compiler design where each variable is assigned only once, it simplifies the data flow analysis and enabling efficient optimizations.

`Preemption`  - A strategy where a higher-priority task or process can interrupt a lower-priority task or process currently executing on a system resource (such as a CPU) before it completes its execution.

`Method-based JIT` - Compiler translate one method at a time to machine code.
