# OneAPI and Agilex Technical Setup
## Configuration-of-oneAPI-FPGA-Runtime-for-DE10-Agilex

```bash
# Installing Intel oneAPI Base Toolkit. Download the offline installer for Intel oneAPI Base Toolkit and execute it as follows:
# We get the 2025.2.0 version of Intel oneAPI Base Toolkit offline installer - Linux. I could not find a way to install any early versions.
cd /
sudo wget https://registrationcenter-download.intel.com/akdlm/IRC_NAS/bd1d0273-a931-4f7e-ab76-6a2a67d646c7/intel-oneapi-base-toolkit-2025.2.0.592_offline.sh
ls # Should see intel-oneapi-base-toolkit-2025.2.0.592_offline.sh

# Installing Intel FPGA Add-on for oneAPI Base Toolkit
# We get the FPGA support package offline installer - Linux
sudo wget https://registrationcenter-download.intel.com/akdlm/IRC_NAS/ff129224-aa19-48f7-96d4-ad12d2d427f9/intel-fpga-support-for-compiler-2025.0.0.591_offline.sh
ls # Should also now see intel-fpga-support-for-compiler-2025.0.0.591_offline.sh

# Make the offline installers executable
sudo chmod +x intel-oneapi-base-toolkit-2025.2.0.592_offline.sh
sudo chmod +x intel-fpga-support-for-compiler-2025.0.0.591_offline.sh

# Run the installers
sudo ./intel-oneapi-base-toolkit-2025.2.0.592_offline.sh
# "The following tools have been processed successfully: 
#  Intel® oneAPI Base Toolkit 2025.2
#  Download location: /root/intel"
sudo ./intel-fpga-support-for-compiler-2025.0.0.591_offline.sh
# "The following tools have been processed successfully:
#  FPGA Support Package for the Intel® oneAPI DPC++/C++ Compiler 2025.0.0
#  Download location: /root/intel"

# Clean up
sudo rm intel-fpga-support-for-compiler-2025.0.0.591_offline.sh
sudo rm intel-oneapi-base-toolkit-2025.2.0.592_offline.sh

# Installing Additional Libraries and Tools
# Some samples and build systems require development tools not included in the base toolkit. Install the following:
sudo apt update
sudo apt -y install cmake pkg-config build-essential libtinfo5 libncurses5

# Installing USB Blaster II Driver
# To enable the USB Blaster II device:
sudo nano /etc/udev/rules.d/51-usbblaster.rules
# Add the following lines:
#    # USB-Blaster
#    ENV{ID_BUS}=="usb", ENV{ID_VENDOR_ID}=="09fb", ENV{ID_MODEL_ID}=="6001", MODE="0666"
#    ENV{ID_BUS}=="usb", ENV{ID_VENDOR_ID}=="09fb", ENV{ID_MODEL_ID}=="6002", MODE="0666"
#    ENV{ID_BUS}=="usb", ENV{ID_VENDOR_ID}=="09fb", ENV{ID_MODEL_ID}=="6003", MODE="0666"
#    # USB-Blaster II
#    ENV{ID_BUS}=="usb", ENV{ID_VENDOR_ID}=="09fb", ENV{ID_MODEL_ID}=="6010", MODE="0666"
#    ENV{ID_BUS}=="usb", ENV{ID_VENDOR_ID}=="09fb", ENV{ID_MODEL_ID}=="6810", MODE="0666"
# Then reload udev rules or reboot.

# Do Quartus Install - You need to install Quartus 21.2 (not other versions)
# Previously, this was not necessary. Now we need to install it separately because of the divorce between Intel and Altera.
sudo wget 'https://downloads.intel.com/akdlm/software/acdsinst/21.2/72/ib_tar/Quartus-pro-21.2.0.72-linux-complete.tar'
# Extract
tar -xf Quartus-pro-21.2.0.72-linux-complete.tar
./setup_pro.sh 
export QUARTUS_ROOTDIR=/intelFPGA_pro/21.2/quartus/

# Installing the DE10-Agilex Board Support Package (BSP)
# Unzip the BSP and move it to the oneAPI board directory:
cd ~
wget https://download.terasic.com/downloads/cd-rom/de10-agilex/linux_BSP/DE10_Agilex_revC_linux_BSP_21.2.zip
unzip DE10_Agilex_revC_linux_BSP_21.2.zip
# Password: ###################### REDACTED ################################
rm DE10_Agilex_revC_linux_BSP_21.2.zip
sudo mkdir -p /opt/intel/oneapi/intelfpgadpcpp/latest/board/
sudo mv de10_agilex/ /opt/intel/oneapi/intelfpgadpcpp/latest/board/

# Setting up the Board and Environment
# Source the oneAPI environment:
cd intel/oneapi
chmod +x setvars.sh
source setvars.sh
cd /
# Add to ./bashrc file
echo 'source /opt/intel/oneapi/setvars.sh' >> ~/.bashrc

# Run the bring-up script:
cd /opt/intel/oneapi/intelfpgadpcpp/latest/board/de10_agilex/bringup/B2E2_8GBx4/
./bringup_fpga.sh fpga

# Verify PCIe device visibility:
lspci -d 1172: 
# 01:00.0 Processing accelerators: Altera Corporation Device 35b4 (rev 01)
export PATH=/intelFPGA_pro/21.2/hld/host/linux64/bin:$PATH
aocl diagnose
# No board support packages installed. I think this is not good ...

# Download vector add
oneapi-cli
mkdir build
cd build
cmake ..
make cpu-gpu
./vector-add-buffers
# Output:
# Running on device: AMD Ryzen 3 3200G with Radeon Vega Graphics
# Vector size: 10000
# [0]: 0 + 0 = 0
# [1]: 1 + 1 = 2
# [2]: 2 + 2 = 4
# ...
# [9999]: 9999 + 9999 = 19998
# Vector add successfully completed on device.

# Note:
# Using oneapi-cli 2025 does not have any FPGA options.
```

