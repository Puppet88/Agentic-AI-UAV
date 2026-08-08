# MEC Offloading Formulation Primitives

> Scope note: These are reusable domain primitives with alternatives and caveats. They are not an ordered formulation for a particular evaluation prompt.
## Retrieval guidance

Task-service requests should be decomposed into subtopics: processor-mode selection; communication-path activation; local, aerial, and remote capacities; uplink and relay capacities; mode and path gates; no service before arrival; cumulative computing work; cumulative transmitted data; completion and deadline logic; route-induced service availability; and energy paid by the modeled devices. A remote processor reached through a UAV requires both data delivery and compute service. Retrieve by subtopic; this file is not an ordered offloading answer.


## 1. Local notation for task-service units

This file uses the following generic notation.

- $\mathcal{Q}$: task set; index $q$.
- $\mathcal{V}$: UAV set; index $v$.
- $\mathcal{H}$: slot set; index $h$.
- $a_q$: arrival slot of task $q$.
- $\Delta$: slot duration in seconds.
- $W_q$: workload in work-units.
- $L_q$: input-data size in bits.
- $m_q^{\mathrm{loc}}\in\{0,1\}$: local-execution mode.
- $m_{q,v}^{\mathrm{air}}\in\{0,1\}$: execution on UAV $v$.
- $m_{q,v}^{\mathrm{rem}}\in\{0,1\}$: remote execution through UAV $v$.
- $g_{q,v}\in\{0,1\}$: UAV $v$ is the first communication hop.
- $f_{q,h}^{\mathrm{loc}}\ge0$: local compute rate in work-units per second.
- $f_{q,v,h}^{\mathrm{air}}\ge0$: UAV compute rate in work-units per second.
- $f_{q,v,h}^{\mathrm{rem}}\ge0$: remote compute rate in work-units per second.
- $r_{q,v,h}^{\mathrm{up}}\ge0$: uplink nominal rate in bits per second.
- $r_{q,v,h}^{\mathrm{relay}}\ge0$: relay nominal rate in bits per second.
- $\eta_{q,v,h}^{\mathrm{up}}$: dimensionless uplink effective-rate factor.
- $\eta_{v,h}^{\mathrm{relay}}$: dimensionless relay effective-rate factor.
- $\chi_{q,h}\in\{0,1\}$: task $q$ has completed by the end of slot $h$.
- $\pi_{v,s(q),h}\in\{0,1\}$: UAV $v$ serves the source station of task $q$ in slot $h$.

## 2. Exactly-one execution mode

### Modeling purpose

Each task should select one processor path.

$$
m_q^{\mathrm{loc}} +\sum_{v\in\mathcal{V}}m_{q,v}^{\mathrm{air}} +\sum_{v\in\mathcal{V}}m_{q,v}^{\mathrm{rem}} =1, \quad \forall q\in\mathcal{Q}.
$$

### Interpretation

The first term selects device execution. The second group selects an onboard UAV processor. The third group selects remote execution through one UAV relay.

### Applicability

Use equality when every task must choose a mode. Use a less-than-or-equal relation with an admission variable when dropping tasks is allowed.

### Caveat

Exactly-one-mode selection does not guarantee that enough compute or data service is provided. Capacity, gating, and cumulative sufficiency are still required.

## 3. First-hop activation

A task uses UAV $v$ as the first communication hop when it is executed on that UAV or forwarded through it:

$$
g_{q,v} = m_{q,v}^{\mathrm{air}}+m_{q,v}^{\mathrm{rem}}, \quad \forall q,v.
$$

This equality is valid because the exactly-one-mode rule prevents both terms on the right from being one simultaneously.

## 4. Per-slot local computing capacity

Let $d(q)$ denote the source device of task $q$. Let $F_d^{\mathrm{loc}}>0$ be local capacity in work-units per second.

$$
\sum_{q:d(q)=d,\ a_q\le h} f_{q,h}^{\mathrm{loc}} \le F_d^{\mathrm{loc}}, \quad \forall d,\ h.
$$

### Interpretation

The left side is the total local compute rate allocated by device $d$ in slot $h$. The right side is its available rate.

## 5. Per-slot UAV computing capacity

Let $F_v^{\mathrm{air}}>0$ be UAV $v$ compute capacity in work-units per second.

$$
\sum_{q:a_q\le h} f_{q,v,h}^{\mathrm{air}} \le F_v^{\mathrm{air}}, \quad \forall v,h.
$$

This constraint limits aggregate onboard processing in each slot.

## 6. Per-slot remote computing capacity

Let $F^{\mathrm{rem}}>0$ be aggregate remote capacity in work-units per second.

$$
\sum_{v\in\mathcal{V}} \sum_{q:a_q\le h} f_{q,v,h}^{\mathrm{rem}} \le F^{\mathrm{rem}}, \quad \forall h.
$$

