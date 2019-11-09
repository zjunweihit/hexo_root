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
It's a different from C/C++ programming, which is row major matrix.

* e.g. `C = alpha * A * B + beta * C` as `C = A * B`, alpha= 1.0f，beta = 0.0f.
```
Prepare data A, B in column major and fixed shape as below:
A (m x k) * B (k x n) = C (m x n)
   2 x 3       3 x 2       2 x 2

hipblasSgemm(handle,
             trans_A, trans_B,
             m, n, k,
             &alpha,
             da, lda,
             db, ldb,
             &beta,
             dc, ldc)
```
* NN => lda = m, ldb = k
```
| 0 2 4 | * | 6 3 |  =  | 26  8 |
| 1 3 5 |   | 5 2 |     | 41 14 |
            | 4 1 |
```
* NT => lda = m, ldb = n
```
| 0 2 4 | * | 6 5 |  =  | 16 10 |
| 1 3 5 |   | 4 3 |     | 28 19 |
            | 2 1 |

original B (n x k): 3 x 2

| 6 4 2 |
| 5 3 1 |

BT in sgemm for matrix multiplication

|6 5 |
|4 3 |
|2 1 |

```
* TN => lda = k, ldb = k
```
| 0 1 2 | * | 6 3 |  =  | 13 4  |
| 3 4 5 |   | 5 2 |     | 58 22 |
            | 4 1 |

original A (k x m): 3 x 2

| 0 3 |
| 1 4 |
| 2 5 |

BT in sgemm for matrix multiplication

| 0 1 2 |
| 3 4 5 |
```
* TT => lda = k, ldb = n
```
| 0 1 2 | * | 6 5 |  =  |  8  5 |
| 3 4 5 |   | 4 3 |     | 44 32 |
            | 2 1 |

orignal A (k x m): 3 x 2 => AT (m x k)

| 0 3 | T  | 0 1 2 |
| 1 4 | => | 3 4 5 | in sgemm for matrix multiplication
| 2 5 |

orignal B (n x k): 2 x 3 => BT (k x n)

| 6 4 2 | T  | 6 5 |
| 5 3 1 | => | 4 3 | in sgemm for matrix multiplication
             | 2 1 |

```

Leading Dimension:
  * The first index in the dimension. A(20, 10), lda=20, row number.
  * If we set `HIPBLAS_OP_T`, it will be the column number, otherwise, it's row number.
  * The number of items in next line at the same position as the current line in matrix order(row major or column major)

# Reference
* [Row- and column-major order](https://en.wikipedia.org/wiki/Row-_and_column-major_order)
