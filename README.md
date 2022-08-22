# Advance Physical Design Using OpenLANE/Sky130

This project is done in the course ["Advanced Physical Design using OpenLANE/Sky130"](https://www.vlsisystemdesign.com/advanced-physical-design-using-openlane-sky130/) by VLSI System Design Corporation. In this project a complete RTL to GDSII flow for PicoRV32a SoC is executed with Openlane using Skywater130nm PDK. Custom designed standard cells with Sky130 PDK are also used in the flow. Timing Optimisations are carried out. Slack violations are removed. DRC is verified.


## Table of Contents
1. [Overview](#overview)
2. [Inception of Opensource EDA](#inception-of-opensource-eda)
   - [How to Talk to computers?](#how-to-talk-to-computers)
   - [SOC Design & OpenLANE](#soc-design--openlane)
     - [Components of opensource digital ASIC design](#components-of-opensource-digital-asic-design)
     - [Simplified RTL2GDS Flow](#simplified-rtl2gds-flow)
     - [OpenLANE ASIC Flow](#openlane-asic-flow)
   - [Opensource EDA tools](#opensource-eda-tools)
     - [OpenLANE design stages](#openlane-design-stages)
     - [OpenLANE Files](#openlane-files)
     - [Invoking OpenLANE](#invoking-openlane)
     - [Design Preparation Step](#design-preparation-step)
     - [Review of files & Synthesis step](#design-preparation-step#review-of-files-&-synthesis-step)
3. [Floorplanning & Placement and library cells](#floorplanning-&-placement-and-library-cells)
   - [Floorplanning considerations](#floorplanning-considerations)
     - [Utilization Factor & Aspect Ratio](#utilization-factor--aspect-ratio)
     - [Pre-placed cells](#pre-placed-cells)
     - [Decoupling capacitors](#decoupling-capacitors)
     - [Power Planning](#power-planning)
     - [Pin Placement](#pin-placement)
     - [Floorplan run on OpenLANE & view in Magic](#floorplan-run-on-openlane--view-in-magic)
   - [Placement](#placement)
     - [Placement Optimization](#placement-optimization)
     - [Placement run on OpenLANE & view in Magic](#placement-run-on-openlane--view-in-magic)
   - [Standard Cell Design Flow](#standard-cell-design-flow)
   - [Standard Cell Characterization Flow](#standard-cell-characterization-flow)
   - [Timing Parameter Definitions](#timing-parameter-definitions)
4. [Design library cell](#design-library-cell)
   - [SPICE Deck creation & Simulation](#spice-deck-creation--simulation)
   - [CMOS inverter Switching Threshold Vm](#cmos-inverter-switching-threshold-vm)
   - [16 Mask CMOS Fabrication](#16-mask-cmos-fabrication)
   - [Inverter Standard cell Layout & SPICE extraction](#inverter-standard-cell-layout--spice-extraction)
   - [Inverter Standard cell characterization](#inverter-standard-cell-characterization)
   - [Magic Features & DRC rules](#magic-features--drc-rules)
5. [Timing Analysis & CTS](#timing-analysis--cts)
   - [Create port definition](#create-port-definition)
   - [Standard Cell LEF generation](#standard-cell-lef-generation)
   - [Integrating custom cell in OpenLANE](#integrating-custom-cell-in-openlane)
   - [Post-synthesis timing analysis](#post-synthesis-timing-analysis)
   - [Clock Tree Synthesis](#clock-tree-synthesis)
6. [Final steps in RTL2GDS](#final-steps-in-rtl2gds)
   - [Power Distribution Network generation](#power-distribution-network-generation)
   - [Routing](#routing)
   - [GDSII](#gdsii)
   - [LEF](#lef)
7. [Design folder](#design-folder)   
8. [Differences from older OpenLANE versions](#differences-from-older-openlane-versions)
9. [Summary]{#summary)
   - [VLSI NON INTERACTIVE OPENLANE FLOW](#vlsi-non-interactive-openlane-flow)
   - [VLSI INTERACTIVE OPENLANE FLOW](#vlsi-interactive-openlane-flow)
11. [Acknowledgements](#acknowledgements)
12. [Author](#author)


## Overview
OpenLANE is an opensource tool or flow used for opensource tape-outs. The OpenLANE flow comprises a variety of tools such as Yosys, ABC, OpenSTA, Fault, OpenROAD app, Netgen and Magic which are used to harden chips and macros, i.e. generate final GDSII from the design RTL. The primary goal of OpenLANE is to produce clean GDSII with no human intervention. OpenLANE has been tuned to function for the Google-Skywater130 Opensource Process Design Kit.

## Inception of Opensource EDA

### How to talk to computers?

The RISC-V Instruction Set Architecture (ISA) is a language used to talk to computers whose hardware is based on RISC-V core. If a user wishes to run a certain application software on a computer, its corresponding C/C++/Java program must be converted into instructions by the compliler. The ouput of the compiler is hardware dependent. These instructions go as inputs to the assembler which outputs binary language that the hardware logic in the chip layout can make sense of. According to the bits received, the digital logic consisting of gates performs the function required by the user of the application software.

### SoC Design & OpenLANE

#### Components of opensource digital ASIC design
The design of digital Application Specific Integrated Circuit (ASIC) requires three enablers or elements - Resistor Transistor Logic Intellectual Property (RTL IPs), Electronic Design Automation (EDA) Tools and Process Design Kit (PDK) data.

![ASIC](https://user-images.githubusercontent.com/86701156/124005001-35a32a80-d9f6-11eb-8fcc-0917ad337699.PNG)

- Opensource RTL Designs: github, librecores, opencores
- Opensource EDA tools: QFlow, OpenROAD, OpenLANE
- Opensource PDK data: Google Skywater130 PDK

The ASIC flow objective is to convert RTL design to GDSII format used for final layout. The flow is essentially a software also known as automated PnR (Place & route).

#### Simplified RTL2GDS Flow

![RTL2GDS flow](https://user-images.githubusercontent.com/86701156/124006238-a139c780-d9f7-11eb-8da9-6069b055fbe0.PNG)

![RTLtoGDSIIflow](https://user-images.githubusercontent.com/83152452/131134578-5cd34ec9-a388-476b-aa4b-914c250d7ec9.png)

- Synthesis: RTL Converted to gate level netlist using standard cell libraries (SCL)
- Floor & Power Planning: Planning of silicon area to ensure robust power distribution
- Placement: Placing cells on floorplan rows aligned with sites
  - Global Placement: for optimal position of cells
  - Detailed Placement: for legal positions
- Routing: Valid patterns for wires
- Signoff: Physical (DRC, LVS) and Timing verifications (STA)

#### OpenLANE ASIC Flow

![3  opensource ASIC digital designs-1](https://user-images.githubusercontent.com/83152452/185787620-8c999b89-2580-477d-aa20-1156d3e996c8.png)

![OpenLaneflow](https://user-images.githubusercontent.com/83152452/131135115-46148ff1-9489-48f6-a334-6702c25def59.png)

From conception to product, the ASIC design flow is an iterative process that is not static for every design. The details of the flow may change depending on ECO’s, IP requirements, DFT insertion, and SDC constraints, however the base concepts still remain. The flow can be broken down into 11 steps:

1. Architectural Design – A system engineer will provide the VLSI engineer with specifications for the system that are determined through physical constraints. 
   The VLSI engineer will be required to design a circuit that meets these constraints at a microarchitecture modeling level.

2. RTL Design/Behavioral Modeling – RTL design and behavioral modeling are performed with a hardware description language (HDL). 
   EDA tools will use the HDL to perform mapping of higher-level components to the transistor level needed for physical implementation. 
   HDL modeling is normally performed using either Verilog or VHDL. One of two design methods may be employed while creating the HDL of a microarchitecture:
   
    a. RTL Design – Stands for Register Transfer Level. It provides an abstraction of the digital   circuit using:
   
   - i. 	 Combinational logic
   - ii. 	 Registers
   - iii.  Modules (IP’s or Soft Macros)
 
    b. 	Behavioral Modeling – Allows the microarchitecture modeling to be performed with behavior-based modeling in HDL. This method bridges the gap between C and HDL allowing HDL design to be performed

3. RTL Verification - Behavioral verification of design

4. DFT Insertion - Design-for-Test Circuit Insertion

5. Logic Synthesis – Logic synthesis uses the RTL netlist to perform HDL technology mapping. The synthesis process is normally performed in two major steps:

     - GTECH Mapping – Consists of mapping the HDL netlist to generic gates what are used to perform logical optimization based on AIGERs and other topologies created 
       from the generic mapped netlist.
       
     - Technology Mapping – Consists of mapping the post-optimized GTECH netlist to standard cells described in the PDK
  
6. Standard Cells – Standard cells are fixed height and a multiple of unit size width. This width is an integer multiple of the SITE size or the PR boundary. Each standard cell comes with SPICE, HDL, liberty, layout (detailed and abstract) files used by different tools at different stages in the RTL2GDS flow.

7. Post-Synthesis STA Analysis: Performs setup analysis on different path groups.

8. Floorplanning – Goal is to plan the silicon area and create a robust power distribution network (PDN) to power each of the individual components of the synthesized netlist. In addition, macro placement and blockages must be defined before placement occurs to ensure a legalized GDS file. In power planning we create the ring which is connected to the pads which brings power around the edges of the chip. We also include power straps to bring power to the middle of the chip using higher metal layers which reduces IR drop and electro-migration problem.

9. Placement – Place the standard cells on the floorplane rows, aligned with sites defined in the technology lef file. Placement is done in two steps: Global and Detailed. In Global placement tries to find optimal position for all cells but they may be overlapping and not aligned to rows, detailed placement takes the global placement and legalizes all of the placements trying to adhere to what the global placement wants.

10. CTS – Clock tree synteshsis is used to create the clock distribution network that is used to deliver the clock to all sequential elements. The main goal is to create a network with minimal skew across the chip. H-trees are a common network topology that is used to achieve this goal.

11. Routing – Implements the interconnect system between standard cells using the remaining available metal layers after CTS and PDN generation. The routing is performed on routing grids to ensure minimal DRC errors.

The Skywater 130nm PDK uses 6 metal layers to perform CTS, PDN generation, and interconnect routing.

### Opensource EDA tools

OpenLANE utilises a variety of opensource tools in the execution of the ASIC flow:
Task | Tool/s
------------ | -------------
RTL Synthesis & Technology Mapping | [yosys](https://github.com/YosysHQ/yosys), abc
Floorplan & PDN | init_fp, ioPlacer, pdn and tapcell
Placement | RePLace, Resizer, OpenPhySyn & OpenDP
Static Timing Analysis | [OpenSTA](https://github.com/The-OpenROAD-Project/OpenSTA)
Clock Tree Synthesis | [TritonCTS](https://github.com/The-OpenROAD-Project/OpenLane)
Routing | FastRoute and [TritonRoute](https://github.com/The-OpenROAD-Project/TritonRoute) 
SPEF Extraction | [SPEF-Extractor](https://github.com/HanyMoussa/SPEF_EXTRACTOR)
DRC Checks, GDSII Streaming out | [Magic](https://github.com/RTimothyEdwards/magic), [Klayout](https://github.com/KLayout/klayout)
LVS check | [Netgen](https://github.com/RTimothyEdwards/netgen)
Circuit validity checker | [CVC](https://github.com/d-m-bailey/cvc)

#### OpenLANE design stages

1. Synthesis
	- `yosys` - Performs RTL synthesis
	- `abc` - Performs technology mapping
	- `OpenSTA` - Performs static timing analysis on the resulting netlist to generate timing reports
2. Floorplan and PDN
	- `init_fp` - Defines the core area for the macro as well as the rows (used for placement) and the tracks (used for routing)
	- `ioplacer` - Places the macro input and output ports
	- `pdn` - Generates the power distribution network
	- `tapcell` - Inserts welltap and decap cells in the floorplan
3. Placement
	- `RePLace` - Performs global placement
	- `Resizer` - Performs optional optimizations on the design
	- `OpenDP` - Perfroms detailed placement to legalize the globally placed components
4. CTS
	- `TritonCTS` - Synthesizes the clock distribution network (the clock tree)
5. Routing
	- `FastRoute` - Performs global routing to generate a guide file for the detailed router
	- `CU-GR` - Another option for performing global routing.
	- `TritonRoute` - Performs detailed routing
	- `SPEF-Extractor` - Performs SPEF extraction
6. GDSII Generation
	- `Magic` - Streams out the final GDSII layout file from the routed def
	- `Klayout` - Streams out the final GDSII layout file from the routed def as a back-up
7. Checks
	- `Magic` - Performs DRC Checks & Antenna Checks
	- `Klayout` - Performs DRC Checks
	- `Netgen` - Performs LVS Checks
	- `CVC` - Performs Circuit Validity Checks

#### OpenLANE Files

The openLANE file structure looks something like this:

- skywater-pdk: contains PDK files provided by foundry
- open_pdks: contains scripts to setup pdks for opensource tools 
- sky130A: contains sky130 pdk files

#### Invoking OpenLANE and Design Preparation 

Openlane can be invoked using docker command followed by opening an interactive session.
flow.tcl is a script that specifies details for openLANE flow.

```
docker
./flow.tcl -interactive
package require openlane 0.9
```

```
prep -design picorv32a
```

![6  openlane step - prep -design picorv32a](https://user-images.githubusercontent.com/83152452/185787631-bbd5fb79-ce07-4d4c-a058-23894c290c5d.png)


#### Review of files & Synthesis step
* A "runs" folder is generated within the picorv32a folder.
* The merged file is created during the merging operation in the pircorv32a design preparation (it merges lef and techlef files)
* Next, we run the synthesis of picorv32a design in the openlane interactive terminal:

`run_synthesis`

![openlane - placement - run_synthesis](https://user-images.githubusercontent.com/83152452/185788656-fd086122-f98e-4fc0-bd4e-a711555c8435.png)


* The yosys and ABC tools are utilised to convert RTL to gate level netlist.
* Calcuation of Flop Ratio:
```
Flop ratio = Number of D Flip flops 
             ______________________
             Total Number of cells
```

```
dfxtp_4 = 1613,
Number of cells = 14876,
Flop ratio = 1613/14876 = 0.1084 = 10.84%
```
* We may check the success of the synthesis step by checking the synthesis folder for the synthesized netlist file (.v file)
* The synthesis statistics report can be accessed within the reports directory. It is usually the last yosys file since files are listed chronologically by date of modification.
* The synthesis timings report are as follows:

![7  synthesis report-timing](https://user-images.githubusercontent.com/83152452/185787646-954e92b2-c2c0-47ed-be00-ccabd63b3cd6.png)

![7  synthesis report-timing-1](https://user-images.githubusercontent.com/83152452/185787648-8672040d-418b-4202-9d3b-a148259c20a0.png)

* The synthesis power report is as follows:

![7  report_power](https://user-images.githubusercontent.com/83152452/185787655-bf47f6ef-eb12-412f-9e3b-ff5da6946190.png)



## Floorplanning & Placement and library cells

### Floorplanning considerations

#### Utilization Factor & Aspect Ratio  

Two parameters are of importance when it comes to floorplanning namely, Utilisation Factor and Aspect Ratio. They are defined as follows:

```
Utilisation Factor =  Area occupied by netlist
                     __________________________
                        Total area of core
```

```
Aspect Ratio =  Height
               ________
                Width
```
                                  
A Utilisation Factor of 1 signifies 100% utilisation leaving no space for extra cells such as buffer. However, practically, the Utilisation Factor is 0.5-0.6. Likewise, an Aspect ratio of 1 implies that the chip is square shaped. Any value other than 1 implies rectanglular chip.

#### Pre-placed cells

Once the Utilisation Factor and Aspect Ratio has been decided, the locations of pre-placed cells need to be defined. Pre-placed cells are IPs comprising large combinational logic which once placed maintain a fixed position. Since they are placed before placement and routing, the are known as pre-placed cells.

#### Decoupling capacitors

Pre-placed cells must then be surrounded with decoupling capacitors (decaps). The resistances and capacitances associated with long wire lengths can cause the power supply voltage to drop significantly before reaching the logic circuits. This can lead to the signal value entering into the undefined region, outside the noise margin range. Decaps are huge capacitors charged to power supply voltage and placed close the logic circuit. Their role is to decouple the circuit from power supply by supplying the necessary amount of current to the circuit. They pervent crosstalk and enable local communication.

#### Power Planning

Each block on the chip, however, cannot have its own decap unlike the pre-placed macros. Therefore, a good power planning ensures that each block has its own VDD and VSS pads connected to the horizontal and vertical power and GND lines which form a power mesh.

#### Pin Placement

The netlist defines connectivity between logic gates. The place between the core and die is utilised for placing pins. The connectivity information coded in either VHDL or Verilog is used to determine the position of I/O pads of various pins. Then, logical placement blocking of pre-placed macros is performed so as to differentiate that area from that of the pin area.

#### Floorplan run on OpenLANE & view in Magic

* Importance files in increasing priority order:

1. ```floorplan.tcl``` - System default envrionment variables
2. ```conifg.tcl```
3. ```sky130A_sky130_fd_sc_hd_config.tcl```

* Floorplan envrionment variables or switches:

1. ```FP_CORE_UTIL``` - floorplan core utilisation
2. ```FP_ASPECT_RATIO``` - floorplan aspect ratio
3. ```FP_CORE_MARGIN``` - Core to die margin area
4. ```FP_IO_MODE``` - defines pin configurations (1 = equidistant/0 = not equidistant)
5. ```FP_CORE_VMETAL``` - vertical metal layer
6. ```FP_CORE_HMETAL``` - horizontal metal layer

***Note: Usually, vertical metal layer and horizontal metal layer values will be 1 more than that specified in the files***
 
 To run the picorv32a floorplan in openLANE:
 ```
 run_floorplan
 
 ```
 ![openlane - placement - run_floorplan](https://user-images.githubusercontent.com/83152452/185788662-a73dca29-98a9-4b78-9297-61f8f9bc7a47.png)
 

Post the floorplan run, a .def file will have been created within the ```results/floorplan``` directory. 
We may review floorplan files by checking the ```floorplan.tcl```. 
The system defaults will have been overriden by switches set in ```conifg.tcl``` and further overriden by switches set in ```sky130A_sky130_fd_sc_hd_config.tcl```.
 
To view the floorplan, Magic is invoked after moving to the ```results/floorplan``` directory:

```
magic -T /home/devipriya/OpenLane/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.min.lef def read picorv32a.def &
```

![openlane - floorplan -2](https://user-images.githubusercontent.com/83152452/185788777-f89ce25e-55aa-45c2-be4b-50c2176a127b.png)
 
* One can zoom into Magic layout by selecting an area with left and right mouse click followed by pressing "z" key. 
* Various components can be identified by using the ```what``` command in tkcon window after making a selection on the component.
* Zooming in also provides a view of decaps present in picorv32a chip.
* The standard cell can be found at the bottom left corner.


![openlane - floorplan -3- zoom in view - merged min lef - 1](https://user-images.githubusercontent.com/83152452/185788801-8e6d99b4-fec5-4d9d-8f4d-4e4bb16e8d17.png)

![openlane - floorplan -3- zoom in view - merged min lef - tap cells](https://user-images.githubusercontent.com/83152452/185788804-0256235e-17f9-43c3-af3b-3a9421c3c8d9.png)

![openlane - floorplan -3- zoom in view - merged min lef](https://user-images.githubusercontent.com/83152452/185788806-a7fd6c16-963d-4e4f-bacc-f827e105a250.png)


### Placement 

#### Placement Optimization

The next step in the OpenLANE ASIC flow is placement. The synthesized netlist is to be placed on the floorplan. Placement is perfomed in 2 stages:

1. Global Placement: It finds optimal position for all cells which may not be legal and cells may overlap. Optimization is done through reduction of half parameter wire length.
2. Detailed Placement: It alters the position of cells post global placement so as to legalise them.

Legalisation of cells is important from timing point of view. 

#### Placement run on OpenLANE & view in Magic

Command:

```
run_placement
```

![openlane - placement - run_placement-1](https://user-images.githubusercontent.com/83152452/185789104-a2284118-7e56-439e-abac-bb1ba669e72b.png)

The objective of placement is the convergence of overflow value. If overflow value progressively reduces during the placement run it implies that the design will converge and placement will be successful. Post placement, the design can be viewed on magic within ```results/placement``` directory:

```
magic -T /home/devipriya/OpenLane/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.max.lef def read picorv32a.def &

```
![openlane - placement - magic -2](https://user-images.githubusercontent.com/83152452/185789110-2c7019f0-98ce-464a-afd0-5259454b5bb1.png)

Zoomed-in views of the standard cell placement:

![openlane - placement - magic -2 - zoom in view](https://user-images.githubusercontent.com/83152452/185789116-71d0859c-72f8-4ffb-aee3-76ecc1850cdf.png)

***Note: Power distribution network generation is usually a part of the floorplan step. However, in the openLANE flow, floorplan does not generate PDN. The steps are - floorplan, placement CTS and then PDN***


### Standard Cell Design Flow

Standard cell design flow involves the following:

1. Inputs: PDKs, DRC & LVS rules, SPICE models, libraries, user-defined specifications. 
2. Design steps: Circuit design, Layout design (Art of layout Euler's path and stick diagram), Extraction of parasitics, Characterization (timing, noise, power).
3. Outputs: CDL (circuit description language), LEF, GDSII, extracted SPICE netlist (.cir), timing, noise and power .lib files

### Standard Cell Characterization Flow

A typical standard cell characterization flow includes the following steps:

1. Read in the models and tech files
2. Read extracted spice netlist
3. Recognise behaviour of the cell
4. Read the subcircuits
5. Attach power sources
6. Apply stimulus to characterization setup
7. Provide necessary output capacitance loads
8. Provide necessary simulation commands

The opensource software called GUNA can be used for characterization. Steps 1-8 are fed into the GUNA software which generates timing, noise and power models.

### Timing Parameter Definitions

Timing defintion | Value
------------ | -------------
slew_low_rise_thr  | 20% value
slew_high_rise_thr |  80% value
slew_low_fall_thr | 20% value
slew_high_fall_thr | 80% value
in_rise_thr | 50% value
in_fall_thr | 50% value
out_rise_thr | 50% value
out_fall_thr | 50% value

```
rise delay =  time(out_fall_thr) - time(in_rise_thr)

Fall transition time: time(slew_high_fall_thr) - time(slew_low_fall_thr)

Rise transition time: time(slew_high_rise_thr) - time(slew_low_rise_thr)
```

A poor choice of threshold points leads to neative delay value. Therefore a correct choice of thresholds is very important.

## Design library cell

OpenLANE allows users to make changes to environment variables on the fly. For instance, if we wish to change the pin placement from equidistant to some other style of placement we may do the following in the openLANE flow:

```
set ::env(FP_IO_MODE) 2
```

### SPICE Deck creation & Simulation

A SPICE deck includes information about the following:

1. Model description
2. Netlist description
3. Component connectivity
4. Component values
5. Capacitance load
6. Nodes
7. Simulation type and parameters
8. Libraries included

### CMOS inverter Switching Threshold Vm

Thw sitching threshold of a CMOS inverter is the point on the transfer characteristic where Vin equals Vout (=Vm). At this point both PMOS and NMOS are in ON state which gives rise to a leakage current

### 16 Mask CMOS Fabrication

The 16-mask CMOS process consists of the following steps:

1. Selection of subtrate: Secting the body/substrate material.
2. Creating active region for transistors: Isolation between active region pockets by SiO2 and Si3N4 deposition followed by photolithography and etching.
3. N-well and P-well formation: Ion implanation by Boron for P-well and by Phosphorous for N-well formation.
4. Formation of gate terminal: NMOS and PMOS gates formed by photolithography techniques.
5. LDD (lightly doped drain) formation: LDD formed to prevent hot electron effect.
6. Source & drain formation: Screen oxide added to avoid channelling during implants followed by Aresenic implantation and annealing.
7. Local interconnect formation: Removal of screen oxide by HF etching. Deposition of Ti for low resistant contacts.
8. Higher level metal formation: CMP for planarization followed by TiN and Tungsten deposition. Top SiN layer for chip protection.

### Inverter Standard cell Layout & SPICE extraction

The Magic layout of a CMOS inverter will be used so as to intergate the inverter with the picorv32a design. To do this, inverter magic file is sourced from [vsdstdcelldesign](https://github.com/nickson-jose/vsdstdcelldesign) by cloning it within the ```openlane_working_dir/openlane``` directory as follows:

```
git clone https://github.com/nickson-jose/vsdstdcelldesign
```

This creates a vsdstdcelldesign named folder in the openlane directory.

To invoke magic to view the ```sky130_inv.mag``` file, the sky130A.tech file must be included in the command along with its path. To ease up the complexity of this command, the tech file can be copied from the magic folder to the vsdstdcelldesign folder.

The sky130_inv.mag file can then be invoked in Magic very easily:

```
magic -T sky130A.tech sky130_inv.mag &
```

![1  cmos inverter -1-mg layout-command](https://user-images.githubusercontent.com/83152452/185789425-d49e008b-acdf-40f0-9b87-f8c8fe33b969.png)

![1  cmos inverter -1-mg layout](https://user-images.githubusercontent.com/83152452/185789429-3d2715ad-ad33-4ae3-9efe-861a712d8033.png)

In Sky130 the first layer is called the local interconnect layer or Locali.

To verify whether the layout is that of CMOS inverter, verification of P-diffusiona nd N-diffusion regions with Polysilicon can be observed.

Other verification steps are to check drain and source connections. The drains of both PMOS and NMOS must be connected to output port (here, Y) and the sources of both must be connected to power supply VDD (here, VPWR).

**LEF or library exchange format:**
A format that tells us about cell boundaries, VDD and GND lines. It contains no info about the logic of circuit and is also used to protect the IP.

**SPICE extraction:**
Within the Magic environment, following commands are used in tkcon to achieve .mag to .spice extraction:

```
extract all
ext2spice cthresh 0 rethresh 0
ext2spice
```

![2  tkcon window commands - ext spice](https://user-images.githubusercontent.com/83152452/185789439-e7a512a3-e4d7-467f-9baf-0f03fcb990dd.png)

This generates the ```sky130_in.spice``` file. This SPICE deck is edited to include ```pshort.lib``` and ```nshort.lib``` which are the PMOS and NMOS libraries respectively. In addition, the minimum grid size of inverter is measured from the magic layout and incorporated into the deck as: ```.option scale=0.01u```. The model names in the MOSFET definitions are changed to ```pshort.model.0``` and ```nshort.model.0``` respectively for PMOS and NMOS. 

Finally voltage sources and simulation commands are defined as follows:

```
VDD VPWR 0 3.3V
VSS VGND 0 0
Va A VGND PUSLE(0V 3.3V 0 0.1ns 0.1 ns 2ns 4ns)
.tran 1n 20n
.control
run 
.endc
.end
```

The final sky130_inv.spice file is modified to:

![3  spice file](https://user-images.githubusercontent.com/83152452/185789670-f5d25504-f809-404f-ab6f-a60214661134.png)

For simulation, ngspice is invoked in the terminal:

```
ngspice sky130_inv.spice
```

The output "y" is to be plotted with "time" and swept over the input "a":

```
plot y vs time a
```

![3  ngspice command on terminal](https://user-images.githubusercontent.com/83152452/185789474-f6f9ab3c-adcc-48d9-a988-8b5ff5bc7f49.png)

The waveform obtained is as shown:

![3  ngspice plot y vs time a](https://user-images.githubusercontent.com/83152452/185789476-9b49eb13-8a60-4166-a4f6-5acf58178c3b.png)

![3  ngspice plot y vs time a - graph](https://user-images.githubusercontent.com/83152452/185789489-1cc709cb-395b-4db7-894c-e90cbe8553c5.png)

The spikes in the output at switching points is due to low capacitance loads. This can be taken care of by editing the spice deck to increase the load capacitance value.

### Inverter Standard cell characterization

Four timing parameters are used to characterize the inverter standard cell:

1. Rise transition: Time taken for the output to rise from 20% of max value to 80% of max value
2. Fall transition- Time taken for the output to fall from 80% of max value to 20% of max value
3. Cell rise delay = time(50% output rise) - time(50% input fall)
4. Cell fall delay  = time(50% output fall) - time(50% input rise)

The above timing parameters can be computed by noting down various values from the ngspice waveform.

```Rise transition = (2.23843 - 2.17935) = 59.08ps```

```Fall transition = (4.09291 - 4.05004) = 42.87ps```

```Cell rise delay = (2.20636 - 2.15) = 56.36ps```

```Cell fall delay = (4.07479 - 4.05) = 24.79ps```



## Timing Analysis & CTS

A requirement for ports as specified in ```tracks.info``` is that they should be at intersection of horizontal and vertical tracks. The CMOS Inverter ports A and Y are on li1 layer. It needs to be ensured that they're on the intersection of horizontal and vertical tracks. We access the tracks.info file for the pitch and direction information:

![image](https://user-images.githubusercontent.com/55539862/185734357-8cd10f22-9a23-4cef-af83-f029b344b473.png)

To ensure that ports lie on the intersection point, the grid spacing in Magic (tkcon) must be changed to the li1 X and li1 Y values. Convergence of grid and tracks can be achieved using the following command:

```
grid 0.46um 0.34um 0.23um 0.17um
```

![1  grid activation](https://user-images.githubusercontent.com/83152452/185789868-4a759399-9ee7-4b94-8b11-93283ed4ebfa.png)

![2  grid cells with tkcon window command nd magic layout changes](https://user-images.githubusercontent.com/83152452/185789878-18b441ec-af32-4f3f-b123-2908b864c6fb.png)


### Create port definition

Once the layout is ready, the next step is extracting LEF file for the cell. However, certain properties and definitions need to be set to the pins of the cell which aid the placer and router tool. For LEF files, a cell that contains ports is written as a macro cell, and the ports are the declared PINs of the macro. Our objective is to extract LEF from a given layout (here of a simple CMOS inverter) in standard format. Defining port and setting correct class and use attributes to each port is the first step. 

The easiest way to define a port is through Magic Layout window and following are the steps:

- In Magic Layout window, first source the .mag file for the design (here inverter). Then **Edit >> Text** which opens up a dialogue box.

![2  port definition - A](https://user-images.githubusercontent.com/83152452/185789881-929deae7-d4eb-4842-9829-13a286be397a.png)

- For each layer (to be turned into port), make a box on that particular layer and input a label name along with a sticky label of the layer name with which the port needs to be associated. Ensure the Port enable checkbox is checked and default checkbox is unchecked as shown in the figure:

![2  port definition - Y](https://user-images.githubusercontent.com/83152452/185789884-8da9ac43-e273-4c94-9f68-2edbf77fcded.png)

In the above two figures, port A (input port) and port Y (output port) are taken from locali (local interconnect) layer. Also, the number in the textarea near enable checkbox defines the order in which the ports will be written in LEF file (0 being the first).

- For power and ground layers, the definition could be same or different than the signal layer. Here, ground and power connectivity are taken from metal1 (Notice the sticky label).

![2  port definition - VPWR](https://user-images.githubusercontent.com/83152452/185789887-94710eac-40c3-4759-9d85-19aba74d69bf.png) 

![2  port definition - VGND](https://user-images.githubusercontent.com/83152452/185789892-7cf30d76-b01b-4a42-8b05-0c2e8c772d04.png)


### Standard Cell LEF generation

Before the CMOS Inverter standard cell LEF is extracted, the purpose of ports must be defined:

Select port A in magic:
```
port class input
port use signal
```
Select Y area
```
port class output
port class signal
```
Select VPWR area
```
port class inout
port use power
```

Select VGND area
```
port class inout
port use ground
```

LEF extraction can be carried out in tkcon as follows:

```
lef write
```

![3  lef file extraction](https://user-images.githubusercontent.com/83152452/185790311-f5a68ba1-9e1d-47b1-ae47-d7a6f1a723d2.png)

This generates ```sky130_vsdinv.lef``` file.


### Integrating custom cell in OpenLANE

In order to include the new standard cell in the synthesis, copy the sky130_vsdinv.lef file to the ```designs/picorv32a/src``` directory  
Since abc maps the standard cell to a library abc there must be a library that defines the CMOS inverter. The ```sky130_fd_sc_hd_typical.lib``` file from ```vsdstdcelldesign/libs``` directory needs to be copied to the ```designs/picorv32a/src``` directory (Note: the slow and fast library files may also be copied).

Next, ```config.tcl``` must be modified:
```
set ::env(LIB_SYNTH) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130/sky130_fd_sc_hd__typical.lib"
set ::env(LIB_SLOWEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130/sky130_fd_sc_hd__slow.lib"
set ::env(LIB_FASTEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130/sky130_fd_sc_hd__fast.lib"
set ::env(LIB_TYPICAL) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130/sky130_fd_sc_hd__typical.lib"

set ::env(EXTRA_LEFS) [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/*.lef]
```

```modified config.tcl file:```

```

# User config
set ::env(DESIGN_NAME) "picorv32a"

# Change if needed
set ::env(VERILOG_FILES) "./designs/picorv32a/src/picorv32a.v"
set ::env(SDC_FILES) "./designs/picorv32a/src/picorv32a.sdc"


# turn off clock
set ::env(CLOCK_PERIOD) "5.000"
set ::env(CLOCK_PORT) "clk"

set ::env(CLOCK_MET) $::env(CLOCK_PORT) 


set ::env(LIB_SYNTH) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib "
set ::env(LIB_MIN) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__fast.lib"
set ::env(LIB_MAX) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__slow.lib "
set ::env(LIB_TYPICAL) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"

set ::env(EXTRA_LEFS) [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/*.lef]

set filename $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/$::env(PDK)_$::env(STD_CELL_LIBRARY)_config.tcl
#set filename $::env(DESIGN_DIR)/$::env(PDK)_$::env(STD_CELL_LIBRARY)_config.tcl
if { [file exists $filename] == 1 } {
      source $filename
}
```

In order to integrate the standard cell in the OpenLANE flow, invoke openLANE as usual and carry out following steps:

```
prep -design picorv32a -tag RUN_2022.08.17_16.22.21 -overwrite
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs
run_synthesis
```

![4  merging lef files in picorv command](https://user-images.githubusercontent.com/83152452/185790538-36dc28f1-b75e-4312-83de-4db1dd9564d5.png)

Next floorplan is run, followed by placement:

```
run_floorplan
run_placement
```

To check the layout invoke magic from the ```results/placement``` directory:

```
magic -T /home/devipriya/OpenLane/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.max.lef def read picorv32a.def &
```

Since the custom standard cell has been plugged into the openLANE flow, it would be visible in the layout.

![sky130_vsdinv](https://user-images.githubusercontent.com/83152452/185794679-cd0de4df-30a6-46bd-b6dd-c73a727d85f4.jpeg)


### Post-synthesis timing analysis

Timing analysis is carried out outside the openLANE flow using OpenSTA tool. For this, a new file ```pre_sta.conf``` is created. This file would be reqiured to carry out the STA analysis. Invoke OpenSTA outside the openLANE flow as follows:

```
sta pre_sta.conf
```

Since clock tree synthesis has not been performed yet, the analysis is with respect to ideal clocks and only setup time slack is taken into consideration. The slack value is the difference between data required time and data arrival time. The worst slack value must be greater than or equal to zero. If a negative slack is obtained, following steps may be followed:
1. Change synthesis strategy, synthesis buffering and synthesis sizing values 
2. Review maximum fanout of cells and replace cells with high fanout

The lef file generated:

![5  generated lef file](https://user-images.githubusercontent.com/83152452/185791680-36453e1a-5b7b-44e4-866f-4d0be2e63b8e.png)

```
VERSION 5.7 ;
  NOWIREEXTENSIONATPIN ON ;
  DIVIDERCHAR "/" ;
  BUSBITCHARS "[]" ;
MACRO sky130_vsdinv
  CLASS CORE ;
  FOREIGN sky130_vsdinv ;
  ORIGIN 0.000 0.000 ;
  SIZE 1.380 BY 2.720 ;
  SITE unithd ;
  PIN A
    DIRECTION INPUT ;
    USE SIGNAL ;
    ANTENNAGATEAREA 0.165600 ;
    PORT
      LAYER li1 ;
        RECT 0.060 1.180 0.510 1.690 ;
    END
  END A
  PIN Y
    DIRECTION OUTPUT ;
    USE SIGNAL ;
    ANTENNADIFFAREA 0.287800 ;
    PORT
      LAYER li1 ;
        RECT 0.760 1.960 1.100 2.330 ;
        RECT 0.880 1.690 1.050 1.960 ;
        RECT 0.880 1.180 1.330 1.690 ;
        RECT 0.880 0.760 1.050 1.180 ;
        RECT 0.780 0.410 1.130 0.760 ;
    END
  END Y
  PIN VPWR
    DIRECTION INOUT ;
    USE POWER ;
    PORT
      LAYER nwell ;
        RECT -0.200 1.140 1.570 3.040 ;
      LAYER li1 ;
        RECT -0.200 2.580 1.430 2.900 ;
        RECT 0.180 2.330 0.350 2.580 ;
        RECT 0.100 1.970 0.440 2.330 ;
      LAYER mcon ;
        RECT 0.230 2.640 0.400 2.810 ;
        RECT 1.000 2.650 1.170 2.820 ;
      LAYER met1 ;
        RECT -0.200 2.480 1.570 2.960 ;
    END
  END VPWR
  PIN VGND
    DIRECTION INOUT ;
    USE GROUND ;
    PORT
      LAYER li1 ;
        RECT 0.100 0.410 0.450 0.760 ;
        RECT 0.150 0.210 0.380 0.410 ;
        RECT 0.000 -0.150 1.460 0.210 ;
      LAYER mcon ;
        RECT 0.210 -0.090 0.380 0.080 ;
        RECT 1.050 -0.090 1.220 0.080 ;
      LAYER met1 ;
        RECT -0.110 -0.240 1.570 0.240 ;
    END
  END VGND
  ```


### Clock Tree Synthesis

The purpose of building a clock tree is enable the clock input to reach every element and to ensure a zero clock skew. H-tree is a common methodology followed in CTS.
Before attempting a CTS run in TritonCTS tool, if the slack was attempted to be reduced in previous run, the netlist may have gotten modified by cell replacement techniques. Therefore, the verilog file needs to be modified using the ```write_verilog``` command. Then, the synthesis, floorplan and placement is run again. To run CTS use the below command:

```
run_cts
```

The CTS run adds clock buffers in therefore buffer delays come into picture and our analysis from here on deals with real clocks. Setup and hold time slacks may now be analysed in the post-CTS STA anlysis in OpenROAD within the openLANE flow:

```
openroad
write_db pico_cts.db
read_db pico_cts.db
read_verilog /openLANE_flow/designs/picorv32a/runs/03-07_11-25/results/synthesis/picorv32a.synthesis_cts.v
read_liberty $::env(LIB_SYNTH_COMPLETE)
link_design picorv32a
read_sdc /openLANE_flow/designs/picorv32a/src/my_base.sdc
set_propagated_clock (all_clocks)
report_checks -path_delay min_max -format full_clock_expanded -digits 4
```

Slack at the end of STA for typical corner:

![6  cts report-1](https://user-images.githubusercontent.com/83152452/185791682-94f98a0a-56ce-4409-9a66-796675ac5d39.png)


## Final steps in RTL2GDS

### Power Distribution Network generation

Unlike the general ASIC flow, Power Distribution Network generation is not a part of floorplan run in OpenLANE. PDN must be generated after CTS and post-CTS STA analyses:

```
gen_pdn
```

We can confirm the success of PDN by checking the current def environment variable: ``` echo $::env(CURRENT_DEF) ```

![pdn def report-1](https://user-images.githubusercontent.com/83152452/185791987-6a53d110-667f-4243-a27f-056ed20e0154.png)


- `gen_pdn` - Generates the Power Distribution network
- The power distribution network has to take the `design_cts.def` as the input def file.
- This will create the grid and the straps for the Vdd and the ground. These are placed around the standard cells.
- The standard cells are designed such that it's height is multiples of the space between the Vdd and the ground rails. Here, the pitch is `2.72`. Only if the above conditions are adhered it is possible to power the standard cells.
- The power to the chip, enters through the `power pads`. There is each for Vdd and Gnd
- From the pads, the power enters the `rings`, through the `via`
- The `straps` are connected to the ring. Vdd straps are connected to the Vdd ring and the Gnd Straps are connected to the Gnd ring. There are horizontal and the vertical straps
- Now the power has to be supplied from the straps to the standard cells. The straps are connected to the `rails` of the standard cells
- If macros are present then the straps attach to the `rings` of the macros via the `macro pads` and the pdn for the macro is pre-done.
- There are definitions for the straps and the railss. In this design straps are at metal layer 4 and 5 and the standard cell rails are at the metal layer 1. Vias connect accross the layers as required.


### Routing 

OpenLANE uses the TritonRoute tool for routing. There are 2 stages of routing:

1. Global routing: Routing region is divided into rectangle grids which are represented as course 3D routes (Fastroute tool).

![global routing](https://user-images.githubusercontent.com/83152452/185791992-5d0dadb2-4dc1-493a-bd24-b531298d6442.png)

3. Detailed routing: Finer grids and routing guides used to implement physical wiring (TritonRoute tool).

![detailed routing](https://user-images.githubusercontent.com/83152452/185791995-c8204a98-b61b-48a3-a1ec-642da35d7589.png)

Features of TritonRoute:

1. Honouring pre-processed route guides
2. Assumes that each net satisfies inter guide connectivity
3. Uses MILP based panel routing scheme
4. Intra-layer parallel and inter-layer sequential routing framework

Running routing step in TritonRoute as part of openLANE flow:

```
run_routing
```

- `run_routing` - To start the routing
- The options for routing can be set in the `config.tcl` file. 
- The optimisations in routing can also be done by specifying the routing strategy to use different version of `TritonRoute Engine`. There is a trade0ff between the optimised route and the runtime for routing.
- For the default setting picorv32a takes approximately 30 minutesaccording to the current version of TritonRoute.
- This routing stage must have the `CURRENT_DEF` set to `pdn.def`
- The two stages of routing are performed by the following engines:
    - Global Route : Fast Route
    - Detailed Route : Triton Route
- Fast Route generates the routing guides, whereas Triton Route uses the Global Route and then completes the routing with some strategies and optimisations for finding the best possible path connect the pins.

## GDSII

GDS Stands for Graphic Design Standard. This is the file that is sent to the foundry and is called as "tape-out". 

*Fact- Earlier, the GDS files were written on magnetic tapes and sent out to the foundry and hence the name "tape-out"*

In openLane use the command `magic`

The GDSII file is generated in the `results/signoff/magic` directory.

No DRC errors are found.

The layout pictures are shown below:

![3  picorv32a mag layout - without black metal](https://user-images.githubusercontent.com/83152452/185795420-9af80389-7205-4a18-ac1b-0f95bd0f9d5c.png)

![3  picorv32a mag layout](https://user-images.githubusercontent.com/83152452/185795424-bb2cf6ee-87ef-4e60-ac54-5e793d8cf3f4.png)

Zoom in view:

![3  picorv32a mag zoom in view layout](https://user-images.githubusercontent.com/83152452/185795427-f2a5b5c1-00ef-4c39-aaf5-81e04eb9cb50.png)


### LEF 

```picorv32a.lef.mag``` file generated is shown below:

![4  picorv32a lef mag layout](https://user-images.githubusercontent.com/83152452/185795429-6fbb2ed0-e787-46d0-a4f7-198ae3bd8fd2.png)

## Design folder


```

picorv32a
├── config.tcl
├── runs
│   ├── RUN_2022.08.17_16.22.21
│   │   ├── config.tcl
│   │   ├── logs
│   │   │   ├── cts
│   │   │   ├── cvc
│   │   │   ├── floorplan
│   │   │   ├── klayout
│   │   │   ├── magic
│   │   │   ├── placement
│   │   │   ├── routing
│   │   │   └── synthesis
│   │   ├── reports
│   │   │   ├── cts
│   │   │   ├── cvc
│   │   │   ├── floorplan
│   │   │   ├── klayout
│   │   │   ├── magic
│   │   │   ├── placement
│   │   │   ├── routing
│   │   │   └── synthesis
│   │   ├── results
│   │   │   ├── cts
│   │   │   ├── cvc
│   │   │   ├── floorplan
│   │   │   ├── klayout
|   |   |   ├── signoff
│   │   │   ├── magic
│   │   │   ├── placement
│   │   │   ├── routing
│   │   │   └── synthesis
│   │   └── tmp
│   │       ├── cts
│   │       ├── cvc
│   │       ├── floorplan
│   │       ├── klayout
│   │       ├── magic
│   │       ├── placement
│   │       ├── routing
│   │       └── synthesis
├── src
|   ├── picorv32a.v

```

## Differences from older OpenLANE versions

- The reports in the latest version is not generated in the terminal, we have to verify them from the reports/results folder.
- SPEF extraction need not be externally performed in the new version. It has been integrated into the OpenLANE flow.


## Summary

### VLSI NON INTERACTIVE OPENLANE FLOW

``` 

cd OpenLane/ 
make mount 
{ If Error occurs use the below commands in OpenLane directory:
sudo chown $USER /var/run/docker.sock 
PYTHON_BIN=python3 make mount
}

./flow.tcl -design picorv32a 

```

STEPS RUNNING:

```

[STEP 1] : Running Synthesis.
[STEP 2] : Running Single-Corner Static Timing Analysis.
[STEP 3] : Running Initial Floorplanning, Setting Core Dimensions.
[STEP 4] : Running IO Placement.
[STEP 5] : Running Power planning with power {VPWR} and ground {VGND}.
[STEP 6] : Generating PDN.
[STEP 7] : Performing Random Global Placement.
[STEP 8] : Running Placement Resizer Design Optimizations.
[STEP 9] : Writing Verilog.
[STEP 10] : Running Detailed Placement.
[STEP 11] : Running Placement Resizer Timing Optimizations.
[STEP 12] : Writing Verilog, Routing.
[STEP 13] : Running Global Routing Resizer Timing Optimizations.
[STEP 14] : Writing Verilog.
[STEP 15] : Running Detailed Placement.
[STEP 16] : Running Global Routing, Starting FastRoute Antenna Repair Iterations.
[STEP 17] : Running Fill Insertion.
[STEP 18] : Writing Verilog.
[STEP 19] : Running Detailed Routing, No DRC violations after detailed routing.
[STEP 20] : Writing Verilog, Running parasitics-based static timing analysis.
[STEP 21] : Running SPEF Extraction at the min process corner.
[STEP 22] : Running Multi-Corner Static Timing Analysis at the min process corner.
[STEP 23] : Running SPEF Extraction at the max process corner.
[STEP 24] : Running Multi-Corner Static Timing Analysis at the max process corner.
[STEP 25] : Running SPEF Extraction at the nom process corner...
[STEP 26] : Running Single-Corner Static Timing Analysis at the nom process corner...
[STEP 27] : Running Multi-Corner Static Timing Analysis at the nom process corner...
[STEP 28] : Running Magic to generate various views, Streaming out GDS-II with Magic, Generating MAGLEF views...
[STEP 29] : Streaming out GDS-II with Klayout...
[STEP 30] : Running XOR on the layouts using Klayout...
[STEP 31] : Running Magic Spice Export from LEF...
[STEP 32] : Writing Powered Verilog.
[STEP 33] : Writing Verilog.
[STEP 34] : Running LEF LVS.
[STEP 35] : Running Magic DRC, Converting Magic DRC Violations to Magic Readable Format, Converting Magic DRC Violations to Klayout Database, Converting DRC Violations to RDB Format, No DRC violations after GDS streaming out, Running Antenna Checks.
[STEP 36] : Running OpenROAD Antenna Rule Checker.
[STEP 37] : Running CVC, Saving final set of views, 
Saving runtime environment, 
Generating final set of reports, Created manufacturability report at 'designs/riscv/runs/RUN_2022.06.07_10.39.52/reports/manufacturability.rpt', 
Created metrics report at 'designs/riscv/runs/RUN_2022.06.07_10.39.52/reports/metrics.csv', 
There are no max slew violations in the design at the typical corner, There are no hold violations in the design at the typical corner, There are no setup violations in the design at the typical corner.

[SUCCESS]: Flow complete.

```


### VLSI INTERACTIVE OPENLANE FLOW

```

cd OpenLane/ 
make mount 
{ If Error occurs use the below commands in OpenLane directory:
sudo chown $USER /var/run/docker.sock 
PYTHON_BIN=python3 make mount
}

./flow.tcl -interactive
package require openlane 0.9
prep -design picorv32a
run_synthesis
run_floorplan
run_placement
run_cts
run_routing
run_magic
run_magic_spice_export
run_magic_drc
run_netgen
run_magic_antenna_check

```


## Acknowledgements 

- [The OpenROAD Project](https://github.com/The-OpenROAD-Project/OpenLane)
- [Nickson Jose](https://github.com/nickson-jose/vsdstdcelldesign)
- [Kunal Ghosh](https://github.com/kunalg123)


## Author

- [A Devipriya](https://github.com/Devipriya1921), Bachelor of Engineering in Electronics and Communication Engineering, Bangalore

