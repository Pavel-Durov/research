# Efficient Interpretation using Quickening

Source: https://publications.sba-research.org/publications/dls10.pdf

## Overview

This paper presents a collection of non-jit techniques that can be used to improve the efficiency of interpreters. All of the examples are based on Python 3.1.

##  Optimisations

### Inlined comparison instructions

General comparison instruction (COMPARE_OP) depends on its arguments to determine which comparison operation to execute at runtime. 

This dynamic overhead is reduced by creating type-specific and inlined comparison functions. In the paper, Figure 1 illustrates the original COMPARE_OP and INCA_CMP_LONG, Long type-specific, inlined instruction.


### Unfolding instructions

Same as with Inlined comparison instructions, there are more Python instructions with behaviour defined by the arguments.

In the paper, the BUILD_TUPLE instruction is used as an example.
Their research analysis showed that the most frequent arguments for BUILD_TUPLE are 2 or 3. Therefore these instructions can be rewritten as argument-specific versions such as BUILD_TUPLE_THREE.


### Reduced instruction format

The instructions format includes additional memory. This memory is used to cache function addresses or data objects.  

They found out that some of the Python instructions didn't use the preserved space at all and the proposed new instruction format where reserved space is removed. This optimisation reduces the size of instructions by half, but it means that we lose the cache optimisation therefore we need an alternative implementation.

#### Inlining Data Object References in LOAD_CONST

For LOAD_CONST instructions that load objects to the lower memory area (first 32-bit), indirection can be replaced by direct assignment to the instruction argument. This optimisation removes the need to fetch the references via the instruction argument.

Similar optimisation is done for LOAD_GLOBAL global values are inlined. The only difference here is that we need to take into account the possibility of global value changes. This is addressed by using an indirect cache store and adaptation of LOAD_GLOBAL and STORE_GLOBAL instructions.


### Local variables caching

LOAD instructions are frequently used in stack-based interpreters.
LOAD_FAST instruction is used to load local variables. 
This optimisation caches variables from the Python stack in the interpreter stack. 

The main idea is to allocate the most frequently executed LOAD instructions to the n available additional local caching variables.

The estimation of which instructions will be executed most often is done by counting the number of occurrences.

In loops, we assume that LOAD and STORE instructions within the loop-body will be executed more often. The deeper the loops are nested, the more often we assume that these instructions will be executed.

To give occurrences inside loops higher scores, the nesting level is multiplied by 100 whenever we enter a loop and divided by 100 whenever the loop is terminated.


### Reducing Reference Counting 

With this optimisation, we remove redundant reference count increments, especially if they are decremented right after the increment.

Reference counts are eliminated from operation implementations and specific instruction sequences.


## Evaluation

The paper evaluation shows evaluation on two platforms: Intel i7 and IBM PowerPC 970.

One of the comparisons is the number of reference count operations - increment and decrement.

The other measure is the relative speedup between different interpreter optimisations: Threaded Code, ECOOP'10 (inline caching only), and DLS'10 (not sure what DLS stands for).

The evaluation shows that these optimisations were able to eliminate up to 2/3 of the increment and a 1/2 of the decrement instructions. 

Speedups are up to 2.18 when combined with the threaded code dispatch technique (Section 5.5).

## Notes

- Mention of threaded-code optimization technique. Something I encountered for the first time in the last Journal reading group on 2024/03/01 = -https://arxiv.org/abs/2106.12496

## Questions

- What's "manual unrolling"? "This manual unrolling of interpreter instructions enables the compiler to perform more optimizations on this block.". Section 3.2.

- Is there a difference between "unfolding" and "outlining" instructions/functions?

- What are the evaluation acronyms that stand for ECOOP'10, and DLS'10?
