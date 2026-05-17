# Manta M8P v1.1 & CB1 Deployment Guide (Debian 13 Platform)

This repository provides the configuration profiles, automated environment deployment tooling, and low-level firmware flashing instructions required to run a modernized Klipper stack on a **BIGTREETECH Manta M8P v1.1** powered by a **CB1 Compute Module** running a custom Python 3.13 / Debian 13 baseline.

---

## <center><h1>Required Items before starting:</h1></center>

* **Hardware:** 
  * BIGTREETECH Manta M8P v1.1 Mainboard (STM32H723 variant)
  * BIGTREETECH CB1 Compute Module
  * High-quality MicroSD Card (8GB or larger)
* **Base OS Image:** `CB1_Debian13_minimal_kernel7.0_20260430.img.xz`
* **Flashing Tool:** BalenaEtcher, Raspberry Pi Imager, or `dd`

---

## 1. Base OS Setup

1. Flash your MicroSD card with the `CB1_Debian13_minimal_kernel7.0_20260430.img.xz` image.
2. Configure your Wi-Fi network settings in the `system.cfg` or `wpa_supplicant.conf` file on the boot partition before unmounting if utilizing a wireless connection.
3. Insert the card into the CB1 slot, mount the module onto the Manta M8P layout, power on the system, and establish an SSH terminal handshake:
   ```bash
   ssh biqu@<your_cb1_ip>
   

```

---

## 2. Automated Klipper Deployment (The "Vibe Code" Method)

Modern Debian 13 layers enforce strict PEP 668 environment policies and introduce core dependency updates that break legacy Klipper installers (specifically causing compilation blocks on older version locks for `cffi` and `greenlet`).

The standalone deployment script below automates the entire mitigation strategy by building an un-isolated virtual environment, patching modern requirements hooks, running low-level C-extension adjustments for Python 3.13, and spinning up the systemd unit files.

### Installation Steps

1. Create a new deployment helper script:
```bash
nano deploy_klipper.sh

```


2. Copy and paste the entire bash script block below into the editor view:

```bash
#!/usr/bin/env bash
# ==============================================================================
# K1 Klipper Deployment & Python 3.13 Repair Script (New Edition)
# Target Host: BigTreeTech CB1 running Armbian Linux (Debian 13 Trixie baseline)
# ==============================================================================
set -e # Exit immediately if any command fails

# Color configurations for clean logging output
GREEN='\033[0;32m'
BLUE='\033[0;34m'
YELLOW='\033[1;33m'
RED='\033[0;31m'
NC='\033[0m' # No Color

log_info() { echo -e "${BLUE}[INFO]${NC} $1"; }
log_success() { echo -e "${GREEN}[SUCCESS]${NC} $1"; }
log_warn() { echo -e "${YELLOW}[WARN]${NC} $1"; }
log_error() { echo -e "${RED}[ERROR]${NC} $1"; exit 1; }

log_info "Starting Klipper deployment and Python 3.13 environment alignment..."

# 1. Swap the Klipper repository using explicit home folder structures
cd ~
if [ -d "/home/biqu/klipper" ]; then
    BACKUP_NAME="/home/biqu/klipper_fresh_backup_$(date +%s)"
    log_warn "Existing Klipper directory found. Moving to: ${BACKUP_NAME}"
    mv /home/biqu/klipper "$BACKUP_NAME"
fi

log_info "Cloning fresh sbtoonz/k1_klipper branch..."
git clone [https://github.com/sbtoonz/k1_klipper.git](https://github.com/sbtoonz/k1_klipper.git) /home/biqu/klipper

# 2. Open the requirements file to remove old version locks before executing the install
log_info "Patching Python version requirements locks for Debian 13 compatibility..."
sed -i 's/cffi==1.14.6/cffi>=1.17.1/g' /home/biqu/klipper/scripts/klippy-requirements.txt
sed -i 's/greenlet==2.0.2/greenlet>=3.1.1/g' /home/biqu/klipper/scripts/klippy-requirements.txt

# 3. Clean and initialize the Python 3 virtual environment
log_info "Re-initializing isolated PEP 668 compliant virtual environment..."
rm -rf /home/biqu/klippy-env
python3 -m venv /home/biqu/klippy-env

# 4. Upgrade foundational tooling inside the environment execution track
log_info "Upgrading core packaging tools (pip, setuptools, wheel)..."
/home/biqu/klippy-env/bin/pip install --upgrade pip setuptools wheel

# 5. Install the unlocked dependencies along with modern NumPy for Python 3.13
log_info "Installing unlocked requirements and compiling NumPy wheels..."
/home/biqu/klippy-env/bin/pip install -r /home/biqu/klipper/scripts/klippy-requirements.txt
/home/biqu/klippy-env/bin/pip install --upgrade numpy

# 6. Manually compile the custom branch background C-extensions accurately
log_info "Compiling low-level kinematic optimization helper modules..."
cd /home/biqu/klipper/klippy/chelper
gcc -shared -fPIC -O2 -Wall \
  -I/usr/include/python3.13 \
  -I/home/biqu/klippy-env/include \
  -o c_helper.so *.c
cd ~

# 7. Run runtime imports verification checks
log_info "Running internal core ecosystem verification checks..."
if /home/biqu/klippy-env/bin/python -c "import cffi; import greenlet; import numpy; print('Validation check successful')"; then
    log_success "Environment integrity check passed cleanly!"
else
    log_error "Environment verification failed. Critical core modules missing."
fi

# 8. Force clean the systemd daemon states and kick off the Klipper host engine
log_info "Reloading systemd engines and recycling host service tracker..."
sudo systemctl daemon-reload
sudo systemctl restart klipper.service

# 9. Verify the final active status profile
sleep 2
if systemctl is-active --quiet klipper.service; then
    log_success "===================================================="
    log_success "  KLIPPER SERVICE IS NOW ACTIVE AND RUNNING FINE!   "
    log_success "===================================================="
else
    log_error "Klipper service failed to maintain an active runtime state. Check /tmp/klippy.log"
fi

```

