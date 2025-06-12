# eclypse_z7_adc_adc_linux
A suite of tools, libraries and projects used to get dual ZMOD adcs as linux devices on the Eclypse Z7 Platform


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