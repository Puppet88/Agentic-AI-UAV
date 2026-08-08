# Completion and Deadline Primitives

> Scope note: These are reusable domain primitives with alternatives and caveats. They are not an ordered formulation for a particular evaluation prompt.

## 1. Purpose

A task model should distinguish four questions:

1. In which slot does the task complete?
2. Has the task completed by a given slot?
3. Does the task complete by the end of the horizon?
4. Does the task complete before its deadline?

These questions require different variables.

## 2. Local notation

- $\mathcal{Q}$: task set; index $q$.
- $\mathcal{H}=\{0,1,\ldots,H-1\}$: slot set.
- $\Delta$: slot duration in seconds.
- $a_q$: arrival slot.
- $A_q=a_q\Delta$: arrival time in seconds.
- $D_q$: relative deadline in seconds.
- $\bar D_q=A_q+D_q$: absolute deadline in seconds.
- $\delta_{q,h}\in\{0,1\}$: task $q$ completes in slot $h$.
- $c_q\in\{0,1\}$: task $q$ completes by the horizon.
- $\chi_{q,h}\in\{0,1\}$: task $q$ has completed by the end of slot $h$.
- $z_q\in\{0,1\}$: task $q$ completes on time.
- $T_q\ge0$: modeled completion time in seconds.

## 3. Completion-slot indicator and single completion

### Modeling purpose

A task should have at most one completion slot.

$$
\sum_{h=a_q}^{H-1}\delta_{q,h}\le1, \quad \forall q.
$$

If every admitted task must complete, equality may be used together with an admission variable.

### Caveat

Without a single-completion condition, a task can be declared complete in several slots and receive duplicate reward.

## 4. Completion status

Define horizon completion status as:

$$
c_q = \sum_{h=a_q}^{H-1}\delta_{q,h}, \quad \forall q.
$$

### Interpretation

- $c_q=1$ means one completion slot is selected.
- $c_q=0$ means no completion occurs within the horizon.

### Applicability

Use this variable when completed-late tasks must be distinguished from unfinished tasks.

## 5. Cumulative completion indicator

For each slot $h\ge a_q$:

$$
\chi_{q,h} = \sum_{\ell=a_q}^{h}\delta_{q,\ell}.
$$

### Interpretation

$\chi_{q,h}=1$ from the completion slot onward. Before completion it is zero.

### Use

This variable is useful for cumulative service sufficiency and for preventing allocations after completion.

## 6. Completion-time encoding

Let $t_h^{\mathrm{end}}=(h+1)\Delta$ be the end time of slot $h$. Let $T^{\mathrm{pen}}\ge T^{\mathrm{hor}}$ be a finite convention for unfinished tasks.

A generic encoding is:

$$
T_q = \sum_{h=a_q}^{H-1}t_h^{\mathrm{end}}\delta_{q,h} + T^{\mathrm{pen}}(1-c_q).
$$

### Term-by-term interpretation

- The sum selects the end time of the unique completion slot.
- The second term assigns a defined penalty time when no completion slot is selected.

### Units

Every term is in seconds.

### Caveat

The unfinished-task time is a modeling convention, not a physical completion event. State it clearly when flow time is reported.

## 7. Absolute and relative deadlines

The relative deadline is $D_q$ seconds after arrival. The absolute deadline is:

$$
\bar D_q=A_q+D_q.
$$

Do not compare a completion time in seconds directly with a slot index.

## 8. Exact slot-based on-time indicator

When deadline feasibility can be determined from slot end times, define:

$$
z_q = \sum_{h:a_q\le h,\ t_h^{\mathrm{end}}\le\bar D_q} \delta_{q,h}.
$$

### Interpretation

Only completion slots that end no later than the deadline contribute to $z_q$.

### Benefit

This definition exactly distinguishes on-time and late completion without a Big-M constraint.

## 9. Implication or Big-M deadline form

An alternative is:

$$
T_q \le \bar D_q + M_q^{\mathrm{time}}(1-z_q).
$$

Also require:

$$
z_q\le c_q.
$$

