# Objective-Design Primitives

> Scope note: These are reusable domain primitives with alternatives and caveats. They are not an ordered formulation for a particular evaluation prompt.
## Retrieval guidance

Build an objective from independently selected benefit and cost components. Potential benefits include collected-item value, admitted-task value, completed-task value, and on-time service value. Potential costs include rejection, deadline miss, tardiness, flow time, route length, normalized processor occupation, normalized link occupation, and energy. The user description determines which components are included. Retrieve each selected component separately and normalize incompatible units. This topic map is not a prescribed combined objective.


## 1. Modeling purpose

A joint logistics-computation model usually has several competing goals. It may reward physical collection and timely task service. It may penalize missed deadlines, delay, resource use, or energy consumption.

The objective should state which outcomes are desirable and which outcomes are costly. It should also avoid adding quantities with incompatible units without normalization or scaling.

## 2. Criterion-indexed scalarization

Let $\mathcal{B}$ be the set of benefit criteria and $\mathcal{C}$ the set of cost criteria. For each benefit $b\in\mathcal{B}$, let $U_b(x)$ be a normalized utility. For each cost $c\in\mathcal{C}$, let $C_c(x)$ be a normalized cost. A generic scalarization is:

$$
\max_{x\in\mathcal{X}} \left[ \sum_{b\in\mathcal{B}}\lambda_b U_b(x) - \sum_{c\in\mathcal{C}}\rho_c C_c(x) \right].
$$

Here $x$ denotes all decisions, $\mathcal{X}$ is the feasible set, and $\lambda_b,\rho_c\ge0$ are nonnegative weights. This expression does not prescribe which criteria must be used. The user request determines the membership of $\mathcal{B}$ and $\mathcal{C}$.

### Interpretation

- A positive weighted utility rewards a desirable outcome.
- A negative weighted cost discourages an undesirable outcome.
- A hard safety or physical requirement belongs in $\mathcal{X}$, not only in the objective.
- Terms with different physical units should be normalized or converted to comparable score units.

### Applicability

Weighted scalarization is suitable when trade-offs are allowed. Use lexicographic optimization, $\epsilon$-constraints, or hard requirements when one criterion has strict priority.

## 3. Logistics collection utility

Let $\mathcal{S}$ be the station set. Let $v_s^{\mathrm{col}}\ge 0$ be the value of serving station $s$. Let $\kappa_s\in\{0,1\}$ indicate that collection credit is earned.

A generic logistics benefit is:

$$
U^{\mathrm{log}} = \sum_{s\in\mathcal{S}} v_s^{\mathrm{col}}\kappa_s.
$$

### Interpretation

- $v_s^{\mathrm{col}}$ may be a monetary value, priority score, or normalized reward.
- $\kappa_s=1$ contributes the station value.
- $\kappa_s=0$ contributes nothing.

The model must state when $\kappa_s$ can equal one. Possible conventions include successful station service or successful return of the item to the depot.

### Units

If $v_s^{\mathrm{col}}$ is measured in monetary units, then $U^{\mathrm{log}}$ has monetary units. If the objective uses normalized terms, divide by a positive reference value such as $\sum_s v_s^{\mathrm{col}}$.

### Caveat

Do not reward assignment alone when collection requires an actual route and depot return. Link the credit variable to the required physical events.

## 4. On-time task-completion utility

Let $\mathcal{Q}$ be the task set. Let $z_q\in\{0,1\}$ indicate that task $q$ is completed by its deadline. Let $\omega_q\ge 0$ be an optional task-priority weight.

A generic quality-of-service benefit is:

$$
U^{\mathrm{qos}} = \sum_{q\in\mathcal{Q}} \omega_q z_q.
$$

If all tasks are equally important, use $\omega_q=1$.

### Interpretation

Each on-time task adds its priority weight. A late or unfinished task adds zero.

### Caveat

