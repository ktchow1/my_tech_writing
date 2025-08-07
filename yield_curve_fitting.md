h2. Piecewise linear regression for yield curve fitting

Given a set of bond data currently tradable in the market:

{latex}
\{T_n, b_n, a_n, c_n, d_n\} \quad \text{for all } n \in \{1, N\}
{latex}

where:

* \(T_n\) — bond maturity  
* \(b_n\) — current best bid of bond \(n\)  
* \(a_n\) — current best ask of bond \(n\)  
* \(c_n\) — previous close price of bond \(n\)  
* \(d_n\) — dv01 of bond \(n\)  

---

h3. Objective

The objective is to fit the yield curve:

{latex}
y(\tau | y_{\tau_0}, y_{\tau_1}, y_{\tau_2}, \dots, y_{\tau_M})
{latex}

with \(\tau\) being any tenor point on the yield curve.  
The curve is parameterized by the yield at a set of predefined time points:

{latex}
\{y_{\tau_m}\}, \quad m \in [0, M]
{latex}

This is a piecewise linear model with \(M\) control points, common control points are:

{latex}
\tau_0 = 0,\quad
\tau_1 = 1d,\quad
\tau_2 = 1M,\quad
\tau_3 = 3M,\quad
\tau_4 = 6M,\quad
\tau_5 = 1Y, \ \text{etc.}
{latex}

---

h3. Step 1 — Convert bond prices to yields

The first step involves converting observed bond prices to bond yields using methods like bisection or Newton-Raphson (solving the Internal-Rate-Return equation).  
Alternatively, we can approximate the price–yield relation using a linear term of the Taylor series, as long as the pivot is close enough.  
Here, we take the previous day closing price as the pivot. For bond \(n\):

{latex}
m_n = \frac{1}{2}(b_n + a_n) = c_n + d_n y_{\tau_n}
{latex}

---

h3. Notation

* \(\{y_{\tau_n}\},\ n \in [1, N]\) — observations / data points  
* \(\{y_{\tau_m}\},\ m \in [0, M]\) — parameters / control points  
* \(y(\tau|\dots)\) — regression of yield curve at \(\tau\)  

---

h3. Objective function

We minimize the weighted sum of price-error squares w.r.t. control points:

{latex}
\min_{\{y_{\tau_m}\}} \sum_{n=1}^N w_n \left( c_n + d_n y_{\tau_n} - \frac{1}{2}(b_n + a_n) \right)^2
{latex}

Substitute interpolation:

{latex}
y_{\tau_n} = (1 - \mu_{\tau_n}) y_{\tau_m} + \mu_{\tau_n} y_{\tau_{m+1}}
{latex}

So:

{latex}
\min_{\{y_{\tau_m}\}} \sum_{n=1}^N w_n \left( c_n + d_n \big[ (1 - \mu_{\tau_n}) y_{\tau_m} + \mu_{\tau_n} y_{\tau_{m+1}} \big] - \frac{1}{2}(b_n + a_n) \right)^2
\tag{1}
{latex}

where:

{latex}
\mu_{\tau_n} = \frac{T_n - \tau_m}{\tau_{m+1} - \tau_m}, \quad
w_n = \left( \frac{1}{a_n - b_n} \right)^2
{latex}

---

h3. Matrix form

{latex}
\min_{Y} (AY - B)^\top W (AY - B)
\tag{2}
{latex}

where:

{latex}
A = 
\begin{bmatrix}
0 & \frac{d_1 (\tau_2 - T_1)}{\tau_2 - \tau_1} & \frac{d_1 (T_1 - \tau_1)}{\tau_2 - \tau_1} & \dots & 0 & 0 \\
0 & 0 & \frac{d_2 (\tau_3 - T_2)}{\tau_3 - \tau_2} & \frac{d_2 (T_2 - \tau_2)}{\tau_3 - \tau_2} & \dots & 0 \\
\vdots & \vdots & \vdots & \vdots & & \vdots \\
0 & 0 & \dots & d_N \frac{\tau_{M+1} - T_N}{\tau_{M+1} - \tau_M} & d_N \frac{T_N - \tau_M}{\tau_{M+1} - \tau_M}
\end{bmatrix}
{latex}

{latex}
Y = 
\begin{bmatrix}
y_{\tau_0} \\
y_{\tau_1} \\
y_{\tau_2} \\
\vdots \\
y_{\tau_M}
\end{bmatrix}
\quad
B =
\begin{bmatrix}
\frac{1}{2}(b_1 + a_1) - c_1 \\
\frac{1}{2}(b_2 + a_2) - c_2 \\
\frac{1}{2}(b_3 + a_3) - c_3 \\
\vdots \\
\frac{1}{2}(b_N + a_N) - c_N
\end{bmatrix}
{latex}

---

h3. Step 2 — Transformation to delta-yield space

We apply a linear transformation to the data space and the parameter space (to handle the case when all bonds have similar tenor).

Transformation matrix \(T\):

{latex}
T =
\begin{bmatrix}
1 & 0 & 0 & 0 & \dots & 0 \\
-1 & 1 & 0 & 0 & \dots & 0 \\
0 & -1 & 1 & 0 & \dots & 0 \\
\vdots & \vdots & \vdots & \ddots & \ddots & \vdots \\
0 & 0 & 0 & 0 & \dots & -1 & 1
\end{bmatrix}
{latex}

{latex}
A' = A T^{-1} = 
\begin{bmatrix}
1 & 0 & 0 & \dots \\
1 & 1 & 0 & \dots \\
1 & 1 & 1 & \dots \\
\vdots & \vdots & \vdots & \ddots
\end{bmatrix}
{latex}

{latex}
Y' = T Y =
\begin{bmatrix}
y_{\tau_0} \\
y_{\tau_1} - y_{\tau_0} \\
y_{\tau_2} - y_{\tau_1} \\
\vdots \\
y_{\tau_M} - y_{\tau_{M-1}}
\end{bmatrix}
{latex}

---

h3. Step 3 — Regularization

We minimize:

{latex}
J(Y') = (A'Y' - B)^\top W (A'Y' - B) + \lambda Y'^\top Y'
\tag{4}
{latex}

Optimality condition:

{latex}
A'^\top W A' Y'_{\text{opt}} + \lambda Y'_{\text{opt}} = A'^\top W B
{latex}

{latex}
Y'_{\text{opt}} = (A'^\top W A' + \lambda I)^{-1} (A'^\top W B)
{latex}

---

h3. Step 4 — Adjusted mid-price

The yield curve adjusted mid price \(m_{n,adj}\) of bond \(n\) is:

{latex}
A'_{n,adj} Y'_{\text{opt}} = B_{n,adj} = m_{n,adj} - c_n
{latex}

{latex}
m_{n,adj} = A'_{n,adj} Y'_{\text{opt}} + c_n
{latex}

where:

{latex}
A'_{n,adj} = [0, \ d_n \frac{\tau_2 - T_n}{\tau_2 - \tau_1}, \ d_n \frac{T_n - \tau_1}{\tau_2 - \tau_1}, \ 0, \dots, 0]
{latex}

assuming \(\tau_1 \leq T_n \leq \tau_2\).
