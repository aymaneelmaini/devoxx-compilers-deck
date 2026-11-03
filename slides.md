---
theme: apple-basic
title: Compilers for Beginners
info: |
  Devoxx lightning talk (20 min): a beginner's introduction to how compilers work.
transition: fade
drawings:
  persist: false
duration: 20min
colorSchema: both
---


---
layout: intro
---

# What's the hardest part of implementing a programming language?


<!--
~1 min. Let the question land and pause so people can guess before you reveal the answer.

"Hello, I'm Aymane, a software engineer at iMedia24. I'm interested in systems and compilers engineering, and I love to dig into this stuff in my free time. Today's topic is a beginner introduction to how compilers work. We're going to open the black box and see what really happens."
-->

---

# Hi, I'm Aymane

<div class="mt-8 text-2xl max-w-2xl">

Software engineer at iMedia24

Building software for clients across all of Europe

</div>

<div class="mt-8 max-w-2xl">
<img src="/imedia24-logo.svg" alt="iMedia24" class="h-40 w-auto" />
</div>

<v-click>

<div class="mt-6 text-xl opacity-70">

Interested in compilers and systems engineering

</div>

</v-click>

<img src="/aymane.jpg" alt="Aymane" class="absolute right-12 top-64 w-56 h-56 rounded-full object-cover" />

<!--
~1 min. Keep the intro short. Name the company once.

Full script: "Hello, I'm Aymane, a software engineer at iMedia24. I'm interested in systems and compilers engineering, and I love to dig into this stuff in my free time."
-->

---

# How a compiler works

<img src="/crafting-interpreters-pipeline.png" alt="Compiler pipeline diagram from Crafting Interpreters" class="mx-auto mt-6 max-h-[64vh] w-auto" />

<div class="text-sm opacity-50 mt-4">Diagram: <i>Crafting Interpreters</i> by Robert Nystrom, craftinginterpreters.com</div>

<!--
~1 min. Script: "Every compiler follows this journey. Let's walk through it."

Point at the three phases: front end (scanning, parsing, analysis), middle end (IR, optimization), back end (code generation, runtime). Then zoom into each stage.
-->

---

# Scanning

<div class="grid grid-cols-[auto_1fr] gap-x-8 gap-y-3 mt-8 text-lg">
<div class="opacity-60">Input</div><div>source code</div>
<div class="opacity-60">Output</div><div>tokens</div>
<div class="opacity-60">What it does</div><div>breaks source into meaningful chunks</div>
</div>

<div class="text-2xl font-mono mt-6">total = price * qty;</div>

<v-click>

<div class="flex flex-wrap gap-3 mt-10 font-mono text-lg">
  <span class="px-3 py-2 border rounded-lg">IDENTIFIER</span>
  <span class="px-3 py-2 border rounded-lg">EQUAL</span>
  <span class="px-3 py-2 border rounded-lg">IDENTIFIER</span>
  <span class="px-3 py-2 border rounded-lg">STAR</span>
  <span class="px-3 py-2 border rounded-lg">IDENTIFIER</span>
  <span class="px-3 py-2 border rounded-lg">SEMICOLON</span>
</div>

</v-click>

<v-click>

<div class="mt-10 text-lg">Caught here: <span class="font-mono">"price * qty;</span> → unclosed string literal</div>

</v-click>

<!--
~2 min. Script: "Scanning breaks source code into tokens: identifiers, operators, punctuation. Whitespace and comments are thrown away."

Running example: `total = price * qty;`. If the closing quote is missing, the scanner can't finish the string token, so it reports "unclosed string literal" right here. The parser never sees this line.
-->

---

# Up next: Parsing

<div class="relative mx-auto mt-4 w-fit max-w-full">
<img src="/crafting-interpreters-pipeline.png" alt="Compiler pipeline mountain" class="block max-h-[62vh] w-auto" />

<div class="absolute" style="left:14.80%;top:32.41%;width:16.45%;height:15.61%"><div class="hl h-full w-full rounded-full border-4 border-red-500"></div></div>
</div>


