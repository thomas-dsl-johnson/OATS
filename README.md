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

**Intel® oneAPI Base Toolkit 2025.2**

Get started guide: https://www.intel.com/content/www/us/en/docs/oneapi-base-toolkit/get-started-guide-linux/current/overview.html

Developer forum: https://community.intel.com/t5/Software-Development-Tools/ct-p/software-dev-tools

Release notes: https://www.intel.com/content/www/us/en/developer/articles/release-notes/intel-oneapi-toolkit-release-notes.html

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
