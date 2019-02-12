---
title: Matrix Multiplication by sgemm
date: 2019-02-12 12:20:17
categories:
  - Compute
tags:
  - BLAS
---

Introduce matrix maltiplication by BLAS library SGEMM API.

<!-- more -->

# Get the output in column major

For compatibility BLAS keeps the column major style as matrix stroage as same as FORTRAN.
But it's a bit hard for C/C++ programmers, who are familiar with row major matrix.

e.g. `C = alpha * A * B + beta * C` as `C = A * B`, alpha= 1.0f，beta = 0.0f.
```
A (m x k) * B (k x n) = C (m x n)
   2 x 3       3 x 2       2 x 2

hipblasSgemm(handle,
             HIPBLAS_OP_T, HIPBLAS_OP_T,
             m, n, k,
             &alpha,
             da, lda, // lda = k
             db, ldb, // ldb = n
             &beta,
             dc, ldc) // ldc = m

CPU(row major):

| 0 1 2 |   | 6 5 |     |  8  5 |
| 3 4 5 |   | 4 3 |     | 44 32 |
            | 2 1 |

GPU(column major):

| 0 3 |     | 6 4 2 |   |  8  5 |
| 1 4 |     | 5 3 1 |   | 44 32 |
| 2 5 |      =>T           |
 => T                      | copy to CPU as matrix transpose
                           V
CPU(row major):
                        |  8 44 |
                        |  5 32 |

A(m x k) -> GPU: AT(k x m) -> HIPBLAS_OP_T -> GPU: A(m x k)
B(k x n) -> GPU: BT(n x k) -> HIPBLAS_OP_T -> GPU: B(k x n)

do sgemm
GPU: C(m x n) -> copy to CPU -> CPU: CT( n x m )
```

Leading Dimension:
  * The first index in the dimension. A(20, 10), lda=20, row number.
  * If we set `HIPBLAS_OP_T`, it will be the column number, otherwise, it's row number.
  * Get the Leading Dimension with *the matrix in GPU* for multiplication.

Summary:
```
C = A * B

利用cublasSgemm的参数 transa（transb） 和 lda（ldb）的设置来共同解决存储方式改变的问题。
关于A,B的参数（行和列）是in GPU时的matrix.

        CPU     GPU
    (1) A, B
    cublasSgemm('t','t', c_row, c_col, a_col, alpha, A, a_col, B, b_col, beta, C, c_row);
    如果求AT*B，只需将A的t改为n

    (2) B       A
    cublasSgemm('n','t', c_row, c_col, a_col, alpha, A, a_row, B, b_col, beta, C, c_row);

    (3) A       B
    cublasSgemm('t','n', c_row, c_col, a_col, alpha, A, a_col, B, b_row, beta, C, c_row);

    (4)         A, B
    cublasSgemm('n','n', c_row, c_col, a_col, alpha, A, a_row, B, b_row, beta, C, c_row);

得到的结果C都是按列存储的（拷贝出host内存之前转置一下，可以考虑使用, 例如cublasSgeam()(矩阵加法),
进行一次1.0 * AT + 0.0 * B的参数设定, 利用内置的转置功能(注意这里的1和0), 来进行将A转换成AT）。

如果是CPU创建的矩阵，要设置T，随之ld使用列，反之用行。
```

# Get the output in row major

As we know, blas uses column major matrix. If going to get the C in row major as result, we can do `CT = BT * AT`, copy CT to CPU and get C directly.
```
A (m x k) * B (k x n) = C (m x n)
   2 x 3       3 x 2       2 x 2

BT(n x k) * AT(k x m) = CT(n x m) in GPU
   2 x 3       3 x 2       2 x 2

hipblasSgemm(handle,
             HIPBLAS_OP_N, HIPBLAS_OP_N,
             n, m, k,
             &alpha,
             db, ldb, // ldb = n
             da, lda, // lda = k
             &beta,
             dc, ldc) // ldc = n
```

# Reference
* [Row- and column-major order](https://en.wikipedia.org/wiki/Row-_and_column-major_order)
* [cuda 矩阵乘法函数之cublasSgemm](https://blog.csdn.net/u011460059/article/details/56506987)
