# 7-OpenLANE Directory structure in detail

## Exploration

### Openlane Directory

````bash
beaver@openlanevm:~/Desktop/work/tools/openlane_working_dir/openlane$ ls
AUTHORS.md       docs        install          regression_results     tests
configuration    empty       Jenkinsfile      requirements_dev.txt   venv
CONTRIBUTING.md  env.py      klayoutrc        requirements_lint.txt  vsdstdcelldesign
default.nix      flake.lock  LICENSE          requirements.txt
dependencies     flake.nix   Makefile         run_designs.py
designs          flow.tcl    Makefile.backup  runs
docker           gui.py      README.md        scripts
beaver@openlanevm:~/Desktop/work/tools/openlane_working_dir/openlane$ ```
````

### PDK

#### Formats for tools

```bash
beaver@openlanevm:~/Desktop/work/tools/openlane_working_dir/pdks/sky130A$ ls
libs.ref  libs.tech  SOURCES
beaver@openlanevm:~/Desktop/work/tools/openlane_working_dir/pdks/sky130A$ ls libs.tech/
irsim  klayout  magic  netgen  ngspice  openlane  qflow  xcircuit  xschem
beaver@openlanevm:~/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.ref/sky130_fd_sc_hd$ ls
cdl  doc  gds  lef  lib  mag  maglef  spice  techlef  verilog
```

### Opening the Openlane ENV

Navigating to the openlane directory, running `sudo make mount` opens the docker container.

```bash
beaver@openlanevm:~/Desktop/work/tools/openlane_working_dir/openlane$ sudo make mount
[sudo] password for beaver:
cd /home/beaver/Desktop/work/tools/openlane_working_dir/openlane && \
	docker run --rm -v /root:/root -v /home/beaver/Desktop/work/tools/openlane_working_dir:/home/beaver/Desktop/work/tools/openlane_working_dir -v /empty:/home/beaver/Desktop/work/tools/openlane_working_dir/openlane/install -w /home/beaver/Desktop/work/tools/openlane_working_dir/openlane -v /root/.volare:/root/.volare -e PDK_ROOT=/root/.volare -e PDK=sky130A  --user 0:0 -e DISPLAY= -v /tmp/.X11-unix:/tmp/.X11-unix -v /root/.Xauthority:/.Xauthority --network host --security-opt seccomp=unconfined -ti efabless/openlane:e0d2e618a8834e733d28dfcc02fa166941be7e71-arm64v8
OpenLane Container:/home/beaver/Desktop/work/tools/openlane_working_dir/openlane%
```

### Designs

Openlane bundles separate preconfigured designs:

```bash
OpenLane Container:/home/beaver/Desktop/work/tools/openlane_working_dir/openlane% ls designs/ci
APU			 blabla		      mem_1r1w       test_sram_macro  y_huff
PPU			 caravel_upw		      picorv32a      tt05_i2c_bert    zipdiv
Readme.md		 gcd			      regfile_2r1w   usb
aes			 inverter		      reproducibles  usb_cdc_core
aes_core		 io_placer		      s44	     wbqspiflash
aes_user_project_wrapper  manual_macro_placement_test  salsa20	     xtea
```

## Flow Design Preparation

### Opening the Flow ENV

The flow environment should be launched within the Openlane ENV.

It can be launched with:

```bash
OpenLane Container:/home/beaver/Desktop/work/tools/openlane_working_dir/openlane% ./flow.tcl -interactive
OpenLane e0d2e618a8834e733d28dfcc02fa166941be7e71
All rights reserved. (c) 2020-2023 Efabless Corporation and contributors.
Available under the Apache License, version 2.0. See the LICENSE file for more details.

