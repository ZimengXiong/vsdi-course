Classic Lee's algorithm is computationally expensive and memory-intensive, explores entire grid space For modern chips with millions of nets, too slow and often replaced or augmented by more advanced, faster algorithms like A\* search or used in combination with global routing strategies

## Global Routing vs Detail Routing and TritonRoute/FastRoute

Routing entire chip at once too complex, problem universally broken down into two stages:mentation

## Routing, Maze Routing, and Lee's Algorithm

Routing are process of creating physical wire connections (metal traces and vias) between pins of all cells in design, based on logical connections defined in netlist Optimization problem: for every net, goal is find shortest path between source and target pins while obeying design rules and avoiding obstacles

### Lee's Algorithm (Maze Routing)

Lee's algorithm is one of foundational algorithms for solving "shortest path" problem on grid Operates in two main phases:

#### Expansion (Wave Propagation):

- Routing area is conceptualized as a grid Starting pin is Source (S) and destination pin is Target (T)
- Algorithm begins at Source cell and labels all its immediate, non-diagonal neighbors with number '1'
- It then finds all unlabeled neighbors of all cells marked '1' and labels them '2'
- This process continues iteratively, creating a "wavefront" of increasing numbers that expands outwards from source, flowing around any obstacles (existing cells, blockages)
- Expansion phase ends when wavefront reaches Target cell

#### Traceback:

- Starting from Target cell, algorithm finds an adjacent cell with label exactly one less than its own (eg if Target is labeled '9', it looks for adjacent '8')
- It then moves to that cell and repeats process, moving from '8' to '7', '7' to '6', and so on, until it traces continuous path back to Source
- This traced path represents shortest possible connection on grid

#### Path Optimization:

- If multiple traceback paths exist, algorithm prioritizes those with fewest bends
- L-shaped routes (one bend) are highly preferred over Z-shaped routes (two bends) because each bend introduces via, which adds resistance and delay

#### Limitations:

While effective, classic Lee's algorithm is computationally expensive and memory-intensive, as it explores entire grid space For modern chips with millions of nets, it's too slow and is often replaced or augmented by more advanced, faster algorithms like A\* search or used in combination with global routing strategies

## Global Routing vs Detail Routing and TritonRoute/FastRoute

Because routing entire chip at once is too complex, problem is universally broken down into two stages:

### Global Routing (Engine: FastRoute)

**Goal:** Plan general path for each net without drawing actual wires

**Process:** Chip area is divided into large rectangular regions called G-cells Global router determines which sequence of G-cells each net will pass through

**Output:** Result is not physical route, but set of "routing guides"—collection of coarse rectangular shapes for each net that define channel or corridor for final wires Assigns routing resources and helps manage overall congestion

### Detail Routing (Engine: TritonRoute)

**Goal:** Perform final, DRC-clean routing within corridors defined by global router

**Process:** Detail router takes routing guides from FastRoute as input Job is to create exact metal segments and vias to connect pins of a net, ensuring final wires stay within their assigned guides

**Output:** Final, physically-connected layout with all wires and vias in place Output is DEF (Design Exchange Format) file

## Features of the TritonRoute Detail Router

TritonRoute is sophisticated detail router with several key features and concepts:

### Pre-processed Route Guides

Before routing, TritonRoute cleans up guides from FastRoute

- **Splitting & Merging:** It splits guides that run in "non-preferred" direction for given metal layer and merges adjacent guides to create more efficient routing space
- **Goal:** Goal is ensure that as much wire length as possible runs in preferred direction for each layer (eg vertical for Metal 1, horizontal for Metal 2) to minimize parasitic capacitance between layers

### Inter-guide Connectivity

Router must understand how different guides for same net connect to each other, either by touching on same layer or by overlapping on adjacent layers, indicates via is needed

### Intra-layer Parallel & Inter-layer Sequential Routing

#### Intra-layer Parallel:

- On single metal layer, routing area is divided into "panels"
- Routing within these panels (eg all even-numbered panels, then all odd-numbered panels) performed in parallel to accelerate process

#### Inter-layer Sequential:

- Routing proceeds one layer at time
- All Metal 1 routing finished before Metal 2 routing begins, then Metal 3 begins, and so on
- Sequential approach simplifies problem

### Access Points (APs) and Access Point Clusters (APCs)

- **Access Point (AP):** Specific, DRC-correct, on-track location where connection can be made (eg where via can be placed to connect M2 and M3)
- **Access Point Cluster (APC):** Set of all possible APs for given pin or wire segment Router's main optimization task is to find lowest-cost path to connect APCs of net together
- **Optimization:** TritonRoute uses Mixed-Integer Linear Programming (MILP) based approach to solve this complex optimization problem, finding best combination of wire segments and vias to connect all APCs for net with minimum cost (wire length and via count)

## Design Rule Check (DRC) for Routing

Design Rule Check (DRC) ensures that layout is physically manufacturable Router must adhere to complex set of rules provided by foundry

### Key DRC Rules:

- **Minimum Width:** Minimum allowable width of metal trace Wire narrower than this may not be printed correctly during fabrication
- **Minimum Spacing:** Minimum gap required between two adjacent metal traces on same layer to prevent them from shorting together
- **Signal Shorts:** Fatal error where two different nets are physically connected Routers solve this by using multiple metal layers If two nets need to cross, one is routed on different layer (eg M2) and uses vias to connect back to its original layer (eg M1) on other side
- **Via Rules:** Vias, which connect different metal layers, have their own set of rules, including minimum via size and required metal enclosure (metal on layers above and below must extend slightly beyond via cut)

## Lab Steps: Running the P&R Flow in OpenLane

### 1. Power Distribution Network (PDN) Generation

Before signal routing, power grid must be created

**Command:** In OpenLane TCL shell, run:

```tcl
gen_pdn
```

**What it Does:** Command reads PDN configuration and generates power rings, straps, and standard cell rails Connects power infrastructure from pads down to individual standard cells Process updates current DEF file

### 2. Running Routing

**Command:**

```tcl
run_routing
```

**Process:** Command initiates two-stage routing process:

- **Global Routing (FastRoute):** Generates routing guides (fastroute.guide)
- **Detail Routing (TritonRoute):** Creates final wire connections based on guides

**Verification:** After run completes, check log for final number of DRC violations Strategy used (ROUTING_STRATEGY = 0) fast but may leave few violations Violation details saved in `results/routing/<design_name>.drc`

### 3. Parasitic Extraction (SPEF Generation)

After routing, physical R and C values of wires must be extracted for accurate timing analysis

**Tool:** Standalone Python-based SPEF extractor is used

**Command:** Command requires paths to final merged.lef file (which contains physical information for all cells) and final routed def file

```bash
# From within spef_extractor directory
python3 main.py <path_to_merged.lef> <path_to_routed.def>
```

**Output:** .spef file created in same directory as .def file File contains detailed parasitic network for every net in design

### 4. Post-Route Static Timing Analysis (STA)

Final and most accurate timing analysis, uses real physical data from final layout

#### Configuration:

OpenSTA script must be updated:

- **Netlist:** Use final post-routing netlist, which is generated after antenna diode insertion (eg `...routing.v`)
- **Parasitics:** Add command to read extracted parasitics:

```tcl
read_spef <path_to_routing_results.spef>
```

#### Analysis:

Running STA with SPEF file provides sign-off quality timing report Shows whether design still meets setup and hold timing requirements after accounting for real delays of clock tree and signal interconnects If violations still exist, further ECOs or full re-run with different settings may be necessary
