# Variant and Boundary Modeling Patterns

> Scope note: This file is a deletion-and-substitution guide. It does not provide a complete answer for any held-out prompt.

## 1. How to use this file

For every changed system boundary, perform six checks:

1. Which physical entities and services still exist?
2. Which decision variables remain meaningful?
3. Which variables become fixed parameters?
4. Which constraint families must be removed or replaced?
5. Which objective components remain measurable?
6. Which route-to-service couplings still exist?

Do not leave variables or constraints for a service that has been removed.

## 2. Full joint routing and MEC

**Keep:** route assignment, arcs, timing, service availability, processor modes, communication paths, capacities, completion, deadlines, and applicable energy terms.

**Remove:** nothing by default; remove only components excluded by the user.

**Objective:** may combine logistics utility and task-service criteria after normalization.

**Coupling:** route and timing decisions create service availability for UAV-dependent computation and communication.

## 3. Routing-only problem

**Keep:** station assignment, route arcs, depot logic, payload, distance, timing, connectivity, and route energy if modeled.

**Remove:** task modes, compute and communication allocations, completion, deadlines, service-window resource gates, and digital-service energy.

**Objective:** collection value, route length, mission time, or propulsion energy.

**Coupling:** no MEC coupling remains.

## 4. Fixed routes with optimized scheduling

**Keep:** task modes, communication paths, resource allocation, completion, deadlines, and energy caused by digital service.

**Fix:** route arcs, station assignments, arrival times, and departure times as parameters.

**Replace:** endogenous presence variables may be replaced by a fixed availability parameter.

**Objective:** task completion, delay, resource use, or service energy.

**Coupling:** scheduling remains constrained by route-derived availability, but routing is not optimized.

## 5. Fixed service windows without route details

**Keep:** slot-level availability parameters, task modes, capacities, cumulative service, completion, and deadlines.

**Remove:** route arcs, depot constraints, payload, distance, route timing, and subtour constraints.

**Objective:** MEC-related criteria only, unless exogenous logistics scores are supplied.

**Coupling:** only the fixed availability-to-resource gate remains.

## 6. MEC-only with continuous infrastructure

**Keep:** task arrivals, processor selection, capacity, cumulative work, completion, and deadlines.

**Remove:** UAV routing, station assignment, service-window variables, and UAV mobility energy.

**Communication:** retain only links that physically exist.

**Coupling:** no route-induced availability remains.

## 7. Local and UAV computing only

**Keep:** local mode, UAV mode, uplink path, local/UAV capacities, service-window gates, completion, deadlines, and applicable UAV compute/reception energy.

**Remove:** remote mode, relay/backhaul variables, remote capacity, relay sufficiency, and remote-server terms.

**Objective:** remove remote-resource terms.

## 8. No remote or cloud execution

This is equivalent to removing the remote processor and every communication hop used only for remote execution. Do not retain a relay variable merely because an uplink still exists for UAV execution.

## 9. No UAV computing

**Keep:** local mode and, if allowed, remote execution through a UAV communication path.

**Remove:** onboard-compute mode, onboard compute allocation, UAV compute capacity, and onboard computing energy.

**Coupling:** uplink and relay may still depend on station presence.

## 10. Remote execution through a UAV relay

**Keep:** remote mode, first-hop activation, uplink, relay/backhaul, remote compute, data-delivery causality, and service availability for required UAV communication.

**Remove:** direct device-to-remote variables unless that path also exists.

**Warning:** selecting a remote processor does not by itself deliver input data.

## 11. Direct device-to-cloud execution

**Keep:** direct-link variables and capacity if the physical system supports them.

**Remove or relax:** UAV-presence gates for the direct path.

**Do not mix:** direct and UAV-assisted paths without a path-selection variable.

## 12. No UAV energy budget

**Remove:** mission energy inequality, energy-component variables, and residual-energy constraints.

**Keep:** payload, distance, timing, and resource capacities if still physical limits.

**Objective:** remove energy cost unless energy is still measured externally.

## 13. Propulsion-only energy

**Keep:** movement and dwell energy.

**Remove:** onboard computing, reception, and relay energy from the UAV battery model.

**Warning:** digital-service allocations may still be constrained by capacity even when their energy is ignored.

## 14. Deadline-only service objective

**Keep:** completion and on-time indicators, deadline logic, and all feasibility constraints needed to process tasks.

**Remove from objective:** flow-time and resource-occupation terms unless the user requests tie-breaking criteria.

