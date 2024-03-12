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
