# VSD Course - VLSI System Design

### [Day 1 - Introduction to OpenLANE and SKY130 PDK](Day1/)

- **Theory**: [Day1-Theory.md](Day1/Day1-Theory.md)

  - Introduction to QFN-48 Package, chip components, and IPs
  - RISC-V ISA fundamentals
  - Open-source digital ASIC design components
  - Complete RTL to GDSII flow overview
  - OpenLANE and Strive chipsets introduction

- **Command Line Practice**: [Day1-CommandLine.md](Day1/Day1-CommandLine.md)
  - OpenLANE directory structure exploration
  - PDK organization and tool formats
  - Environment setup and Docker container usage
  - Design preparation and synthesis execution
  - Flip-flop ratio calculations

### [Day 2 - Floorplanning and Placement](Day2/)

- **Theory**: [Day2-Theory.md](Day2/Day2-Theory.md)

  - Floorplanning concepts and utilization factors
  - Power planning and pin placement strategies
  - Placement algorithms and optimization
  - Standard cell characterization flow

- **Command Line Practice**: [Day2-CommandLine.md](Day2/Day2-CommandLine.md)
  - Floorplan execution and configuration
  - Magic layout tool usage
  - Placement analysis and optimization
  - SPICE simulation and characterization

### [Day 3 - Standard Cell Design and Characterization](Day3/)

- **Combined Documentation**: [Day3-Theory+CommandLine.md](Day3/Day3-Theory+CommandLine.md)
  - CMOS fabrication process (16-mask process)
  - Standard cell design methodology
  - Magic layout tool advanced features
  - SPICE model integration
  - Cell characterization and timing analysis

### [Day 4 - Timing Analysis and Clock Tree Synthesis](Day4/)

- **Combined Documentation**: [Day4-Theory+CommandLine.md](Day4/Day4-Theory+CommandLine.md)
  - Grid and track information management
  - LEF file generation and integration
  - Clock tree design and timing analysis
  - Clock gating and delay table concepts
  - Static timing analysis with OpenSTA
  - Clock tree synthesis using TritonCTS
  - H-tree algorithms and signal integrity

### [Day 5 - Routing and Physical Implementation](Day5/)

- **Documentation**: [Day5.md](Day5/Day5.md)
  - Routing algorithms (Lee's Algorithm, Maze Routing)
  - Global vs detailed routing strategies
  - TritonRoute and FastRoute integration
  - Design Rule Check (DRC) for routing
  - Power distribution network generation
  - Parasitic extraction and post-route STA
