---
title: "How Poko works"
date: 2021-12-01T15:30:00-06:00
draft: false
---

The Kotlin compiler's job is to convert source code into byte code. Compiler plugins like [Poko](https://github.com/drewhamilton/Poko)
are able to intercept a representation of the source code during compilation and affect the
produced byte code. To this end, Poko's purpose is fairly straightforward: for any source class with
the `@Poko` annotation, produce byte code that uses that class's primary constructor properties in
three overridden functions.

To complicate things, Kotlin added a new compiler backend in 1.4, and made that new backend the
default in 1.5. So depending on the mode in which the compiler runs, the byte code output is
completely different: either classic JVM byte code or IR byte code. Poko supports both, effectively
meaning this plugin's core functions must be written twice.

When the Kotlin compiler starts executing, it loads `ComponentRegistrar` services, which is how
plugins can register themselves. [`PokoComponentRegistrar`](https://github.com/drewhamilton/Poko/blob/main/poko-compiler-plugin/src/main/kotlin/dev/drewhamilton/poko/PokoComponentRegistrar.kt)
registers both the non-IR [`PokoCodegenExtension`](https://github.com/drewhamilton/Poko/blob/main/poko-compiler-plugin/src/main/kotlin/dev/drewhamilton/poko/codegen/PokoCodegenExtension.kt)
and the IR [`PokoIrGenerationExtension`](https://github.com/drewhamilton/Poko/blob/main/poko-compiler-plugin/src/main/kotlin/dev/drewhamilton/poko/ir/PokoIrGenerationExtension.kt)
here. Depending on the consuming project's compiler settings, only one of these will actually be
used by the compiler.

### IR function generation

`PokoIrGenerationExtension` executes for every Poko consumer who compiles with `useIR = true`, which
is possible from Kotlin 1.4 and is the default in Kotlin 1.5. Poko first added support for this in
0.5.0, which supports Kotlin 1.4.20.

For every function that gets compiled, the extension checks if its owner class has the `@Poko`
annotation and meets the requirements for Poko classes (has a primary constructor with at least one
member property, etc.). If so, and if the function is one of the standard `toString`, `equals`, and
`hashCode` functions, the plugin determines the class's primary constructor properties and uses
those properties to override that function.

This approach fundamentally depends on every class already having these three functions by
default—it only intercepts the byte code generation after the compiler is already creating each
function. This is possible because all JVM classes include each of these three functions, implicitly
inherited from the `Object` base class. Code inside [`PokoMembersTransformer`](https://github.com/drewhamilton/Poko/blob/main/poko-compiler-plugin/src/main/kotlin/dev/drewhamilton/poko/ir/PokoMembersTransformer.kt),
like `generateToStringMethodBody`, is used to generate each function body by calling Kotlin compiler
APIs that represent IR byte code constructs.

### Non-IR function generation

`PokoCodegenExtension` executes for every Poko consumer who compiles with `useIR = false`, which is
the default for Kotlin 1.4 and can still be set in Kotlin 1.5 and 1.6.

For every class that gets compiled, the extension checks if that class has the `@Poko` annotation
and meets the requirements for Poko classes. If so, it determines the class's primary constructor
properties and uses those properties to override the `toString`, `equals`, and `hashCode` functions
in that class.

Since every Java class has a default implementation for each of these functions, Poko finds the
existing function declaration and mutates it with the new byte code. Internal `FunctionGenerator`
classes, such as [`ToStringGenerator`](https://github.com/drewhamilton/Poko/blob/main/poko-compiler-plugin/src/main/kotlin/dev/drewhamilton/poko/codegen/ToStringGenerator.kt),
are used to generate each function body by calling Kotlin compiler APIs that represent Java byte code
constructs.

---

If you want to learn more about how Poko works, your best bet is to browse the [source code](https://github.com/drewhamilton/Poko/tree/main/poko-compiler-plugin/src/main/kotlin/dev/drewhamilton/poko)
and the [tests](https://github.com/drewhamilton/Poko/blob/main/poko-compiler-plugin/src/test/kotlin/dev/drewhamilton/poko/PokoCompilerPluginTest.kt).
If you have questions, comments, or ideas, feel free to bring them to the [discussion board](https://github.com/drewhamilton/Poko/discussions)!
