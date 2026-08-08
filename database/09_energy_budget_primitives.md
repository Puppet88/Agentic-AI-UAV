# UAV Energy-Budget Primitives

> Scope note: These are reusable domain primitives with alternatives and caveats. They are not an ordered formulation for a particular evaluation prompt.
## Retrieval guidance

Energy modeling begins with the battery boundary. Candidate UAV-side components include movement or flight energy, hovering or dwell energy, onboard computing energy, receive-side uplink energy, relay or backhaul transmission energy, and auxiliary avionics energy. Choose distance-based, power-time, energy-per-work, or energy-per-bit forms according to available data. Include only components paid by the UAV and combine them in joules under a mission or per-slot budget.


## 1. Purpose

A UAV mission can consume energy through movement, hovering, onboard computing, wireless reception, and relay transmission. The energy model should identify which device pays each energy cost and should use coefficients with explicit units.

Energy is a physical quantity measured in joules. It is different from normalized resource occupation.

## 2. Local notation

- $\mathcal{V}$: UAV set; index $v$.
- $\mathcal{N}$: routing-node set; indices $i,j$.
- $\mathcal{S}$: station set; index $s$.
- $\mathcal{H}$: slot set; index $h$.
- $x_{v,i,j}\in\{0,1\}$: selected route arc.
- $\ell_{ij}$: distance in meters.
- $t_{v,s}^{\mathrm{arr}}$, $t_{v,s}^{\mathrm{dep}}$: station times in seconds.
- $\pi_{v,s,h}\in\{0,1\}$: slot-level station presence.
- $f_{q,v,h}^{\mathrm{air}}$: onboard compute rate in work-units per second.
- $r_{q,v,h}^{\mathrm{up}}$: uplink allocation in bits per second.
- $r_{q,v,h}^{\mathrm{relay}}$: relay allocation in bits per second.
- $\eta_{q,v,h}^{\mathrm{up}}$, $\eta_{v,h}^{\mathrm{relay}}$: dimensionless effective-rate factors.
- $\Delta$: slot duration in seconds.
- $E_v^{\max}$: available battery energy in joules.

## 3. Total mission energy budget

A generic additive budget is:

$$
E_v^{\mathrm{move}} + E_v^{\mathrm{hover}} + E_v^{\mathrm{cpu}} + E_v^{\mathrm{up}} + E_v^{\mathrm{relay}} + E_v^{\mathrm{aux}} \le E_v^{\max}, \quad \forall v\in\mathcal{V}.
$$

### Interpretation

- $E_v^{\mathrm{move}}$: propulsion energy during travel.
- $E_v^{\mathrm{hover}}$: propulsion or service energy while stationary.
- $E_v^{\mathrm{cpu}}$: onboard processing energy.
- $E_v^{\mathrm{up}}$: UAV-side energy associated with receiving or supporting uplink traffic.
- $E_v^{\mathrm{relay}}$: energy for forwarding data.
- $E_v^{\mathrm{aux}}$: optional avionics, sensing, or baseline electronics energy.

Every term is in joules.

### Applicability

Include only energy components paid from the modeled UAV battery. Device transmit energy and remote-server energy should be modeled separately if they are outside the UAV battery.

## 4. Distance-based movement-energy approximation

Let $\epsilon_v^{\mathrm{move}}$ be movement energy per meter in joules per meter.

$$
E_v^{\mathrm{move}} = \epsilon_v^{\mathrm{move}} \sum_{i\in\mathcal{N}} \sum_{j\in\mathcal{N}:j\ne i} \ell_{ij}x_{v,i,j}.
$$

### Units

Joules per meter multiplied by meters gives joules.

### Applicability

This linear approximation is useful when speed and payload effects are fixed or absorbed into the coefficient.

### Caveat

It may be inaccurate when propulsion power varies strongly with speed, wind, payload, or acceleration.

## 5. Power-time movement model

Let $P_{v,i,j}^{\mathrm{fly}}$ be average flight power in watts on arc $i\rightarrow j$. Let $\tau_{v,i,j}=\ell_{ij}/\bar v_v$ be travel time in seconds.

$$
E_v^{\mathrm{move}} = \sum_{i,j:i\ne j} P_{v,i,j}^{\mathrm{fly}} \tau_{v,i,j} x_{v,i,j}.
$$

