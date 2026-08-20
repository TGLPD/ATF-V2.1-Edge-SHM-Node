
# What are Bases in Linear Algebra
In linear algebra, a **basis** (plural: bases) is essentially a minimal set of fundamental building blocks that can be used to construct a vector space. You can think of a basis as the "coordinate system" for that space.

For a set of vectors to qualify as a basis for a vector space, it must strictly satisfy two conditions:

1. **They must span the space:** By scaling (multiplying by a number) and adding the basis vectors together, you can reach absolutely _any_ point in the vector space.
    
2. **They must be linearly independent:** None of the basis vectors are redundant. You cannot create one of the basis vectors by scaling and adding the others together.
    

If you remove a vector from a basis, they will no longer span the space. If you add another vector to a basis, the set will no longer be linearly independent.

## The Standard Basis

The most common basis you will encounter is the **standard basis**.

In a 2D plane ($\mathbb{R}^2$), the standard basis consists of two vectors of length 1 pointing exactly along the x-axis and y-axis:

- $\mathbf{e}_1 = \begin{pmatrix} 1 \\ 0 \end{pmatrix}$
    
- $\mathbf{e}_2 = \begin{pmatrix} 0 \\ 1 \end{pmatrix}$
    

Every vector in 2D space can be written as a unique combination of these two. For example, the vector $\begin{pmatrix} 3 \\ 4 \end{pmatrix}$ is simply exactly $3\mathbf{e}_1 + 4\mathbf{e}_2$.

The number of vectors in a basis tells you the **dimension** of the space. Because it takes exactly two basis vectors to define $\mathbb{R}^2$, it is 2-dimensional.

You aren't restricted to the standard basis. You can choose any two vectors to be your basis in a 2D plane, as long as they aren't pointing in the exact same (or directly opposite) direction.

You can use the interactive tool below to manipulate basis vectors and see how they can span a 2D space:

> **Key insight:** A basis gives you a unique way to describe any vector. If you change your basis, you change the coordinates of everything in that space, which is a foundational concept for things like data compression and computer graphics.