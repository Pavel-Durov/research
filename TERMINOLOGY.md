## Terminology

`Inlining` - Replaces a function call site with the body of the function as part of optimisation?

`Outlining` - Reverse of inlining? Replaces  body of a function with a function call as part of de-optimisation?

`Entry block` - The first block of a function in LLVM IR

`Mappable Block` - LLVM IR Basicblock that we can see its instructions

`Unmappable Block` - LLVM IR Basicblock that we can't see its instructions

`Meta-tracing` - recording and compilation of the code is automatic.

`Tracing` - manual code that records a trace and compiles it.
