---
layout: post
title: "Optimization techniques: Cache memory access" 
date: 2023-05-10 17:06:00
description: "Cache memory accesses optimization techniques are strategies used to improve program performance by minimizing the number of cache misses."
categories: Development
tags: c/c++ performance
---

The aim of this post is to better understand the functioning of processor cache memories and how to optimize the code, in this case in C or C++, minimizing the number of cache misses. To do this, a total of 3 techniques will be presented using an example of matrix multiplication.

## Unoptimized matrix multiplication

In this case, we have an example of multiplication of 2 matrices. The result of this multiplication will be stored in a third matrix. The variables that store the values of the matrices are A, B, and C. On the other hand, N is the maximum size of the matrix. So the matrices are of size NxN. Below is the main operation source code.

```c
for (j = 0; j < N; j++)
    for (k = 0; k < N; k++)
        for (i = 0; i < N; i++)
            C[i][j] += A[i][k] * B[k][j];
```

Now we compile the program with GCC and run it using the 'time' program to measure the time it took to perform the computation. At the time of compilation, we introduced the "-O0" parameter to indicate to the compiler that we do not want any kind of optimization, since it is possible that several of the techniques that will be shown below are applied automatically by the compiler.

```bash
$ gcc -O0 code.c -o run
$ time ./run
# output: TODO: complete results
```

As we can see, the program took a few seconds to calculate the result. This is logical because the program has an algorithmic complexity of n^3, with 3 nested loops for each matrix. However, as we will see now, a large part of this time is spent on memory accesses, which are very slow.

After seeing the program and running the simple version without any problems, let's see the different optimizations to be made.


## Loop interchange

This technique consists of reordering the nested loops, so that data access is made in an order that favors cached blocks. For this, it is important to take into account the size of the cache blocks of our processor.

```c
// ---------- unoptimized code ----------
// for (j = 0; j < N; j++)
//     for (k = 0; k < N; k++)
//         for (i = 0; i < N; i++)
//             C[i][j] += A[i][k] * B[k][j];
// --------------------------------------

for (i = 0; i < N; i++)
    for (k = 0; k < N; k++)
        for (j = 0; j < N; j++)
            C[i][j] += A[i][k] * B[k][j];
```

As we can see, we have changed the order of the loops so that they go in the same order as the accesses of matrix C (unlike the unoptimized code). This way, we manage to perform the iterations of matrix C by rows instead of columns, reducing the number of cache misses. This is because cache blocks are filled with information from rows, so in the previous way, we were jumping from row to row, causing a cache miss for each access.

As in the previous case, we compile and run the program. The performance has improved compared to the previous execution, as we can see.

```bash
$ gcc -O0 code_loop_interchange.c -o run_loop_interchange
$ time ./run_loop_interchange
# output: TODO: complete results
```

## Loop blocking

With the previous case, we learned how to take advantage of the spatial locality of data and the order of access to minimize memory access by utilizing cache and the way it brings data. However, we can still take advantage of temporal locality and the time that the cache stores data. In this way, we can initially produce a series of cache misses to bring a set of information and then access it massively in the cache.

The case of matrix multiplication is a perfect example, as for each row of matrix A, we need to access the equivalent column of matrix B. To optimize both accesses, we can form blocks with width equal to the cache block. Thus, the first time we read a block, we will get a high number of cache misses, but for the rest of the operations to be performed on the block, there will be no cache misses. To implement this, we simply choose a block width size which we will call "NB". This parameter defines the blocks that we are going to traverse.


```c
// ---------- loop interchange ----------
// for (i = 0; i < N; i++)
//     for (k = 0; k < N; k++)
//         for (j = 0; j < N; j++)
//             C[i][j] += A[i][k] * B[k][j];
// --------------------------------------

for (i0 = 0; i0 < N; i0+=NB)
    for (k0 = 0; k0 < N; k0+=NB)
        for (j0 = 0; j0 < N; j0+=NB)
            for (i = 0; i < (i0+NB); i++)
                for (k = k0; k < (k0+NB); k++)
                    for (j = j0; j < (j0+NB); j++)
                        C[i][j] += A[i][k] * B[k][j];
```

As in the previous case, we compile and run the program. The performance has improved compared to the previous execution, as we can see.

```bash
$ gcc -O0 code_blocking.c -o run_blocking
$ time ./run_blocking
# output: TODO: complete results
```

## Loop unrolling

The unrolling technique is possibly the simplest of all the techniques seen previously. Its goal is to modify the loop structure in order to reduce the number of instructions to be executed due to the iterators. To make this case more readable, I will apply this optimization to the code optimized with Loop Interchange, although it could also be applied to the blocking optimization technique code.

```c
// ---------- loop interchange ----------
// for (i = 0; i < N; i++)
//     for (k = 0; k < N; k++)
//         for (j = 0; j < N; j++)
//             C[i][j] += A[i][k] * B[k][j];
// --------------------------------------

for (i = 0; i < N; i++)
    for (k = 0; k < N; k+=2)
        for (j = 0; j < N; j+=2) {
            C[i][j]   += A[i][k]   * B[k][j];
            C[i][j+1] += A[i][k]   * B[k][j+1];
            C[i][j]   += A[i][k+1] * B[k+1][j];
            C[i][j+1] += A[i][k+1] * B[k+1][j+1];
        }
```

As we can see in this example, we are reducing the number of additions that are performed due to the "for" loops. This means that each loop has to iterate by doing an "iterator+=2" instead of going one by one, but computationally it is not something that affects us.

It is true that we could write the matrix multiplication code line by line. However, we must consider the disadvantages that this entails. The first disadvantage is the readability and potential source of bugs that all the hardcoded operations in the matrix multiplication can bring. The second is the size of the code itself, which will be considerably increased. And finally, there is the issue of code reuse. In this case, we can do it since the matrix we are multiplying has an even width. However, if the unrolling we want to perform is not a multiple of the matrix size, we will end up with a segmentation fault, limiting us to the matrices we can multiply.

As in the previous case, we compile and run the program. The performance has improved compared to the loop interchange execution, as we can see.

```bash
$ gcc -O0 code_unrolling.c -o run_unrolling
$ time ./run_unrolling
# output: TODO: complete results
```