3. Save and close the editor (`CTRL+O`, `Enter` to save, then `CTRL+X` to exit).
4. Update script execution access rules:
```bash
chmod +x deploy_klipper.sh


```



```

5. Trigger the pipeline deployment run:
   ```bash
   ./deploy_klipper.sh
   

```

---

## 3. Configure and Compile Klipper Firmware

Now that the host environment is normalized, build the microcontroller unit (MCU) microcode tailored specifically for the **STM32H723** variant of the Manta M8P v1.1. This establishes a high-bandwidth USB-to-CAN bus bridge layer directly through the board's infrastructure layout.

1. Navigate to the core Klipper workspace:
```bash
cd ~/klipper
make menuconfig


```



```

2. Mirror these low-level parameters precisely within the selection fields:
   * **`[*] Enable extra low-level configuration options`**
   * **Micro-controller Architecture:** `STMicroelectronics STM32`
   * **Processor model:** `STM32H723`
   * **Bootloader offset:** `128KiB bootloader`
   * **Clock Reference:** `25 MHz crystal`
   * **Communication interface:** `USB to CAN bus bridge (USB on PA11/PA12)`
   * **CAN bus interface:** `CAN bus (on PD0/PD1)`
   * **CAN bus speed:** `1000000`

3. Save changes and exit (`Q`, then confirm with `Y`).
4. Generate the compiled flash payload binary file:
   ```bash
   make clean
   make
   

```

The compiled artifact will be outputted cleanly directly to your localized path at `~/klipper/out/klipper.bin`.

---

## 4. Hardware Flash Execution (DFU Mode)

Deploy the newly generated binary over the internal interface by forcing the M8P's microcontroller to interact using a native hardware DFU mapping track.

### Enter DFU Mode Physically:

1. Locate the **BOOT0** button and the **RST** (Reset) hardware switches on the surface of the Manta M8P mainboard.
2. Press down and **hold** the **BOOT0** button.
3. Click and **release** the **RST** button once.
4. **Release** your hold on the **BOOT0** button.

### Validate Connection Profile:

Verify that the underlying Linux operating system has safely detected the bootloader interface hardware ID signature:

```bash
lsusb

```

Ensure a record pointing to device map code `0483:df11` is explicitly listed in your active USB topology view.

### Execute the Flash:

Flash the binary down to the chip architecture over the DFU mapping pipe:

```bash
make flash FLASH_DEVICE=0483:df11

```

### Understanding Flashing Diagnostic Output:

During the write process, the installation macro will monitor write percentages up to 100%, and then inevitably end with a validation exception looking similar to this:

```text
Download    [=========================] 100%        31336 bytes
Download done.
File downloaded successfully
dfu-util: Error during download get_status
Failed to flash to 0483:df11: Error running dfu-util
make: *** [src/stm32/Makefile:100: flash] Error 255

```

> **CRITICAL REASSURANCE:** Receiving an `Error during download get_status` or `Error 255` notice at the conclusion of flash operations is completely normal behavior for STM32 configurations. The error triggers because the microcontroller initializes the newly deployed Klipper application immediately after the download phase, dropping its standard USB bootloader connection handle before the host logging system can process a normal closure handshake. **Your firmware has been written successfully.**

---

## 5. Maintenance and Post-Flash Initialization

1. Press the **RST** (Reset) button on the Manta M8P physical board to initialize the newly flashed firmware stack runtime parameters.
2. If future OS system upgrades modify underlying Python library packages or cause disconnection bugs across the runtime stack over time, refresh the host environment cleanly at any moment via your script hook:
```bash
./deploy_klipper.sh

```
