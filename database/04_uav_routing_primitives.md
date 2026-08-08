# UAV Routing Formulation Primitives

> Scope note: These are reusable domain primitives with alternatives and caveats. They are not an ordered formulation for a particular evaluation prompt.
## Retrieval guidance

Routing requests should be decomposed into subtopics: station assignment; service or collection credit; incoming and outgoing arc consistency; depot activation and return; unused vehicles; self-loop exclusion; payload and route-distance limits; start, arrival, departure, return, travel-time, service-duration, and mission-horizon logic; and one connectivity or subtour-elimination mechanism. Alternatives include time-propagation, order variables, cut sets, and commodity flow. Retrieve the relevant subsection for each requested subtopic rather than treating the file as an ordered route model.


## 1. Local notation for routing units

This file uses the following generic notation.

- $\mathcal{V}$: vehicle set; index $v$.
- $\mathcal{S}$: service-station set; index $s$.
- $o$: depot node.
- $\mathcal{N}=\{o\}\cup\mathcal{S}$: routing nodes; indices $i,j$.
- $y_{v,s}\in\{0,1\}$: station $s$ is assigned to vehicle $v$.
- $x_{v,i,j}\in\{0,1\}$: vehicle $v$ uses arc $i\rightarrow j$.
- $a_v^{\mathrm{use}}\in\{0,1\}$: vehicle $v$ is used.
- $\kappa_s\in\{0,1\}$: station service or collection receives credit.
- $\ell_{ij}\ge 0$: distance in meters.
- $w_s\ge 0$: station payload in kilograms.
- $Q_v\ge 0$: vehicle payload capacity in kilograms.
- $\Lambda_v^{\max}\ge 0$: distance budget in meters.
- $\bar v_v>0$: speed in meters per second.
- $\sigma_s\ge 0$: minimum service duration in seconds.
- $t_{v,s}^{\mathrm{arr}},t_{v,s}^{\mathrm{dep}}\ge 0$: arrival and departure times in seconds.
- $t_v^{\mathrm{start}},t_v^{\mathrm{return}}\ge 0$: depot start and return times in seconds.
- $T^{\mathrm{hor}}>0$: mission horizon in seconds.

## 2. Station assignment

### Modeling purpose

The assignment relation limits how many vehicles are responsible for a station.

A common at-most-one rule is:

$$
\sum_{v\in\mathcal{V}}y_{v,s}\le 1, \quad \forall s\in\mathcal{S}.
$$

### Interpretation

The left side counts assigned vehicles. The right side allows zero or one assignment.

### Applicability

Use this constraint when a station may be skipped and duplicate service is prohibited.

Use equality to one only when every station must be served. Allow a larger upper bound when cooperative or repeated service is permitted.

## 3. Service-credit linkage

### Modeling purpose

A station should not receive collection credit unless it is assigned and served.

A one-way linkage is:

$$
\kappa_s\le \sum_{v\in\mathcal{V}}y_{v,s}, \quad \forall s\in\mathcal{S}.
$$

### Interpretation

If no vehicle is assigned, the right side is zero and $\kappa_s$ must be zero.

### Caveat

This relation alone does not prove that the item reaches the depot. If depot delivery is required for credit, add route and return logic or define credit using a stronger completion event.

## 4. Incoming and outgoing route consistency

### Modeling purpose

An assigned station should have one incoming and one outgoing route arc for the responsible vehicle.

A generic pair of constraints is:

$$
\sum_{i\in\mathcal{N}:i\ne s}x_{v,i,s}=y_{v,s}, \quad \forall v,s,
$$

$$
\sum_{j\in\mathcal{N}:j\ne s}x_{v,s,j}=y_{v,s}, \quad \forall v,s.
$$

### Interpretation

- If $y_{v,s}=0$, no selected arc enters or leaves station $s$ for vehicle $v$.
- If $y_{v,s}=1$, exactly one selected arc enters and exactly one selected arc leaves.

### Caveat

Flow consistency does not always eliminate disconnected cycles. A separate connectivity mechanism is still required.

## 5. Depot balance and unused vehicles

### Modeling purpose

A used vehicle should leave the depot once and return once. An unused vehicle should do neither.

