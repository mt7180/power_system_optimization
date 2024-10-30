# Optimal Power Flow (OPF): 
_Power System Optimization_  

OPF problems are used to model the entire power system network topology by representing the transmission line interconnections between different nodes for power system optimization. Each node, also known as bus, represents a specific location, where a given set of generators, consumers and storage systems inject or withdraw power into/from the network. It aims to find the optimal operating conditions for the power system network by minimizing the (short-run production) costs of meeting electricity demand within the network of connected nodes, subject to transmission flow limits, including various electrical parameters such as voltage angles, reactance, etc. 
The full alternating current or "AC" OPF is a non-convex optimization problem and turns out to be an extremely hard problem to solve. Hence, linearized versions (LOPF) are used in applications with high computational complexity where it would be impossible to use the full load flow equations. One of them is the so called "DC-OPF". The DC-OPF approximation ignores the reactive power but still works satisfactorily for bulk power transmission networks as long as such networks are not operated at the brink of instability or under heavily loaded conditions ([source](https://github.com/Power-Systems-Optimization-Course/power-systems-optimization/blob/30c3463a375f27928430581cb416a5361cee31fa/Notebooks/06-Optimal-Power-Flow.ipynb)). But voltage drops or overloaded lines due to reactive power flows cannot be considered ([source](https://www.tugraz.at/fileadmin/user_upload/tugrazExternal/f560810f-089d-42d8-ae6d-8e82a8454ca9/files/lf/Session_B2/224_LF_Recht.pdf)).

In general power flows along a transmission line l from bus i to bus j are driven by voltage phase angle differences, denoted by $\theta_i$ and $\theta_j$. The three basic assumptions used to derive the linearized DC-OPF approximation from the underlying AC-OPF problem are:

- The resistance for each branch is negligible relative to the reactance, and can therefore be approximated as ~0.
- The voltage magnitude at each bus is constant and equal to the base voltage (e.g. equal to 1 p.u). -> simplifying the normally non-linear power flow equations.
- The voltage angle difference $(\theta_j - \theta_i)$ across any branch from bus i to j is sufficiently small such that $cos(\theta_i - \theta_j) \approx 1$ and $sin(\theta_i - \theta_j) \approx (\theta_i - \theta_j)$.   
  
($\theta$ is measured in radians.)  
_for details on power flow theory see: https://nworbmot.org/courses/es-24/es-5-power_flow.pdf_


## General power flow concept (Single Period ED Optimal DC Power Flow): 
As a first step, the single period approach of DC-OPF (without storage and variable renewable generators) is formulated:  

The transmission lines (from node i to node j / where each node (=bus) could stand for one country, for example) are treated as a transport model with controllable dispatch (coupled source & sink), constrained by energy conservation at each node. The absolute flows on these transmission line interconnections cannot exceed the line capacities due to thermal limits.

Sets:   

$$
\begin{align*}
&N = \text{set of all nodes in the network} \\
&N^i = \text{set of all nodes connected to node i} \\
&G = \text{set of all generators in the network} \\
&G^i = \text{set of all generators at node i}
\end{align*}
$$
  
Objective Function:
  
$$ \min \sum_{i \in N, g \in G} VarCost_{i,g} \cdot P_{i,g} $$
subject to:  

$$ \begin{align}
    \sum_{g \in G^i} P_{g} - Demand_i = \sum_{j \in N^i} f_{i,j} \quad \leftrightarrow \lambda_i & \quad \forall i \in N \quad &\text{nodal power balance} \\
    f_{i,j} = \frac{\theta_i-\theta_j}{X_{i,j}} \quad & \quad \forall i \in N, j \in N^i &\text{power flow from node i to node j} \\
    P_{g}^{min} \leq P_g \leq P_g^{max} & \quad \forall g \in G & \quad \text{generator limits} \\
    |f_{i,j}| \leq F^{max}_{i,j} & \quad \forall i \in N, j \in N^i & \quad \text{transmission capacity limits} \\
    \theta_{slack} = 0 & & \quad \text{reference bus voltage angle}
\end{align}
$$


Parameters:

- $P^{min}_g$, the minimum operating bounds for the generator (based on engineering or natural resource constraints)
- $P^{max}_g$, the maximum operating bounds for the generator (based on engineering or natural resource constraints)
- $Demand_i$ at node i in MW
- $VarCost_{i,g}$, variable generation costs
- $F^{max}_{i,j}$ transmission capacity (for the line between node i and node j)
- $X_{i,j}$, reactance of line between node i and node j

Variables:
- $P_g$, generation dispatch by (thermal) unit g in MW
- $f_{i,j}$, power flow from node i to node j in MW
- $\theta_i$ voltage angle in node i
- $\lambda_i$, locational marginal price (LMP) in node i

Market price λ is the shadow price of the balance constraint, i.e. the cost of supply for an extra increment 1 MW of demand. ([source](https://nworbmot.org/courses/es-24/es-8-electricity_markets.pdf#page=37)) (=> increase in the objective function if demand increases)

Not considered here:   
    - fixed annualized costs for generators and transmission capacities  
    - losses 

Next, a multi-period techno-economic cost linear optimization problem for the capacity investment and dispatch of renewable and thermal generators, as well as storage that minimizes the total annual system costs is formulated, which is used in [Schlachtberger et al. 2017](https://arxiv.org/pdf/1704.05492) with pypsa.  

But first, we need some theory:

## Linear Optimal Power Flow (LOPF) with network constraints for Kirchhoff’s Voltage Law (KVL) and Kirchhoff’s Current Law (KCL)

- conventionally the linearization of the relations between power flows in the network and power injection at the buses is expressed indirectly through auxiliary variables that represent the voltage angles at the buses (see above). For pypsa a new formulation of the LOPF problem is introduced which uses the power flows directly, decomposed using graph theoretic techniques into flows on a spanning tree and flows around closed cycles in the network. This formulation involves fewer decision variables and fewer constraints than the angle-based formulation resulting in shorter calculation times to solve the LOPF formulation [Hörsch et al. 2018](https://www.sciencedirect.com/science/article/abs/pii/S0378779617305138):

| [Source: Lecture ES by T. Brown](https://nworbmot.org/courses/es-24/es-5-power_flow.pdf)  |   |
|---|---|
|$K_{i,l} = \begin{cases} 1 & \text{if edge l starts at node i} \\ −1 & \text{if edge l ends at node i} \\ 0 & \text{otherwise} \end{cases}$ | the node-edge **incidence matrix** $K \in \mathbb{R}^{N×L}$ for a directed graph (every edge has an orientation) with N nodes and L edges |
|$$ \sum_{i \in N}{p_i} = 0 \quad$$ | power balance in the whole network of nodes N - all injected power should be consumed in the network, otherwise the network would be in imbalance.|
|$$p_i = \sum_{l \in L}{K_{il}f_l \quad \forall i \in N}$$ | **Kirchhoff’s Current Law (KCL)** inforces energy conservation at each node (the power imbalance equals what goes out minus what comes in) for the linear setting.|
|$$\sum_{i \in N}{K_{il}}=0 \quad \forall l \in L$$| for a given edge l, the corresponding columns of the incidence matrix sums up to zero, since every edge starts at some node (+1) and ends at some node (-1)|
|$$\sum_{i \in N}{K_{il}\theta_i} \quad \forall l \in L$$| the voltage difference across edge l |
|$$KC = 0 $$| Since the flow in a closed cycle is balanced (the flow that enters each node along the edges of the cycle is balanced by the flow that exits)|
|$$(1)$$ $$\sum_{l \in L}{C_{l,c}}\sum_{i \in N}{K_{il}\theta_i} = 0 \quad \forall c \in C $$ | **Kirchhoff's Voltage Law (KVL)** inforces that the directed sum of voltage differences around each closed cycle add up to zero. <br>The cycles are organized in a **cycle basis matrix** $C_{l,c}$ , where c labels each cycle. |

with:
- N = set of all nodes i in the network
- L = set of all edges/ branches l in the network
- C = set of all cycles c in the network


<br>

The equations for DC circuits (Ohm's Law) and "linear power flow" in AC circuits are analogous:  

$$\begin{align*}
    I = \frac{V_i-V_j}{R} &\leftrightarrow f_l = \frac{\theta_i-\theta_j}{x_l}
\end{align*}$$   
  
with: Active power flow $f_l$ per edge l, Voltage angle $\theta_i$ at node i and Reactance X  

The voltage differece across edge l written in the logic of the incidence matrix K:  
$$\begin{align*}
    f_l &= \frac{\theta_i-\theta_j}{x_l} \\
        &= \frac{1}{x_l} \sum_i{K_{i,l}\theta_i} \\
    \Leftrightarrow f_l x_l &= \sum_i{K_{i,l}\theta_i} \quad (2)
\end{align*}$$
> instead of L different $f_l$ variables, $f_l$ depends only on N voltage angles $\theta_i$  
> and further: since the flow doesn't shift under constant shift, we can choose a slack/ reference node, such that $\theta_1=0$ 

with (1) + (2) KVL now becomes:
$$\begin{align*}\Rightarrow\sum_l{C_{l,c} f_l x_l} = 0 \quad \forall c\end{align*}$$

-> the power flows are used directly in the power balance, decomposed using graph theoretic techniques into flows on a spanning tree and flows around closed cycles in the network. This formulation involves fewer decision variables and fewer constraints than the angle-based formulation resulting in shorter calculation times to solve the LOPF formulation.

With this switch in the formulation of the power balance, in the following the  LOPF formulation for a multi-period techno-economic cost linear optimization problem with renewable and thermal generators, as well as storage is given according to [Schlachtberger et al. 2017](https://arxiv.org/pdf/1704.05492)/ pypsa. (see also [Brown, Hörsch, Schlachtberger 2018](https://arxiv.org/pdf/1707.09913))

## Multi-Period Linear Power Flow Equations
_with EES and renewable generators_




Sets:   
$$
\begin{align*}
&N = \text{set of all nodes i in the network} \\
&L = \text{set of all branches/ lines l in the network} \\
&G = \text{set of all generators g in the network} \\
\end{align*}
$$
  
Objective Function:
  
$$ \min  
    \Biggr[ 
        \sum_{i \in N, g \in G, t} w_t \cdot VarCost_{i,g} \cdot P_{i,g,t} +  
        \sum_{i \in N, g \in G} FixedCost_{i, g} \cdot CAP_{i,g} +
        \sum_{l \in L} FixedCost_l \cdot F_l^{max}
    \Biggl]
$$
subject to:  

$$ \begin{align}
    \sum_{g \in G} P_{i,g,t} + P_{i,t}^{dch} - P_{i,t}^{ch} - Demand_{i,t} = \sum_{l \in L} K_{il} f_{l,t} \quad \leftrightarrow w_t\lambda_{i,t} & \quad \forall i, t & \quad &\text{nodal power balance} \\
    P_{i,g}^{min} \leq P_{i,g,t} \leq P_{i,g}^{max} & \quad \forall i, g, t & \quad &\text{conventional generator limits} \\
    0 \leq P_{i,g,t} \leq \Lambda_{i,g,t} P_{i,g}^{max} & \quad \forall i, g, t & \quad &\text{renewable generator limits} \\
    soc_{i,s,t} = soc_{i,s,t-1} + (\eta P_{i,t}^{ch} - \frac{P_{i,t}^{dch}} {\eta})\cdot w_t & \quad \forall i,s,t & \quad &\text{storage stage of charge balance}\\
    0 \leq soc_{i,t} \leq soc_{i}^{max} & \quad \forall i, t & \quad &\text{storage capacity limits} & \quad \forall i & \quad &\text{} \\
    0 \leq P_{i,t}^{ch} \leq P^{ch,max}_i & \quad \forall i,t & \quad &\text{storage chargin limits} \\
    0 \leq P_{i,t}^{dch} \leq P^{dch,max}_i & \quad \forall i,t & \quad &\text{storage discharging limits} \\
    |f_{l,t}| \leq F^{max}_{l} & \quad \forall l, t & \quad &\text{transmission capacity limits} \\
    \sum_l{C_{l,c} f_l x_l} = 0 & \quad \forall c & \quad &\text{cycle-based KVL} \\
    \theta_{slack} = 0 & & \quad &\text{reference bus voltage angle} \\
    P_{i,g,t} - P_{i,g,t-1} \leq RampUp_{i,g} & \quad \forall i,g, t > 0 & \quad &\text{ramp up limit}\\
    P_{i,g,t-1} - P_{i,g,t} \leq RampDn_{i,g} &  \quad \forall i, g, t > 0 & \quad &\text{ramp down limit}
    
\end{align}
$$


Parameters:

- $P^{min}_g$ minimum operating bounds for the generator (based on engineering or natural resource constraints)
- $P^{max}_g$ maximum operating bounds for the generator (based on engineering or natural resource constraints)
- $Demand_{i,t}$ demand at node i and time t in MW
- $VarCost_{i,g}$ generator operational/ variable costs in €/MWh
- $FixedCost_{i,g}$ generator capital/ fixed costs in €/MW
- $CAP_g$ power capacity of generator g at node i in MW
- $F^{max}_{l}$ transmission capacity of branch l in MW
- $w_t$,
- $C_{l,c}$ , L×(L−N +1) cycle basis matrix
- $K_{il}$,
- $\Lambda_{i,g,t}$
- $\eta$

Variables:
- $P_g$, generation dispatch by (thermal) unit g in MW
- $P_{i,s,t}^{ch}$
- $P_{i,s,t}^{dch}$
- $soc$
- $f_{l}$, power flow in branch l at time t in MW
- $\theta_i$ voltage angle in node i
- $\lambda_i$, locational marginal price (LMP) in node i



### References
- https://nworbmot.org/courses/es-24/es-5-power_flow.pdf
- https://pypsa.readthedocs.io/en/latest/user-guide/optimal-power-flow.html
- https://home.engineering.iastate.edu/~jdm/ee553-Old/LMP.pdf
- https://arxiv.org/pdf/1704.05492
- https://www.sciencedirect.com/science/article/abs/pii/S0378779617305138
- https://github.com/Power-Systems-Optimization-Course/power-systems-optimization/blob/30c3463a375f27928430581cb416a5361cee31fa/Notebooks/06-Optimal-Power-Flow.ipynb
- https://www.tugraz.at/fileadmin/user_upload/tugrazExternal/f560810f-089d-42d8-ae6d-8e82a8454ca9/files/lf/Session_B2/224_LF_Recht.pdf