<style>
.hl { animation: pulse 1.4s ease-in-out infinite; }
@keyframes pulse {
  0%, 100% { transform: scale(1); opacity: 1; }
  50% { transform: scale(1.08); opacity: 0.5; }
}
</style>

<!--
~0.5 min. Script: "Next, parsing. It takes the tokens and builds a tree that shows their structure."
-->

---

# Parsing

<div class="grid grid-cols-[auto_1fr] gap-x-8 gap-y-3 mt-8 text-lg">
<div class="opacity-60">Input</div><div>tokens</div>
<div class="opacity-60">Output</div><div>AST</div>
<div class="opacity-60">What it does</div><div>gives the tokens structure; precedence is the tree's shape</div>
</div>



<div class="mt-8 text-2xl font-mono">3 + 5 * 8</div>

<div class="mt-6">

```text
    +
   / \
  3   *
     / \
    5   8
```

</div>

<v-click>

<div class="mt-6 text-xl">Precedence is the shape of the tree.</div>

</v-click>

<!--
~2 min. Script: "Parsing gives the tokens structure. Precedence is baked into the tree's shape: multiplication ends up deeper, so it's evaluated first."

Recursive descent is the most common hand-written approach: one function per precedence level. Show the code only if asked; the live demo covers the same idea.
-->

---

# Parsing errors

<div class="grid grid-cols-[auto_1fr] gap-x-8 gap-y-3 mt-8 text-lg">
<div class="opacity-60">Input</div><div>tokens</div>
<div class="opacity-60">Output</div><div>AST, or an error</div>
<div class="opacity-60">What it does</div><div>checks the sequence against the grammar</div>
</div>

<div class="mt-8 text-2xl font-mono">total = price * ;</div>

<v-click>

<div class="mt-6 text-xl font-mono">illegal start of expression</div>

</v-click>

<!--
~1.5 min. Script: "Same running example, one token missing. The right operand is gone. Every token is valid on its own, but the sequence breaks the grammar, so the parser reports 'illegal start of expression'."
-->

---

# Parsing statements

<div class="grid grid-cols-[auto_1fr] gap-x-8 gap-y-3 mt-8 text-lg">
<div class="opacity-60">Input</div><div>tokens</div>
<div class="opacity-60">Output</div><div>AST with nested statements</div>
<div class="opacity-60">What it does</div><div>builds statements, nested inside each other</div>
</div>

<div class="text-lg opacity-70">Not just expressions</div>

<div class="mt-8 text-2xl font-mono">if (qty > 0) total = price * qty;</div>

<div class="mt-6">

```text
If
├─ condition: Binary (>)
│   ├─ qty
│   └─ 0
└─ then: Assign (total)
    └─ Binary (*)
        ├─ price
        └─ qty
```

</div>

<v-click>

<div class="mt-6 text-xl font-mono">')' expected</div>

</v-click>

<!--
~1.5 min. Script: "The parser builds statements too, and nests them. An `if` is a rule: the keyword `if`, an opening parenthesis, a condition, a closing parenthesis, then a statement."

If the closing parenthesis is missing, the parser stops at that point and reports "')' expected". Every developer has seen this message.
-->

---

# Up next: Static analysis

<div class="relative mx-auto mt-4 w-fit max-w-full">
<img src="/crafting-interpreters-pipeline.png" alt="Compiler pipeline mountain" class="block max-h-[62vh] w-auto" />

<div class="absolute" style="left:27.96%;top:3.60%;width:16.45%;height:14.89%"><div class="hl h-full w-full rounded-full border-4 border-red-500"></div></div>
</div>

<!--
~0.5 min. Script: "Next, static analysis. The tree is checked for meaning before the program runs."
-->

---

# Static analysis

<div class="grid grid-cols-[auto_1fr] gap-x-8 gap-y-3 mt-8 text-lg">
<div class="opacity-60">Input</div><div>AST</div>
<div class="opacity-60">Output</div><div>checked AST</div>
<div class="opacity-60">What it does</div><div>resolves names, binds variables to scopes, checks types</div>
</div>

