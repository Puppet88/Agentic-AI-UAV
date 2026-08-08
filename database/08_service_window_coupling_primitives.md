# Service-Window Coupling Primitives

> Scope note: These are reusable domain primitives with alternatives and caveats. They are not an ordered formulation for a particular evaluation prompt.
## Retrieval guidance

Service-window coupling has four distinct questions: how route timing creates a station-availability interval; how that interval becomes a slot-level presence variable; which computing and communication resources are gated by presence; and whether onboard processing or relay may continue after departure using stored data. Decide whether windows are fixed or endogenous before selecting formulas. Local processing normally remains independent of UAV presence.


## 1. Purpose

A route-induced service window connects physical mobility with digital service. A UAV can provide station-dependent computation and communication only when it is physically available at the relevant station.

This coupling is central to joint routing and MEC models. Without it, the scheduling layer may use a UAV remotely at times and locations that are physically impossible.

## 2. Local notation

- $\mathcal{V}$: UAV set; index $v$.
- $\mathcal{S}$: station set; index $s$.
- $\mathcal{H}$: slot set; index $h$.
- $\Delta$: slot duration in seconds.
- $t_h^{\mathrm{ref}}$: reference time of slot $h$, such as slot start, midpoint, or end.
- $y_{v,s}\in\{0,1\}$: station $s$ is assigned to UAV $v$.
- $t_{v,s}^{\mathrm{arr}}$, $t_{v,s}^{\mathrm{dep}}$: arrival and departure times in seconds.
- $\pi_{v,s,h}\in\{0,1\}$: UAV $v$ is present and serving station $s$ in slot $h$.
- $s(q)$: source station of task $q$.
- $f_{q,v,h}^{\mathrm{air}}$: UAV compute allocation in work-units per second.
- $r_{q,v,h}^{\mathrm{up}}$: device-to-UAV rate in bits per second.
- $r_{q,v,h}^{\mathrm{relay}}$: UAV-to-remote rate in bits per second.

## 3. Assignment and service eligibility

### Modeling purpose

A UAV should not serve a station that is not assigned to it.

$$
\pi_{v,s,h}\le y_{v,s}, \quad \forall v,s,h.
$$

### Interpretation

If $y_{v,s}=0$, presence must be zero in every slot. Assignment is necessary but not sufficient for presence.

## 4. One station at a time

A UAV cannot provide station-local service at several stations simultaneously:

$$
\sum_{s\in\mathcal{S}}\pi_{v,s,h}\le1, \quad \forall v,h.
$$

### Applicability

Use this form when each UAV has one location and one station-local service association per slot.

It may be changed if overlapping coverage regions allow a UAV to serve several stations from one location.

## 5. Presence linked to arrival and departure interval

### One-way feasibility implication

If $\pi_{v,s,h}=1$, the slot reference time must lie inside the station interval:

$$
t_{v,s}^{\mathrm{arr}} \le t_h^{\mathrm{ref}} + M^{\mathrm{time}}(1-\pi_{v,s,h}),
$$

$$
t_{v,s}^{\mathrm{dep}} \ge t_h^{\mathrm{ref}} - M^{\mathrm{time}}(1-\pi_{v,s,h}).
$$

### Interpretation

When $\pi_{v,s,h}=1$, the Big-M terms vanish and the reference time lies between arrival and departure. When $\pi=0$, the constraints are relaxed.

### Caveat

These inequalities ensure that selected presence is physically valid. They do not force every slot inside the interval to have $\pi=1$. If exact interval coverage is required, add a stronger discretization rule or define presence directly from fixed windows.

## 6. Service-duration consistency in slots

When each active presence slot represents $\Delta$ seconds of service, a useful consistency relation is:

$$
\sum_{h\in\mathcal{H}}\pi_{v,s,h}\Delta \le t_{v,s}^{\mathrm{dep}}-t_{v,s}^{\mathrm{arr}} + M^{\mathrm{time}}(1-y_{v,s}).
$$

### Interpretation

The total selected slot service should fit inside the continuous dwell interval.

A lower-bound or equality version may be used when every portion of the dwell interval must be represented by slots.

## 7. Fixed service-window variant

When routes and station times are already known, define a parameter $\bar\pi_{v,s,h}\in\{0,1\}$.

- $\bar\pi_{v,s,h}=1$ means UAV $v$ is available at station $s$ in slot $h$.
- $\bar\pi_{v,s,h}=0$ means it is unavailable.

The scheduling model then uses $\bar\pi$ in resource gates. It does not optimize assignment, route arcs, arrival time, or departure time.

## 8. UAV computing gated by presence

### Modeling purpose

Onboard computing for a station task is available only while the UAV serves that station, unless the task and data are allowed to travel with the UAV after upload.

A strict station-window gate is:

$$
f_{q,v,h}^{\mathrm{air}} \le F_v^{\mathrm{air}}\pi_{v,s(q),h}.
$$

### Interpretation

