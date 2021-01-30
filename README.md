# **Advanced Physical Design Workshop using openLANE sky130 PDK**

## **About The Project**

This project is a documentation of VSD Advanced Physical Design Workshop using OpenLANE and Skywater PDK.

OpenLANE is an automated RTL to GDSII flow based on several components including OpenROAD, Yosys, Magic, Netgen, Fault, OpenPhySyn, SPEF-Extractor and custom methodology scripts for design exploration and optimization. The flow performs full ASIC implementation steps from RTL all the way down to GDSII

## **Getting Started**

References to how to setup the tool chain is given below followed by a log of what I worked on during the workshop.

**Installation Reference :** <https://github.com/nickson-jose/openlane_build_script>

## **RTL to GDSII Introduction**

**ASIC Flow (RTL to GDSII) Introduction:**

ASIC stands for Application Specific Integrated Circuit. ASIC design flow is the process in which a design is conceptualized and finally realised.

1.  Architectural Design -- A system engineer will provide the VLSI engineer with specifications for the system that are determined through physical constraints. The VLSI engineer will be required to design a circuit that meets these constraints at a microarchitecture modeling level.

2.  RTL Design/Behavioral Modeling -- RTL design and behavioral modeling are performed with a hardware description language (HDL). EDA tools will use the HDL to perform mapping of higher-level components to the transistor level needed for physical implementation. HDL modeling is normally performed using either Verilog or VHDL. One of two design methods may be employed while creating the HDL of a microarchitecture:

> a\. RTL Design -- Stands for Register Transfer Level. It provides an abstraction of the digital circuit using:

-   i\. Combinational logic

-   ii\. Registers