<div class="mt-8 text-2xl font-mono">total = pricee * qty;</div>

<v-click>

<div class="mt-6 text-xl font-mono">cannot find symbol: pricee</div>

</v-click>

<!--
~1.5 min. Script: "Static analysis checks things that need context, before anything runs. It resolves each name to a declaration and binds variables to scopes."

The syntax is valid, so the parser accepts it. Only the analysis stage knows `pricee` was never declared.

Contrast: Python reports the same typo as a NameError, but only when that line runs. Same mistake, different stage, depending on the language.
-->

---

# Up next: Optimization

<div class="relative mx-auto mt-4 w-fit max-w-full">
<img src="/crafting-interpreters-pipeline.png" alt="Compiler pipeline mountain" class="block max-h-[62vh] w-auto" />

<div class="absolute" style="left:38.65%;top:22.21%;width:14.25%;height:16.81%"><div class="hl h-full w-full rounded-full border-4 border-red-500"></div></div>
</div>

<!--
~0.5 min. Script: "Next, optimization. The compiler rewrites the program to do less work."
-->

---

# Optimization

<div class="grid grid-cols-[auto_1fr] gap-x-8 gap-y-3 mt-8 text-lg">
<div class="opacity-60">Input</div><div>IR</div>
<div class="opacity-60">Output</div><div>faster code, same behavior</div>
<div class="opacity-60">What it does</div><div>folds constants, removes dead code, inlines calls</div>
</div>

<div class="grid grid-cols-3 gap-6 mt-10 text-center">
<div class="border rounded-xl p-6">

**Constant folding**

<div class="font-mono mt-3">2 + 3 → 5</div>

</div>
<div class="border rounded-xl p-6">

**Dead code removal**

<div class="mt-3 opacity-70">Unreachable code is dropped</div>

</div>
<div class="border rounded-xl p-6">

**Inlining**

<div class="mt-3 opacity-70">Small functions copied in place</div>

</div>
</div>

<!--
~1.5 min. Script: "The optimizer rewrites the program to do less work, with the same behavior. The result is faster code."

Stress: the optimizer must never change what the program does. That's where the hard bugs hide.
-->

---

# Up next: Code generation

<div class="relative mx-auto mt-4 w-fit max-w-full">
<img src="/crafting-interpreters-pipeline.png" alt="Compiler pipeline mountain" class="block max-h-[62vh] w-auto" />

<div class="absolute" style="left:73.19%;top:33.25%;width:20.29%;height:10.80%"><div class="hl h-full w-full rounded-full border-4 border-red-500"></div></div>
</div>

<!--
~0.5 min. Script: "Next, code generation. We pick the target: bytecode, native code, or another language."
-->

---

# Code generation

<div class="grid grid-cols-[auto_1fr] gap-x-8 gap-y-3 mt-8 text-lg">
<div class="opacity-60">Input</div><div>optimized IR</div>
<div class="opacity-60">Output</div><div>bytecode, native code or another language</div>
<div class="opacity-60">What it does</div><div>picks the target</div>
</div>

<div class="grid grid-cols-3 gap-6 mt-10 text-center">
<div class="border rounded-xl p-6">

**Bytecode → VM**

<div class="mt-3 opacity-70">JVM, Lua</div>

</div>
<div class="border rounded-xl p-6">

**Native code**

<div class="mt-3 opacity-70">C, Rust</div>

</div>
<div class="border rounded-xl p-6">

**Transpile**

<div class="mt-3 opacity-70">TypeScript → JavaScript</div>

</div>
</div>

<v-click>


<div class="mt-10 text-xl">An intermediate representation (IR) sits in the middle.</div>

</v-click>

<!--
~2 min. Script: "This is the decision point: what do we output? Bytecode for a VM, native machine code, or another high-level language."

The IR matters most: each front end targets the IR and each back end reads it, so N languages and M targets need N + M pieces, not N × M compilers. LLVM is the best-known example.
-->

---

# Why the IR matters

Kotlin has one frontend and several backends