### Interpretation

If $z_q=1$, completion time must be within the deadline. If $z_q=0$, the bound is relaxed.

### Caveat

This form alone does not force $z_q$ to become one when a task completes on time. A reward for $z_q$ or an exact equivalence relation is needed if that direction matters.

## 10. Completed-late and unfinished indicators

Define a late-completion indicator:

$$
\ell_q^{\mathrm{late}}=c_q-z_q.
$$

Define an unfinished indicator:

$$
\upsilon_q=1-c_q.
$$

Under valid completion and on-time logic:

- $z_q=1$ means completed on time;
- $\ell_q^{\mathrm{late}}=1$ means completed late;
- $\upsilon_q=1$ means unfinished by the horizon.

These categories are mutually exclusive when $z_q\le c_q$.

## 11. Completion linked to cumulative computing work

Let $S_{q,h}^{\mathrm{cpu}}$ be cumulative computing work delivered by slot $h$, measured in work-units. Let $W_q$ be task workload.

A completion claim should require:

$$
S_{q,h}^{\mathrm{cpu}} \ge W_q\chi_{q,h}.
$$

### Interpretation

When completion has been claimed by slot $h$, cumulative computing work must reach the workload. Before completion, the right side is zero.

## 12. Completion linked to cumulative data delivery

For a communication-dependent mode, let $S_{q,h}^{\mathrm{data}}$ be cumulative required data delivered by slot $h$, measured in bits.

$$
S_{q,h}^{\mathrm{data}} \ge L_q\chi_{q,h}.
$$

This constraint is separate from computing sufficiency because $L_q$ and $W_q$ have different units.

## 13. Tasks arriving at different slots

All completion and cumulative sums should start at $a_q$:

$$
\sum_{h=a_q}^{H-1}(\cdot).
$$

Allocations for $h<a_q$ should be zero. This can be enforced by indexing or by an arrival-availability parameter.

## 14. Flow-time calculation

For a completed task:

$$
\Phi_q=T_q-A_q.
$$

The unit is seconds.

For unfinished tasks, $\Phi_q$ depends on the selected penalty-time convention. An alternative is to compute flow time only for completed tasks and use a separate unfinished penalty.

## 15. Hard and soft deadline models

### Hard deadline

A mandatory task can be required to finish on time:

$$
z_q=1.
$$

This can make the model infeasible when resources are insufficient.

### Soft deadline

Allow $z_q=0$ and penalize misses or lateness in the objective. This preserves feasibility but allows service-level violations.

## 16. Completion does not imply deadline satisfaction

The following relation is generally valid:

$$
z_q\le c_q.
$$

The reverse relation is not generally valid. A task may be completed after its deadline.

Only define $c_q=z_q$ when the model deliberately treats late completion as equivalent to non-completion.

## 17. Common logical inconsistencies

- Defining $z_q$ as completion status and later describing it as on-time status.
- Allowing several completion slots.
- Claiming completion without enough compute service.
- Claiming remote completion without enough data delivery.
- Using a completion time that is undefined for unfinished tasks.
- Comparing seconds with slot indices.
- Treating completed-late and unfinished tasks as identical without stating it.
- Allowing service before arrival.
- Rewarding $c_q$ and $z_q$ as if both always represent the same event.
- Using a Big-M value without a valid time bound.

## 18. Variant guidance

- **Deadline-only evaluation:** use $z_q$ and omit late-completion value if late tasks have no benefit.
- **Completion-only evaluation:** use $c_q$ and remove deadline variables.
- **Completed-late tasks are useful:** retain $c_q$, $z_q$, and $\ell_q^{\mathrm{late}}$.
- **Hard deadlines:** require $z_q=1$ for mandatory tasks.
- **Soft deadlines:** penalize $1-z_q$ or tardiness.
- **Continuous time:** replace completion-slot binaries with a continuous finish time and logical completion status.
- **Non-preemptive jobs:** link completion time to start time and uninterrupted processing duration.
- **Stochastic tasks:** completion variables exist only for realized or scenario-specific tasks.
