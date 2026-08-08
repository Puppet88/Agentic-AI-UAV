# Notation Design Primitives

> Scope note: These are reusable domain primitives with alternatives and caveats. They are not an ordered formulation for a particular evaluation prompt.
## Retrieval guidance

Use this file to design a notation system. For a joint mobility-computing model, common symbol groups include entity sets, routing nodes and depot, vehicles, stations, devices, tasks, arrival slots, processing slots, processor modes, communication paths, route arcs, assignments, presence indicators, compute rates, link rates, completion variables, deadlines, capacities, energy quantities, objective weights, and Big-M constants. Retrieve narrower subsections when definitions are needed. Do not copy one unified table as an answer.


## 1. Purpose

Notation design is part of model correctness. A good notation system separates physical entities, time indices, binary logic, continuous resource allocations, and derived quantities. Every symbol should have one meaning, one domain, and a clear unit.

This file presents one generic convention. A formulation may use different symbols, but it should preserve the distinctions described here.

## 2. Core sets and indices

| Symbol | Type | Meaning |
|---|---|---|
| $\mathcal{V}$ | set | mobile vehicles or UAVs |
| $v$ | index | a vehicle in $\mathcal{V}$ |
| $\mathcal{S}$ | set | service stations |
| $s$ | index | a station in $\mathcal{S}$ |
| $o$ | node | depot or base node |
| $\mathcal{N}=\{o\}\cup\mathcal{S}$ | set | routing nodes |
| $i,j$ | indices | origin and destination nodes in $\mathcal{N}$ |
| $\mathcal{D}$ | set | task-generating devices |
| $d$ | index | a device in $\mathcal{D}$ |
| $\mathcal{Q}$ | set | realized computation tasks |
| $q$ | index | a task in $\mathcal{Q}$ |
| $\mathcal{H}$ | set | scheduling slots |
| $h,\ell$ | indices | current slot and summation slot |
| $\mathcal{P}$ | set | generic processors when a unified processor index is useful |
| $p$ | index | a processor in $\mathcal{P}$ |
| $\mathcal{L}$ | set | communication links or link classes |
| $e$ | index | a communication link in $\mathcal{L}$ |

### Chunk-local mapping symbols

- $d(q)$: source device of task $q$.
- $s(d)$: station hosting device $d$.
- $s(q)=s(d(q))$: source station of task $q$.
- $a_q$: task-generation or arrival slot.

## 3. Time parameters

| Symbol | Domain and unit | Meaning |
|---|---|---|
| $H$ | positive integer | number of slots |
| $\Delta$ | seconds | slot duration |
| $a_q$ | integer in $\mathcal{H}$ | task-generation slot |
| $A_q=a_q\Delta$ | seconds | task arrival time |
| $D_q$ | seconds | relative deadline duration |
| $\bar D_q=A_q+D_q$ | seconds | absolute deadline |
| $T^{\mathrm{hor}}$ | seconds | mission or scheduling horizon |

### Important distinction

The arrival slot $a_q$ identifies when a task is generated. The current processing slot $h$ identifies when a decision is made. A formulation should not use one index for both roles when tasks can persist across multiple slots.

## 4. Routing parameters and variables

### Parameters

| Symbol | Domain and unit | Meaning |
|---|---|---|
| $\ell_{ij}$ | nonnegative meters | distance from node $i$ to node $j$ |
| $\bar v_v$ | positive meters/second | travel speed of vehicle $v$ |
| $w_s$ | nonnegative kilograms | item weight at station $s$ |
| $Q_v$ | nonnegative kilograms | payload capacity of vehicle $v$ |
| $\Lambda_v^{\max}$ | nonnegative meters | route-distance budget |
| $\sigma_s$ | nonnegative seconds | minimum service duration at station $s$ |
| $v_s^{\mathrm{col}}$ | score or monetary units | collection value of station $s$ |

### Binary variables

| Symbol | Domain | Meaning |
|---|---|---|
| $a_v^{\mathrm{use}}$ | $\{0,1\}$ | vehicle $v$ is activated |
| $y_{v,s}$ | $\{0,1\}$ | station $s$ is assigned to vehicle $v$ |
| $x_{v,i,j}$ | $\{0,1\}$ | vehicle $v$ uses directed arc $i\rightarrow j$ |
| $\kappa_s$ | $\{0,1\}$ | station service or collection is credited |
| $\pi_{v,s,h}$ | $\{0,1\}$ | vehicle $v$ is present and serving station $s$ in slot $h$ |

### Continuous timing variables

