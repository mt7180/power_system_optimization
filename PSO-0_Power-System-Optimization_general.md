# Power System Optimization

Power system optimization addresses the challenges of the planning and operating electrical power systems, including generation capacity planning, minimizing operational costs and reducing emissions.  

#### Overview of Key Models in Power System Optimization
_The models/ problem formulations differ in complexity and therefore in detail and computational requirements:_

- **Basic Capacity Expansion Problem**: fundamental in planning for future system capacity needs, focusing on minimizing investment and operational costs over time.
- **Generation Expansion Planning (GEP)** : Similar to Capacity Expansion but usually considers a broader range of scenarios and longer time horizons, addressing long-term investment decisions.
- **Economic Dispatch (ED)**: core operational model that ensures power generation is allocated optimally among generators to minimize cost.
- **Economic-Emission Dispatch**: variation of economic dispatch that integrates emissions or other environmental concerns into the objective function.
- **Dynamic Economic Dispatch (DED)**: ED over a certain period of time to meet time dependant power demands at minimum operating cost. 
- **Optimal Power Flow (OPF)**: models the physics of electricity flows in power networks, adding a layer of complexity and more realism to the Economic Dispatch (ED) problem. It represents the network's topology, including transmission line connections between nodes (buses) and electrical parameters like resistance and reactance. The full Alternating Current (AC) OPF is non-convex and extremely difficult (NP-hard) to solve, so operators often use a simplified, linearized version called Direct Current (DC) OPF. This approximation is effective for most transmission networks unless they are heavily loaded or near instability.minimizes the short-run production costs of meeting electricity demand at a number of connected locations from a given set of generators subject to various technical and transmission network flow limit constraints. See also: AC-OPF / DC-OPF
- **AC Optimal Power Flow (AC-OPF)**: in its natural form, it requires the introduction of complex numbers to formulate voltage constraints. AC-OPF aims for optimizing real and reactive power flows while considering the full nonlinearities of alternating current (AC) power flow equations, such as voltage and phase angle constraints.
- **DC Optimal Power Flow (DC-OPF)**: Special case of those for AC networks, with the difference that all quantities are real. DC OPF problem approximates the AC OPF problem by making additional assumptions to produce linear constraints. While the additional assumptions result in a potential loss of solution accuracy, they make the DC OPF problem much easier to solve. Commonly used for large-scale applications due to its reduced complexity.
- **Security-Constrained Optimal Power Flow (SCOPF)**: extension of OPF that incorporates contingency analysis to ensure the system can remain secure (meet demand) even if certain lines or generators fail. 
- **Distributed Optimal Power Flow (D-OPF)**: Accounts for distributed energy resources (DERs), such as rooftop solar or small-scale storage, and optimizes power flow with these added complexities.
- **Multi-Period Optimal Power Flow (MOPF)**: Extends OPF by optimizing over multiple time periods (e.g., daily, weekly), allowing for better integration of time-varying aspects like demand and renewable generation.
- **Unit Commitment**: Important for scheduling which generators should be online to meet demand in a cost-effective manner, considering startup/shutdown costs and constraints.
- **Security-Constrained Unit Commitment (SCUC)**: extends Unit Commitment to consider contingencies and system security, ensuring that the scheduled generation can meet load requirements even after certain contingencies.
- **Stochastic/Robust Unit Commitment**: Addresses uncertainty in load, renewable generation, or system failures. These models are crucial with the increasing penetration of variable renewable energy sources (like wind and solar).
- **Optimal Transmission Expansion Planning (TEP)**: Focuses on determining the optimal placement and capacity of new transmission lines to minimize overall system costs while ensuring system reliability.
- **Optimal Reactive Power Dispatch (ORPD)**: An extension of OPF that focuses on optimizing reactive power flow to minimize losses and maintain voltage levels within prescribed limits.
- **Hydrothermal Coordination**: Models the joint operation of hydro and thermal power plants, which is important in systems with significant hydro generation.
- **Market-based OPF**: Considers market mechanisms like locational marginal pricing (LMP) for optimal pricing and dispatch decisions.
- **Co-optimization of Energy and Reserves**: Simultaneously optimizes energy and reserve markets, crucial in modern grids where balancing services are becoming increasingly necessary.