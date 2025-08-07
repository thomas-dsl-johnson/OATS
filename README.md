# Configuration-of-oneAPI-FPGA-Runtime-for-DE10-Agilex

```bash
# remove the current oneapi install
cd /opt/intel/ 
sudo rm -rf oneapi/

# Get the 2025.2.0 version of Intel oneAPI Base Toolkit offline installer - Linux
# I could not find a way to install any early versions.
cd /
sudo wget https://registrationcenter-download.intel.com/akdlm/IRC_NAS/bd1d0273-a931-4f7e-ab76-6a2a67d646c7/intel-oneapi-base-toolkit-2025.2.0.592_offline.sh
ls # Should see intel-oneapi-base-toolkit-2025.2.0.592_offline.sh

# get the FPGA support package offline installer - Linux
sudo wget https://registrationcenter-download.intel.com/akdlm/IRC_NAS/ff129224-aa19-48f7-96d4-ad12d2d427f9/intel-fpga-support-for-compiler-2025.0.0.591_offline.sh
ls # Should also now see intel-fpga-support-for-compiler-2025.0.0.591_offline.sh

# make the offline installers executable
sudo chmod +x intel-oneapi-base-toolkit-2025.2.0.592_offline.sh
sudo chmod +x intel-fpga-support-for-compiler-2025.0.0.591_offline.sh

# run the installer
sudo ./intel-oneapi-base-toolkit-2025.2.0.592_offline.sh
# "The following tools have been processed successfully: 
#  Intel® oneAPI Base Toolkit 2025.2
#  Download location: /root/intel"
sudo ./intel-fpga-support-for-compiler-2025.0.0.591_offline.sh
# "The following tools have been processed successfully:
#  FPGA Support Package for the Intel® oneAPI DPC++/C++ Compiler 2025.0.0
#  Download location: /root/intel"

# clean up
sudo rm intel-fpga-support-for-compiler-2025.0.0.591_offline.sh
sudo rm intel-oneapi-base-toolkit-2025.2.0.592_offline.sh

# source setvars.sh
cd intel/oneapi
chmod +x setvars.sh
source setvars.sh
cd /
# add to ./bashrc file
echo 'source /opt/intel/oneapi/setvars.sh' >> ~/.bashrc

# Do Quartus Install 
# End of Quartus Install
export QUARTUS_ROOTDIR=/intelFPGA_pro/21.2/quartus/

# Install DE10-Agilex Board Support Package
cd ~
wget https://download.terasic.com/downloads/cd-rom/de10-agilex/linux_BSP/DE10_Agilex_revC_linux_BSP_21.2.zip
unzip DE10_Agilex_revC_linux_BSP_21.2.zip
# Password: ***REMOVED***
rm DE10_Agilex_revC_linux_BSP_21.2.zip
sudo mkdir -p /opt/intel/oneapi/intelfpgadpcpp/latest/board/
sudo mv de10_agilex/ /opt/intel/oneapi/intelfpgadpcpp/latest/board/
cd /opt/intel/oneapi/intelfpgadpcpp/latest/board/de10_agilex/bringup/B2E2_8GBx4/
./bringup_fpga.sh fpga

#
lspci -d 1172: 
# 01:00.0 Processing accelerators: Altera Corporation Device 35b4 (rev 01)
export PATH=/intelFPGA_pro/21.2/hld/host/linux64/bin:$PATH
aocl diagnose
# No board support packages installed.

# Download vector add
oneapi-cli
mkdir build
cd build
cmake ..
make cpu-gpu
./vector-add-buffers
# Running on device: AMD Ryzen 3 3200G with Radeon Vega Graphics
# Vector size: 10000
# [0]: 0 + 0 = 0
# [1]: 1 + 1 = 2
# [2]: 2 + 2 = 4
# ...
# [9999]: 9999 + 9999 = 19998
# Vector add successfully completed on device.
```

**Intel® oneAPI Base Toolkit 2025.2**

Get started guide: https://www.intel.com/content/www/us/en/docs/oneapi-base-toolkit/get-started-guide-linux/current/overview.html

Developer forum: https://community.intel.com/t5/Software-Development-Tools/ct-p/software-dev-tools

Release notes: https://www.intel.com/content/www/us/en/developer/articles/release-notes/intel-oneapi-toolkit-release-notes.html

