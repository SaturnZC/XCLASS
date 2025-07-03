# XCLASS 安裝指南

本文件分為兩部分，分別說明在 Ubuntu 22.04 LTS 及 Apple Silicon (M3) macOS 上安裝 XCLASS (v1.4.3) 的詳細步驟。兩套系統的安裝流程與遇到的問題、解決辦法完全分開敘述。

**參考文件**：[XCLASS 官方手冊](https://xclass-pip.astro.uni-koeln.de/manual/installing.html)

---

# 目錄

- [Ubuntu 22.04 LTS 安裝流程](#ubuntu-2204-lts-安裝流程)
- [安裝過程中常見問題與解決方案](#2-安裝過程中常見問題與解決方案)
- [推薦的依賴版本組合](#3-推薦的依賴版本組合)
- [安裝步驟總結](#4-安裝步驟總結)
- [測試安裝](#5-測試安裝)
- [專業建議](#6-專業建議)
- [在 Mac M3 上安裝 XCLASS (v1.4.3)](#在-mac-m3-上安裝-xclass-v143)
- [參考連結](#參考連結)

---

# Ubuntu 22.04 LTS 安裝流程

## 1. 安裝前準備

- **作業系統**：Ubuntu 22.04 LTS
- **Python 版本**：3.10.x（建議 [deadsnakes PPA](https://launchpad.net/~deadsnakes/+archive/ubuntu/ppa)）
- **OpenMPI 版本**：1.8.6（需自行編譯）
- **XCLASS 來源**：離線安裝包 `xclass_pip_off/`
- **注意**：本指南不使用虛擬環境，所有操作於主系統進行，請評估風險。

### 系統與基礎工具安裝

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y build-essential git cmake gfortran libfftw3-dev libgsl-dev \
libhdf5-dev liblapack-dev libblas-dev zlib1g-dev libcurl4-openssl-dev libgomp1
```

### Python 3.10 安裝與設定

```bash
sudo add-apt-repository ppa:deadsnakes/ppa -y
sudo apt update
sudo apt install -y python3.10 python3.10-dev python3.10-venv python3-pip
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.10 1
sudo update-alternatives --config python3  # 選擇 Python 3.10
```

### OpenMPI 1.8.6 編譯安裝(可以先試試新版的openmpi，如果不行在降回舊版自行編譯)

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

### pip 路徑設定

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

---

## 2. 安裝過程中常見問題與解決方案

### Pillow 導入錯誤

- **錯誤訊息**：`ImportError: cannot import name '_imaging' from 'PIL'`
- **解決方法**：
    ```bash
    python3 -m pip uninstall Pillow PIL -y
    python3 -m pip install Pillow
    ```

### setuptools 版本不兼容

- **錯誤訊息**：`AttributeError: install_layout`
- **解決方法**：
    ```bash
    sudo python3 -m pip uninstall setuptools -y
    sudo python3 -m pip install setuptools==58.0.4
    ```

### matplotlib API 衝突

- **錯誤訊息**：`ImportError: cannot import name 'register_cmap'`
- **解決方法**：
    ```bash
    python3 -m pip install matplotlib==3.5.3
    ```

### NumPy 主要版本不兼容

- **錯誤訊息**：`A module that was compiled using NumPy 1.x cannot be run in NumPy 2.x`
- **解決方法**：
    ```bash
    python3 -m pip install numpy==1.25.0
    ```

### astropy 與 regions 依賴衝突

- **解決方法**：
    ```bash
    python3 -m pip install astropy==5.1 regions==0.10
    ```

---

## 3. 推薦的依賴版本組合

| 套件         | 版本      |
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

## 4. 安裝步驟總結

```bash
# 1. 卸載潛在衝突套件
python3 -m pip uninstall Pillow matplotlib numpy astropy setuptools -y

# 2. 安裝精確版本的核心依賴
python3 -m pip install setuptools==58.0.4 numpy==1.25.0 matplotlib==3.5.3 astropy==5.1 Pillow==11.3.0

# 3. 安裝其他依賴
python3 -m pip install scipy spectral_cube regions PyQt5 h5py lxml emcee ultranest

# 4. 安裝 XCLASS 離線包
cd ~/Desktop/xclass/xclass_pip_off
python3 -m pip install . -v
```

---

## 5. 測試安裝

進入 Python 互動式命令列，測試導入：

```python
import xclass
from xclass import task_DatabaseQuery, task_myXCLASS, task_myXCLASSFit
from xclass.task_myXCLASSFit import task_myXCLASSMapFit
# 預期所有導入均成功，無錯誤報告。
```

---

# 在 Mac M3 上安裝 XCLASS (v1.4.3)

**目標**：成功在配備 Apple Silicon (M3 晶片) 的 macOS 系統上安裝 XCLASS 軟體包及其所有必要組件。

**參考文件**：[XCLASS 官方手冊](https://xclass-pip.astro.uni-koeln.de/manual/installing.html)

### 問題概述

在 Mac M3 環境下，直接依照官方 pip 安裝指南會遇到多個編譯錯誤，主要來自於 Fortran/C 編譯器、Python 介面（numpy.f2py）、Xcode Command Line Tools SDK 及 Homebrew 安裝的 GCC/Gfortran 之間的相容性及路徑設定問題。

常見錯誤包括：

- `ModuleNotFoundError: No module named 'numpy'`（編譯時找不到 numpy）
- `unknown type name 'FILE'`、`unknown type name 'size_t'`（C 編譯錯誤）
- `gfortran: error: unrecognized command-line option '--fcompiler=gnu95'`（Fortran 編譯錯誤）

---

### 1. 前置準備（Homebrew 安裝依賴）

安裝 Homebrew（如尚未安裝）：

```zsh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

安裝必要依賴：
```zsh
brew install gcc@12
brew install open-mpi
brew install python@3.10
```

安裝 Python 依賴（建議指定版本以確保與 XCLASS 相容）：

```zsh
/opt/homebrew/bin/python3.10 -m pip install numpy==1.25.0 scipy matplotlib==3.5.3 astropy==5.1
```
> **說明**：這些套件版本是根據 XCLASS 官方建議及相容性測試選擇，能避免因新版 API 變動導致的安裝或執行錯誤。

--- 
### 2. XCLASS 環境變數設定
添加環境變數到zshrc。
```zsh
vim ~/.zshrc
```
```zsh
# --- XCLASS 環境變數設定 ---

# 設定 Fortran 和 C 編譯器路徑，明確指向 Homebrew 安裝的版本
export FC="/opt/homebrew/bin/gfortran"
export CC="/opt/homebrew/bin/gcc-12"

# 強制 GCC 使用其自己的標準函式庫頭文件路徑，避免與系統 SDK 衝突
# 這些變數中的 `$(...)` 會自動獲取 Homebrew `gcc@12` 的實際版本和您的 macOS 核心版本
export CFLAGS="-I/opt/homebrew/Cellar/gcc@12/$(brew list --version gcc@12 | cut -d' ' -f2)/lib/gcc/aarch64-apple-darwin$(uname -r | cut -d'.' -f1)/12/include -isystem /usr/local/include -isystem /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk/usr/include -isystem /Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include"
export CPPFLAGS="-I/opt/homebrew/Cellar/gcc@12/$(brew list --version gcc@12 | cut -d' ' -f2)/lib/gcc/aarch64-apple-darwin$(uname -r | cut -d'.' -f1)/12/include -isystem /usr/local/include -isystem /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk/usr/include -isystem /Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include"

# 設定連結器旗標
export LDFLAGS="-L/opt/homebrew/lib -lcurl -lz"

# 確保 Homebrew 的 bin 目錄和 Homebrew Python 的 bin 目錄在 PATH 的最前面，以便優先使用它們
export PATH="/opt/homebrew/bin:$PATH"
export PATH="/opt/homebrew/opt/python@3.10/bin:$PATH"

# (可選但建議) 增加 OpenMP 堆疊大小，有助於平行化計算
ulimit -s unlimited
export KMP_STACKSIZE='3999M'
export OMP_STACKSIZE='3999M'
export GOMP_STACKSIZE='3999M'
# --- XCLASS 環境變數設定 結束 ---
```

重新讀取zshrc

```zsh
source ~/.zshrc
```

### 3.修改 XCLASS 原始碼的 Makefile 檔案

XCLASS 的安裝腳本 setup.py 會調用位於不同子目錄下的 Makefile 進行 Fortran 程式碼和 Python 介面的編譯。這些 Makefile 需要手動修改，以確保 numpy.f2py 總是透過 Homebrew 安裝的 Python 3.10 執行。

請針對以下兩個檔案進行精確修改：
`~/yourpathto/xclass_pip_off/xclass/xclass/interface/Makefile`

`~/yourpathto/xclass_pip_off/xclass/xclass/lite/Makefile`
修改步驟 (對每個檔案重複執行)：

a. 進入對應的目錄：
例如：
```zsh
cd ~/yourpathto/xclass_pip_off/xclass/xclass/interface/
```
b. 備份 Makefile：

```zsh
cp Makefile Makefile.bak
```
c. 編輯 Makefile：

```zsh
vim Makefile
```
d. 修改 FCPY 的定義：
在檔案開頭附近，找到或新增以下行，確保 FCPY 指向 Homebrew Python 的 f2py：

```zsh
#interface/Makefile
# set f2py compiler and flags
FCPY = python3 -m numpy.f2py # old

FCPY = /opt/homebrew/bin/python3.10 -m numpy.f2py # new
```

以及lite 裡FC的需重新定義成FCPY
```zsh
#lite/Makefile
# set f2py compiler and flags
FC = python3 -m numpy.f2py # old

FCPY = /opt/homebrew/bin/python3.10 -m numpy.f2py # new
```

e. 修改lite/Makefile 中 f2py 和 sig 規則中的調用：
這是解決 gfortran: error: unrecognized command-line option 的關鍵。請找到 Makefile 中定義 f2py: 和 sig: 目標的部分，並確保它們是使用 $(FCPY) 來調用 f2py，而不是直接使用 $(FC)。

修改前 (old)：
```zsh
#lite/Makefile
f2py:
	$(FC) -c $(FFLAGS) -m $(EXEC) $(FSRC) # <- 這裡直接使用了 FC，會導致 gfortran 錯誤

sig:
	$(FC) -h CalculateXCLASS.pyf -m $(EXEC) $(FSRC) # <- 這裡也需要修改
```
修改後 (new)：
```zsh 
#lite/Makefile
f2py:
	$(FCPY) -c $(FFLAGS) -m $(EXEC) $(FSRC)

sig:
	$(FCPY) -h CalculateXCLASS.pyf -m $(EXEC) $(FSRC)
```

---

### 5. 執行最終安裝

切換至 `xclass_pip_off` 根目錄，執行：

```bash
cd ~/yourpathto/xclass/xclass_pip_off/
python3.10 -m pip install . -v
```

如無致命錯誤，XCLASS 即安裝完成。


## 參考連結

- [Xclass 官方安裝說明](https://xclass-pip.astro.uni-koeln.de/manual/installing.html#required-packages)