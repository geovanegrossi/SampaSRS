# SampaSRS

## How to build

The code has the following build dependencies

- C++17 compiler
- Git
- CMake (3.14 or later)
- [ROOT](https://root.cern/) (6.26 or later)
- [libpcap](https://www.tcpdump.org/index.html)
- SDL2 (for sampa_gui)

### Ubuntu dependencies

    sudo apt install build-essential git cmake libpcap-dev libsdl2-dev

### Windows dependencies

Download the precompiled binaries from [WinPcap](https://www.winpcap.org/install/bin/WpdPack_4_1_2.zip).

To build you also need to pass the location of the extracted binaries to CMake:

    cmake .. -DPCAP_ROOT_DIR=PathTo_libpcap_DevPack <remaining flags>

### Build

    mkdir build && cd build
    cmake .. -DCMAKE_INSTALL_PREFIX=../install
    cmake --build . --target install

### Cluster build

It's possible to build only the reconstruction code to be able to run in a cluster environment, removing `libpcap` and `SDL2` as dependencies. To accomplish that run CMake with the following flags:

    mkdir build && cd build
    cmake .. -DCMAKE_INSTALL_PREFIX=../install -DSAMPA_BUILD_ACQUISITION=OFF -DSAMPA_NATIVE_OPTIMIZATION=OFF
    cmake --build . --target install

The `SAMPA_BUILD_ACQUISITION` flag will disable all acquisition-related code, including the GUI. If the code needs to run on a different machine than the one it was compiled you probably should also disable the native optimizations, setting `SAMPA_NATIVE_OPTIMIZATION` to off. This will ensure the code is compatible with multiple processor architectures, with some performance cost.

### Running on Linux

>[!IMPORTANT]
>The executables `sampa_acquisition` and `sampa_gui` needs special permissions to read raw network sockets. You can run those as root with `sudo` or you can give them the permission to run as a normal user with:

    sudo setcap cap_net_raw=pe sampa_acquisition
    sudo setcap cap_net_raw=pe sampa_gui

## Running the acquisition

To start a acquisition first configure the system using `sampa_control`. After that, execute `sampa_gui`. You can choose to save the file or not. 

    ./sampa_control config.txt
    sudo ./sampa_gui

The file _config.txt_ contains the different I2C commands to configure each SAMPA. An example is provided but different applications will have different configuration files. The complete list of commands and addresses can also be found at `include/sampasrs/slow_control.hpp` 

## Processing data

The data processing chain starts with `sampa_decoder`. Use it to process _.raw_ files into _.root_ TTres. To create the pedestal files for non-zero suppression runs (this is the only feature available at the moment - zero suppression runs under construction :hammer_and_wrench:) run the following command:

    ./sampa_decoder pedestal_file.raw

You will find that along with a text file containing the pedestal information, a graph and a small calculation to check integrity was made. 

To process the data produced by the detector first decode the acquisition file with `sampa_decoder`. To generate the clusters and reconstruct events execute `clustering`

    ./clustering <pedestal_file.txt> <data_file.root>