An on-time indicator is not the same as a completion indicator. A task may finish after its deadline. Such a task is completed but not on time.

## 5. Deadline-miss cost

A simple miss cost is:

$$
C^{\mathrm{miss}} = \sum_{q\in\mathcal{Q}} \mu_q(1-z_q),
$$

where $\mu_q\ge 0$ is a miss-severity weight.

### Interpretation

- If $z_q=1$, task $q$ contributes zero miss cost.
- If $z_q=0$, task $q$ contributes $\mu_q$.

### Compatibility with completion reward

The completion reward and miss penalty can coexist. However, they partly duplicate the same incentive. For equal weights, maximizing $z_q$ and minimizing $1-z_q$ produce similar ranking effects.

Use both only when there is a clear reason. For example, the reward may represent service value, while the miss penalty represents a stronger service-level violation.

### Caveat

A miss indicator may include both late and unfinished tasks. If the application distinguishes them, define separate late and unfinished costs.

## 6. Flow-time or delay cost

Let $A_q$ be task arrival time in seconds. Let $T_q$ be its modeled completion time in seconds.

A generic flow time is:

$$
\Phi_q=T_q-A_q.
$$

A weighted aggregate delay cost is:

$$
C^{\mathrm{delay}} = \sum_{q\in\mathcal{Q}} \vartheta_q^{\mathrm{delay}}\Phi_q,
$$

where $\vartheta_q^{\mathrm{delay}}\ge 0$ is a delay weight.

### Interpretation

- $T_q$ is the time at which processing is considered complete.
- $A_q$ is the time at which the task becomes available.
- Their difference is the time spent in the system.

### Units

$\Phi_q$ is measured in seconds. The coefficient $\vartheta_q^{\mathrm{delay}}$ must convert seconds into objective score if the other terms use different units.

### Unfinished-task convention

An unfinished task has no physical completion time. The model must define a convention, such as:

$$
T_q=T^{\mathrm{pen}} \quad\text{when task }q\text{ is unfinished},
$$

where $T^{\mathrm{pen}}$ is a finite penalty time no smaller than the horizon. This keeps flow-time accounting well-defined.

### Caveat

Do not use an undefined completion time for unfinished tasks. Do not subtract an arrival slot index from a completion time in seconds.

## 7. Normalized computing occupation

Let $\mathcal{P}$ be the processor set. Let $F_p>0$ be processor $p$ capacity in work-units per second. Let $f_{q,p,h}\ge 0$ be the compute rate allocated to task $q$ in slot $h$.

The utilization of processor $p$ in slot $h$ is:

$$
u_{p,h}^{\mathrm{cpu}} = \frac{\sum_{q}f_{q,p,h}}{F_p}.
$$

It is dimensionless. Under a valid capacity constraint, it lies in $[0,1]$.

A generic average computing occupation is:

$$
\widehat C^{\mathrm{cpu}} = \frac{1}{H|\mathcal{P}|} \sum_{h\in\mathcal{H}} \sum_{p\in\mathcal{P}} u_{p,h}^{\mathrm{cpu}}.
$$

### Interpretation

The numerator is the allocated compute rate. The denominator is the corresponding capacity. The ratio measures the fraction of that processor occupied in the slot.

## 8. Normalized communication occupation

Let $\mathcal{L}$ be the link set. Let $B_e>0$ be the nominal capacity of link $e$ in bits per second. Let $r_{q,e,h}\ge 0$ be the allocated rate.

The link utilization is:

$$
u_{e,h}^{\mathrm{net}} = \frac{\sum_q r_{q,e,h}}{B_e}.
$$

A generic average communication occupation is:

$$
\widehat C^{\mathrm{net}} = \frac{1}{H|\mathcal{L}|} \sum_{h\in\mathcal{H}} \sum_{e\in\mathcal{L}} u_{e,h}^{\mathrm{net}}.
$$

The total normalized resource cost can be formed as:

