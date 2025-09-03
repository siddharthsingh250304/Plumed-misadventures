Got it! Here’s the **full guide** properly formatted in **Markdown** with fenced code blocks for clear display on any Markdown renderer:

---

# Plumed-misadventures

# A Complete Guide to Installing GROMACS, PLUMED, and MLCVS with GPU Support

This guide will walk you through the complete setup of a GPU-accelerated simulation environment for the PLUMED Masterclass tutorial. It is designed to work on Ubuntu/Debian-based systems and incorporates solutions to common installation issues.

---

### Step 1: System Prerequisites (The Foundation)

First, install all the necessary compilers, build tools, and development headers required by the scientific software. This prevents many common errors.

```bash
# Update your package list
sudo apt update

# Install essential build tools, git, cmake, and specific compilers
# We install gcc-10 and g++-10 specifically to ensure compatibility with the CUDA Toolkit.
sudo apt install build-essential cmake git wget python3-dev python3-pip gcc-10 g++-10

# Install essential Python build packages using pip
pip3 install --upgrade pip setuptools cython
```

---

### Step 2: Install the NVIDIA CUDA Toolkit

This provides the core libraries and compiler (nvcc) for GPU acceleration.

```bash
# Install the standard CUDA toolkit from the Ubuntu repositories.
# We will use the g++-10 compiler from Step 1 to ensure compatibility.
sudo apt install nvidia-cuda-toolkit
```

---

### Step 3: Download and Configure LibTorch (GPU Version)

LibTorch is the C++ library for PyTorch, used by PLUMED for machine learning integration.

```bash
# Navigate to your home directory for a clean workspace
cd ~

# Download the LibTorch library with CUDA support. This version is known to be stable.
wget https://download.pytorch.org/libtorch/cu117/libtorch-cxx11-abi-shared-with-deps-1.13.1%2Bcu117.zip

# Unzip the library and remove the zip file
unzip libtorch-cxx11-abi-shared-with-deps-1.13.1+cu117.zip
rm libtorch-cxx11-abi-shared-with-deps-1.13.1+cu117.zip

# Set environment variables in a script for easy activation
export LIBTORCH=${PWD}/libtorch

echo "export CPATH=${LIBTORCH}/include/torch/csrc/api/include/:${LIBTORCH}/include/:${LIBTORCH}/include/torch:\$CPATH" > ${LIBTORCH}/sourceme.sh
echo "export INCLUDE=${LIBTORCH}/include/torch/csrc/api/include/:${LIBTORCH}/include/:${LIBTORCH}/include/torch:\$INCLUDE" >> ${LIBTORCH}/sourceme.sh
echo "export LIBRARY_PATH=${LIBTORCH}/lib:\$LIBRARY_PATH" >> ${LIBTORCH}/sourceme.sh
echo "export LD_LIBRARY_PATH=${LIBTORCH}/lib:\$LD_LIBRARY_PATH" >> ${LIBTORCH}/sourceme.sh

# Add this script to your .bashrc so it's loaded automatically in all future terminals
echo ". ${LIBTORCH}/sourceme.sh" >> ~/.bashrc
```

---

### Step 4: Install PLUMED

Compile PLUMED, linking against the LibTorch library.

```bash
# Navigate to your home directory
cd ~

# Activate the LibTorch environment in your current terminal
source ~/libtorch/sourceme.sh

# Clone the latest version of PLUMED from GitHub
git clone https://github.com/plumed/plumed2.git
cd plumed2

# Configure the build. This will find LibTorch and Python automatically.
./configure --enable-libtorch --enable-modules=all

# Compile PLUMED using 4 processor cores
make -j4

# Source the PLUMED environment file
. sourceme.sh

# Add the PLUMED environment file to your .bashrc for future sessions
echo ". ${PWD}/sourceme.sh" >> ~/.bashrc
```

---

### Step 5: Install GROMACS with GPU Support

Download and compile a version of GROMACS compatible with your PLUMED installation.

```bash
# Navigate to your home directory
cd ~

# Download and Extract the Correct GROMACS Version
progname=gromacs-2022.5
wget http://ftp.gromacs.org/pub/gromacs/$progname.tar.gz
tar xvf $progname.tar.gz
mv $progname $progname-source
export GROMACS_ROOT=${PWD}/gromacs
cd $progname-source

# Patch GROMACS with PLUMED
plumed --no-mpi patch -p -f --runtime -e $progname

# Configure the Build
mkdir build
cd build

cmake .. \
  -DCMAKE_C_COMPILER=gcc-10 \
  -DCMAKE_CXX_COMPILER=g++-10 \
  -DGMX_GPU=CUDA \
  -DGMX_MPI=OFF \
  -DGMX_BUILD_OWN_FFTW=ON \
  -DCMAKE_INSTALL_PREFIX=${GROMACS_ROOT} \
  -DGMX_DEFAULT_SUFFIX=ON \
  -DGMX_DOUBLE=OFF \
  -DGMX_OPENMP=ON

# Compile and Install GROMACS
make -j4
make install

# Clean Up and Finalize Environment
cd ../..
rm -rf $progname.tar.gz $progname-source

source ${GROMACS_ROOT}/bin/GMXRC.bash
echo ". ${GROMACS_ROOT}/bin/GMXRC.bash" >> ~/.bashrc
```

---

### Step 6: Final Setup (Python Environment & Tutorial Data)

Create a dedicated Python environment for running the analysis and notebooks.

```bash
conda create -n masterclass22-05
conda activate masterclass22-05
conda install numpy pandas matplotlib scikit-learn pytorch torchvision torchaudio -c pytorch -c conda-forge
```

After creating the environment, install mlcvs and download the tutorial data:

```bash
# Make sure your 'masterclass22-05' environment is active
cd ~
git clone https://github.com/luigibonati/mlcvs.git -b v0.1.1
cd mlcvs/
pip install .
cd ~

# Download the tutorial data
git clone https://github.com/luigibonati/masterclass-plumed.git
```

---

**Installation Complete!**
You are now fully set up and ready to begin the tutorial. Navigate to the `masterclass-plumed` directory and start the Jupyter notebooks.
