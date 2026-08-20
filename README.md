# Os-and-cloud-managment-
# Zone Job-Scheduler & Cloud Security Project

##
For production deployment across the zone-controller jobs, **Priority Scheduling with Ageing** is selected as the primary algorithm family.

### Reasons Other Families Are Less Suitable:
1. **FCFS (First-Come, First-Served)**: Less suitable due to its high average waiting time caused by convoy effects; when long tasks arrive early the shorter tasks that are ready to be excecuted but force to wait until the long task is done. (e.g., `Z1-J01` with burst time 8 arrives at t=0).
2. **SJF / SRTF Family**: Less suitable because heavy processes like `Z2-J01` (burst time 9) suffer severe starvation when short jobs continually arrive in other zones; as they have less burst time than 9, they will be queued first.
3. **Round Robin**: Less suitable due to excessive context-switch overhead; at quantum 3, it generates 16 context switches across 17 slices compared to only 10 switches at quantum 6, wasting real OS CPU cycles on state saving.

---
*Blueprint documentation is at `docs/architecture_blueprint.md`.*
