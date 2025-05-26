A function $f$ from a space $E$ to a space $F$ (typically normed vector spaces or metric spaces in this context) is continuous at a point $a \in E$ if, for every $\epsilon > 0$, there exists some $\delta > 0$ such that for all $x \in E$, if the distance between $x$ and $a$ is less than $\delta$, then the distance between $f(x)$ and $f(a)$ is less than $\epsilon$.

More formally, using norms: Let $E$ and $F$ be normed spaces. A function $f: E \to F$ is continuous at a point $a \in E$ if: For every $\epsilon > 0$, there exists a $\delta > 0$ such that for every $x \in E$, if $|x - a|_E < \delta$, then $|f(x) - f(a)|_F < \epsilon$.

A function is said to be continuous if it is continuous at every point in its domain.

Alternatively, a function $f$ is continuous at $a$ if the limit of $f(x)$ as $x$ approaches $a$ is equal to $f(a)$: $$ \lim\_{x \to a} f(x) = f(a) $$

Example 1: A Continuous Function
Function: $f(x) = 2x + 1$, where $f: \mathbb{R} \to \mathbb{R}$.

Proof of Continuity: We want to show that $f(x)$ is continuous at any arbitrary point $a \in \mathbb{R}$. According to the definition, for every $\epsilon > 0$, we need to find a $\delta > 0$ such that if $|x - a| < \delta$, then $|f(x) - f(a)| < \epsilon$.

Let's analyze $|f(x) - f(a)|$: $|f(x) - f(a)| = |(2x + 1) - (2a + 1)|$ $= |2x + 1 - 2a - 1|$ $= |2x - 2a|$ $= |2(x - a)|$ $= 2|x - a|$

We want $2|x - a| < \epsilon$. This implies $|x - a| < \frac{\epsilon}{2}$.

So, we can choose $\delta = \frac{\epsilon}{2}$. Now, let's verify. Assume $|x - a| < \delta$. Since $\delta = \frac{\epsilon}{2}$, we have $|x - a| < \frac{\epsilon}{2}$. Then, $2|x - a| < 2 \left(\frac{\epsilon}{2}\right)$. So, $2|x - a| < \epsilon$. This means $|f(x) - f(a)| < \epsilon$.

Thus, for any $a \in \mathbb{R}$ and any $\epsilon > 0$, if we choose $\delta = \frac{\epsilon}{2}$, then $|x - a| < \delta$ implies $|f(x) - f(a)| < \epsilon$. Therefore, $f(x) = 2x + 1$ is continuous for all $x \in \mathbb{R}$.

Example 2: A Discontinuous Function
Function: The Heaviside step function, $H(x)$, defined as: $H(x) = \begin{cases} 0 & \text{if } x < 0 \ 1 & \text{if } x \ge 0 \end{cases}$ where $H: \mathbb{R} \to \mathbb{R}$.

Proof of Discontinuity at $a = 0$: We want to show that $H(x)$ is not continuous at $a = 0$. To do this, we need to show that there exists an $\epsilon > 0$ such that for every $\delta > 0$, there is an $x$ such that $|x - 0| < \delta$ but $|H(x) - H(0)| \ge \epsilon$.

Let's choose $\epsilon = \frac{1}{2}$ (any value between 0 and 1 would work). We know $H(0) = 1$. So we need to show that for any $\delta > 0$, there's an $x$ with $|x| < \delta$ such that $|H(x) - 1| \ge \frac{1}{2}$.

Consider any $\delta > 0$. We can always choose an $x$ such that $-\delta < x < 0$. For example, let $x = -\frac{\delta}{2}$. For this $x$:

$|x - 0| = |-\frac{\delta}{2}| = \frac{\delta}{2} < \delta$. (The condition on $x$ is met)
Since $x < 0$, $H(x) = 0$.
Then, $|H(x) - H(0)| = |0 - 1| = |-1| = 1$.
Is $1 \ge \epsilon$? Since we chose $\epsilon = \frac{1}{2}$, yes, $1 \ge \frac{1}{2}$.

So, we found an $\epsilon = \frac{1}{2}$ such that for any $\delta > 0$, we can find an $x$ (e.g., $x = -\delta/2$) for which $|x - 0| < \delta$ but $|H(x) - H(0)| \ge \epsilon$. Therefore, $H(x)$ is not continuous at $x = 0$.

Example 3: Another Continuous Function (using the limit definition)
Function: $g(x) = x^2$, where $g: \mathbb{R} \to \mathbb{R}$.

Proof of Continuity at an arbitrary point $a \in \mathbb{R}$ using limits: We need to show that $\lim_{x \to a} g(x) = g(a)$. $g(a) = a^2$. We need to show $\lim_{x \to a} x^2 = a^2$.

$\lim_{x \to a} x^2 = \lim_{x \to a} (x \cdot x)$ Using the limit property that the limit of a product is the product of the limits (if they exist): $= (\lim_{x \to a} x) \cdot (\lim_{x \to a} x)$ $= a \cdot a$ $= a^2$ Since $\lim_{x \to a} g(x) = a^2 = g(a)$, the function $g(x) = x^2$ is continuous for all $x \in \mathbb{R}$.
