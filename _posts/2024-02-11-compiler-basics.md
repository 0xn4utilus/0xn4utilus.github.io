---
title: Compiler Basics
author: nautilus
date: 2024-02-11 21:30:30 +0530
categories: [Compiler,Development]
tags: [compiler]
---
Consise post about the various stages in the compilation process particularly that of C using clang.

## Overview of Compiler

Computer needs binaries or executables to run any program. These binaries are nothing but sequence of 0s and 1s or instructions that are to be executed by the CPU. These instructions depend on the computer architecture and are called [Instruction Set Architecture (ISA)](https://en.wikipedia.org/wiki/Instruction_set_architecture). There are different ISAs like x86, ARM, MIPS, etc. and thus we need machine specific binaries. A compiler is a program that translates high-level programming language into machine-specific binaries be executed by a computer.

## Different Phases of Compilation

- Lexical Analysis: Takes in the source code written in high-level language as a input and scans it to generate a stream of [Tokens](https://learn.microsoft.com/en-us/cpp/cpp/character-sets). The characters are scanned one by one and we try to classify them in pre defined categories called lexemes(for example keywords, identifiers, operators, etc.). After lexemes are recognised, they are defined as Tokens.
- Syntax Analysis: The stream of Tokens are then parsed against the rules of the language's [grammar](https://en.wikibooks.org/wiki/Introduction_to_Programming_Languages/Grammars) to form an Abstract Syntax Tree. We only check if the tokens generated are in the correct order(for example the parentheses are matched or not).
- Semantic Analysis: The constructed AST is logically validated. Type checking, Scope resolution, duplicate variable declaration checking, control flow analysis, etc happens here.
- Intermediate Code Generation: There are a number of OS and computer architectures and thus directly converting from source code to machine-specific binaries is not fessible. Intermediate Representation is machine-independent and thus it is optimal to perform optimizations on IR then coverting it to binaries.
- Code Generation: IR is converted into target machine code and we get an executable.

## How clang compiles C code?

Let us now see various stages involved in compiling a basic C HelloWorld! code.

```c
#include <stdio.h>

int main() {
    printf("Hello, World!\n");
    return 0;
}

```

### Lexical Analysis 

Run `clang -fsyntax-only -Xclang -dump-tokens inputfile.c` to see the tokens generated. Note the additional information associated with each token.

```c
...
... preprocessor tokens
...
int 'int'        [LeadingSpace] Loc=</usr/include/stdio.h:886:32>
r_paren ')'             Loc=</usr/include/stdio.h:886:35>
semi ';'                Loc=</usr/include/stdio.h:886:36>
int 'int'        [StartOfLine]  Loc=<helloW.c:3:1>
identifier 'main'        [LeadingSpace] Loc=<helloW.c:3:5>
l_paren '('             Loc=<helloW.c:3:9>
r_paren ')'             Loc=<helloW.c:3:10>
l_brace '{'      [LeadingSpace] Loc=<helloW.c:3:12>
identifier 'printf'      [StartOfLine] [LeadingSpace]   Loc=<helloW.c:4:5>
l_paren '('             Loc=<helloW.c:4:11>
string_literal '"Hello, World!\n"'              Loc=<helloW.c:4:12>
r_paren ')'             Loc=<helloW.c:4:29>
semi ';'                Loc=<helloW.c:4:30>
return 'return'  [StartOfLine] [LeadingSpace]   Loc=<helloW.c:5:5>
numeric_constant '0'     [LeadingSpace] Loc=<helloW.c:5:12>
semi ';'                Loc=<helloW.c:5:13>
r_brace '}'      [StartOfLine]  Loc=<helloW.c:6:1>
eof ''          Loc=<helloW.c:6:2>
```

### AST Generatation

We can view the AST using `clang -Xclang -ast-dump inputfile.c` or `clang -Xclang -ast-dump=json inputfile.c`.

```c
.
. 
.
`-FunctionDecl 0x1364e90 <helloW.c:3:1, line:6:1> line:3:5 main 'int ()'
  `-CompoundStmt 0x1365058 <col:12, line:6:1>
    |-CallExpr 0x1364fd0 <line:4:5, col:29> 'int'
    | |-ImplicitCastExpr 0x1364fb8 <col:5> 'int (*)(const char *, ...)' <FunctionToPointerDecay>
    | | `-DeclRefExpr 0x1364f30 <col:5> 'int (const char *, ...)' Function 0x1349938 'printf' 'int (const char *, ...)'
    | `-ImplicitCastExpr 0x1365010 <col:12> 'const char *' <NoOp>
    |   `-ImplicitCastExpr 0x1364ff8 <col:12> 'char *' <ArrayToPointerDecay>
    |     `-StringLiteral 0x1364f50 <col:12> 'char[15]' lvalue "Hello, World!\n"
    `-ReturnStmt 0x1365048 <line:5:5, col:12>
      `-IntegerLiteral 0x1365028 <col:12> 'int' 0
```

### IR Code Generatation

To view the Intermediate Representation run `clang -S -emit-llvm inputfile.c` it will geneate a `.ll` file.

```c
; ModuleID = 'helloW.c'
source_filename = "helloW.c"
target datalayout = "e-m:e-p270:32:32-p271:32:32-p272:64:64-i64:64-f80:128-n8:16:32:64-S128"
target triple = "x86_64-pc-linux-gnu"

@.str = private unnamed_addr constant [15 x i8] c"Hello, World!\0A\00", align 1

; Function Attrs: noinline nounwind optnone uwtable
define dso_local i32 @main() #0 {
  %1 = alloca i32, align 4
  store i32 0, i32* %1, align 4
  %2 = call i32 (i8*, ...) @printf(i8* noundef getelementptr inbounds ([15 x i8], [15 x i8]* @.str, i64 0, i64 0))
  ret i32 0
}

declare i32 @printf(i8* noundef, ...) #1

attributes #0 = { noinline nounwind optnone uwtable "frame-pointer"="all" "min-legal-vector-width"="0" "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="x86-64" "target-features"="+cx8,+fxsr,+mmx,+sse,+sse2,+x87" "tune-cpu"="generic" }
attributes #1 = { "frame-pointer"="all" "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="x86-64" "target-features"="+cx8,+fxsr,+mmx,+sse,+sse2,+x87" "tune-cpu"="generic" }

!llvm.module.flags = !{!0, !1, !2, !3, !4}
!llvm.ident = !{!5}

!0 = !{i32 1, !"wchar_size", i32 4}
!1 = !{i32 7, !"PIC Level", i32 2}
!2 = !{i32 7, !"PIE Level", i32 2}
!3 = !{i32 7, !"uwtable", i32 1}
!4 = !{i32 7, !"frame-pointer", i32 2}
!5 = !{!"Ubuntu clang version 14.0.0-1ubuntu1.1"}
```

### Assembly Code Generatation

Finally we can see the machine specific assembly code using `clang -S inputfile.c` it is dependent on target machine. You can use `--target` flag to generate assembly for different architectures. Notice the differences in ARM and x86 assembly instructions.

#### x86 architecture, Ubuntu OS

```c
        .text
        .file   "helloW.c"
        .globl  main                             # -- Begin function main
        .p2align        4, 0x90
        .type   main,@function
main:                                   # @main
        .cfi_startproc
# %bb.0:
        pushq   %rbp
        .cfi_def_cfa_offset 16
        .cfi_offset %rbp, -16
        movq    %rsp, %rbp
        .cfi_def_cfa_register %rbp
        subq    $16, %rsp
        movl    $0, -4(%rbp)
        leaq    .L.str(%rip), %rdi
        movb    $0, %al
        callq   printf@PLT
        xorl    %eax, %eax
        addq    $16, %rsp
        popq    %rbp
        .cfi_def_cfa %rsp, 8
        retq
.Lfunc_end0:
        .size   main, .Lfunc_end0-main
        .cfi_endproc
                                        # -- End function
        .type   .L.str,@object                  # @.str
        .section        .rodata.str1.1,"aMS",@progbits,1
.L.str:
        .asciz  "Hello, World!\n"
        .size   .L.str, 15

        .ident  "Ubuntu clang version 14.0.0-1ubuntu1.1"
        .section        ".note.GNU-stack","",@progbits
        .addrsig
        .addrsig_sym printf
```

#### ARM Architecture

```c
        .text
        .syntax unified
        .eabi_attribute 67, "2.09"      @ Tag_conformance
        .cpu    arm7tdmi
        .eabi_attribute 6, 2    @ Tag_CPU_arch
        .eabi_attribute 8, 1    @ Tag_ARM_ISA_use
        .eabi_attribute 9, 1    @ Tag_THUMB_ISA_use
        .eabi_attribute 34, 0   @ Tag_CPU_unaligned_access
        .eabi_attribute 17, 1   @ Tag_ABI_PCS_GOT_use
        .eabi_attribute 20, 1   @ Tag_ABI_FP_denormal
        .eabi_attribute 21, 0   @ Tag_ABI_FP_exceptions
        .eabi_attribute 23, 3   @ Tag_ABI_FP_number_model
        .eabi_attribute 24, 1   @ Tag_ABI_align_needed
        .eabi_attribute 25, 1   @ Tag_ABI_align_preserved
        .eabi_attribute 38, 1   @ Tag_ABI_FP_16bit_format
        .eabi_attribute 18, 4   @ Tag_ABI_PCS_wchar_t
        .eabi_attribute 26, 2   @ Tag_ABI_enum_size
        .eabi_attribute 14, 0   @ Tag_ABI_PCS_R9_use
        .file   "helloW.c"
        .globl  main                            @ -- Begin function main
        .p2align        2
        .type   main,%function
        .code   32                              @ @main
main:
        .fnstart
@ %bb.0:
        push    {r11, lr}
        mov     r11, sp
        sub     sp, sp, #8
        mov     r0, #0
        str     r0, [sp]                        @ 4-byte Spill
        str     r0, [sp, #4]
        ldr     r0, .LCPI0_0
        bl      printf
                                        @ kill: def $r1 killed $r0
        ldr     r0, [sp]                        @ 4-byte Reload
        mov     sp, r11
        pop     {r11, lr}
        bx      lr
        .p2align        2
@ %bb.1:
.LCPI0_0:
        .long   .L.str
.Lfunc_end0:
        .size   main, .Lfunc_end0-main
        .cantunwind
        .fnend
                                        @ -- End function
        .type   .L.str,%object                  @ @.str
        .section        .rodata.str1.1,"aMS",%progbits,1
.L.str:
        .asciz  "Hello, World!\n"
        .size   .L.str, 15

        .ident  "Ubuntu clang version 14.0.0-1ubuntu1.1"
        .section        ".note.GNU-stack","",%progbits
        .addrsig
        .addrsig_sym printf
```

The assembly code generated depends on the optimization flags viz. `-O1`, `-O2`, `-O3`, `-Ofast`, `-Os`, `-Oz`, etc. read more [here](https://clang.llvm.org/docs/CommandGuide/clang.html).

## Additional Resources

- [https://karkare.github.io/cs335/](https://karkare.github.io/cs335/)
- [https://www.cs.princeton.edu/courses/archive/spr03/cs320/schedule.htm](https://www.cs.princeton.edu/courses/archive/spr03/cs320/schedule.htm)
- [https://iitd-plos.github.io/col729/refs/ALSUdragonbook.pdf](https://iitd-plos.github.io/col729/refs/ALSUdragonbook.pdf)