The right side is zero when the UAV is absent. Therefore, the compute allocation is forced to zero.

### Applicability

Use this strict form when onboard processing is tied to station service.

If a UAV may continue processing an already uploaded task after leaving, gate the upload by presence and model onboard task carriage separately. Do not silently assume both interpretations.

## 9. Device-to-UAV uplink gated by presence

$$
r_{q,v,h}^{\mathrm{up}} \le B_v^{\mathrm{up}}\pi_{v,s(q),h}.
$$

### Interpretation

A source device can upload through UAV $v$ only when that UAV serves the source station in slot $h$.

### Caveat

Assignment alone is not enough. The UAV must be present in the current slot.

## 10. UAV-to-remote relay gated by presence

A strict coupled gate is:

$$
r_{q,v,h}^{\mathrm{relay}} \le B_v^{\mathrm{relay}}\pi_{v,s(q),h}.
$$

### Applicability

Use this form when the model allows relay activity only during the station service interval.

If the UAV can forward stored data after departure, introduce an onboard data-buffer state. In that model, the uplink remains station-window dependent, while later relay activity depends on stored data and backhaul availability rather than station presence.

## 11. Cloud computation after data delivery

Remote computation may continue after the UAV leaves the station if all required input data have already reached the remote processor.

A common assumption is:

- uplink is gated by source-station presence;
- relay is gated by either station presence or an explicit UAV data buffer;
- remote compute is gated by remote-mode selection and data-arrival causality, but not necessarily by current station presence.

### Caveat

Do not gate remote compute by presence automatically unless that is the intended system behavior. Conversely, do not allow remote compute before the input data reach the remote processor.

## 12. Local computing independence

Local computation usually does not require a UAV service window:

$$
f_{q,h}^{\mathrm{loc}} \text{ is not multiplied by } \pi_{v,s(q),h}.
$$

Local service may still depend on task arrival, device capacity, selected mode, and completion status.

## 13. Route-to-MEC forward coupling

The coupling chain is:

$$
(x,y,t^{\mathrm{arr}},t^{\mathrm{dep}}) \longrightarrow \pi \longrightarrow (f^{\mathrm{air}},r^{\mathrm{up}},r^{\mathrm{relay}}).
$$

### Interpretation

- Route and timing decisions create station availability.
- Presence variables represent that availability at slot level.
- Resource gates translate availability into MEC feasibility.

This is operational coupling. An algorithm may still solve the routing and scheduling parts sequentially.

## 14. Energy and service-window interaction

Longer service windows can increase:

- hovering or station-service energy;
- available uplink time;
- available onboard-compute time;
- available relay time under a strict-window assumption.

A generic hovering-energy relation is:

$$
E_v^{\mathrm{hover}} = P_v^{\mathrm{hover}} \sum_{s,h}\pi_{v,s,h}\Delta,
$$

where $P_v^{\mathrm{hover}}$ is in watts. The resulting energy is in joules.

### Caveat

Do not count the same dwell time once through continuous arrival/departure duration and again through presence slots unless the two terms represent different physical costs.

## 15. Endogenous versus fixed service windows

### Endogenous windows

- $x$, $y$, arrival times, departure times, and $\pi$ are decision variables.
- Routing and scheduling are jointly coupled.
- Timing and presence consistency constraints are required.

### Fixed windows

- Route and timing are inputs.
- $\bar\pi$ is a parameter.
- The scheduling problem retains resource, completion, and deadline constraints.
- Route feasibility constraints are outside the scheduling model.

## 16. Common coupling errors

- Allowing UAV compute while the UAV is at another station.
- Allowing uplink based on assignment but not current presence.
- Allowing direct remote execution when only a UAV-assisted path exists.
- Gating local computation by UAV presence without a system reason.
- Treating assignment, route arc, and presence as the same variable.
- Defining presence only by time but not by station assignment.
- Allowing one UAV to serve several distant stations in one slot.
- Forcing remote compute to stop after UAV departure without stating that assumption.
- Allowing post-departure relay without an onboard data buffer.
- Counting service-window time twice in energy.

## 17. Variant guidance

- **Fixed service windows:** replace $\pi$ with parameter $\bar\pi$ and remove route-to-window constraints.
- **Routing-only:** remove $\pi$ if no station-slot service is modeled.
- **MEC-only:** keep fixed-window parameters and all scheduling gates.
- **Local-only computing:** remove UAV and relay gates.
- **UAV compute without cloud:** retain onboard-compute and uplink gates. Remove relay and remote compute.
- **Cloud through UAV only:** retain uplink and relay gates. Remove onboard compute mode if unavailable.
- **Post-departure onboard processing:** gate upload by presence and add a task-onboard state.
- **Post-departure relay:** add a data-buffer balance and backhaul-availability model.
- **Overlapping station coverage:** replace one-station-at-a-time logic with coverage and radio-sharing constraints.
- **Continuous time:** use interval-overlap variables or event-based service decisions instead of slot presence.
