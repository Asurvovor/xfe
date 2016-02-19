$$
\begin{align}
\frac{\partial{ \left( f(x) + \frac{\rho}{2} {\Vert Ax - v \Vert}_2^2 \right) }} {\partial{x}}
& = \frac{\partial{\left( \lambda{\Vert x \Vert}_1 + \frac{\rho}{2} {\Vert Ax - v \Vert}_2^2\right)}} {\partial{x}} \
& = \frac{\partial{(\lambda {\Vert x \Vert}_1)}} {\partial{x}} + \underline{ \frac{\rho}{2} \cdot \frac{(Ax)^T(Ax) - 2(Ax)^Tv + {\Vert v \Vert}_2^2} {\partial{x}} } \
& = -\lambda I + \underline{ \rho A^TAx - \rho A^T v } \
& = \rho A^TAx - (\lambda I + \rho A^T v) = 0
\end{align} \qquad(2.8)
$$
