# Compiler overview


## A brief tour of how alb compiles programs:

Compilation process begins in [Driver.hs](https://github.com/habit-lang/alb/blob/master/src/Driver.hs), the entrypoint
of alb. When alb is invoked, it will:

- Build options from passed in command line arguments and/or .alb dotfile.
- Build a list of filepaths with `findFiles` from each path passed in, searching for valid .hb/.lhb files if path to
directories were given.
- After a list of filepaths have been constructed, load the files and attempt to lex and parse them by invoking
`readProgram` (TODO: break down and explain steps of lexing/parsing), and perform "Import Chasing";
For each file, it will check the list of "requires" and building a list of dependencies to other files. 
- Then after parsing is done and no errors are encountered, build the compiler pipeline and execute it.
This is where the magic happens.

alb's compiler pipeline is defined in the
[`buildPipeline` function](https://github.com/habit-lang/alb/blob/master/src/Driver.hs#L293),
which consists of the following "phases" (which themselves are made up of smaller "passes"):

- [Desugaring](#desugaring)
- [Kind Inference](#kind-inference-and-type-inference)
- [Type Inference](#kind-inference-and-type-inference)
- [Specialization]()
- [Normalization]()
- [LC Generation]()

Keen readers will also notice two additional stages in `buildPipeline`, LLVMed and Compiled. At the end of the frontend 
phases which ends with LC Generation, alb's job is done. The generated LC code is merely passed on to
[milc](https://github.com/habit-lang/mil-tools/) to do the hard work of emitting LLVM IR and machine code.

In this chapter we will look into how each of these frontend phases transform the code from a bird's eye view,
and in more detail in later chapters.

### Desugaring:

First, the compiler will attempt to "desugar" the program; decomposing higher-lever features into core language 
constructs. As they will no longer have to deal with syntactic sugar, this step helps simplify the code for 
subsequent phases (and passes). In alb, we have the following passes:

- [fixityProgram](https://github.com/habit-lang/alb/blob/master/src/Analyzer/Fixity.hs): Resolve infix operators, 
i.e. rewriting 1 + 2 into applicative form (+) 1 2.
- [freshenProgram](https://github.com/habit-lang/alb/blob/master/src/Analyzer/Freshening.hs): "Freshen" by assigning
unique name to each variable bindings, i.e. when there are variables that shadows another, give it a new unique name
so as it does not conflict.
- [eliminateTuplesProgram](): TODO
- [patternTuplesProgram](): TODO 
- [generateTuples](): TODO
- [desugarProgram](): TODO
- [rewriteFunctionalNotation](): TODO

By the end of the desugaring phase, the program will have been translated into a simpler, compiler-friendly subset
of Habit, dubbed `IMPEG`. We use `IMPEG` to do [Kind Inference and Type Inference](#kind-inference-and-type-inference).

> **Intermediate representations**
> 
> As with most compilers, alb works with several intermediate representation (IR) internally during the compilation
> process. Language features that might complicate compilation are stripped away, what's left is a representation
> that is less error-prone to work with and easier to apply crucial transformations and optimizations on.
> 
> During compilation, a Habit program have gone through transformations into a couple different IRs,
> each optimized for different phases: `IMPEG`, `XMPEG`, `LC`, `MIL` and `LLVM IR`.
> (TODO: why is blockquote styling broken?)
>
> (TODO: why is blockquote styling broken?)


### Kind Inference and Type Inference:

TODO
