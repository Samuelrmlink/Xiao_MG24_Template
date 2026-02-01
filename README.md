# Xiao MG24 Template
Intended to assist with spinning up new projects using the Seeed Studio Xiao MG24 module with Zephyr RTOS. 

## Enabled Features
* Bluetooth LE
* SMP SVR
* Mcuboot
* Bluetooth OTA (via Mcuboot & SMP SVR)
* NUS Service (Nordic Semi's UART over Bluetooth LE)
* Shell via NUS Service

## Zephyr Environment Setup

Usage of Ubuntu is assumed here. WSL2 and/or Docker/Podman may be helpful.

Install dependencies from APT

```bash
sudo apt update
sudo apt upgrade
sudo apt install --no-install-recommends git cmake ninja-build gperf ccache dfu-util device-tree-compiler wget python3-dev python3-venv python3-tk xz-utils file make gcc gcc-multilib g++-multilib libsdl2-dev libmagic1
```


Install Zephyr Workspace (Generic)

```bash
mkdir ~/zephyrproject
python3 -m venv ~/zephyrproject/.venv
source ~/zephyrproject/.venv/bin/activate
pip install west
west init -m https://github.com/zephyrproject-rtos/zephyr --mr v4.2.0 ~/zephyrproject
cd ~/zephyrproject
west update
west zephyr-export
west packages pip --install
cd ~/zephyrproject/zephyr
west sdk install
west blobs fetch
```


Install OpenOCD fork (includes support for CMSIS-DAPv2 and EFRMG24)

```bash
git clone --depth 1 -b arduino-0.12.0-rtx5 https://github.com/facchinm/OpenOCD ~/zephyrproject/tools/src-openocd
cd ~/zephyrproject/tools/src-openocd
sudo apt install -y make libtool pkg-config autoconf automake texinfo libhidapi-dev libusb-1.0-0-dev
mkdir -p ~/zephyrproject/tools/openocd
./bootstrap
./configure --prefix $HOME/zephyrproject/tools/openocd
make -j4
make install
# Optionally: rm -rf ~/zephyrproject/tools/src-openocd
```


Download this project into your Zephyr Workspace

```bash
cd ~/zephyrproject
git clone https://github.com/Samuelrmlink/Xiao_MG24_Template.git Xiao_test
```


## Build Project

Compile Project

```bash
cd ~/zephyrproject
source ~/zephyrproject/zephyr/zephyr-env.sh
source ~/zephyrproject/.venv/bin/activate
west build -b xiao_mg24 --pristine --sysbuild Xiao_test -- -DEXTRA_CONF_FILE="overlay-bt.conf" -DCONFIG_MCUBOOT_SIGNATURE_KEY_FILE=\"bootloader/mcuboot/root-rsa-2048.pem\" -DCONFIG_BOOTLOADER_MCUBOOT=y
west flash --openocd=/home/samuel/.platformio/packages/tool-openocd/bin/openocd --openocd-search=/home/samuel/.platformio/packages/tool-openocd/share/openocd/scripts/
```


Flash Project

```bash
west flash --openocd=$HOME/zephyrproject/tools/openocd/bin/openocd --openocd-search=$HOME/zephyrproject/tools/openocd/share/openocd/scripts/
```


OTA Project via nRF Connect App
* Copy the '~/zephyrproject/build/Xiao_test/zephyr/zephyr.signed.bin' to your iOS/Android phone
* Open the 'nRF Connect' app
* Connect to the 'Xiao Test App' device from the scanner interface
* Tap 'DFU' and then select the 'zephyr.signed.bin' file from your Internal Storage
* Select mode: 'Test and Confirm' - which will automatically install & validate the OTA firmware upgrade