Watts multiplied by seconds gives joules.

### Variant

If speed is a decision, both power and travel time can depend on speed. The resulting model may be nonlinear.

## 6. Hovering or station-service energy

### Continuous-time form

Let $P_v^{\mathrm{hover}}$ be hovering power in watts. Introduce a nonnegative service-duration variable $\phi_{v,s}$ in seconds. Link it to station service with:

$$
0\le \phi_{v,s}\le T^{\mathrm{hor}}y_{v,s}.
$$

When station $s$ is assigned, impose $\phi_{v,s}=t_{v,s}^{\mathrm{dep}}-t_{v,s}^{\mathrm{arr}}$ using bounded linear equalities or two Big-M inequalities. Then define:

$$
E_v^{\mathrm{hover}} = P_v^{\mathrm{hover}} \sum_{s\in\mathcal{S}}\phi_{v,s}.
$$

This avoids multiplying a continuous dwell time by a binary assignment variable.

### Slot-based form

$$
E_v^{\mathrm{hover}} = P_v^{\mathrm{hover}} \sum_{s\in\mathcal{S}} \sum_{h\in\mathcal{H}} \pi_{v,s,h}\Delta.
$$

### Caveat

Use one dwell-time representation unless the terms capture different energy components. Otherwise, service time may be counted twice.

## 7. Linear onboard-computing energy

Let $\epsilon_v^{\mathrm{cpu}}$ be energy per work-unit in joules per work-unit.

$$
E_v^{\mathrm{cpu}} = \epsilon_v^{\mathrm{cpu}} \sum_{q} \sum_{h} f_{q,v,h}^{\mathrm{air}}\Delta.
$$

### Interpretation

$f\Delta$ is computing work. Multiplying by energy per work-unit gives joules.

### Applicability

This form is suitable when energy per completed work-unit is approximately constant.

## 8. Frequency-dependent computing alternative

A more general dynamic-voltage-and-frequency model may use:

$$
E_v^{\mathrm{cpu}} = \sum_{h} \xi_v^{\mathrm{cpu}} \left(F_{v,h}^{\mathrm{used}}\right)^3 \Delta,
$$

where $F_{v,h}^{\mathrm{used}}=\sum_q f_{q,v,h}^{\mathrm{air}}$ and $\xi_v^{\mathrm{cpu}}$ has units that convert cubic rate times seconds into joules.

### Caveat

This is nonlinear. Use it only when processor frequency scaling is part of the model and coefficient units are defined.

## 9. Uplink-related UAV energy

An uplink involves device transmission and UAV reception. If the budget accounts for UAV receiver or link-support energy, let $\epsilon_v^{\mathrm{rx}}$ be joules per successfully received bit.

$$
E_v^{\mathrm{up}} = \epsilon_v^{\mathrm{rx}} \sum_q\sum_h \eta_{q,v,h}^{\mathrm{up}} r_{q,v,h}^{\mathrm{up}} \Delta.
$$

### Units

Effective rate times slot duration gives received bits. Joules per bit multiplied by bits gives joules.

### Caveat

Do not describe this as device transmit energy unless the device battery is included in the same budget. The energy owner must be explicit.

## 10. Relay or backhaul energy

Let $\epsilon_v^{\mathrm{tx}}$ be relay-transmission energy in joules per delivered bit.

$$
E_v^{\mathrm{relay}} = \epsilon_v^{\mathrm{tx}} \sum_q\sum_h \eta_{v,h}^{\mathrm{relay}} r_{q,v,h}^{\mathrm{relay}} \Delta.
$$

This term is active only when relay traffic is allocated. Mode and service-window gates should enforce that logic.

## 11. Power-based communication alternative

If transmit power is modeled explicitly, let $P_{q,v,h}^{\mathrm{tx}}$ be watts and $a_{q,v,h}^{\mathrm{relay}}\in\{0,1\}$ indicate relay activity.

$$
E_v^{\mathrm{relay}} = \sum_q\sum_h P_{q,v,h}^{\mathrm{tx}} a_{q,v,h}^{\mathrm{relay}} \Delta.
$$

This form may be combined with a rate-power channel model. It is more general but may be nonlinear.

## 12. Mission-level accumulation

Each energy component should accumulate over the same mission boundary. If a task continues beyond vehicle return, its post-mission energy should not be assigned to the completed vehicle mission unless the model explicitly allows it.

