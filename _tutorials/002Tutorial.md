---
title: "Matrix multiplication" 
layout: post
date: 23-10-2022

usemathjax: true

image: 
  path: tutorials/images/002MatrixMultiplication.png 
  thumbnail: tutorials/images/002MatrixMultiplication.png
  caption: "A multiplication"
  
  categories:
     -intermediate
     -math
     -algorithms
  tags:
     -intermediate
     -math
     -algorithms
---

<h2> How to do it fast. And what is “fast”. </h2>

Due to a <a href="https://deepmind.google/discover/blog/discovering-novel-algorithms-with-alphatensor/"> novel result </a> in AI, every programmer group I am in has been mentioning matrix multiplication this week. How it is so impressive that an AI allows us to do this faster than before . . . But wait, what exactly was it before? What is faster? How can we measure how fast an algorithm runs? Why there are so many Colts in Deathloop? In this entry, I will try to answer 3 of the 4 last questions.

First I will talk about matrix multiplication. Well, first what is a matrix.

<h2> What is a matrix? How and why do we multiply them? </h2>

A $$n×m$$ matrix is a grid of $$n$$ cells and $$m$$ columns that have a number on every one of its entries. Vectors are just $$n×1$$ matrices. Now, why do we use Matrices in game development in the first place? Well, for explaining that we need to define matrix multiplication. For $$A$$ a $$n×m$$ matrix and B a $$m×l$$ matrix, we define $$AB$$ by the recursion described below. If $$A$$ and $$B$$ can be decompose in the submatrices,

$$
A = \begin{bmatrix}A_{11} & A_{12}\\A_{21} & A_{22}\end{bmatrix} B = \begin{bmatrix}B_{11} & B_{12}\\B_{21} & B_{22}\end{bmatrix}
$$

then

$$
AB = \begin{bmatrix}A_{11}B_{11} +A_{12}B_{21}& A_{11}B_{12} A_{12} B_{22} \\A_{21}B_{11} + A_{22} B_{21} & A_{21}B_{12} + A_{22} B_{22}\end{bmatrix}
$$

While the formula may seem weirdly artificial it is quite handy. Suppose we have $$v$$ a vector that denotes a two-dimensional position (let us say, the position of your main character) and you want to rotate it by an angle $$\theta$$. You could do some fancy algorithm to compute the new position. Or you could just do the multiplication $$R_\theta v$$, where,

$$
R_\theta = \begin{bmatrix} \cos(\theta) & -\sin(\theta)\\ \sin(\theta) & \cos(\theta) \end{bmatrix}
$$

And yup, rotating $$v$$ gets much easier. No need to do any fancy trigonometry (unless you want to convince yourself the last formula works, which you can do <a href="https://en.wikipedia.org/wiki/Rotation_matrix"> here </a>). In general, matrices are useful because they reduce the transformation of vectors into multiplications. And we know how to do multiplication, it is the algorithm we wrote above.

NOTE: You might have implemented the multiplication algorithm as two for loops. It is an <a href="https://i.redd.it/vrsnwsgpg0761.png"> exercise to the reader </a> to check that the recursive formula I described above is the same once you unroll all the recursions.

<h2> How we measure speed of an algorithm? </h2>

This is tricky, because we want to measure the algorithm, not the machine performance. Every machine is going to run at a different speed. What takes two seconds in an Xbox Series X may take weeks on a cellphone. So, we need to measure that does not just depend on the machine we are using to run the algorithm.

A way of doing that is by counting the number of operations an algorithm makes as a function of the size of the input.

Let us take matrix multiplication as an example and let us take square matrices to make it simpler. Our way of measuring the size of the input will be the number of columns of the matrices. Now, we have your recursive algorithm of matrix multiplication. So, if $$f(n)$$ is the number of operations we do by multiplying two matrices of size $$n×n$$, then, the recursive algorithm is telling us that,

$$
f(n) = 8 f\left(\frac{n}{2} \right)
$$

