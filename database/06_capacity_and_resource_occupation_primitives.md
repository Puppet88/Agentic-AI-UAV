# Capacity and Resource-Occupation Primitives

> Scope note: These are reusable domain primitives with alternatives and caveats. They are not an ordered formulation for a particular evaluation prompt.

## 1. Purpose

Capacity constraints prevent simultaneous allocations from exceeding a physical resource. Resource-occupation measures describe how heavily each resource is used. They are useful in objectives, reports, and ablation studies.

Capacity and occupation are related but not identical:

- A capacity constraint is a hard feasibility rule.
- An occupation measure is a derived ratio or cost.

## 2. Local notation

- $\mathcal{Q}$: task set.
- $\mathcal{H}$: slot set with $H=|\mathcal{H}|$.
- $\mathcal{D}$: device set.
- $\mathcal{V}$: UAV set.
- $d(q)$: source device of task $q$.
- $f_{q,h}^{\mathrm{loc}}$: local compute allocation in work-units per second.
- $f_{q,v,h}^{\mathrm{air}}$: UAV compute allocation in work-units per second.
- $f_{q,v,h}^{\mathrm{rem}}$: remote compute allocation in work-units per second.
- $F_d^{\mathrm{loc}}$, $F_v^{\mathrm{air}}$, $F^{\mathrm{rem}}$: positive compute capacities in work-units per second.
- $r_{q,v,h}^{\mathrm{up}}$, $r_{q,v,h}^{\mathrm{relay}}$: communication allocations in bits per second.
- $B_v^{\mathrm{up}}$, $B_v^{\mathrm{relay}}$: positive communication capacities in bits per second.

All allocation variables are nonnegative.

## 3. Aggregate local computing capacity

### Modeling purpose

A device may serve several of its own tasks in one slot, but the total rate cannot exceed its processor capacity.

$$
\sum_{q:d(q)=d}f_{q,h}^{\mathrm{loc}} \le F_d^{\mathrm{loc}}, \quad \forall d\in\mathcal{D},\ h\in\mathcal{H}.
$$

### Interpretation

The left side is the total local compute rate assigned in the slot. The right side is the available local rate.

### Units

Both sides are in work-units per second.

## 4. Aggregate UAV computing capacity

$$
\sum_{q\in\mathcal{Q}}f_{q,v,h}^{\mathrm{air}} \le F_v^{\mathrm{air}}, \quad \forall v\in\mathcal{V},\ h\in\mathcal{H}.
$$

This constraint limits concurrent onboard processing. It should be combined with execution-mode and service-window gates.

## 5. Aggregate remote computing capacity

For one shared remote processor:

$$
\sum_{v\in\mathcal{V}} \sum_{q\in\mathcal{Q}}f_{q,v,h}^{\mathrm{rem}} \le F^{\mathrm{rem}}, \quad \forall h\in\mathcal{H}.
$$

### Caveat

The vehicle index in $f_{q,v,h}^{\mathrm{rem}}$ may identify the relay path, not a distinct remote processor. Therefore, all such allocations should share the same remote capacity unless separate remote servers are explicitly modeled.

## 6. Uplink capacity

$$
\sum_{q\in\mathcal{Q}}r_{q,v,h}^{\mathrm{up}} \le B_v^{\mathrm{up}}, \quad \forall v,h.
$$

The left side is the total nominal uplink rate assigned through UAV $v$ in slot $h$.

## 7. Relay or backhaul capacity

$$
\sum_{q\in\mathcal{Q}}r_{q,v,h}^{\mathrm{relay}} \le B_v^{\mathrm{relay}}, \quad \forall v,h.
$$

### Applicability

Use separate uplink and relay constraints when the two hops use different radios, spectra, or bottlenecks.

Use a joint capacity only when both directions consume one shared physical resource. The joint rule should then be stated explicitly.

## 8. Processor utilization

For any processor $p$ with capacity $F_p>0$, define:

$$
u_{p,h}^{\mathrm{cpu}} = \frac{\sum_q f_{q,p,h}}{F_p}.
$$

### Interpretation

- The numerator is allocated compute rate.
- The denominator is available compute rate.
- The ratio is dimensionless.
- A valid capacity constraint gives $0\le u_{p,h}^{\mathrm{cpu}}\le1$.

### Zero-capacity safeguard

Do not define the ratio when $F_p=0$. Remove the unavailable processor or define its mode as disabled. A zero denominator must not be hidden by a small numerical constant.

## 9. Link utilization

