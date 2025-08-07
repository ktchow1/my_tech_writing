# Piecewise Linear Regression for Yield Curve Fitting

Given a set of bond data currently tradable in the market:

$$
\{T_n, b_n, a_n, c_n, d_n\}, \quad \text{for all } n \in \{1, \dots, N\}
$$

Where:
- $T_n$ is bond maturity  
- $b_n$ is current best bid of bond $n$  
- $a_n$ is current best ask of bond $n$  
- $c_n$ is previous close price of bond $n$  
- $d_n$ is DV01 of bond $n$  

---

The objective is to fit the yield curve $y(\tau\,|\,y_0, y_1, \dots, y_M)$ with $\tau$ being any tenor point on the yield curve.  
The curve is parameterized by yields at a set of predefined time points:

$$
\{y_m\}, \quad m \in \{0, 1, \dots, M\}
$$

This is a piecewise linear model with $M$ control points. Common control points are:

$$
\tau = 0,\ 1\text{D},\ 1\text{M},\ 3\text{M},\ 6\text{M},\ 1\text{Y}, \dots
$$

---

### Convert Bond Prices to Yields

Using methods like bisection or Newton-Raphson on the IRR equation. Alternatively, use linear approximation from Taylor series (if pivot is close). Taking previous close as pivot:

$$
m_n = \frac{b_n + a_n}{2} = c_n + d_n y_n
$$

Where:
- $\{y_n\},\ n \in [1, N]$ are observations/data points  
- $\{y_m\},\ m \in [0, M]$ are parameters/control points  
- $y(\tau\,|\dots)$ is the regression output at $\tau$

---


### Objective: Minimize Weighted Price Error

Minimize:

$$
\min_{\{y_m\}} \sum_n w_n \left( c_n + d_n y_n - \frac{b_n + a_n}{2} \right)^2
$$

With piecewise interpolation:

$$
\min_{\{y_m\}} \sum_n w_n \left( c_n + d_n \left[(1 - \mu_{T_n}) y_m + \mu_{T_n} y_{m+1} \right] - \frac{b_n + a_n}{2} \right)^2 \tag{1}
$$

Where $T_n$ lies between two tenor points $y_m \leq y(T_n) \leq y_{m+1}$, and:

$$
w_n = \frac{1}{|a_n - b_n|}, \quad \mu_{T_n} = \frac{T_n - \tau_m}{\tau_{m+1} - \tau_m}
$$

---

### Matrix Formulation

Rewriting Equation (1) as:

$$
\min_Y (AY - B)^T W (AY - B) \tag{2}
$$

Where:
- $A$ is the design matrix based on interpolation weights
- $Y$ is the vector of control points ($y_0, \dots, y_M$)
- $B$ is the vector of $(b_n + a_n)/2 - c_n$
- $W$ is a diagonal matrix with $w_n$ entries

---

### Linear Transformation

To transform into delta yields:

$$
\min_{Y'} (A' Y' - B)^T W (A' Y' - B) \tag{3}
$$

Where $Y = T Y'$ and $A' = A T$ with:

$$
T =
\begin{bmatrix}
1 & 0 & 0 & \dots & 0 \\
-1 & 1 & 0 & \dots & 0 \\
0 & -1 & 1 & \dots & 0 \\
\vdots & \vdots & \vdots & \ddots & 0 \\
0 & 0 & 0 & \dots & 1
\end{bmatrix}
$$

---

### Regularization

Adding a regularization term:

$$
J(Y') = (A' Y' - B)^T W (A' Y' - B) + \lambda {Y'}^T Y' \tag{4}
$$

Setting gradient to zero:

$$
A'^T W A' Y' + \lambda Y' = A'^T W B
$$

So the solution is:

$$
Y' = (A'^T W A' + \lambda I)^{-1} A'^T W B
$$
# Piecewise Linear Regression for Yield Curve Fitting

Given a set of bond data currently tradable in the market:

$$
\{T_n, b_n, a_n, c_n, d_n\}, \quad \text{for all } n \in \{1, \dots, N\}
$$

Where:
- $T_n$ is bond maturity  
- $b_n$ is current best bid of bond $n$  
- $a_n$ is current best ask of bond $n$  
- $c_n$ is previous close price of bond $n$  
- $d_n$ is DV01 of bond $n$  

---

The objective is to fit the yield curve $y(\tau\,|\,y_0, y_1, \dots, y_M)$ with $\tau$ being any tenor point on the yield curve.  
The curve is parameterized by yields at a set of predefined time points:

$$
\{y_m\}, \quad m \in \{0, 1, \dots, M\}
$$

This is a piecewise linear model with $M$ control points. Common control points are:

$$
\tau = 0,\ 1\text{D},\ 1\text{M},\ 3\text{M},\ 6\text{M},\ 1\text{Y}, \dots
$$

---

### Convert Bond Prices to Yields

Using methods like bisection or Newton-Raphson on the IRR equation. Alternatively, use linear approximation from Taylor series (if pivot is close). Taking previous close as pivot:

$$
m_n = \frac{b_n + a_n}{2} = c_n + d_n y_n
$$

Where:
- $\{y_n\},\ n \in [1, N]$ are observations/data points  
- $\{y_m\},\ m \in [0, M]$ are parameters/control points  
- $y(\tau\,|\dots)$ is the regression output at $\tau$

---

### Objective: Minimize Weighted Price Error

Minimize:

$$
\min_{\{y_m\}} \sum_n w_n \left( c_n + d_n y_n - \frac{b_n + a_n}{2} \right)^2
$$

Or with piecewise linear interpolation:

$$
\min_{\{y_m\}} \sum_n w_n \left( c_n + d_n \left[ (1 - \mu_n) y_{m} + \mu_n y_{m+1} \right] - \frac{b_n + a_n}{2} \right)^2 \tag{1}
$$

Where:
- $\mu_n$ is the interpolation weight  
- $w_n$ is inverse of spread  

---

### Matrix Formulation

Define:

$$
\min (AY - B)^T W (AY - B) \tag{2}
$$

Where:
- $A$ is $N \times (M+1)$ design matrix  
- $Y$ is the $(M+1) \times 1$ parameter vector  
- $B$ is the $N \times 1$ output vector  
- $W$ is a diagonal matrix of weights

---

### Linear Transformation on Parameter Space

To allow long-term bonds to impact short-term points:

$$
Y = TY' \quad \text{(e.g., delta yields)}
$$

Then:

$$
\min (ATY' - B)^T W (ATY' - B) = \min (A'Y' - B)^T W (A'Y' - B) \tag{3}
$$

Where $T$ is a lower triangular matrix with 1 on the diagonal and -1 below it.

---

### Regularization

Introduce penalty term $\lambda$:

$$
J(Y') = (A'Y' - B)^T W (A'Y' - B) + \lambda Y'^T Y' \tag{4}
$$

To solve:

$$
\nabla J(Y') = 2A'^T W A' Y' - 2A'^T W B + 2\lambda Y' = 0
$$

Solving:

$$
Y' = (A'^T W A' + \lambda I)^{-1} A'^T W B
$$

---

### Reconstruct Mid-Price from Fitted Yield Curve

Adjusted mid-price:

$$
m_{n, \text{adj}} = A'_n Y' + c_n
$$

Assuming $\tau_m \leq T_n \leq \tau_{m+1}$ and $A'_n = [0\ d_n(1 - \mu_n)\ d_n \mu_n\ 0\ \dots\ 0]$
