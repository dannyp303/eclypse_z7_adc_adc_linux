# eclypse_z7_adc_adc_linux
A suite of tools, libraries and projects used to get dual ZMOD adcs as linux devices on the Eclypse Z7 Platform


## Table of Contents
- [docker-petalinux](#docker-petalinux)
    - docker build for petalinux with xilinx 2019.1, which was used to create the hardware.
- [Eclypse-Z7-HW](#eclypse-z7-hw)
    - A fork of the Eclypse-Z7-HW repo with a zmod_adc_adc/master branch containing the hardware project files for the dual adc setup.
- [Eclypse-Z7-OS](#eclypse-z7-os)
    - A fork of the Eclypse-Z7-OS repo with a zmod_adc_adc/master branch containing the petalinux build project for the above hardware project.
- [py_ez7_udmabuf](#py-ez7-udmabuf)
    - Source for python bindings that can be used to interact with the zmod adcs. Mostly copied from [miyo's py_eclypse_z7 project](https://github.com/miyo/py_eclypse_z7/tree/main) with alterations to use u-dma-buf instead of the xilinx axidma drivers, as the axidma drivers do not allow us to use two AXI with seperate dma ranges.

## Hardware Description
 The primary goal of this project was to be able to read simultaneously from both ADCs. Some modifications to the original design by Digilent had to be made to facilitate this. 

Below we have the block diagram for our hardware. Things to note are the usage of two sepeate AXI lines. This sepeareates both adcs so they can write simultaneously and so we can recieve signals like buffer full or transfer complete from each adc individually.
![Block Diagram](res/block_diagram.png)

Below is the address editor for our hardware. Here we descretely split our 1G memory space into two seperate 512 Mb regions so that they do not step on eachother.
![Address Editor](res/address_editor.png)

## Operating System Description
The main caveate with our operating system design is that xilinx's axidma driver does not support multiple AXI lines. While i'm not sure the internal reason for why this is designed the way it is, the reason for only one AXI line being operable is that the dma buffer space used is allocated by the system, giving it a random address in memory. Since we split our buffer ranges into two, one ADC's buffer will be outside of its DMA range. 

To solve this, we add the [u-dma-buf](https://github.com/ikwzm/udmabuf) kernel module as a bitbake recipe. This kernel module allows us to statically define dma buffer memory regions in our device tree and access them through user space devices like `/dev/udmabuf0`.

#### NOTE: The bitbake recipe currently does not properly install the .ko. Instead find the module in the kernel build and install it manually on the running system.


## Cross Compiling for the Eclypse Z7
It is highly recommended to cross compile all user space code for the eclypse in a crossenv in petalinux using the SDK for your build.

### Crossenv setup
1. First we have to build the sdk. To do this, after building your image with `petalinux-build` we run `petalinux-build --sdk`.
2. After we have our sdk, we source the sdk env with `source /opt/petalinux-sdk/environment-setup-cortexa9t2hf-neon-xilinx-linux-gnueabi`
3. Make sure you have the same python version you are going to install installed in the petalinux build system. 
    - The recommended way to do this is to build from source natively first inside the build system, then build the same source in the cross env.
4. Build the cross env with `python3 -m crossenv /path/to/host/python3.11 /path/to/crossenv-dir`
5. Activate the crossenv with `source /path/to/crossenv-dir/bin/activate`
6. Set your environment variables used in compilation to match the sdk like so:
    ```
    export CC=arm-linux-gnueabihf-gcc
    export CXX=arm-linux-gnueabihf-g++
    export AR=arm-linux-gnueabihf-ar
    export RANLIB=arm-linux-gnueabihf-ranlib
    export READELF=arm-linux-gnueabihf-readelf
    export STRIP=arm-linux-gnueabihf-strip
    export SYSROOT=/opt/petalinux-sdk/sysroots/cortexa9t2hf-neon-xilinx-linux-gnueabi
    export CFLAGS="--sysroot=$SYSROOT"
    export LDFLAGS="--sysroot=$SYSROOT"
    ```
7. Use `file` to make sure your programs are ARM executables.
8. Compile or even pip install the packages you need, compress them, and send them to your Eclypse-Z7 to be used.