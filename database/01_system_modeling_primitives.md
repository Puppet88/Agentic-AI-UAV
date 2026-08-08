# System Modeling Primitives for Joint UAV Logistics and MEC

> Scope note: These are reusable domain primitives with alternatives and caveats. They are not an ordered formulation for a particular evaluation prompt.

## 1. Modeling purpose

A system model converts a narrative operating scenario into entities, processes, decisions, resources, and feasibility relationships. For a joint UAV-logistics and MEC problem, the key modeling challenge is that one mobile platform supports two functions:

- physical service, such as visiting stations and collecting items;
- digital service, such as receiving data, processing tasks, or relaying data to remote computing infrastructure.

The two functions share mobility time, battery energy, onboard computing capacity, and wireless communication capacity.


## Corpus-use boundary

This file describes a problem class, not one required formulation. A generated model should be derived from the user narrative. When an evaluation set exists, the corpus should have been frozen before that set was authored.

## 2. Generic system boundary

A generic system can contain the following elements.

- A vehicle set $\mathcal{V}$ containing UAVs or other mobile service vehicles.
- A service-node set $\mathcal{S}$ containing stations that may require logistics and digital service.
- A depot node $o$ where vehicles start and finish a mission.
- A routing-node set $\mathcal{N}=\{o\}\cup\mathcal{S}$.
- A device set $\mathcal{D}$ containing task-generating devices.
- A task set $\mathcal{Q}$ containing realized computation requests.
- A slot set $\mathcal{H}=\{0,1,\ldots,H-1\}$ with slot duration $\Delta$ seconds.
- A remote processor denoted by $p^{\mathrm{rem}}$, such as a terrestrial edge cluster or cloud server.

A device-to-station mapping $s(d)\in\mathcal{S}$ identifies the station that hosts device $d$. A task-to-device mapping $d(q)\in\mathcal{D}$ identifies the source of task $q$. Therefore, the source station of task $q$ is $s(q)=s(d(q))$.

### Applicability

This abstraction applies when devices are associated with known service nodes and a mobile platform must be physically present to provide short-range service.

It should be changed when devices move independently, when a station can be served remotely without vehicle presence, or when the remote processor has a direct device link.

## 3. Physical logistics layer

The physical layer represents vehicle movement and station service.

Typical decisions include:

- whether station $s$ is assigned to vehicle $v$;
- whether vehicle $v$ travels directly from node $i$ to node $j$;
- when vehicle $v$ arrives at and departs from station $s$;
- whether the item at station $s$ is collected;
- whether vehicle $v$ is present at station $s$ in slot $h$.

Typical limits include:

- payload capacity in kilograms;
- travel distance in meters;
- mission duration in seconds;
- battery energy in joules;
- minimum service duration in seconds.

### Modeling caveat

Station assignment, route arcs, and slot-level presence are different concepts. Assignment states which vehicle is responsible for a station. A route arc states which movement occurs. Presence states when a vehicle is available at a station. These variables should not be used interchangeably.

## 4. Device computing layer

Each device may execute a task locally. Local execution consumes device computing capacity but normally does not require a UAV service window.

A local processor is commonly described by:

- a compute-rate capacity in work-units per second;
- a task workload in work-units;
- a per-slot allocation in work-units per second;
- an optional device-energy cost.

Local processing is independent of UAV presence unless the application explicitly requires UAV coordination, data acquisition, or power transfer.

## 5. Aerial MEC layer

A UAV may provide onboard computation while it serves a station. The following conditions are normally required:

- the task selects a UAV-execution mode;
- the selected UAV is assigned or otherwise eligible to serve the source station;
- task input data reaches the UAV;
- the UAV is present during the required uplink and onboard-processing slots;
- UAV computing and energy capacities are respected.

The aerial layer can reduce remote communication delay. It also consumes UAV battery energy and onboard resources.

## 6. Terrestrial or cloud computing layer

A remote processor may execute tasks that are forwarded through a UAV-assisted path. A common two-hop path is:

1. device to serving UAV;
2. serving UAV to remote processor.

Remote execution therefore requires communication-path feasibility in addition to remote compute capacity.

### Applicability

The two-hop model applies when the device has no direct remote link and the UAV acts as the access point and relay.

It does not apply when a direct device-to-cloud link is explicitly available. In that case, a separate direct-link mode, capacity, and energy model should be introduced.

### Modeling caveat

Remote compute capacity alone does not make cloud execution feasible. The input data must first be delivered through every required hop. The model should distinguish communication service from computing service.

## 7. Product collection and station service

A station may have a logistics value $v_s^{\mathrm{col}}$ and an item weight $w_s$.