<div class="mt-2 flex justify-center">
<img src="/IR-diagram.png" alt="Kotlin and other JVM languages share one bytecode backend" style="height: 330px; width: auto" />
</div>

<v-click>

<div class="mt-3 text-xl">The frontend scans, parses and checks Kotlin once. Only the backend changes.</div>

</v-click>

<!--
~1.5 min. Script: "Kotlin is a good example of the IR paying off. One frontend, three backends. The JVM bytecode is the decoupling point: any language that compiles to it runs on the same VM, which is why Java, Scala and Clojure share it."
-->

---

# Up next: Runtime

<img src="/crafting-interpreters-pipeline.png" alt="Compiler pipeline mountain" class="mx-auto mt-4 max-h-[64vh] w-auto" />

<!--
~0.5 min. Script: "Next, the runtime. This is where the program actually executes."
-->

---

# Runtime

<div class="grid grid-cols-[auto_1fr] gap-x-8 gap-y-3 mt-8 text-lg">
<div class="opacity-60">Input</div><div>compiled code</div>
<div class="opacity-60">Output</div><div>a running program</div>
<div class="opacity-60">What it does</div><div>manages memory, garbage collection and runtime checks</div>
</div>

<div class="grid grid-cols-3 gap-6 mt-10 text-center">
<div class="border rounded-xl p-6">

**Memory management**

<div class="mt-3 opacity-70">allocates objects and tracks where they live</div>

</div>
<div class="border rounded-xl p-6">

**Garbage collection**

<div class="mt-3 opacity-70">finds objects nothing points to anymore and frees them</div>

</div>
<div class="border rounded-xl p-6">

**Runtime checks**

<div class="mt-3 opacity-70">array bounds, null access, failed casts</div>

</div>
</div>


<!--
~1.5 min. Script: "The runtime runs your program and does three jobs. It manages memory: it allocates objects. It garbage-collects: it frees objects nothing points to anymore. And it runs checks that only make sense while the program runs, like array bounds."

Index 10 on a three-element array compiles fine. The check only happens when the line runs, so the error shows up at runtime.
-->

---

# Compiler flavors

| Flavor | Example |
| --- | --- |
| **Single-pass** | Turbo Pascal |
| **Tree-walk interpreter** | jlox |
| **Bytecode + VM** | JVM, Lua |
| **Transpiler** | TypeScript |

<!--
~2 min. Script: "Now the names make sense. Each flavor is a different mix of the stages we just saw."

Map each one back to the pipeline: a tree-walk interpreter skips code generation and executes the AST directly; a JIT generates machine code while the program runs.
-->

---

# Live demo

<div class="mt-6">
<ParseDemo />
</div>

<!--
~3 min. Type `1 + 2 * 3`, then `(1 + 2) * 3`. Point out how precedence changes the tree shape.

Backup: if the live demo fails, show the static tree from the parsing slide.
-->

---

# Real world

- Flex (lexer generator)
- Bison (parser generator)
- LLVM (backend)

<v-click>

<div class="mt-10 text-xl opacity-70">Rust, javac: written in their own language</div>

</v-click>

<!--
~1 min. Script: "You don't build all of this from scratch. Tools exist for every stage. Many languages use LLVM as their backend, and many compilers are self-hosted: the Rust compiler is written in Rust, and javac is written in Java."
-->

---
layout: statement
---

# Compilers aren't magic.

<v-click>

They're a pipeline of well-understood stages.

</v-click>

<!--
~0.5 min. Script: "Understanding this helps you read source code, debug, and appreciate language design."
-->

---

# Resources

- *Crafting Interpreters*, Robert Nystrom
- *Language Implementation Patterns*, Terence Parr
- Read the source: Lua, V, Kotlin

<div class="text-sm opacity-50 mt-8">Pipeline diagram adapted from <i>Crafting Interpreters</i> by Robert Nystrom.</div>

<!--
~0.5 min. Script: point at the book first, then the source code to read.
-->

---
layout: intro
---

# Thank you

## Questions?


<!--
Script: "Thank you for staying with me."

Timings add up to about 20 min. Skip the demo if time is short.
-->