If the remote infrastructure contains several servers, use server-specific capacities and assignment variables.

## 7. Uplink and relay capacity

Let $B_v^{\mathrm{up}}>0$ and $B_v^{\mathrm{relay}}>0$ be nominal link capacities in bits per second.

$$
\sum_{q:a_q\le h}r_{q,v,h}^{\mathrm{up}} \le B_v^{\mathrm{up}}, \quad \forall v,h,
$$

$$
\sum_{q:a_q\le h}r_{q,v,h}^{\mathrm{relay}} \le B_v^{\mathrm{relay}}, \quad \forall v,h.
$$

### Caveat

The uplink and relay are different resources. One capacity constraint should not be used for both unless they physically share the same channel and the shared-use rule is explicitly modeled.

## 8. Compute allocation gated by execution mode

### Local gate

$$
0\le f_{q,h}^{\mathrm{loc}} \le F_{d(q)}^{\mathrm{loc}}m_q^{\mathrm{loc}}.
$$

### UAV gate

$$
0\le f_{q,v,h}^{\mathrm{air}} \le F_v^{\mathrm{air}}m_{q,v}^{\mathrm{air}}.
$$

### Remote-compute gate

$$
0\le f_{q,v,h}^{\mathrm{rem}} \le F^{\mathrm{rem}}m_{q,v}^{\mathrm{rem}}.
$$

### Interpretation

A zero mode variable forces the corresponding allocation to zero. A selected mode permits allocation up to the physical capacity.

## 9. Communication allocation gated by path

### Uplink gate

$$
0\le r_{q,v,h}^{\mathrm{up}} \le B_v^{\mathrm{up}}g_{q,v}.
$$

### Relay gate

$$
0\le r_{q,v,h}^{\mathrm{relay}} \le B_v^{\mathrm{relay}}m_{q,v}^{\mathrm{rem}}.
$$

### Interpretation

The uplink is required for UAV execution and remote execution through a UAV. The relay is required only for the remote mode.

### Caveat

Do not activate relay allocation for UAV-only execution. Do not allow remote execution without a valid first hop and relay path.

## 10. No service before task arrival

Define the known arrival-availability parameter by:

$$
A_{q,h}^{\mathrm{arr}}=\mathbf{1}\{h\ge a_q\}.
$$

Thus, $A_{q,h}^{\mathrm{arr}}=1$ at and after the arrival slot, and it is zero before arrival. Each allocation can be bounded by this parameter. For example:

$$
f_{q,h}^{\mathrm{loc}} \le F_{d(q)}^{\mathrm{loc}}A_{q,h}^{\mathrm{arr}}.
$$

Similar bounds apply to UAV compute, remote compute, uplink, and relay allocations.

### Purpose

This prevents a task from receiving service before it exists.

## 11. Optional no-service-after-completion rule

Define $\chi_{q,h-1}=1$ when task $q$ was already completed before slot $h$. Set $\chi_{q,-1}=0$.

A generic post-completion gate is:

$$
f_{q,h}^{\mathrm{loc}} \le F_{d(q)}^{\mathrm{loc}}(1-\chi_{q,h-1}).
$$

Equivalent gates can be used for other allocations.

### Applicability

Use this when service after completion has no meaning and allocations should be tightly bounded.

It may be omitted if the objective already penalizes unnecessary resources and the additional constraints are not needed.

## 12. Cumulative local computing sufficiency

### Modeling purpose

A task cannot claim completion before receiving enough computing work.

For each slot $h\ge a_q$:

$$
\sum_{\ell=a_q}^{h} f_{q,\ell}^{\mathrm{loc}}\Delta \ge W_q\chi_{q,h} - M_q^{\mathrm{work}}(1-m_q^{\mathrm{loc}}).
$$

### Units

- $f\Delta$ is in work-units.
- $W_q$ is in work-units.
- $M_q^{\mathrm{work}}$ is also in work-units.

### Interpretation

If local mode is selected and completion is claimed by slot $h$, cumulative local service must reach the workload.

## 13. Cumulative UAV computing sufficiency

For each candidate UAV $v$ and slot $h\ge a_q$:

$$
\sum_{\ell=a_q}^{h} f_{q,v,\ell}^{\mathrm{air}}\Delta \ge W_q\chi_{q,h} - M_q^{\mathrm{work}}(1-m_{q,v}^{\mathrm{air}}).
$$

The right side is active only for the selected UAV-execution mode.

## 14. Cumulative remote computing sufficiency

For each candidate relay UAV $v$ and slot $h\ge a_q$:

$$
\sum_{\ell=a_q}^{h} f_{q,v,\ell}^{\mathrm{rem}}\Delta \ge W_q\chi_{q,h} - M_q^{\mathrm{work}}(1-m_{q,v}^{\mathrm{rem}}).
$$

Remote computing service and communication delivery are separate requirements. Both must be satisfied for remote completion.

