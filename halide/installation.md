# Installation
Note that this is based on the travis build for Ubuntu 14.04LTS as found in .travis.yml.

### Install Dependencies
```sh
sudo add-apt-repository ppa:ubuntu-toolchain-r/test
sudo apt-get update
sudo apt-get install g++-4.9 
sudo apt-get install build-essential git zlib1g-dev libedit-dev libpng-dev libjpeg-dev graphviz git
```

### Set needed environment variables
First set `BUILD_DIR` to wherever LLVM, Clang, and CoreIR should be built. 
```sh
export BUILD_DIR="~"

export LLVM_VERSION=5.0.0
export BUILD_SYSTEM=MAKE 
export CXX_=g++
export CC_=gcc
export CXX=${CXX_}
export CC=${CC_}

export LLVM_CONFIG=${BUILD_DIR}/llvm/bin/llvm-config
export CLANG=${BUILD_DIR}/llvm/bin/clang

export COREIRCONFIG="g++"
export COREIR_DIR=${BUILD_DIR}/coreir
```

### Install LLVM 5.0 and Clang 5.0
```sh
wget https://releases.llvm.org/${LLVM_VERSION}/clang+llvm-${LLVM_VERSION}-linux-x86_64-ubuntu14.04.tar.xz
tar xf clang+llvm-${LLVM_VERSION}-linux-x86_64-ubuntu14.04.tar.xz
mv clang+llvm-${LLVM_VERSION}-linux-x86_64-ubuntu14.04 ${BUILD_DIR}/llvm
```

### Build CoreIR
```sh
cd ${BUILD_DIR}
git clone https://github.com/rdaly525/coreir.git
make -C coreir -j2
```

### Build Halide compiler
```sh
cd ${BUILD_DIR}
git clone https://github.com/StanfordAHA/Halide-to-Hardware.git
cd Halide_CoreIR

# may need to specify full path to coreir
export COREIR_DIR="/home/<user>/coreir"
make -j2
```