One can unroll that formula, and get that then $$f(n)=Cn^3$$. The constant $$C$$ is unimportant for now. It is going to depend on how fast our machine can multiply numbers, and not on the algorithm itself. That is why sometimes that is denoted by $$f(n)=\mathcal{O}(n^3)$$, that is, up to some constant our number of operations increases as a cube with the size of the input. Also, the quantity $$f$$
 is called the asymptotic complexity and is the most used way of measuring the speed of an algorithm.

<h2> How fast we can do multiplication? </h2>

Any sane person would look at the matrix multiplication formula and state there is no better way to do the multiplication than the recursive algorithm. Or that any other way to do it is equivalent.

Well, turns out there is a guy that was insane enough to realize that was not optimal. Recall that we can write,


$$
A = \begin{bmatrix}A_{11} & A_{12}\\A_{21} & A_{22}\end{bmatrix} B = \begin{bmatrix}B_{11} & B_{12}\\B_{21} & B_{22}\end{bmatrix}
$$

and with this we define

$$
M_1 = (A_{11} + A_{22})(B_{11}+B_{22}) \\
M_2 = (A_{21} + A_{22})B_{11} \\
M_3 = A_{11}(B_{12}-B_{22}) \\
M_4 = A_{22}(B_{21}-B_{11}) \\
M_5 = (A_{11}+A_{12})B_{22} \\
M_6 = (A_{21}-A_{11})(B_{11}+B_{12}) \\
M_7 = (A_{12}-A_{22})(B_{21}+B_{22})
$$

then we can actually get the surprising formula:

$$
AB = \begin{bmatrix} M_1 + M_4 - M_5 + M_7 & M_3 + M_5 \\ M_2 + M_4 & M_1 - M_2 + M_3 + M_6 \end{bmatrix}
$$

This is known as the Strassen algorithm for multiplication. How fast is this? Note that we reduce the multiplication of two matrices to the multiplication of 7 matrices of half the size. Then,

$$
f(n) = 7 f\left(\frac{n}{2} \right)
$$

Which <a href="https://makeameme.org/meme/when-your-math-08b3029fb4"> gives </a> that $$f(n)=\mathcal{O}(n^{log_2(7)})=\mathcal{O}(n^{2.807…})$$, which is better than the recursive algorithm! That means that, at least for bigger matrices, the weird Strassen algorithm is better than the standard multiplication algorithm.

<h2> Is this the fastest way to multiply matrices!? </h2>

The surprising answer is . . . we do not know. Since the output matrix has $$ \mathcal{O}(n^2) $$ number of entries any algorithm can at most run by doing that number of operations. But what is the exact best order? We do not know. For a long recap of what is known click <a href="https://en.wikipedia.org/wiki/Computational_complexity_of_matrix_multiplication"> here </a>, but so far, we know matrix multiplication can be obtained with an algorithm that does $$\mathcal{O}(n^{2.3728..})$$ operations. There is a series of conjectures (ie, things that are believed to be true) about the complexity of matrix multiplication, a popular being that in the best case it is $$O(n^2log(n))$$.

The note that I posted at the start of this entry is interesting because of different reasons. First, as I already mentioned, Strassen’s way of multiplying matrices is not the best one known for big matrices, but it was the best one for small ones for the last 55 years. That is, for matrices with less than 100 columns, the constants in the other algorithms are massive, and in reality, Strassen’s algorithm should be preferred in that case. However, this has just changed, as the new AlphaTensor algorithm is the first improvement we have in those small matrix multiplication in 55 years!

Another interesting point is the following. So far AlphaTensor gives an algorithm for fixed-size matrices, so it is impossible to get the asymptotic complexity of the algorithm it gives. If those methods could be relaxed and analysed for matrices of arbitrary size, this could mean we have an improvement in the asymptotic complexity of matrix multiplication. Could we finally get to the believed $$ \mathcal{O}(n^2log(n))$$ complexity? Only time and bunch of bored mathematicians will tell.