**Avoid double counting:** one on-time reward and one miss penalty may be redundant when they use the same indicator.

## 15. No resource-occupation objective

**Keep:** physical capacity constraints.

**Remove:** normalized compute/link utilization from the objective.

**Warning:** deleting a utilization penalty does not permit capacity violation.

## 16. Single UAV

**Simplify:** remove the vehicle index where clarity improves.

**Keep:** depot route, station sequence, capacity, timing, service availability, and energy.

**Remove:** inter-UAV exclusivity or coordination rules that no longer apply.

## 17. Multiple UAVs

**Keep:** per-UAV routes, capacities, timing, service windows, and energy.

**Add when required:** station exclusivity, collision avoidance, shared-spectrum constraints, or fleet-level budgets.

**Do not assume:** homogeneous UAVs unless stated.

## 18. No payload constraint

**Remove:** payload variables and carrying-capacity inequalities.

**Keep:** route feasibility, distance, timing, and energy.

**Objective:** collection value remains valid if items have no modeled mass limit.

## 19. No travel-distance constraint

**Remove:** explicit route-distance budget.

**Keep:** travel time and propulsion energy if they are still modeled.

**Warning:** an energy budget may indirectly limit distance, but it is not identical unless the energy model makes that equivalence explicit.

## 20. Hard deadlines

**Keep:** deadline inequalities as mandatory feasibility conditions for admitted tasks.

**Remove or reduce:** miss penalties when misses are forbidden.

**Admission:** use an admission variable if infeasible tasks may be rejected.

## 21. Soft deadlines

**Keep:** completion time and lateness or on-time indicators.

**Objective:** penalize lateness, misses, or tardiness.

**Allow:** completed-late tasks when useful to the application.

## 22. Continuous-time model

**Keep:** event times, route timing, continuous service intervals, and continuous resource integrals or event-based allocations.

**Remove:** slot indices and slot-completion variables.

**Replace:** slot presence with interval-overlap logic.

**Units:** integrals of rate over time produce work or transmitted data.

## 23. Slot-based model

**Keep:** slot duration, arrival slots, per-slot capacities, cumulative sums, and slot-level presence.

**Replace:** continuous travel/service events with compatible slot approximations when exact event times are not used.

**Warning:** define whether a slot represents its start, end, or full interval.

## 24. Deterministic task arrivals

**Treat:** arrival times and task attributes as known parameters.

**Keep:** scheduling and resource constraints.

**Remove:** stochastic expectation or chance constraints unless another uncertainty remains.

## 25. Stochastic task arrivals

**Choose one approach:** scenario-based stochastic programming, robust bounds, chance constraints, online control, or an MDP.

**Do not use future information:** an online policy cannot allocate resources to unseen tasks.

**Evaluation:** report the arrival distribution or scenario-generation method.

## 26. Optional task admission

Introduce $a_q^{\mathrm{adm}}\in\{0,1\}$. Replace exactly-one-mode equality by:

$$
\sum_{p\in\mathcal{P}}m_{q,p}=a_q^{\mathrm{adm}}.
$$

Gate resource allocations and completion by $a_q^{\mathrm{adm}}$. The objective may reward admission or penalize rejection.

## 27. Multiple visits or repeated service

A single assignment variable and one-in/one-out flow may be insufficient. Use visit indices, time-expanded nodes, or route-position variables. Service windows and collected quantity must be linked to each visit rather than only to the station.

## 28. Post-departure onboard processing

Gate the initial upload by station presence. Introduce a task-onboard or uploaded-data state. UAV computation after departure then depends on the onboard state and UAV compute capacity, not current station presence.

## 29. Post-departure relay

Introduce a UAV data buffer. The buffer increases with received uplink data and decreases with relay transmission. Relay after departure is feasible only when buffered data and backhaul availability exist.

## 30. Output-return traffic

If processed results are non-negligible, add output-data size, downlink or return-path variables, capacity, cumulative delivery, and energy. If output size is assumed negligible, state the assumption explicitly.

## 31. Boundary-control checklist

Before finalizing a model, verify:

- every processor and communication hop exists physically;
- every selected mode has the required data path;
- fixed decisions are parameters, not free variables;
- removed services leave no orphan variables;
- local service is not accidentally gated by UAV presence;
- UAV-dependent service is not available outside its valid window;
- hard constraints are not replaced by weak objective penalties;
- rate-times-time conversions include the correct slot duration;
- completion, on-time status, and admission are distinct when required;
- objective terms match the chosen system boundary.