%
```

### Initialize openlane/load designs

```bash
% package require openlane
0.9
% prep -design designs/ci/picorv32a
[INFO]: Using configuration in 'designs/ci/picorv32a/config.json'...
[INFO]: PDK Root: /root/.volare
[INFO]: Process Design Kit: sky130A
[INFO]: Standard Cell Library: sky130_fd_sc_hd
[INFO]: Optimization Standard Cell Library: sky130_fd_sc_hd
[INFO]: Run Directory: /home/beaver/Desktop/work/tools/openlane_working_dir/openlane/designs/ci/picorv32a/runs/RUN_2025.07.17_18.16.46
[INFO]: Saving runtime environment...
[INFO]: Preparing LEF files for the nom corner...
[INFO]: Preparing LEF files for the min corner...
[INFO]: Preparing LEF files for the max corner...
[WARNING]: PNR_SDC_FILE is not set. It is recommended to write a custom SDC file for the design. Defaulting to BASE_SDC_FILE
[WARNING]: SIGNOFF_SDC_FILE is not set. It is recommended to write a custom SDC file for the design. Defaulting to BASE_SDC_FILE
%
```

This does the following:

- Create a new "RUN", located in `/openlane/designs/ci/picorv32a/runs`

- Sets up the env for openlane flow (synthesis, floorplanning, etc)

### Synthesis

Run synthesis with

```bash
% run_synthesis
[STEP 1]
[INFO]: Running Synthesis (log: designs/ci/picorv32a/runs/RUN_2025.07.17_18.16.46/logs/synthesis/1-synthesis.log)...
[STEP 2]
[INFO]: Running Single-Corner Static Timing Analysis (log: designs/ci/picorv32a/runs/RUN_2025.07.17_18.16.46/logs/synthesis/2-sta.log)...
```

This converts the RTL VHDL into a gate-level netlist of standard cells.

We can preview the synthesis output with

```bash
OpenLane Container:/home/beaver/Desktop/work/tools/openlane_working_dir/openlane% tail designs/ci/picorv32a/runs/RUN_2025.07.17_18.16.46/logs/synthesis/1-synthesis.log -n 20
     sky130_fd_sc_hd__or2_2        294
     sky130_fd_sc_hd__or2b_2        30
     sky130_fd_sc_hd__or3_2         41
     sky130_fd_sc_hd__or3b_2        18
     sky130_fd_sc_hd__or4_2         25
     sky130_fd_sc_hd__or4b_2         5
     sky130_fd_sc_hd__or4bb_2        3
     sky130_fd_sc_hd__xnor2_2       70
     sky130_fd_sc_hd__xor2_2        37

   Chip area for module '\picorv32': 98792.249600

64. Executing Verilog backend.
Dumping module `\picorv32'.

65. Executing JSON backend.

End of script. Logfile hash: a25e75eb99, CPU: user 2.89s system 0.07s, MEM: 78.05 MB peak
Yosys 0.38 (git sha1 543faed9c8c, clang++ 16.0.6 -fPIC -Os)
Time spent: 69% 2x abc (6 sec), 7% 41x opt_expr (0 sec), ...
```

### Calculating flip flop ratio

We can find the flip flop count looking for the `dfxtp` line.

```bash
OpenLane Container:/home/beaver/Desktop/work/tools/openlane_working_dir/openlane% cat designs/ci/picorv32a/runs/RUN_2025.07.17_18.16.46/logs/synthesis/1-synthesis.log | grep "dfxtp"
  cell sky130_fd_sc_hd__dfxtp_2 (noninv, pins=3, area=21.27) is a direct match for cell type $_DFF_P_.
    \sky130_fd_sc_hd__dfxtp_2 _DFF_P_ (.CLK( C), .D( D), .Q( Q));
  mapped 1596 $_DFF_P_ cells to \sky130_fd_sc_hd__dfxtp_2 cells.
     sky130_fd_sc_hd__dfxtp_2     1596
ABC: Scl_LibertyReadGenlib() skipped sequential cell "sky130_fd_sc_hd__dfxtp_2".
ABC: Scl_LibertyReadGenlib() skipped sequential cell "sky130_fd_sc_hd__dfxtp_4".
     sky130_fd_sc_hd__dfxtp_2     1596
```

We can see the number of cells looking for `Number of cells:`, searching with vim

```bash
42.2. Analyzing design hierarchy..
Top module:  \picorv32
Removed 0 unused modules.

43. Printing statistics.

=== picorv32 ===

   Number of wires:               6301
   Number of wire bits:           8874
   Number of public wires:         179
   Number of public wire bits:    2304
   Number of memories:               0
   Number of memory bits:            0
   Number of processes:              0
   Number of cells:               8116
     $_ANDNOT_                    1159
     $_AND_                        399
     $_DFFE_PP_                   1240
     $_DFF_P_                       91
     $_MUX_                       2709
     $_NAND_                       226
     $_NOR_                        175
     $_NOT_                        144
     $_ORNOT_                      165
     $_OR_                        1042
     $_SDFFCE_PN0P_                 34
     $_SDFFCE_PP0P_                  6
     $_SDFFE_PN0N_                   1
     $_SDFFE_PN0P_                 153
     $_SDFFE_PP0P_                   1
/Number of cells:
```
