# Trace-based Just-in-Time Type Specialization for Dynamic Languages

Source: http://www.cs.williams.edu/~freund/cs434-exemplar/gal-trace.pdf

## Summary

The paper introduces TraceMonkey, a tracer designed for dynamic languages, and discusses the theory and implementation of trace-based JIT-type techniques.

The key assumption is that loops in dynamic languages usually have consistent variable value types, which allows typed traces with type annotations and guards to be compiled and reused for hot loops.

Typed traces include type annotations alongside variables and guards to detect dynamic type changes during execution.

The paper outlines several optimisation techniques such as activation record area optimisation, type specialisation, object representation specialisation, function inlining, trace blacklisting, nesting trace algorithm, and trace tree optimizations.

In the evaluation, TraceMonkey achieves a speedup of x25 for integer-heavy benchmarks compared to the SpiderMonkey engine. It also outperforms SFX and V8 engines in 9 out of 26 benchmarks.

## Techniques:

- Activation record area: Boxing and Unboxing values can be expensive. In order to overcome this all intermediate values are recorded in a small activation record area, including local and global variables. The tracer can read and write these variables quickly and independently of the boxing mechanism used by the interpreter. When the trace exits, the values are boxed and copied back to the interpreter structures.

- Type specialization - When the trace is recorded, the recorder emits guard instructions that conditionally exit if at runtime the operation yields a value of a different type from that seen during recording.

- Objects representation specialization -  Instead of traversing the JavaScript prototype chain every time object property is accessed, the tracer uses a shared hashtable with object values that is more performant. Values are of course guarded here.

- Numbers representation specialization: Avoiding redundant type conversions such as Number to Integer and Double.

- Function inlining

- Trace Blacklisting - avoiding spending time on trace compilation for traces that fails to execute.

- Nesting Traces - Avoiding duplicate compilation of outer loop paths.

- Trace Tree Optimization

- Trace stiching