- $v_s^{\mathrm{col}}$ is a benefit, such as priority or collection value. Its unit may be monetary value, score units, or a normalized utility.
- $w_s$ is a physical payload quantity, commonly measured in kilograms.

The model should state when collection is considered complete. Common conventions include:

- collection occurs when the vehicle finishes service at the station;
- collection is credited only if the vehicle returns to the depot;
- collection is credited if the station is assigned and visited.

The selected convention affects routing and objective logic.

## 8. Task arrivals and time discretization

A task $q\in\mathcal{Q}$ may be characterized by:

- arrival slot $a_q\in\mathcal{H}$;
- arrival time $A_q=a_q\Delta$ seconds;
- workload $W_q$ work-units;
- input size $L_q$ bits;
- relative deadline $D_q$ seconds;
- absolute deadline $\bar D_q=A_q+D_q$ seconds.

A slot-based model treats resource allocations as rates that are constant within a slot. If $f_{q,h}$ is a compute rate in work-units per second, then

$$
f_{q,h}\Delta
$$

is the computing work delivered in slot $h$. If $r_{q,h}$ is an effective data rate in bits per second, then

$$
r_{q,h}\Delta
$$

is the data volume delivered in slot $h$.

### Modeling caveat

Workload and input size are different physical quantities. Workload is completed by computing service. Input size is delivered by communication service. One cumulative constraint cannot replace the other.

## 9. Route-induced service availability

Vehicle routing determines where and when UAV-assisted digital service is available. Let $\pi_{v,s,h}\in\{0,1\}$ indicate that vehicle $v$ serves station $s$ in slot $h$.

A generic availability statement is:

$$
\pi_{v,s,h}=1 \quad\Longrightarrow\quad y_{v,s}=1\text{ and vehicle }v\text{ is physically present at station }s\text{ in slot }h.
$$

UAV computing, device-to-UAV transmission, and UAV relay transmission should be gated by this availability when they require station presence.

This is a forward operational coupling:

$$
\text{route and timing decisions} \;\longrightarrow\; \text{service windows} \;\longrightarrow\; \text{MEC feasibility}.
$$

The arrow describes dependency. It does not by itself imply that a training algorithm feeds MEC performance back into route learning.

## 10. Shared resources and trade-offs

The physical and digital services can share:

- mission time;
- UAV energy;
- hovering time;
- onboard compute capacity;
- uplink and relay capacity;
- station service windows.

A longer station visit can provide more digital service, but it may reduce the number of stations that can be visited. More UAV computing can reduce remote traffic, but it consumes onboard energy. More remote execution can reduce onboard compute demand, but it requires communication capacity.

These trade-offs motivate a joint objective or a constrained multi-objective formulation.

## 11. Assumptions that should be stated

A rigorous formulation should explicitly state assumptions such as:

- vehicles are homogeneous or heterogeneous;
- flight speed is fixed or decision-dependent;
- station locations are deterministic;
- all logistics items are ready at mission start or arrive dynamically;
- each station can be visited once or multiple times;
- a vehicle serves at most one station per slot;
- wireless effective-rate factors are known, estimated, or decision-dependent;
- tasks are divisible across slots but not across execution modes;
- preemption is allowed or forbidden;
- cloud processing can continue after the UAV leaves once data are delivered;
- task output-return traffic is negligible or explicitly modeled;
- task arrivals are known realizations or stochastic processes;
- collection credit requires depot return or only station service.

Unstated assumptions often create logical gaps.

## 12. Workflow from narrative to formulation

A practical modeling workflow is:

1. Identify entities and map them to sets.
2. Identify physical and digital events.
3. Separate decisions from parameters.
4. Define the route, service, execution, resource, completion, and energy variables.
5. Select a time representation: continuous time, slots, or a hybrid.
6. Define the benefit and cost categories.
7. Add local feasibility constraints for each subsystem.
8. Add cross-layer coupling constraints.
9. Check units and variable domains.
10. Remove modes and constraints that are not requested.
11. Check boundary cases, such as unused vehicles and unfinished tasks.

## 13. Variant guidance

- **Routing-only:** remove task, compute, communication, and completion variables. Keep physical service and route feasibility.
- **MEC-only with fixed windows:** treat $\pi_{v,s,h}$ as input data. Remove route arcs and route timing decisions.
- **No cloud mode:** remove relay variables and remote compute allocations.
- **No UAV compute mode:** retain uplink and relay only if the UAV still supports remote execution.
- **No energy budget:** remove energy variables and constraints. Keep capacity and timing constraints.
- **Continuous time:** replace slot-indexed allocations with continuous-time rates or event-based variables.
- **Stochastic arrivals:** formulate a scenario, online, robust, or chance-constrained model rather than assuming future realizations are known.