## 15. Cumulative uplink data-volume sufficiency

### Modeling purpose

A task using a UAV first hop must upload its input data before completion can be claimed.

$$
\sum_{\ell=a_q}^{h} \eta_{q,v,\ell}^{\mathrm{up}} r_{q,v,\ell}^{\mathrm{up}} \Delta \ge L_q\chi_{q,h} - M_q^{\mathrm{data}}(1-g_{q,v}).
$$

### Units

The effective factor is dimensionless. Rate times slot duration gives bits. Both sides are in bits.

## 16. Cumulative relay data-volume sufficiency

Remote execution through UAV $v$ also requires the relay hop:

$$
\sum_{\ell=a_q}^{h} \eta_{v,\ell}^{\mathrm{relay}} r_{q,v,\ell}^{\mathrm{relay}} \Delta \ge L_q\chi_{q,h} - M_q^{\mathrm{data}}(1-m_{q,v}^{\mathrm{rem}}).
$$

### Interpretation

A remote task cannot be declared complete unless enough input data have crossed the relay hop.

### Caveat

If the relay sends compressed or transformed data, replace $L_q$ with the appropriate relay-data volume.

## 17. Computing work and transmitted data are different

The following two quantities have different units:

$$
\sum f\Delta \quad\text{in work-units},
$$

$$
\sum \eta r\Delta \quad\text{in bits}.
$$

A correct model uses one set of cumulative constraints for computing work and another set for communication volume. Meeting one requirement does not imply that the other requirement is met.

## 18. Effective-rate factors

An effective-rate factor may represent packet success, channel availability, coding efficiency, or a normalized channel-quality multiplier.

Typical assumptions are:

$$
0\le \eta_{q,v,h}^{\mathrm{up}}\le 1, \qquad 0\le \eta_{v,h}^{\mathrm{relay}}\le 1.
$$

If an effective factor can exceed one because of a different definition, its physical meaning and unit conversion must be stated.

## 19. Service-window gating

UAV compute and UAV-assisted communication normally require physical presence at the source station.

A generic UAV-compute gate is:

$$
f_{q,v,h}^{\mathrm{air}} \le F_v^{\mathrm{air}}\pi_{v,s(q),h}.
$$

An uplink gate is:

$$
r_{q,v,h}^{\mathrm{up}} \le B_v^{\mathrm{up}}\pi_{v,s(q),h}.
$$

A relay gate may also use presence when the relay is permitted only during station service:

$$
r_{q,v,h}^{\mathrm{relay}} \le B_v^{\mathrm{relay}}\pi_{v,s(q),h}.
$$

Remote compute may continue after data delivery if the system permits it. That assumption should be stated separately.

## 20. Optional communication-computation causality

A basic cumulative-volume condition ensures that all input data are delivered by completion. A stronger model may also prevent remote computing from advancing before sufficient data arrive.

One generic principle is:

$$
\frac{1}{W_q} \sum_{\ell=a_q}^{h}f_{q,v,\ell}^{\mathrm{rem}}\Delta \le \frac{1}{L_q} \sum_{\ell=a_q}^{h} \eta_{v,\ell}^{\mathrm{relay}}r_{q,v,\ell}^{\mathrm{relay}}\Delta.
$$

Both sides are dimensionless fractions. This form assumes computation can progress proportionally as data arrive. A non-streaming task may instead require full data delivery before remote processing starts.

## 21. Common MEC modeling errors

- Using one cumulative constraint for both workload and data size.
- Allocating resources before task arrival.
- Allocating local compute to a UAV-selected task.
- Allowing relay traffic without a remote mode.
- Allowing remote compute without uplink and relay delivery.
- Ignoring per-slot aggregate capacity.
- Using raw nominal rate as delivered bits without an effective-rate definition.
- Forgetting slot duration in cumulative service.
- Allowing UAV compute when the UAV is not present.
- Treating cloud compute as automatically available from every station.
- Leaving allocations positive after completion without a reason.

## 22. Variant guidance

- **Local and UAV only:** remove remote mode, relay allocation, and remote compute capacity.
- **No UAV compute:** remove $m^{\mathrm{air}}$ and UAV compute allocation. Keep UAV relay if remote mode remains.
- **No cloud or remote processor:** remove remote mode and all relay constraints.
- **Direct device-to-remote link:** introduce a direct-link mode and capacity. Do not reuse the UAV relay variables.
- **Fixed service windows:** treat $\pi$ as an input parameter.
- **MEC-only:** remove route decisions. Keep presence or connectivity parameters.
- **Single-slot tasks:** cumulative sums reduce to one-slot requirements.
- **Non-preemptive tasks:** introduce start and processing-continuity variables. Rate-sharing alone may allow preemption.
- **Stochastic arrivals:** make decisions online or by scenario. Do not allocate future service to unknown tasks in a deterministic model.
