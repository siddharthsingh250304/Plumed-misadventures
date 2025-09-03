# Plumed-misadventures

code
Markdown
# A Complete Guide to Installing GROMACS, PLUMED, and MLCVS with GPU Support

This guide will walk you through the complete setup of a GPU-accelerated simulation environment for the PLUMED Masterclass tutorial. It is designed to work on Ubuntu/Debian-based systems and incorporates solutions to common installation issues.

---

### **Step 1: System Prerequisites (The Foundation)**

First, we will install all the necessary compilers, build tools, and development headers that are required by the scientific software. This prevents many common errors from ever happening.

```bash
# Update your package list
sudo apt update

# Install essential build tools, git, cmake, and specific compilers
# We install gcc-10 and g++-10 specifically to ensure compatibility with the CUDA Toolkit.
sudo apt install build-essential cmake git wget python3-dev python3-pip gcc-10 g++-10

# Install essential Python build packages using pip
pip3 install --upgrade pip setuptools cython
Step 2: Install the NVIDIA CUDA Toolkit
This provides the core libraries and compiler (nvcc) for GPU acceleration.
code
Bash
# Install the standard CUDA toolkit from the Ubuntu repositories.
# We will use the g++-10 compiler from Step 1 to ensure compatibility.
sudo apt install nvidia-cuda-toolkit
Step 3: Download and Configure LibTorch (GPU Version)
This is the C++ library for PyTorch, which PLUMED uses for its machine learning integration.
code
Bash
# Navigate to your home directory for a clean workspace
cd ~

# Download the LibTorch library with CUDA support. This version is known to be stable.
wget https://download.pytorch.org/libtorch/cu117/libtorch-cxx11-abi-shared-with-deps-1.13.1%2Bcu117.zip

# Unzip the library and remove the zip file
unzip libtorch-cxx11-abi-shared-with-deps-1.13.1+cu117.zip
rm libtorch-cxx11-abi-shared-with-deps-1.13.1+cu117.zip

# --- Create a script to set the required environment variables ---
# This makes it easy to activate the library in any terminal session.
export LIBTORCH=${PWD}/libtorch

echo "export CPATH=${LIBTORCH}/include/torch/csrc/api/include/:${LIBTORCH}/include/:${LIBTORCH}/include/torch:\$CPATH" > ${LIBTORCH}/sourceme.sh
echo "export INCLUDE=${LIBTORCH}/include/torch/csrc/api/include/:${LIBTORCH}/include/:${LIBTORCH}/include/torch:\$INCLUDE" >> ${LIBTORCH}/sourceme.sh
echo "export LIBRARY_PATH=${LIBTORCH}/lib:\$LIBRARY_PATH" >> ${LIBTORCH}/sourceme.sh
echo "export LD_LIBRARY_PATH=${LIBTORCH}/lib:\$LD_LIBRARY_PATH" >> ${LIBTORCH}/sourceme.sh

# Add this script to your .bashrc so it's loaded automatically in all future terminals
echo ". ${LIBTORCH}/sourceme.sh" >> ~/.bashrc
Step 4: Install PLUMED
Now we compile PLUMED, which will link against the LibTorch library we just set up.
code
Bash
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
Step 5: Install GROMACS with GPU Support
This is the most complex step. We will download a version of GROMACS that is compatible with our PLUMED installation and compile it using our specific g++-10 compiler to ensure it works with CUDA.
code
Bash
# Navigate to your home directory
cd ~

# --- Download and Extract the Correct GROMACS Version ---
# We use 2022.5 because it is compatible with the latest PLUMED patches.
progname=gromacs-2022.5
wget http://ftp.gromacs.org/pub/gromacs/$progname.tar.gz
tar xvf $progname.tar.gz
mv $progname $progname-source
export GROMACS_ROOT=${PWD}/gromacs
cd $progname-source

# --- Patch GROMACS with PLUMED ---
plumed --no-mpi patch -p -f --runtime -e $progname

# --- Configure the Build ---
mkdir build
cd build

# This cmake command is specifically tailored to solve all previous issues:
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

# --- Compile and Install GROMACS ---
make -j 4
make install

# --- Clean Up and Finalize Environment ---
cd ..
cd ..
rm -rf $progname.tar.gz $progname-source

source ${GROMACS_ROOT}/bin/GMXRC.bash
echo ". ${GROMACS_ROOT}/bin/GMXRC.bash" >> ~/.bashrc
Step 6: Final Setup (Python Environment & Tutorial Data)
The core software is now installed. The final step is to create a dedicated Python environment for running the analysis and notebooks. Choose either the conda or pip/venv method.
Option A (Using conda):
code
Bash
conda create -n masterclass22-05
conda activate masterclass22-05
conda install numpy pandas matplotlib scikit-learn pytorch torchvision torchaudio -c pytorch -c conda-forge
Option B (Using pip with a virtual environment):
code
Bash
cd ~
python3 -m venv masterclass22-05
source masterclass22-05/bin/activate
pip3 install torch torchvision torchaudio
pip install numpy pandas matplotlib scikit-learn
After creating the environment, install mlcvs and download the tutorial data:
code
Bash
# Make sure your 'masterclass22-05' environment is active
cd ~
git clone https://github.com/luigibonati/mlcvs.git -b v0.1.1
cd mlcvs/
pip install .
cd ~

# Download the tutorial data
git clone https://github.com/luigibonati/masterclass-plumed.git
Installation Complete! You are now fully set up and ready to begin the tutorial. You can navigate to the masterclass-plumed directory and start the Jupyter notebooks.
