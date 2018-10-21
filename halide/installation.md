# Installation
Note that this is based on the travis build for Ubuntu 14.04LTS as found in .travis.yml.

### Install Dependencies
```sh
 sudo apt-get install build-essential git zlib1g-dev libedit-dev libpng-dev libjpeg-dev graphviz g++-4.9
```

### Set needed environment variables
First set `BUILD_DIR` to wherever LLVM, Clang, and CoreIR should be built. 
```sh
export BUILD_DIR=""

export LLVM_VERSION=3.7.1 
export BUILD_SYSTEM=MAKE 
export CXX_=g++-4.9 
export CC_=gcc-4.9
export CXX=${CXX_}
export CC=${CC_}

export LLVM_CONFIG=${BUILD_DIR}/llvm/bin/llvm-config
export CLANG=${BUILD_DIR}/llvm/bin/clang

export COREIRCONFIG="g++-4.9"
export COREIR_DIR=${BUILD_DIR}/coreir
```

### Install LLVM 3.7 and Clang 3.7
```sh
wget http://llvm.org/releases/${LLVM_VERSION}/clang+llvm-${LLVM_VERSION}-x86_64-linux-gnu-ubuntu-14.04.tar.xz
tar xf clang+llvm-${LLVM_VERSION}-x86_64-linux-gnu-ubuntu-14.04.tar.xz
mv clang+llvm-${LLVM_VERSION}-x86_64-linux-gnu-ubuntu-14.04 ${TRAVIS_BUILD_DIR}/llvm
```

### Build CoreIR
```sh
cd ${BUILD_DIR}
git clone https://github.com/rdaly525/coreir.git
make -C coreir -j
```

### Build Halide compiler
```sh
cd ${BUILD_DIR}
git clone https://github.com/jeffsetter/Halide_CoreIR.git
cd Halide_CoreIR
make -j
```
