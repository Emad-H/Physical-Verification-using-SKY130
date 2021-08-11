# Physical Verification using SKY130

![workshop-flyer](Workshop-Flyer.png)

A 5 day cloud based virtual training workshop conducted by VSD-IAT from 11<sup>th</sup> to 15<sup>th</sup> August. The link to the workshop webpage can be found [here](https://www.vlsisystemdesign.com/physical-verification-using-sky130/). Below is a brief, day-wise documentation about the topics covered in the course, along with my implementations of the lab sessions.

## Table of Contents



## Day 1 - Introduction to SkyWater SKY130 and Open-Source EDA Tools

### SkyWater PDK

The SkyWater Open Source PDK is a joint project between Google and the SkyWater Technology Foundry, which provides a fully open source Process Design Kit (PDK), and its related resources. The SkyWater open PDK public repository contains the following:
 - [Documentation](https://skywater-pdk.rtfd.io/) 
 - [PDK Library and files](https://github.com/google/skywater-pdk)
 - as well as the [Community](https://join.skywater.tools/)
 
 ![google-sky-logo](Day1/1-0.png)
 
The "130" in SKY130 stands for the feature size, which is the length of smallest transistor that can be manufactured in the process.

### Open-Source EDA Tools

Open_PDKs is a Makefile based installer that takes files from the SkyWater PDKs and reformats them for a number of open source EDA tools, which can be found at [R. Timothy Edwards' github page](https://github.com/RTimothyEdwards/open_pdks).

Tools currently supported by open_pdks:
- Magic
- Klayout
- Openlane
- Xschem
- Netgen
- Ngspice
- IVerilog
- qflow
- IRSIM
- xcircuit

To install SKY130 PDKs, we must clone the repository and specify the process to compile and install using the following commands.

```
git clone https://github.com/RTimothyEdwards/open_pdks
cd open_pdks
configure --enable-sky130-pdk
make
sudo make install
```

The ```make``` process grabs the SKY130 repository and submodules, as well as a few third party repositories to use in the install. It then builds the libraries from these various repositories.

The libraries supported by open_pdks are:
- Digital standard cells (ex: sky130_fd_sc_hd)
- Primitive devices/analog (ex: sky130_fd_pr)
- I/O cells (ex: sky130_fd_io)
- 3rd party libraries (ex: sky130_ml_xx_hd)

Open_PDKs uses a common installed filesystem structure, where the SkyWater PDKs are placed under the directory ```/usr/share/pdk/sky130A/```. Under this main SK130 PDK directory, are 2 subdirectories ```libs.tech```, which contains all subdirectories for the open source tool setups, and ```libs.ref```, which contains the reference libraries in various formats. The project directory follows a similar format, with a ```project_root/``` directory containing subdirectories for each tool or flow needed.

### Physical Verification and Design Flows

Like functional verification, where we check if the voltages, signals and timings match the specification; physical verification is to check whether you have a mask layout that matches what you think the circuit should be.

### Lab - Checking Tool Installations

1. Magic: It can be run by giving the command ```magic``` in the command line interface. This brings up a layout window and a console window that is a stock tcl interpretor used to run commands for layout and actions. We can get the tcl interpretor in the terminal itself instead of the seperate console window by using the option ```magic -noconsole```. Magic can also be run without the graphics layout window using the option ```magic -dnull - noconsole```, and should be called as such when running from a script. To run magic in batch mode, we use the command ```magic -dnull -noconsole filename.tcl```.

2. Netgen: We can run Netgen using the command ```netgen``` in the terminal. It is completely command driven and has no graphics interface. Its console window is a stock tcl interpretor like magic as well. We can get the tcl interpretor in the terminal itself instead of the seperate console window by using the option ```netgen -noconsole```. To run netgen in batch mode, we use the command ```netgen -batch source filename.tcl```. Netgen also provides a GUI window written in python that can be accessed using ```usr/local/lib/netgen/pyhton/lvs_manager.py```, though this interface hides many useful options that cannot be accessed with just this window itself.

3. Xschem: It is accessed using the command ```xschem``` in the terminal. This should bring up a schematics window. Unlike netgen and magic however, xschem has no seperate console window and uses the native command line terminal for tcl commands. Xschem can be run in batch mode with the command ```xschem --tcl filename.tcl -q```.

4. Ngspice: It can be run using the command ```ngspice``` in the linux command line. Ngspice has its own prompt and runs its own set of interpretor commands that aren't based on tcl. It can be run in batch mode using the option ```ngspice -b```.

Let's get acquainted with these open source tools with a basic example of an inverter design. First, we must create a directory for the design and initialise subdirectories for each of the open source tools that we will be using. This is shown below.

![inv-dir](Day1/1-1.png)

Next, we must set up each directory for its respective tool to run properly with the SkyWater PDKs. This can be done by creating a symbolic link between the just initialised subdirectories and the SKY130 submodules created with the open_pdks installer. This is done using ```ln -s /file_path``` in the desired subdirectory, and is shown below. As we will run ngspice simulations using xschem, so we setup ngspice under the xschem subdirectory itself.

![symlink](Day1/1-2.png)

Let us run Xschem using the command ```xschem```. This brings up a display for xschem with a lot of example schematics, introducing you to SKY130 devices as seen below. Examples can be accessed by clicking the relevant rectangle and pressing the E key on the keyboard. We can return to the menu by pressing CTRL+E. The F key resizes the schematic to fit the window.

![xschem](Day1/1-3.png)

Now, we can check the magic installation using the command ```magic``` in the /mag subdirectory. This should bring up the 2 magic windows, with the layout window displaying "Technology: sky130A", along with many colors and icons displaying the available layers in this technology, as show below.

![magic](Day1/1-4.png)

We can use the command ```magic -d XR``` to invoke a cairo graphics package which uses 3D acceleration to get better rendering than the default graphics. There is also an OpenGL based graphics package that can be accessed using ```magic -d -OGL```. 

Useful Magic Shortcuts:
1. Left and right mouse buttons to adjust the cursor box
2. Shift+Z to zoom out
3. Middle mouse button/P to select a layer (also known as painting)
4. E to erase whatever is present in the cursor box (can also be done ny clicking the middle mouse button on an empty part of the layout)
5. V can be used to view the entire layout
6. CTRL+P opens up the parameter options for the selected device
7. S key can be used to select layers
8. Typing the ```what``` command in the magic console gives information on the selected layer
9. ; key can be used to type commands in the magic console without moving between windows, until the Enter key is pressed

The purple layer "metal1" below shall be most used to connect devices in further lab experiments.

![metal1](Day1/1-5.png)

We can create some devices using the two Devices drop down buttons. Let us select the nmos (mosfet) under "Devices 1" and set the width to 2 um, length to 0.5 um and fingers to 3. This should result in the following device being created.

![params](Day1/1-6.png)

![device](Day1/1-7.png)

### Lab - Creating an Inverter Design

Let's run xschem and open a new schematic under the File option. To add a component we use the Ins key. Select the SkyWater library directory path to access SkyWater components and choose the fd_pr library. To create an inverter, we need a basic nfet and pfet. So let's select a nfet and pfet device from the insert window and place it anywhere in the schematic. 

![nfet-pfet-place](Day1/1-8.png)

As we want to turn the schematic to a layout, it is good practice to keep the circuit parts that will be turned into a layout self-contained so it doesn't have any unnecessary components in it that do not need to be present in the layout. It is also good practice not to use any specific power supply pins as there is no concept of a global supply pin/net in a layout. Not adhering to these practices can cause problems later when verifying layouts.

As pins are not PDK specific, they can be found under the xschem library in the insert window. These are named as ipin.sym, opin.sym and iopin.sym. We can place the pins and use the M key after hovering the component to move them around on the schematic window. The C key can be used to copy components. Similarly, the Del key can be use to erase components. Now, we can make use of the W key to insert wires between components and make connections. Next, we should rename each pin to something sensible using the Q key to bring up the parameter window.

![inv-wired](Day1/1-9.png)

To configure the properties of the devices, we can select them by clicking on them and bringing up the parameter window with the Q key. We change the length in the parameter window to 0.18 as the default value of 0.15 is restricted to sram devices only. We can set the number of fingers to 3, and the width of each finger to 1.5. As we have 3 fingers however, the total width in the parameter window must be set to 3 times of the finger width, which is 4.5. The other parameters are PDK specific so it is best not to alter them unnecessarily.

![inv-nfet-param](Day1/1-10.png)

Similarly, we can adjust the parameters of the pfet to 3 fingers, width of 1 per finger, and a length of 0.18. We must specify the body to be connected to the Vdd pin as it is a 3 pin pfet.

![inv-pfet-param](Day1/1-11.png)

Now, we have a valid schematic for a basic inverter. We can save this using "Save as" and saving it under the local user directory and renaming it to a suitable name.

To functionally validate the schematic, we must create a testbench that is seperate from the schematic. First, we create a symbol for the schematic, as the schematic will appear as a symbol in the testbench. To do this, we click on the Symbol menu and select "Make symbol from schematic". Now, we can create a testbench schematic using the new schematic option and insert the generated symbol from the local directory using the Ins key.

The testbench will be very simple. We will generate a ramp input and watch the output response after connecting the power supplies. To do this, we can insert 2 voltage sources from the default xschem library, one ofr the input and one for the supply. We can connect these and add a GND node to the supply connections. Next, we must create "opins" for the input and ouput signals that we want to see in Ngspice. If done correctly, we should get the following.

![inv-test](Day1/1-12.png)

Next, we set the values for the voltages. The supply voltage is set to 1.8v. For the input voltage, we must set the supply to a piece-wise linear function to get ramp like behaviour as follows.

![inv-pwl](Day1/1-13.png)

Here, the PWL function has voltage and time values that state that the supply will start at 0v, then start to ramp up from 20 ns till it reaches its final value at 900 ns of 1.8v. Next, we must place two more statements for ngspice, but as these arent specific to any component, they must be placed in text boxes. To place a text box, we select the code_shown.sym component under the xschem library.

The first one will specify the location of the device models used in the device schematic. We will use a .lib statement that selects a top level file that tells ngspice where to find all the models and also specifying a simulation corner for all the models. We shall use the typical corner with ```value = ".lib /usr/share/pdk/sky130A/libs.tech/ngspice/sky130.lib.spice tt"```. For the second block, we shall use the statement as follows.

```value = ".control
tran 1n 1u
plot V(in) V(out)
.endc"
```

This will tell ngspice to run a transient simulation for 1 ns and monitor voltages for the in and out pins. Finally, we should get the completed testbench schematic as follows, and save this as inverter_tb.sch.

![inv-tb](Day1/1-14.png)

To generate the netlist we can click on the Netlist button, then we can simulate it in Ngspice by clicking the Simulate button. We should get the following waveform, which confirms that our schematic behaves as an inverter.