A strong activation form is:

$$
\sum_{s\in\mathcal{S}}x_{v,o,s}=a_v^{\mathrm{use}}, \quad \forall v\in\mathcal{V},
$$

$$
\sum_{s\in\mathcal{S}}x_{v,s,o}=a_v^{\mathrm{use}}, \quad \forall v\in\mathcal{V}.
$$

### Interpretation

- $a_v^{\mathrm{use}}=1$ activates one depot departure and one depot return.
- $a_v^{\mathrm{use}}=0$ forces both sums to zero.

### Alternative

A weaker balance form equates departures and returns without forcing a single trip. Use the activation form when each vehicle performs at most one depot-based tour.

## 6. Self-loop exclusion

A route should not select an arc from a node to itself:

$$
x_{v,i,i}=0, \quad \forall v\in\mathcal{V},\ \forall i\in\mathcal{N}.
$$

This prevents meaningless zero-movement arcs. It also avoids ambiguity in timing constraints.

## 7. Payload capacity

### Modeling purpose

The total item weight assigned to a vehicle must not exceed its carrying capacity.

$$
\sum_{s\in\mathcal{S}}w_s y_{v,s}\le Q_v, \quad \forall v\in\mathcal{V}.
$$

### Units

Both sides are measured in kilograms or another consistent mass unit.

### Applicability

This mission-level form applies when all collected items remain onboard until depot return.

It is not valid for multi-trip routes with intermediate unloading unless the payload state is modeled along the route.

## 8. Travel-distance budget

### Modeling purpose

The selected route length must respect the vehicle range limit.

$$
\sum_{i\in\mathcal{N}} \sum_{j\in\mathcal{N}:j\ne i} \ell_{ij}x_{v,i,j} \le \Lambda_v^{\max}, \quad \forall v\in\mathcal{V}.
$$

### Units

The left side and right side are in meters.

### Caveat

A distance budget is not an energy budget. It may approximate propulsion limits, but communication, hovering, and computation energy require separate accounting.

## 9. Mission start and return horizon

A common fixed-start convention is:

$$
t_v^{\mathrm{start}}=0, \quad \forall v.
$$

A mission-horizon condition is:

$$
0\le t_v^{\mathrm{return}}\le T^{\mathrm{hor}}a_v^{\mathrm{use}}, \quad \forall v.
$$

### Interpretation

An unused vehicle has return time zero under this convention. A used vehicle must return within the horizon.

### Variant

If vehicles may depart later, replace the fixed start with a bounded decision variable.

## 10. Travel-time consistency with Big-M linearization

### Modeling purpose

If an arc is selected, the destination arrival time must follow the origin departure time plus travel time.

For station-to-station arcs:

$$
t_{v,j}^{\mathrm{arr}} \ge t_{v,i}^{\mathrm{dep}} +\frac{\ell_{ij}}{\bar v_v} -M^{\mathrm{time}}_{v,i,j}(1-x_{v,i,j}), \quad \forall v,\ i\ne j,\ i,j\in\mathcal{S}.
$$

For depot-to-station movement:

$$
t_{v,s}^{\mathrm{arr}} \ge t_v^{\mathrm{start}} +\frac{\ell_{o,s}}{\bar v_v} -M^{\mathrm{time}}_{v,o,s}(1-x_{v,o,s}).
$$

For station-to-depot movement:

$$
t_v^{\mathrm{return}} \ge t_{v,s}^{\mathrm{dep}} +\frac{\ell_{s,o}}{\bar v_v} -M^{\mathrm{time}}_{v,s,o}(1-x_{v,s,o}).
$$

### Units

Distance divided by speed gives seconds. Every term in each inequality is measured in seconds.

### Big-M selection

$M^{\mathrm{time}}$ must be large enough to relax the relation when the arc is not selected. A safe value should be derived from the mission horizon and time bounds. It should not be an arbitrary extremely large number.

## 11. Minimum service duration

### Modeling purpose

An assigned station requires enough dwell time for logistics service or communication setup.

$$
t_{v,s}^{\mathrm{dep}}-t_{v,s}^{\mathrm{arr}} \ge \sigma_s y_{v,s}, \quad \forall v,s.
$$