-   iii\. Modules (IP's or Soft Macros)

3.  RTL Verification - Behavioral verification of design

4.  DFT Insertion - Design-for-Test Circuit Insertion

5.  Logic Synthesis -- Logic synthesis uses the RTL netlist to perform HDL technology mapping.

6.  Post-Synthesis STA Analysis: Performs setup analysis on different path groups.

7.  Floorplanning -- Goal is to plan the silicon area and create a robust power distribution network (PDN) to power each of the individual components of the synthesized netlist. IO placement is done here too sometimes.

8.  Placement- Place the standard cells on the floorplane rows, aligned with sites defined in the technology lef file. Placement is done in two steps: Global and Detailed. In Global placement tries to find optimal position for all cells but they may be overlapping and not aligned to rows, detailed placement takes the global placement and legalizes all of the placements trying to adhere to what the global placement wants.

9.  CTS - Clock tree synteshsis is used to create the clock distribution network that is used to deliver the clock to all sequential elements. The main goal is to create a network with minimal skew across the chip. H-trees are a common network topology that is used to achieve this goal.

10. Routing -- Implements the interconnect system between standard cells using the remaining available metal layers after CTS and PDN generation. The routing is performed on routing grids to ensure minimal DRC errors.

## **Workshop Introduction**

This workshop focuses on using the open-source RTL2GDS EDA tool, OpenLANE, in conjunction with the Skywater 130nm PDK to perform the full RTL2GDS flow as shown below:![](media/image21.png){width="5.286458880139983in" height="3.7361023622047242in"}

## **Physical Design**

Integrated Circuit (IC) design is split up into Front-end Design using HDLs and Back-end Design or Physical Design. The inputs to physical design are (i) a netlist, (ii) library information on the basic devices in the design, and (iii) a technology file containing the manufacturing constraints.

PDKs: Process Design Kit(PDKs) contains device models, digital standard cell libraries, process design rules, input-output libraries and other important collection of files used for fabrication.

## **Day 1** **Inception of Open Source EDA, openLANE and skywater130 PDK**

![](media/image36.png)

**Skywater-pdk**

![](media/image23.png)

Running OpenLANE:

-   ./flow.tcl is the script which runs the OpenLANE flow

-   -interactive options allows us to run the entire flow in a step-step by step manner

OpenLANE is launched from docker that is setup during installation

![](media/image17.png)

![](media/image41.png)

![](media/image18.png)

Import Package and prepare picorv32a using the following commands

![](media/image7.png)

./flow.tcl -interactive //command to invoke openlane

package require openlane 0.9 //adding packages to openlane

If -tag is not given during preparation new folder with name corresponding to time and date is created.

**Designs Folder:**

![](media/image46.png)

**Directory Hierarchy**

![](media/image49.png)

![](media/image35.png)

![](media/image1.png)

Config.tcl files - Design specific configuration switches used by OpenLANE

There are multiple config.tcl files, where they are given priority(when the same variable is switched in the both .tcl files), foundry\_config.tcl has the highest.

Config.tcl

![](media/image27.png)

Skywater\_config.tcl (higher priority) hence clock period set to 24.73

![](media/image33.png)

**Running Synthesis:**

Uses yosys and abc to synthesize and map to technology library primitives.

![](media/image25.png)

![](media/image50.png)

The synthesis step also performs STA(standard timing analysis) and there is no negative slack in this instance. For more detailed analysis , check reports/synthesis inside the openLANE run.

![](media/image29.png)

![](media/image19.png)

-   picorv32a.synthesis.v in results is the verilog gate netlist file.

## 

## 

## **Day 2 Chip Floorplanning and Standard Cells**

**Floorplan**

![](media/image6.png)

Similar to how synthesis generated picorv32a.synthesis.v, floorplanning generates picorv32a.floorplan.def

![](media/image43.png)

Magic is used to view layout. Tech file corresponds to technology information and lef and def correspond to the actual layout. These three are inputs to magic and we launch by above command.

![](media/image28.png)

### **Placement**

![](media/image39.png)

Similarly we get a def file after placement. This is opened with magic.

The placement:![](media/image26.png)

## **Day 3 Design Library Cell**

Inverter cell by nickson-jose , Through this we learn about making a standard cell. Also, after including this cell into the skywater standard cell library we see a change in timing of the core as this new cell is now mapped to the netlist, this gives us an opportunity to go through STA.

![](media/image52.png)

**Navigation in Magic**

In magic hover over a selected area and press s macro to select that layer. Similarly there are other macros. **What** command in the tkcon window describes what the selected portion of the cell is. D key is used to delete selected areas. We shall delete the area to see what DRC errors arise. Similarly while designing a cell one might encounter such errors and they should be dealt appropriately. Any DRC errors in an area can be read by drc why. One can learn more about magic [[here]{.underline}](http://opencircuitdesign.com/magic/). Magic has continuous DRC checking that means as soon as an error occurs it can be seen in the top bar.

![](media/image31.png)

![](media/image44.png)

![](media/image11.png)

Creating DRC error:

![](media/image12.png)![](media/image8.png)

Fixing the DRC error:

By painting the deleted layer(locali) in that area. Hover over the layers in the right side of magic and press P to pain the selected area with that layer.

![](media/image47.png)

Extracting the inverter to spice file and performing simulations on it:

![](media/image10.png)

Extracted file:

![](media/image9.png)

Modified spice file:

![](media/image42.png)

Option scale should be 0.01u for minimum grid size in the layout

**NGspice :**

![](media/image37.png)

![](media/image3.png)

1.  Drag with right click to zoom and analyse the plot and calculate delays.

## **Day 4 & 5 Layout Timing Analysis and CTS and Routing**

Input and output ports must be at the intersection of vertical and horizontal tracks and width should be an odd multiple of track's horizontal pitch. Height odd multiple of vertical pitch.

![](media/image32.png)

![](media/image38.png)

-   Routing happens on metal layers.

-   Tracks info , PNR tools route through tracks.

-   Later coordinate offset pitch

> Li1 x 0.23 0.46
>
> Li2 y 0.17 0.32
>
> Press G when on the magic layout to activate grid lines.

**Grid Lines:**

![](media/image15.png)

We converge tracks size with grid size to make sure ports are on the intersection of tracks

![](media/image2.png)

Input Port A:

![](media/image51.png)

PR boundary and cell width being odd multiple of tracks.

![](media/image13.png)

Writing the cell into a LEF file:

![](media/image4.png)

![](media/image34.png)

LEF file:

![](media/image24.png)

Copy library files from vsdstdcell to picorv32a source:

![](media/image30.png)

![](media/image40.png)

![](media/image22.png)

![](media/image20.png)

![](media/image48.png)

![](media/image5.png)

After adding the new cell

![](media/image45.png)

To reduce slack one could change design switches given in configuration/README.md, they can be changed on the fly from the openlane command line using set.
![](media/image54.png)

Placement on new synthesized file

![](media/image14.png)

Cts:

![](media/image16.png)

Invoke the openroad from openlane

openroad

To do timing analysis create a db from lef and def files. db once created can be read multiple times.

read_lef loacation of merged.lef read_def location of picorv32a_cts.def write_db After CTS slack is increased. To reduce slack violation edit the variables for clock buffers, replace the buffers.
![](media/image53.png)

Routing:
There are two types of routing in the physical design process, global routing and detailed routing. Global routing allocates routing resources that are used for connections. It also does track assignment for a particular net.

Detailed routing does the actual connections. Different constraints that are to be taken care during the routing are DRC, wire length, timing etc. Openlane tools for routing:

FastRoute - Performs global routing to generate a guide file for the detailed router
TritonRoute - Performs detailed routing
SPEF-Extractor - Performs SPEF extraction
run_routing

## **Acknowledgements**
> Praharsha Mahurkar, Teaching Assistant(VSD Corp. Pvt. Ltd)
> Nickson Jose, Intern(VSD Corp. Pvt. Ltd)
> Kunal Ghosh, Co-founder (VSD Corp. Pvt. Ltd)
