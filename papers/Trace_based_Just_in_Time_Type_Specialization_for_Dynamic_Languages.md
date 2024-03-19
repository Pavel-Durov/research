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


## Questions

### What happened to TraceMonkey (i.e. why does Firefox now use SpiderMonkey)?

TraceMonkey became part of SpiderMonkey in Firefox 3.5 and was later removed in Firefox version 11 in favour of JaegerMonkey.

JaegerMonkey is a method JIT compiler, first released in Firefox 4.

The downside of the TracingMonkey tracing-JIT approach is that it had to switch back and forth between the compiled machine code and the interpreter in some conditions (called "knocked off trace."). This transition happened more often than initially anticipated and had a significant performance impact.

JaegerMonkey engine included:

- JavaScript Method-JIT. This was expected to have a fast baseline JS performance compared to other engines which were also method-JIT based and to be consistent, as there is no need to transition between the machine code and the interpreter.

- TaceMonkey tracing engine for inner loop compilation.

Eventually, JaegerMonkey was replaced later on by IonMonkey.

Sources:
- https://hacks.mozilla.org/2010/03/improving-javascript-performance-with-jagermonkey/

- https://wiki.mozilla.org/JaegerMonkey

- https://wiki.mozilla.org/IonMonkey

- https://en.wikipedia.org/wiki/SpiderMonkey