| Symbol | Domain and unit | Meaning |
|---|---|---|
| $t_v^{\mathrm{start}}$ | nonnegative seconds | vehicle departure time from depot |
| $t_v^{\mathrm{return}}$ | nonnegative seconds | vehicle return time |
| $t_{v,s}^{\mathrm{arr}}$ | nonnegative seconds | arrival time at station $s$ |
| $t_{v,s}^{\mathrm{dep}}$ | nonnegative seconds | departure time from station $s$ |

### Important distinction

Assignment $y_{v,s}$ is a mission-level responsibility. Presence $\pi_{v,s,h}$ is a slot-level physical state. A vehicle may be assigned to a station but not present in every slot.

## 5. Task and mode parameters

| Symbol | Domain and unit | Meaning |
|---|---|---|
| $W_q$ | nonnegative work-units | computing workload of task $q$ |
| $L_q$ | nonnegative bits | input-data size of task $q$ |
| $\omega_q$ | nonnegative score | optional task-priority weight |

### Workload versus input size

- $W_q$ is consumed by a processor. It is compared with cumulative compute service.
- $L_q$ is transported over communication links. It is compared with cumulative delivered bits.

They may be correlated, but they are not interchangeable and need not have the same numerical value.

### Binary execution-mode variables

| Symbol | Domain | Meaning |
|---|---|---|
| $m_q^{\mathrm{loc}}$ | $\{0,1\}$ | task $q$ uses local execution |
| $m_{q,v}^{\mathrm{air}}$ | $\{0,1\}$ | task $q$ executes on vehicle $v$ |
| $m_{q,v}^{\mathrm{rem}}$ | $\{0,1\}$ | task $q$ executes remotely through vehicle $v$ |
| $g_{q,v}$ | $\{0,1\}$ | vehicle $v$ is the first communication hop for task $q$ |

A useful derived relation is $g_{q,v}=m_{q,v}^{\mathrm{air}}+m_{q,v}^{\mathrm{rem}}$ when exactly one execution mode is selected.

### Cloud execution versus communication path

$m_{q,v}^{\mathrm{rem}}=1$ identifies the processor choice. It does not itself deliver the task data. Uplink and relay allocations are separate continuous decisions and must satisfy their own capacity and data-volume constraints.

## 6. Computing-allocation variables and capacities

| Symbol | Domain and unit | Meaning |
|---|---|---|
| $f_{q,h}^{\mathrm{loc}}$ | nonnegative work-units/second | local compute rate allocated to task $q$ |
| $f_{q,v,h}^{\mathrm{air}}$ | nonnegative work-units/second | vehicle compute rate allocated to task $q$ |
| $f_{q,v,h}^{\mathrm{rem}}$ | nonnegative work-units/second | remote compute rate attributed to task $q$ through vehicle $v$ |
| $F_d^{\mathrm{loc}}$ | positive work-units/second | local capacity of device $d$ |
| $F_v^{\mathrm{air}}$ | positive work-units/second | onboard capacity of vehicle $v$ |
| $F^{\mathrm{rem}}$ | positive work-units/second | aggregate remote capacity |

A rate multiplied by slot duration gives delivered work:

$$
\text{work delivered in slot }h=f_{q,h}\Delta.
$$

## 7. Communication-allocation variables and capacities

| Symbol | Domain and unit | Meaning |
|---|---|---|
| $r_{q,v,h}^{\mathrm{up}}$ | nonnegative bits/second | allocated device-to-vehicle nominal rate |
| $r_{q,v,h}^{\mathrm{relay}}$ | nonnegative bits/second | allocated vehicle-to-remote nominal rate |
| $B_v^{\mathrm{up}}$ | positive bits/second | aggregate uplink allocation limit |
| $B_v^{\mathrm{relay}}$ | positive bits/second | aggregate relay allocation limit |
| $\eta_{q,v,h}^{\mathrm{up}}$ | dimensionless, often in $[0,1]$ | uplink effective-rate factor |
| $\eta_{v,h}^{\mathrm{relay}}$ | dimensionless, often in $[0,1]$ | relay effective-rate factor |

The effective delivered bits in a slot are:

$$
\eta_{q,v,h}^{\mathrm{up}}r_{q,v,h}^{\mathrm{up}}\Delta
$$

for the uplink, and

$$
\eta_{v,h}^{\mathrm{relay}}r_{q,v,h}^{\mathrm{relay}}\Delta
$$

for the relay.

If $r$ is already defined as effective throughput, the factor $\eta$ should be omitted to avoid double scaling.

## 8. Completion and deadline variables

| Symbol | Domain and unit | Meaning |
|---|---|---|
| $\delta_{q,h}$ | $\{0,1\}$ | task $q$ completes in slot $h$ |
| $c_q$ | $\{0,1\}$ | task $q$ is completed by the horizon |
| $\chi_{q,h}$ | $\{0,1\}$ | task $q$ has completed by the end of slot $h$ |
| $z_q$ | $\{0,1\}$ | task $q$ is completed on time |
| $T_q$ | nonnegative seconds | completion time under a stated unfinished-task convention |

