# Use Prefix Operators (++i) vs. Postfix Operators (i++)


The ``**C++ Primer**'' book has a very interesting recommendation 
to use postfix operators, only when necessary (on page 148 for the 5th version). 

The authors shared the following in their book.    
>   The prefix version avoids unnecessary work.
>   It increments the value and returns the incremented version. 
>   The postfix operator must store the original value so that it
>   can return the unincremented value as its result." 
>
>   For ints and pointers, the compiler can optimize away this extra work. 
>

After reading this, I started to use prefix instead of postfix more often. 
However, I have always been doubting the statement the authors made in their book,
until this moment.

To finally test the authors' claim, I created the following program. 
```
#include <iostream>
int main() {
        long i=0, j;
        j = i++;
        j = ++i;
        return 0;
}
```

Here is the assembly code for the above short code. 
```
_main:                                  ## @main
        .cfi_startproc
## %bb.0:
        pushq   %rbp
        .cfi_def_cfa_offset 16
        .cfi_offset %rbp, -16
        movq    %rsp, %rbp
        .cfi_def_cfa_register %rbp
        xorl    %eax, %eax
        movl    $0, -4(%rbp)
        movq    $0, -16(%rbp)       // assign 0 to i

                                    // j = i++;
        movq    -16(%rbp), %rcx     // move i from memory to register rcx
        movq    %rcx, %rdx          // copy rcx to rdx
        addq    $1, %rdx            // increment rdx
        movq    %rdx, -16(%rbp)     // store rdx into i
        movq    %rcx, -24(%rbp)     // store rcx into j

                                    // j = ++i;    
        movq    -16(%rbp), %rcx     // move i from memory to rcx
        addq    $1, %rcx            // increment rcx
        movq    %rcx, -16(%rbp)     // move rcx into i
        movq    %rcx, -24(%rbp)     // move rcx into j

        popq    %rbp
        retq
        .cfi_endproc
```
For postfix operators, the compiler indeed stores another copy in rcx
 and then assign it to j. We use one more register for postfix operators.

```
// program 2

#include <iostream>
int main() {
        long i=0, j;
        i++;
        ++i;
        return 0;
}
```

```
_main:                                  ## @main
        .cfi_startproc
## %bb.0:
        pushq   %rbp
        .cfi_def_cfa_offset 16
        .cfi_offset %rbp, -16
        movq    %rsp, %rbp
        .cfi_def_cfa_register %rbp
        xorl    %eax, %eax
        movl    $0, -4(%rbp)
        movq    $0, -16(%rbp)       // assign 0 to i
        
                                    // i++;
        movq    -16(%rbp), %rcx     // move i from memory to rcx
        addq    $1, %rcx            // increment rcx by 1
        movq    %rcx, -16(%rbp)     // store rcx back to i

                                    // ++i;
        movq    -16(%rbp), %rcx
        addq    $1, %rcx
        movq    %rcx, -16(%rbp)

        popq    %rbp
        retq
        .cfi_endproc
```

If we just increment a variable by itself, it seems the 
generated code would be the same for both postfix and prefix operators. 