$$
\widehat C^{\mathrm{res}} = \beta_{\mathrm{cpu}}\widehat C^{\mathrm{cpu}} + \beta_{\mathrm{net}}\widehat C^{\mathrm{net}},
$$

where $\beta_{\mathrm{cpu}},\beta_{\mathrm{net}}\ge 0$.

### Caveats

- Do not add raw work-units per second to raw bits per second.
- Do not divide by a zero capacity.
- Do not count the same allocation in two resource groups unless both physical resources are truly occupied.
- If an allocated rate already represents utilization, do not divide by capacity again.

## 9. Optional energy cost

Let $E_v^{\mathrm{use}}$ be mission energy used by vehicle $v$ in joules. Let $E_v^{\max}>0$ be its energy budget.

A normalized energy cost is:

$$
\widehat C^{\mathrm{energy}} = \frac{1}{|\mathcal{V}|} \sum_{v\in\mathcal{V}} \frac{E_v^{\mathrm{use}}}{E_v^{\max}}.
$$

This term is unitless.

### Applicability

Use it when energy efficiency is an optimization preference. If energy is only a feasibility limit, retain the budget constraint and omit the energy objective term.

### Caveat

Do not penalize energy twice without justification. A hard energy budget and a soft energy cost can coexist, but the paper should state why both are needed.

## 10. Normalization and weight selection

A common min-max normalization for a nonnegative criterion $X$ is:

$$
\widehat X=\frac{X}{X^{\mathrm{ref}}},
$$

where $X^{\mathrm{ref}}>0$ is a meaningful upper bound, maximum possible value, or baseline scale.

Weights can be chosen by:

- policy priorities;
- sensitivity analysis;
- expert elicitation;
- Pareto-front exploration;
- normalization followed by equal weights;
- lexicographic priority rules.

The chosen values should be reported. A claim of reproducibility requires all coefficients to be specified.

## 11. Equivalent minimization form

Let $J(x)$ denote any selected maximization objective assembled from applicable benefit and cost criteria. An equivalent minimization is obtained by changing the sign:

$$
\min_{x\in\mathcal{X}} -J(x).
$$

Using the criterion-indexed notation from Section 2, the same relation can be written as:

$$
\min_{x\in\mathcal{X}} \left[\sum_{c\in\mathcal{C}}\rho_c C_c(x)-\sum_{b\in\mathcal{B}}\lambda_b U_b(x)\right].
$$

The feasible set is unchanged. Only the objective sign convention changes. This generic conversion does not prescribe a fixed list or ordering of criteria.

## 12. Objective-term compatibility checklist

Before combining terms, check:

- Every benefit has a positive sign in a maximization model.
- Every cost has a negative sign in a maximization model.
- All weights are nonnegative.
- Every term is defined for unfinished tasks.
- Different physical units are normalized or converted.
- Completion is not rewarded multiple times without justification.
- Deadline miss and late completion are distinguished when needed.
- Resource occupation is not confused with physical energy.
- Hard constraints are not replaced by weak penalties.

## 13. Variant guidance

- **Deadline-only objective:** retain $U^{\mathrm{qos}}$ or $C^{\mathrm{miss}}$. Remove collection, flow-time, and resource terms if they are outside scope.
- **Routing-only:** retain logistics benefit and optional travel or energy cost. Remove task-service terms.
- **MEC-only with fixed windows:** remove collection benefit. Retain task and resource terms.
- **No communication-resource penalty:** set $\beta_{\mathrm{net}}=0$ or omit that component.
- **No energy preference:** omit $\widehat C^{\mathrm{energy}}$ but keep the energy budget if required.
- **Hard deadlines:** require on-time completion as a constraint for mandatory tasks. A miss cost is then unnecessary.
- **Soft deadlines:** keep a late or miss penalty.
- **Lexicographic optimization:** optimize the highest-priority criterion first, fix its achieved level, and then optimize lower-priority criteria.