### Completion versus on-time completion

$c_q=1$ means the task finishes by the modeled horizon. $z_q=1$ means it finishes no later than its deadline. A completed-late task can have $c_q=1$ and $z_q=0$.

### Completion slot versus cumulative completion

$\delta_{q,h}$ identifies the unique slot in which completion occurs. $\chi_{q,h}$ indicates whether completion has occurred by slot $h$.

## 9. Energy variables and parameters

| Symbol | Domain and unit | Meaning |
|---|---|---|
| $E_v^{\max}$ | nonnegative joules | available vehicle energy |
| $E_v^{\mathrm{move}}$ | nonnegative joules | movement energy |
| $E_v^{\mathrm{hover}}$ | nonnegative joules | hovering or station-service energy |
| $E_v^{\mathrm{cpu}}$ | nonnegative joules | onboard-computing energy |
| $E_v^{\mathrm{up}}$ | nonnegative joules | UAV-side uplink reception or link-support energy |
| $E_v^{\mathrm{relay}}$ | nonnegative joules | relay-transmission energy |
| $E_v^{\mathrm{res}}$ | nonnegative joules | residual energy after the mission |

Energy coefficients must state their units. Examples include joules per meter, watts, joules per work-unit, and joules per delivered bit.

## 10. Objective weights and normalization

Let $\theta_r\ge 0$ denote a nonnegative coefficient for criterion $r$.

Weights are dimensionless only when each criterion has already been normalized. If raw terms with different units are combined, each coefficient must convert or scale its term into a common score.

## 11. Big-M constants

A Big-M value is a parameter, not an unspecified symbol. Use a local name that reflects the bounded quantity, such as:

- $M^{\mathrm{time}}$ in seconds;
- $M^{\mathrm{work}}$ in work-units;
- $M^{\mathrm{data}}$ in bits.

A valid Big-M should be derived from known upper and lower bounds. An excessively large value can weaken a mixed-integer model and create numerical instability.

## 12. Domain declarations

A complete formulation should include explicit domains, such as:

$$
x_{v,i,j},y_{v,s},a_v^{\mathrm{use}},\kappa_s,\pi_{v,s,h}\in\{0,1\}.
$$

$$
m_q^{\mathrm{loc}},m_{q,v}^{\mathrm{air}},m_{q,v}^{\mathrm{rem}},g_{q,v},\delta_{q,h},c_q,\chi_{q,h},z_q\in\{0,1\}.
$$

$$
f_{q,h}^{\mathrm{loc}},f_{q,v,h}^{\mathrm{air}},f_{q,v,h}^{\mathrm{rem}},r_{q,v,h}^{\mathrm{up}},r_{q,v,h}^{\mathrm{relay}}\ge 0.
$$

All time and energy variables should also be nonnegative unless a transformed coordinate system is explicitly used.

## 13. Common notation errors

- Reusing $D_q$ for both deadline and data size.
- Using $\mathcal{V}$ for both a vehicle set and a capacity.
- Using one variable for assignment and physical presence.
- Using one allocation variable for both compute rate and communication rate.
- Treating a task-generation slot as the current service slot.
- Using $c_q$ and $z_q$ as if they had the same meaning.
- Treating a remote execution-mode indicator as proof that data were transmitted.
- Adding physical energy and normalized utilization without scaling.
- Using a Big-M constant without a unit or finite bound.
- Omitting binary or nonnegative domains.

## 14. Variant guidance

- **Continuous-time scheduling:** replace slot variables with start, finish, and continuous service variables.
- **No relay mode:** delete $m_{q,v}^{\mathrm{rem}}$ and relay allocations.
- **Fixed routes:** treat $x$, $y$, and route times as parameters.
- **Fixed service windows:** treat $\pi$ as a parameter.
- **Heterogeneous vehicles:** index speed, payload, compute, communication, and energy parameters by $v$.
- **Single vehicle:** the vehicle index can be suppressed, but keeping it may make later extensions easier.

## 15. Cross-file symbol consistency guard

Use $a_v^{\mathrm{use}}$ only for vehicle activation and $u_{p,h}^{\mathrm{cpu}}$ only for processor utilization. Use $\vartheta_q^{\mathrm{delay}}$ only as an objective weight and $\ell_q^{\mathrm{late}}$ only as a late-completion indicator. Use $o_{v,i}$ for a routing order variable so that task index $q$ remains unambiguous. Use $\xi_v^{\mathrm{cpu}}$ for a nonlinear computing-energy coefficient. These distinctions prevent symbol reuse across independently retrieved chunks.