### Interpretation

- If $y_{v,s}=1$, dwell time is at least $\sigma_s$ seconds.
- If $y_{v,s}=0$, the lower bound becomes zero.

### Caveat

If unassigned timing variables remain free, add bounds that deactivate or harmlessly limit them. Do not let arbitrary unassigned times influence other constraints.

## 12. Timing-based subtour-elimination logic

### Modeling purpose

Positive travel and service times can make a disconnected directed cycle infeasible.

Suppose a selected cycle contains stations $s_1,s_2,\ldots,s_r$ for one vehicle. For each selected cycle arc, timing consistency implies:

$$
t_{v,s_{k+1}}^{\mathrm{arr}} \ge t_{v,s_k}^{\mathrm{dep}}+\tau_{s_k,s_{k+1}},
$$

where $\tau_{i,j}=\ell_{ij}/\bar v_v>0$, and $s_{r+1}=s_1$.

Service duration implies:

$$
t_{v,s_k}^{\mathrm{dep}} \ge t_{v,s_k}^{\mathrm{arr}}+\sigma_{s_k}.
$$

Summing around the cycle gives:

$$
0 \ge \sum_{k=1}^{r} \left(\sigma_{s_k}+\tau_{s_k,s_{k+1}}\right).
$$

This is impossible when the summed travel and service time is strictly positive. Therefore, the cycle cannot exist.

### Required conditions

Timing-based elimination is valid only when:

- every selected station-to-station arc activates the timing relation;
- time variables are shared consistently around the cycle;
- total travel plus service time around every nontrivial cycle is positive;
- self-loops are excluded;
- Big-M values do not accidentally leave selected-arc timing inactive.

### Important warning

Flow constraints alone can permit disconnected station subtours. Do not claim connectivity from flow conservation without a valid additional argument.

### Zero-time warning

If some travel times and all service times on a cycle can be zero, strict time increase is lost. Timing may then fail to eliminate the cycle.

## 13. Explicit subtour-elimination alternatives

When timing cannot guarantee connectivity, use an explicit method.

### Order-variable method

Introduce an order variable $o_{v,s}$ and impose an MTZ-type relation:

$$
o_{v,i}-o_{v,j}+|\mathcal{S}|x_{v,i,j} \le |\mathcal{S}|-1, \quad i\ne j.
$$

The bounds and activation of $o_{v,s}$ must be defined; for assigned stations, a common domain is $1\le o_{v,s}\le |\mathcal{S}|$.

### Cut-set method

For every nonempty proper subset $\mathcal{R}\subset\mathcal{S}$, require at least one selected arc to leave the subset when it contains assigned stations. This is strong but can require separation algorithms.

### Commodity-flow method

Send an artificial flow from the depot to assigned stations. The route arcs limit this flow. A disconnected component cannot receive depot flow.

## 14. Common routing errors

- Treating assignment as proof of a valid depot-connected route.
- Balancing depot arcs without controlling multiple trips.
- Omitting self-loop exclusion.
- Including $i=j$ in a positive travel-time implication.
- Using distance and time in the same expression without speed conversion.
- Crediting collection without a service or return condition.
- Using payload capacity when items can be unloaded mid-route without modeling payload state.
- Claiming subtour elimination from flow constraints alone.
- Using zero service duration and zero travel time while relying on timing for subtour elimination.
- Using an undefined Big-M value.

## 15. Variant guidance

- **Single vehicle:** suppress index $v$ if desired. Keep depot, flow, budget, and connectivity logic.
- **Multiple depot trips:** replace one-tour activation with trip indices or depot-reload logic.
- **No payload constraint:** remove the payload inequality and payload parameters.
- **No distance constraint:** remove the range inequality. Keep time and energy limits if required.
- **Fixed route:** treat $x$ and $y$ as parameters. Keep timing only if arrival and departure times remain decisions.
- **Mandatory service:** replace at-most-one assignment with exactly-one assignment.
- **Repeated station visits:** use visit indices or time-expanded nodes. A single binary assignment is insufficient.
- **Continuous-time route:** retain arrival and departure variables.
- **Slot-based route:** use time-expanded arc or presence variables and ensure travel occupies the correct number of slots.