Upon opening a fresh terminal:
```bash
export QUARTUS_ROOTDIR=/intelFPGA_pro/21.2/quartus/
export AOCL_BOARD_PACKAGE_ROOT=/opt/intel/oneapi/intelfpgadpcpp/latest/board/de10_agilex
export PATH=$QUARTUS_ROOTDIR/bin:$PATH
export LD_LIBRARY_PATH=$AOCL_BOARD_PACKAGE_ROOT/linux64/lib:$LD_LIBRARY_PATH
export PATH=/intelFPGA_pro/21.2/hld/host/linux64/bin:$PATH
aocl diagnose
aocl install

source /opt/intel/oneapi/intelfpgadpcpp/latest/board/de10_agilex/init_env.sh
cd /opt/intel/oneapi/intelfpgadpcpp/latest/board/de10_agilex/bringup/B2E2_8GBx4/
./bringup_fpga.sh fpga
export PATH=/intelFPGA_pro/21.2/hld/host/linux64/bin:$PATH
aocl diagnose
```
## Configuration of Docker Image of OneAPI
Since Intel's acquisition of Altera, the FPGA development tools (Quartus, now under the Altera brand again) and the OneAPI software tools have been on increasingly separate development tracks. You cannot mix and match major versions and expect them to work. So, the plan now is to use Docker.

**What is it:** A Docker image is a self-contained package with a specific Linux OS, all the correct tool versions (Quartus, oneAPI, libraries), and pre-configured environment variables.

**Why is it useful:** It completely isolates the development environment from your host machine, eliminating any chance of version conflicts or incorrect setup.

On FPGA server:
```bash
# 1. Install Docker

# 2. Pull the relevant image from Intel's Docker Hub.
docker pull intel/oneapi-basekit:2022.1.2-devel-ubuntu18.04
# The download is 5.6 Gb, it may take some time

# ... Proceed Similarly to `on local`
# Need to set --device flag on setup
```