For communication link $e$ with capacity $B_e>0$:

$$
u_{e,h}^{\mathrm{net}} = \frac{\sum_q r_{q,e,h}}{B_e}.
$$

This quantity is also dimensionless.

### Nominal versus effective rate

If capacity is defined for nominal allocation, use nominal rates in the utilization ratio. Effective delivered bits may use an additional efficiency factor. Do not mix nominal capacity with effective throughput without explaining the convention.

## 10. Average computing occupation

Let $\mathcal{P}^{+}$ contain only processors with positive capacity.

$$
C^{\mathrm{cpu,avg}} = \frac{1}{H|\mathcal{P}^{+}|} \sum_{h\in\mathcal{H}} \sum_{p\in\mathcal{P}^{+}} u_{p,h}^{\mathrm{cpu}}.
$$

### Interpretation

This is the mean fraction of processor capacity occupied over processors and slots.

### Applicability

Use it when all included processors should contribute equally after normalization.

If processors have different economic importance, use weights.

## 11. Average communication occupation

Let $\mathcal{L}^{+}$ contain links with positive capacity.

$$
C^{\mathrm{net,avg}} = \frac{1}{H|\mathcal{L}^{+}|} \sum_{h\in\mathcal{H}} \sum_{e\in\mathcal{L}^{+}} u_{e,h}^{\mathrm{net}}.
$$

This is dimensionless.

## 12. Weighted aggregate occupation

A combined resource cost can be defined as:

$$
C^{\mathrm{res}} = \beta_{\mathrm{cpu}}C^{\mathrm{cpu,avg}} + \beta_{\mathrm{net}}C^{\mathrm{net,avg}},
$$

where $\beta_{\mathrm{cpu}},\beta_{\mathrm{net}}\ge0$.

### Caveat

The weights should be reported. Otherwise, the relative importance of computing and communication occupation is not reproducible.

## 13. Raw allocation versus normalized occupation

Raw total compute allocation is:

$$
R^{\mathrm{cpu,raw}} = \sum_h\sum_p\sum_q f_{q,p,h}\Delta.
$$

Its unit is work-units.

Raw total communication allocation is:

$$
R^{\mathrm{net,raw}} = \sum_h\sum_e\sum_q r_{q,e,h}\Delta.
$$

Its unit is bits if $r$ is an effective throughput. These raw quantities should not be added directly because their units differ.

Normalized occupation is unitless and can be aggregated after the normalization rule is stated.

## 14. Peak-utilization alternative

A peak measure can control congestion:

$$
C^{\mathrm{cpu,peak}} \ge u_{p,h}^{\mathrm{cpu}}, \quad \forall p,h.
$$

Minimizing $C^{\mathrm{cpu,peak}}$ reduces the maximum processor utilization.

A similar variable can be defined for communication links.

## 15. Time-weighted occupation

If slot durations differ, let $\Delta_h$ be the duration of slot $h$.

$$
C^{\mathrm{cpu,time}} = \frac{1}{\sum_h\Delta_h} \sum_h\Delta_h \left( \frac{1}{|\mathcal{P}^{+}|} \sum_p u_{p,h}^{\mathrm{cpu}} \right).
$$

This avoids treating short and long slots as equally important.

## 16. Avoiding double counting

Common double-counting cases include:

- counting the same remote compute allocation once per relay and again as a shared-cloud allocation;
- counting nominal communication rate and effective delivered rate as two separate resource costs;
- adding per-link utilization and a total-network utilization that contains the same links;
- penalizing both resource allocation and energy generated from the same allocation without explaining the two distinct goals.

## 17. Common errors

- Confusing a capacity constraint with an objective penalty.
- Adding work-units per second to bits per second.
- Dividing by zero capacity.
- Omitting nonnegativity of allocations.
- Using one capacity for local, UAV, and remote processors without a shared-resource assumption.
- Treating uplink and relay as one link without justification.
- Averaging over unavailable resources.
- Counting service before task arrival.
- Using normalized occupation but describing it as physical energy.

## 18. Variant guidance

- **No resource-occupation objective:** keep all capacity constraints and remove occupation costs.
- **Peak-aware scheduling:** add peak-utilization variables.
- **Heterogeneous UAVs:** index all capacities by vehicle.
- **Fixed allocations:** treat rates as parameters and report utilization only.
- **One shared radio:** replace separate link capacities with a joint scheduling constraint.
- **Orthogonal channels:** retain separate capacity constraints.
- **Continuous time:** replace slot sums with time integrals or event intervals.
