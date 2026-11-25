# Mathematical Model of Gen-LM

This document formalizes the architecture of the dynamically growing concept graph underlying the **Gen-LM** project. The model describes the processes of abstraction formation (merges), the birth of new nodes (conflicts), and activation decay.

## 1. State Space

The system at time $t$ is defined by the triple:

$$
\mathcal{S}_t = (V_t, E_t, s_t)
$$

where:
- $V_t$ is a finite set of nodes (concepts);
- $E_t \subseteq V_t \times V_t$ is a set of directed edges (DAG);
- $s_t : V_t \to \mathcal{X} \times \mathbb{N} \times \{0,1\} \times \mathbb{N}$ is the state function.

The state of a node $v \in V_t$:

$$
s_t(v) = \big(x_v,\ e_v,\ w_v,\ \ell_v\big)
$$

where:
- $x_v \in \mathbb{R}^d$ is the **meaning vector** (embedding, $d = 384$ for MiniLM);
- $e_v \in \mathbb{N}$ is the **abstraction level** (energy);
- $w_v \in \{0, 1\}$ is the **freeze flag** ($w_v = 1$ means the node was "used" in a merge);
- $\ell_v \in \mathbb{N}$ is the **component level** (model hierarchy).

Edges have a weight assigned:

$$
\omega_t : E_t \to \mathbb{R}_{>0}
$$

Similarity metric (cosine similarity):

$$
\sigma(x_u, x_v) = \frac{\langle x_u, x_v \rangle}{\|x_u\| \cdot \|x_v\|} \in [-1, 1]
$$

---

## 2. Dynamics: Two Time Scales

The system has **two time scales**:
- **Fast**: node activation, wave decay ($\Delta t = 1$);
- **Slow**: topology changes (merges, spawns, shatters).

### 2.1 Activation Function

Let $a_t : V_t \to \mathbb{R}_{\geq 0}$ be the node activation. The dynamics:

$$
a_{t+1}(v) = \underbrace{\alpha \cdot a_t(v)}_{\text{decay}} + \underbrace{\gamma \sum_{u \in N^-(v)} \frac{\omega(u,v)}{\deg^+(u)} a_t(u)}_{\text{wave propagation}}
$$

where:
- $\alpha \in (0,1)$ is the forgetting factor;
- $\gamma \in (0,1]$ is the transfer coefficient;
- $N^-(v)$ are the incoming neighbors of $v$;
- $\deg^+(u)$ is the out-degree of $u$.

This is **degree normalization** (similar to PageRank): hub nodes transfer less energy to each neighbor.

### 2.2 External Impulse

When text $T$ with embedding $x_T$ is input:

$$
a_t(v) \leftarrow a_t(v) + A_0 \cdot \mathbb{1}\big[\sigma(x_v, x_T) \geq \theta_{\text{inp}}\big]
$$

where $A_0 = 100$ is the impulse strength, and $\theta_{\text{inp}}$ is the hit threshold.

---

## 3. Topology Operations

### 3.1 Merge

Let $u, v \in V_t$. A merge is **allowed** if:

$$
w_u = w_v = 0 \quad \wedge \quad e_u = e_v \quad \wedge \quad \sigma(x_u, x_v) \geq \theta_{\text{merge}}
$$

Then a new node $w$ is born:

$$
\begin{cases}
x_w = \dfrac{x_u + x_v}{2} \\
e_w = e_u + 1 \\
\ell_w = \max(\ell_u, \ell_v) \\
w_w = 0
\end{cases}
$$

Simultaneously:

$$
w_u \leftarrow 1, \quad w_v \leftarrow 1
$$

**Edge inheritance** — from the "strongest" branch:

$$
\text{parent}(w) = \arg\max_{z \in \{u,v\}} \text{chg}(z)
$$

where $\text{chg}(z)$ is the change counter of node $z$.

**Monotonicity**: parents $u, v$ remain in $V_t$ (not deleted), so:

$$
|V_{t+1}| = |V_t| + 1
$$

### 3.2 Spawn (Conflict / Growth)

If a node $p$ has two children $u, v$ such that:

$$
x_u \neq x_v \quad \wedge \quad \text{chg}(u) > 0 \quad \wedge \quad \text{chg}(v) > 0 \quad \wedge \quad (u, v) \notin \text{CW}
$$

($\text{CW}$ is the set of previously conflicted pairs), then a node $z$ is born:

$$
\begin{cases}
x_z = x_u \oplus x_v \ \ (\text{component-wise XOR for bits}) \\
e_z = 0 \\
\ell_z = \max(\ell_u, \ell_v) \\
w_z = 0 \\
\text{chg}(z) = \text{chg}(u) + \text{chg}(v)
\end{cases}
$$

### 3.3 Shatter / Descent

If $w_v = 1$ and $e_v > 0$, the shatter operation applies:

$$
\text{Sh}(v) \to (v_1, v_2)
$$

where:

$$
e_{v_1} = e_{v_2} = e_v - 1, \quad w_{v_i} = 0, \quad x_{v_1}, x_{v_2} \text{ — split of } x_v
$$

This operation ensures **reversibility** (connection to the Collatz conjecture: $n/2$ as a compression analog).

---

## 4. System Invariants

**I1. DAG property.** $\nexists$ cycles in $E_t$.  
Corollary: the reachability relation is a partial order.

**I2. Growth monotonicity.** For any transition $\mathcal{S}_t \to \mathcal{S}_{t+1}$:

$$
|V_{t+1}| \geq |V_t| \quad \wedge \quad \sum_{v} e_v(t+1) \geq \sum_{v} e_v(t)
$$

**I3. Energy monotonicity along edges.** For all $(u, v) \in E_t$:

$$
e_u \leq e_v
$$

**I4. Flag $W$.** If $w_v = 1$, then $v$ no longer participates in merges:

$$
w_v = 1 \implies \forall t' > t : \nexists u, \ v \in \text{Merge}(t')
$$

**I5. No collapse.** If $|V_t| \geq 2$, there is no $t^*$ such that all nodes have the same state and the same $e$.

---

## 5. Asymptotic Properties

### 5.1 Growth Slowing (D4)

Let $M_k$ be the number of merges at energy level $k$. A merge at level $k+1$ requires **two independent merges** at level $k$ yielding the **same** vector $x$:

$$
\mathbb{P}\big(\text{merge}_{k+1}\big) \sim \mathbb{P}\big(\text{merge}_k\big)^2
$$

Therefore, as $k \to \infty$:

$$
M_k \to 0 \quad \text{(geometrically fast)}
$$

This provides **asymptotic slowing** without an external "ceiling".

### 5.2 Arrow of Time

Let $T_w(t)$ be the set of nodes with $e > 0$. Then:

$$
T_w \text{ grows monotonically by inclusion}
$$

This is the **arrow of time**: integration (merge) is irreversible without shattering.

---

## 6. Connection to Known Mathematical Structures

| Our Model | Mathematical Analog |
|---|---|
| $V_t$ with $\sigma$ | metric space with cosine metric |
| $(V_t, E_t)$ DAG | partially ordered set (poset) |
| Merge | operation $\otimes$ in a **tensor category** |
| Level $e$ | **gradation** (graded structure) |
| Slowing $M_k$ | **branching process** in a critical regime |
| Shatter $\text{Sh}$ | gradient inversion (analog of $n/2$ in Collatz) |