On local (M1 MacBook):
```bash
# 1. Install Docker
# Download from https://www.docker.com/products/docker-desktop/
# Mount the .dmg

# 2. Pull the relevant image from Intel's Docker Hub. See https://hub.docker.com/r/intel/oneapi-basekit/tags?page=3
docker pull intel/oneapi-basekit:2022.1.2-devel-ubuntu18.04
# The download is 5.6 Gb, it may take some time

# 3. Set up directory
mkdir ~/docker_oneapi
cd ~/docker_oneapi

# 4. Create the container
docker run -it -v ~/oneapi_project:/host intel/oneapi-basekit:2022.1.2-devel-ubuntu18.04
# Open a new terminal and list containers
docker ps -a
# Copy CONTAINER ID of intel/oneapi-basekit:2022.1.2-devel-ubuntu18.04 e.g. abc123def456
# Create Image
docker commit abc123def456 my_oneapi_image
# Close container and run
docker run -it --name my_oneapi_dev -v ~/oneapi_project:/host my_oneapi_image
# In future
docker start -ai my_oneapi_dev
# n.b. If you ever want to get rid of it and start fresh
docker rm my_oneapi_dev

```


On Cloud VM:
```bash
# We are on Debian 6.1.140-1 (2025-05-22) x86_64

# 1. Install Docker Engine
# Following instructions on: 
# https://docs.docker.com/engine/install/debian/
sudo apt update
sudo apt install apt-transport-https ca-certificates curl software-properties-common
# Use the convenience script
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# 2. Pull the relevant image from Intel's Docker Hub. See https://hub.docker.com/r/intel/oneapi-basekit/tags?page=3
sudo docker pull intel/oneapi-basekit:2022.1.2-devel-ubuntu18.04

# 3. Make dir
mkdir ~/oneapi_fpga_project
cd oneapi_fpga_project

# 4. Create the container
sudo docker run --name my_oneapi_dev -it -v ~/oneapi_fpga_project:/host_project intel/oneapi-basekit:2022.1.2-devel-ubuntu18.04 /bin/bash
# In future
sudo docker start -ai my_oneapi_dev
```

Inside docker:
```bash
# 5. Install dependencies 
apt-get update && apt-get install -y git wget unzip build-essential pkg-config cmake libtinfo5 libncurses5

# 6. Install Quartus
wget https://downloads.intel.com/akdlm/software/acdsinst/21.2/72/ib_tar/Quartus-pro-21.2.0.72-linux-complete.tar
tar -xf Quartus-pro-21.2.0.72-linux-complete.tar
./setup_pro.sh 
export QUARTUS_ROOTDIR=/intelFPGA_pro/21.2/quartus/

# 7.
# Get samples https://github.com/oneapi-src/oneAPI-samples/releases/tag/2022.1.0
apt-get update && apt-get install -y git
# Could not find vector add in the following: git clone -b 2022.1.0 https://github.com/oneapi-src/oneAPI-samples.git
git clone -b 2023.1.0 https://github.com/oneapi-src/oneAPI-samples.git
cd /oneAPI-samples/DirectProgramming/C++SYCL_FPGA/Tutorials/GettingStarted/fpga_compile

## Part 1
## n.b. goal is to compile and run a standard C++ version of vector addition. 
## This version uses no SYCL and runs only on CPU.
## It serves is a non-FPGA starting point.
cd part1_cpp
mkdir build
cd build
cmake ..
make fpga_emu
./vector_add.fpga_emu
# RESULT:
# Started/fpga_compile/part1_cpp/build# ./vector_add.fpga_emu
# add two vectors of size 256
# PASSED

## Part 2
cd ../..
cd part2_dpcpp_functor_usm
mkdir build
cd build
cmake ..
make fpga_emu
```