Mission-level energy is usually:

$$
E_v^{\mathrm{use}} = E_v^{\mathrm{move}} + E_v^{\mathrm{hover}} + E_v^{\mathrm{cpu}} + E_v^{\mathrm{up}} + E_v^{\mathrm{relay}} + E_v^{\mathrm{aux}}.
$$

## 13. Per-slot energy alternative

Define slot energy $e_{v,h}\ge0$ in joules:

$$
e_{v,h} = e_{v,h}^{\mathrm{hover}} + e_{v,h}^{\mathrm{cpu}} + e_{v,h}^{\mathrm{up}} + e_{v,h}^{\mathrm{relay}}.
$$

Then:

$$
\sum_h e_{v,h}+E_v^{\mathrm{move}} \le E_v^{\max}.
$$

This form is useful for online control and residual-energy states.

## 14. Residual energy and energy headroom

Residual energy after planned operations is:

$$
E_v^{\mathrm{res}} = E_v^{\max}-E_v^{\mathrm{use}}.
$$

An energy-headroom ratio is:

$$
\rho_v^{\mathrm{energy}} = \frac{E_v^{\mathrm{res}}}{E_v^{\max}},
$$

provided $E_v^{\max}>0$. The ratio is dimensionless.

This quantity can be passed to a lower scheduling layer as route-induced information.

## 15. Energy dependence on execution mode

Mode gates indirectly control energy:

- no UAV execution implies zero onboard compute allocation and zero related compute energy;
- no UAV first hop implies zero uplink allocation and zero related receiving energy;
- no remote-via-UAV mode implies zero relay allocation and zero relay energy.

Avoid adding separate mode-energy products when the allocation gates already provide the correct zero-energy behavior.

## 16. Energy dependence on service windows

UAV compute and communication allocations should be zero outside valid service windows when those operations require station presence. Therefore, the corresponding energy terms also become zero outside those windows.

Hovering energy is directly related to dwell or presence duration.

## 17. Avoiding double counting

Common double-counting errors include:

- counting propulsion during station service in both movement and hovering terms;
- counting communication energy once from allocated nominal rate and again from effective delivered rate;
- counting UAV receiver energy and device transmitter energy in one UAV-only budget;
- counting baseline electronics power inside every component and again as auxiliary energy;
- using both continuous dwell time and slot presence for the same hovering cost;
- penalizing energy in the objective and describing the same normalized resource cost as energy.

## 18. Coefficient-unit checklist

- $\epsilon^{\mathrm{move}}$: joules per meter.
- $P^{\mathrm{fly}}$: watts.
- $P^{\mathrm{hover}}$: watts.
- $\epsilon^{\mathrm{cpu}}$: joules per work-unit.
- $\epsilon^{\mathrm{rx}}$: joules per received bit.
- $\epsilon^{\mathrm{tx}}$: joules per transmitted bit.
- $\Delta$: seconds.
- Effective-rate factors: dimensionless.

## 19. Common errors

- Adding power in watts directly to energy in joules.
- Omitting slot duration from rate-based energy.
- Treating nominal allocated rate as delivered bits without a stated convention.
- Charging uplink transmit energy to the UAV when the device transmits.
- Allowing communication energy when the related mode is not selected.
- Allowing UAV compute energy outside the service window under a strict-window model.
- Using a distance budget as a complete battery model.
- Leaving energy coefficients without units.
- Defining negative residual energy instead of enforcing the budget.

## 20. Variant guidance

- **No UAV energy budget:** remove all UAV energy variables and the budget. Keep physical route, time, and resource capacities.
- **Propulsion-only energy:** retain movement and hovering. Remove compute and communication energy.
- **Detailed wireless power model:** replace energy-per-bit approximations with transmit-power and channel-rate relations.
- **Device energy included:** add a separate per-device energy budget.
- **Remote-server energy included:** add server energy outside the UAV budget.
- **Fixed route:** movement energy becomes a known parameter. Scheduling still affects compute and communication energy.
- **Fixed service windows:** hovering energy may be fixed. MEC allocations still affect compute and communication energy.
- **Heterogeneous UAVs:** index all powers and coefficients by $v$.
- **Energy as an objective only:** possible, but safety-related battery limits should generally remain constraints.
