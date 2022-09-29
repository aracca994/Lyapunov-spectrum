# Lyapunov-spectrum

Jacobian-free implementation for the computation of the first m largest lyapunov exponents and Kaplan-Yorke dimension of a dynamical system. 
The algorithm requires only the right hand side of the nonlinear governing equations.

The exponents are computed following the orthonormalization algorithm of Benettin et al. (https://doi.org/10.1007/BF02128236). Kaplan-Yorke dimension (https://doi.org/10.1007/BFb0064319).

## Algorithm

In this section, we describe the algorithm used to compute the first $m$ largest Lyapunov exponents of the system. The algorithm requires the integration of the governing equations $m+1$ times, but does not require the computation of the Jacobian of the system.
We consider a nonlinear autonomous dynamical system in the form of 
\begin{equation}
\label{eq:dyn_syst}
    \mathbf{\dot{q}} = \mathbf{f(q)}
\end{equation}
where $\mathbf{q}$ is the system's state and $\mathbf{f}$ is a nonlinear operator. In chaotic solutions, the norm of a perturbation $\mathbf{y}_i$, such that $\mathbf{\hat{q}}_i = \mathbf{\overline{q}} + \mathbf{y}_i$ with $\mathbf{y}_i \ll 1$, grows in time until nonlinear saturation. For small enough times, $t_1 - t_0$, so that we avoid nonlinear saturation, the evolution of  $\mathbf{y}_i$ can be computed as
\begin{equation}
\label{eq: pert}
    \mathbf{y}_i (t_1) = \mathbf{\overline{q}}(t_1) - \mathbf{\hat{q}}_i(t_1),
\end{equation}
where both elements in the right-hand side are computed by solving \eqref{eq:dyn_syst} with initial conditions equal to $\mathbf{\overline{q}}(t_0)$ and $\mathbf{\overline{q}}(t_0)+\mathbf{y}_i(t_0)$, respectively. The average exponential growth rate for the perturbation $\mathbf{y}_i$ between $t_0$ and $t_1$ is
\begin{equation}
    \lambda = \frac{1}{t_1 - t_0}\ln\left(\frac{||\mathbf{y}(t_1)||}{||\mathbf{y}(t_0)||}\right),
\end{equation}
where $||\cdot||$ indicates the $L_2$ norm.
For long enough times, $t_1 \to \infty$ , any perturbation evolves with the same $\Lambda_1$, the dominant Lyapunov exponent 
\begin{equation}
    \Lambda_1 = \lim_{t_1\to\infty}\frac{1}{t_1-t_0}\ln\left(\frac{||\mathbf{y}(t_1)||}{||\mathbf{y}(t_0)||}\right),
\end{equation}
as the component along the direction with maximum growth becomes dominant for sufficiently long times. However, due to saturation of the nonlinear equations (or instability of the linearized equations) for finite times, computing $\Lambda$ is not straightforward.

To compute the growth along the $m$ most unstable directions for long times, \citet{benettin1980lyapunov} proposed to periodically orthonormalize the evolution of the subspace spanned by $m$ different perturbations. The algorithm works as follows. Every $t_{\mathrm{o}}$, we orthonormalize the $m$ perturbations and compute the future evolution of the orthonormalized basis:
\begin{align}
    \mathbf{\Tilde{y}}_1(t) &= \frac{\mathbf{y}_1(t)}{||\mathbf{y}_1(t)||} \quad \mathrm{where} \quad \mathbf{y}_1(t-t_{\mathrm{o}}) = \epsilon \mathbf{\Tilde{y}}_1(t-t_{\mathrm{o}}), \nonumber \\ 
    \vdots &  \nonumber \\
    \mathbf{\Tilde{y}}_i (t) &= \frac{\mathbf{y'}_i(t)}{||\mathbf{y'}_i(t)||}; \quad \mathbf{y'}_i = \mathbf{y}_i - \sum_{j=1}^{i-1} (\mathbf{y}_i^T\mathbf{\Tilde{y}}_j) \mathbf{\Tilde{y}}_j \quad \mathrm{where} \quad \mathbf{y}_i(t-t_{\mathrm{o}}) = \epsilon\mathbf{\Tilde{y}}_i(t-t_{\mathrm{o}}), \nonumber \\
    \vdots &  \nonumber \\
    \mathbf{\Tilde{y}}_m (t) &= \cdots 
\end{align}
where $\epsilon \ll 1$ is selected in order for initial condition to be infinitesimal and $\mathbf{y'}_i(t)$ is computed using \eqref{eq: pert}.
At each orthonormalization, we store the average exponential growths, so that for the $i$-th direction at the $k$-th orthonormalization we have 
\begin{equation}
    \lambda_i^{(k)} = \frac{1}{t_{\mathrm{o}}}\ln\left(\frac{||\mathbf{y}_i(t)||}{||\mathbf{y}_i(t-t_{\mathrm{o}})||}\right)
\end{equation}
where $||\mathbf{y}_1(t-t_{\mathrm{o}})||=\epsilon$. After $N_\mathrm{o}$ orthonormalizations, the Lyapunov exponents are the average of the stored exponential growths
\begin{equation}
    \Lambda_i = \frac{1}{N_\mathrm{o}}\sum_{k=1}^{N_\mathrm{o}}\lambda_i^{(k)}.
\end{equation}


