# XCLASS Installation Guide

This document is divided into two parts, detailing the installation steps for XCLASS (v1.4.3) on Ubuntu 22.04 LTS and Apple Silicon (M3) macOS. The installation processes, encountered issues, and solutions for the two systems are described separately.

**Reference:** [XCLASS Official Manual](https://xclass-pip.astro.uni-koeln.de/manual/installing.html)

---

# Table of Contents

- [Ubuntu 22.04 LTS Installation Process](#ubuntu-2204-lts-installation-process)
- [Common Issues and Solutions During Installation](#2-common-issues-and-solutions-during-installation)
- [Recommended Dependency Versions](#3-recommended-dependency-versions)
- [Installation Steps Summary](#4-installation-steps-summary)
- [Testing the Installation](#5-testing-the-installation)
- [Professional Advice](#6-professional-advice)
- [Installing XCLASS (v1.4.3) on Mac M3](#installing-xclass-v143-on-mac-m3)
- [Reference Links](#reference-links)

---

# Ubuntu 22.04 LTS Installation Process

## 1. Preparation

- **Operating System:** Ubuntu 22.04 LTS
- **Python Version:** 3.10.x (recommended: [deadsnakes PPA](https://launchpad.net/~deadsnakes/+archive/ubuntu/ppa))
- **OpenMPI Version:** 1.8.6 (needs to be compiled manually)
- **XCLASS Source:** Offline installation package `xclass_pip_off/`
- **Note:** This guide does not use virtual environments; all operations are performed on the main system. Please assess the risks.

### System and Basic Tools Installation

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y build-essential git cmake gfortran libfftw3-dev libgsl-dev \
libhdf5-dev liblapack-dev libblas-dev zlib1g-dev libcurl4-openssl-dev libgomp1
```

### Python 3.10 Installation and Setup

```bash
sudo add-apt-repository ppa:deadsnakes/ppa -y
sudo apt update
sudo apt install -y python3.10 python3.10-dev python3.10-venv python3-pip
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.10 1
sudo update-alternatives --config python3  # Select Python 3.10
```

### OpenMPI 1.8.6 Compilation and Installation (You may try a newer version first; if it fails, downgrade and compile manually)

```bash
mkdir -p ~/tmp/openmpi_install && cd ~/tmp/openmpi_install
wget https://www.open-mpi.org/software/ompi/v1.8/downloads/openmpi-1.8.6.tar.gz
tar -xzf openmpi-1.8.6.tar.gz && cd openmpi-1.8.6
./configure --prefix=/opt/openmpi-1.8.6
make -j$(nproc)
echo 'export PATH="/opt/openmpi-1.8.6/bin:$PATH"' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH="/opt/openmpi-1.8.6/lib:$LD_LIBRARY_PATH"' >> ~/.bashrc
source ~/.bashrc
```

### pip Path Setup

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

---

## 2. Common Issues and Solutions During Installation

### Pillow Import Error

- **Error Message:** `ImportError: cannot import name '_imaging' from 'PIL'`
- **Solution:**
    ```bash
    python3 -m pip uninstall Pillow PIL -y
    python3 -m pip install Pillow
    ```

### setuptools Version Incompatibility

- **Error Message:** `AttributeError: install_layout`
- **Solution:**
    ```bash
    sudo python3 -m pip uninstall setuptools -y
    sudo python3 -m pip install setuptools==58.0.4
    ```

### matplotlib API Conflict

- **Error Message:** `ImportError: cannot import name 'register_cmap'`
- **Solution:**
    ```bash
    python3 -m pip install matplotlib==3.5.3
    ```

### NumPy Major Version Incompatibility

- **Error Message:** `A module that was compiled using NumPy 1.x cannot be run in NumPy 2.x`
- **Solution:**
    ```bash
    python3 -m pip install numpy==1.25.0
    ```

### astropy and regions Dependency Conflict

- **Solution:**
    ```bash
    python3 -m pip install astropy==5.1 regions==0.10
    ```

---

## 3. Recommended Dependency Versions

| Package      | Version   |
| ------------ | --------- |
| APLpy        | 2.2.0     |
| astropy      | 5.1       |
| matplotlib   | 3.5.3     |
| numpy        | 1.25.0    |
| Pillow       | 11.3.0    |
| regions      | 0.10      |
| setuptools   | 58.0.4    |
| xclass       | 1.4.3     |

---

## 4. Installation Steps Summary

```bash
# 1. Uninstall potentially conflicting packages
python3 -m pip uninstall Pillow matplotlib numpy astropy setuptools -y

# 2. Install exact versions of core dependencies
python3 -m pip install setuptools==58.0.4 numpy==1.25.0 matplotlib==3.5.3 astropy==5.1 Pillow==11.3.0

# 3. Install other dependencies
python3 -m pip install scipy spectral_cube regions PyQt5 h5py lxml emcee ultranest

# 4. Install XCLASS offline package
cd ~/Desktop/xclass/xclass_pip_off
python3 -m pip install . -v
```

---

## 5. Testing the Installation

Enter the Python interactive shell and test imports:

```python
import xclass
from xclass import task_DatabaseQuery, task_myXCLASS, task_myXCLASSFit
from xclass.task_myXCLASSFit import task_myXCLASSMapFit
# All imports should succeed without errors.
```

---

# Installing XCLASS (v1.4.3) on Mac M3

**Goal:** Successfully install the XCLASS software package and all required components on macOS with Apple Silicon (M3 chip).

**Reference:** [XCLASS Official Manual](https://xclass-pip.astro.uni-koeln.de/manual/installing.html)

### Problem Overview

On Mac M3, following the official pip installation guide directly will result in multiple compilation errors, mainly due to incompatibilities and path issues among Fortran/C compilers, Python interface (numpy.f2py), Xcode Command Line Tools SDK, and Homebrew-installed GCC/Gfortran.

Common errors include:

- `ModuleNotFoundError: No module named 'numpy'` (numpy not found during compilation)
- `unknown type name 'FILE'`, `unknown type name 'size_t'` (C compilation errors)
- `gfortran: error: unrecognized command-line option '--fcompiler=gnu95'` (Fortran compilation error)

---

### 1. Prerequisites (Install Dependencies via Homebrew)

Install Homebrew (if not already installed):

```zsh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Install required dependencies:
```zsh
brew install gcc@12
brew install open-mpi
brew install python@3.10
```

Install Python dependencies (specify versions to ensure compatibility with XCLASS):

```zsh
/opt/homebrew/bin/python3.10 -m pip install numpy==1.25.0 scipy matplotlib==3.5.3 astropy==5.1
```
> **Note:** These package versions are chosen based on XCLASS official recommendations and compatibility tests to avoid installation or runtime errors due to API changes in newer versions.

--- 
### 2. XCLASS Environment Variable Setup
Add environment variables to your zshrc.
```zsh
vim ~/.zshrc
```
```zsh
# --- XCLASS Environment Variable Setup ---

# Set Fortran and C compiler paths, explicitly pointing to Homebrew-installed versions
export FC="/opt/homebrew/bin/gfortran"
export CC="/opt/homebrew/bin/gcc-12"

# Force GCC to use its own standard library header paths to avoid conflicts with system SDK
# The `$(...)` expressions automatically obtain the actual version of Homebrew `gcc@12` and your macOS kernel version
export CFLAGS="-I/opt/homebrew/Cellar/gcc@12/$(brew list --version gcc@12 | cut -d' ' -f2)/lib/gcc/aarch64-apple-darwin$(uname -r | cut -d'.' -f1)/12/include -isystem /usr/local/include -isystem /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk/usr/include -isystem /Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include"
export CPPFLAGS="-I/opt/homebrew/Cellar/gcc@12/$(brew list --version gcc@12 | cut -d' ' -f2)/lib/gcc/aarch64-apple-darwin$(uname -r | cut -d'.' -f1)/12/include -isystem /usr/local/include -isystem /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk/usr/include -isystem /Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include"

# Set linker flags
export LDFLAGS="-L/opt/homebrew/lib -lcurl -lz"

# Ensure Homebrew's bin directory and Homebrew Python's bin directory are at the front of PATH for priority
export PATH="/opt/homebrew/bin:$PATH"
export PATH="/opt/homebrew/opt/python@3.10/bin:$PATH"

# (Optional but recommended) Increase OpenMP stack size for parallel computation
ulimit -s unlimited
export KMP_STACKSIZE='3999M'
export OMP_STACKSIZE='3999M'
export GOMP_STACKSIZE='3999M'
# --- End of XCLASS Environment Variable Setup ---
```

Reload zshrc

```zsh
source ~/.zshrc
```

### 3. Modify XCLASS Source Makefile Files

The XCLASS installation script setup.py calls Makefiles in different subdirectories to compile Fortran code and the Python interface. These Makefiles need to be manually modified to ensure numpy.f2py is always invoked via the Homebrew-installed Python 3.10.

Please make precise modifications to the following two files:
`~/yourpathto/xclass_pip_off/xclass/xclass/interface/Makefile`

`~/yourpathto/xclass_pip_off/xclass/xclass/lite/Makefile`
Modification steps (repeat for each file):

a. Enter the corresponding directory:
For example:
```zsh
cd ~/yourpathto/xclass_pip_off/xclass/xclass/interface/
```
b. Backup the Makefile:

```zsh
cp Makefile Makefile.bak
```
c. Edit the Makefile:

```zsh
vim Makefile
```
d. Modify the definition of FCPY:
Near the top of the file, find or add the following line to ensure FCPY points to Homebrew Python's f2py:

```zsh
#interface/Makefile
# set f2py compiler and flags
FCPY = python3 -m numpy.f2py # old

FCPY = /opt/homebrew/bin/python3.10 -m numpy.f2py # new
```

And in lite, redefine FC as FCPY
```zsh
#lite/Makefile
# set f2py compiler and flags
FC = python3 -m numpy.f2py # old

FCPY = /opt/homebrew/bin/python3.10 -m numpy.f2py # new
```

e. Modify the invocation of f2py and sig rules in lite/Makefile:
This is key to resolving the gfortran: error: unrecognized command-line option issue. Find the f2py: and sig: targets in the Makefile and ensure they use $(FCPY) to invoke f2py, not $(FC).

Before (old):
```zsh
#lite/Makefile
f2py:
    $(FC) -c $(FFLAGS) -m $(EXEC) $(FSRC) # <- Directly uses FC, which causes gfortran error

sig:
    $(FC) -h CalculateXCLASS.pyf -m $(EXEC) $(FSRC) # <- Needs modification as well
```
After (new):
```zsh 
#lite/Makefile
f2py:
    $(FCPY) -c $(FFLAGS) -m $(EXEC) $(FSRC)

sig:
    $(FCPY) -h CalculateXCLASS.pyf -m $(EXEC) $(FSRC)
```

---

### 5. Perform Final Installation

Switch to the `xclass_pip_off` root directory and run:

```bash
cd ~/yourpathto/xclass/xclass_pip_off/
python3.10 -m pip install . -v
```

If there are no fatal errors, XCLASS is successfully installed.


## Reference Links

- [Xclass Official Installation Guide](https://xclass-pip.astro.uni-koeln.de/manual/installing.html#required-packages)
