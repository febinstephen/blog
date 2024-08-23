+++
author = "febin.stephen"
title = "Journey of a C program."
date = "2020-06-10"
description = "My coding lessons."
tags = [
    "c",
    "programming",
]
weight = 10
+++

Here is a piece of note I prepared as part of the C-learning process.<!--more-->Don't you think it is little laborious??Well it's a reality at least for people who are asked to learn C by working out problems in the classic K&R book.The *Source.c* compilation and execution stages are the most complex of all the other languages out there.A typical C-program goes through four stages before it become executable. I am using GCC(GNU compiler collection) to compile and execute my c-programs.C-compilation is a multi-step process.Here I describe stuff which is deep but yet in a short and simple way. As you should all know C was designed to implement the UNIX operating system,most of the UNIX kernel and all of it's supporting tools and libraries are written in C,and of course C is the first choice for system-level programming.

**source.c** is a simple program to demonstrate the translation from source file to executable file. The following command (provided that gcc is installed on your Linux box) compiles **source.c** and creates an executable file called **source**,by default an executable file called a.out is created.
> gcc source.c -o source

```
#include <stdio.h>
#define A 10
main()
{
int a, b, c;
a = b = A;
c = a + b;
printf("%d\n",c);
}
```

The GCC compiler reads **source.c** and translate it to an executable file.The compilation is performed in four sequential steps. We will see each one of them in detail and also show commands to perform each phases separately and see the output to understand the changes occur to source.c

Now, let's perform all four steps to compile and run C program one by one.


### Preprocessing

Before the compiler starts compiling, the source file is processed by a program called **CPP** (C preprocessor). **CPP** is invoked automatically by the compiler. **CPP** converts **source.c** file into another 'modified' or 'expanded' **source.c**. preprocessor command starts with a #. Bit confusing ehh? don't worry let's preprocess our **source.c** with the following command and see the output. Use either of the following commands.

> cpp source.c || gcc -E source.c

You can redirect the above commands to a file with 'i' extension to store and check what all changes occurred to our **source.c**. Below is only a part of the new **source.c**.
Here you can see all the preprocessor directives (macros and header files) are expanded. For instance the statement

a = b = A;

becomes

a = b = 10;

### Compilation proper

In this phase the preprocessed **source.c** is transformed into assembly code. We can explicitly tell gcc to perform this phase by passing the -S option, the following command creates a file called **source.s**. After having created the **source.s**, While looking at assembly code you may note that the assembly code contains a call to the external function printf and other opcodes like addl,movl.

> gcc -S source.c

Below is an extract from **source.c**

```
main:
.LFB0:
.cfi_startproc
pushq %rbp
.cfi_def_cfa_offset 16
.cfi_offset 6, -16
movq %rsp, %rbp
.cfi_def_cfa_register 6
subq $16, %rsp
movl $10, -12(%rbp)
movl -12(%rbp), %eax
movl %eax, -8(%rbp)
movl -12(%rbp), %eax
movl -8(%rbp), %edx
addl %edx, %eax
movl %eax, -4(%rbp)
movl -4(%rbp), %eax
movl %eax, %esi
movl $.LC0, %edi
movl $0, %eax
call printf
leave
.cfi_def_cfa 7, 8
ret
.cfi_endproc
```
### Assembly

In this phase the C source code is turned into an object code file, which is a file ending in **.o** which contains the binary version of the source code. Object code is not directly executable, though. In order to make an executable, you also have to add code for all of the library functions that were #included into the file (this is not the same as including the declarations, which is what #include does). This is the job of the linker. The command to perform this phase is as follows, -c option means to compile the source code file into an object file but not to invoke the linker.

> gcc -c source.c
> as source.s -o source.o

Here assembler(as) converts **source.s** into machine instructions and generates the object file **source.o**

I am not adding extract of source.o because it is purely machine understandable form,and I suppose i'm not posting this blog for any machines to read!

### Linking

Like preprocessor, linker is also a separate program called **ld**, and it is invoked automatically when you use the compiler.

> gcc source.c -o source

Now you have a file called source that you can run and which will hopefully do something cool and/or useful. This is all you need to know to begin compiling your own C programs.Hope you have enjoyed reading this blog!


{{< css.inline >}}

<style>
.canon { background: white; width: 100%; height: auto; }
</style>

{{< /css.inline >}}
