# Installation
Note that this is based on the travis build for Ubuntu 14.04LTS as found in .travis.yml.

### Install Dependencies
```sh
sudo add-apt-repository ppa:ubuntu-toolchain-r/test
sudo apt-get update
sudo apt-get install g++-7
sudo apt-get install build-essential git zlib1g-dev libedit-dev libpng-dev libjpeg-dev graphviz git
```

### Set needed environment variables
First set `BUILD_DIR` to wherever LLVM, Clang, and CoreIR should be built. 
```sh
export BUILD_DIR="~"

export LLVM_VERSION=6.0.0
export BUILD_SYSTEM=MAKE 
export CXX_=g++-7
export CC_=gcc-7
export CXX=${CXX_}
export CC=${CC_}

export LLVM_CONFIG=${BUILD_DIR}/llvm/bin/llvm-config
export CLANG=${BUILD_DIR}/llvm/bin/clang
export PATH=${PATH}:${BUILD_DIR}/llvm-6.0.0/bin

export COREIRCONFIG=${CXX_}
export COREIR_DIR=${BUILD_DIR}/coreir
export COREIR_PATH=${BUILD_DIR}/coreir
export FUNCBUF_DIR=${BUILD_DIR}/BufferMapping/cfunc
export RDAI_DIR=${BUILD_DIR}/rdai
```

### Install LLVM 6.0 and Clang 6.0
```sh
wget https://releases.llvm.org/${LLVM_VERSION}/clang+llvm-${LLVM_VERSION}-linux-x86_64-ubuntu14.04.tar.xz
tar xf clang+llvm-${LLVM_VERSION}-linux-x86_64-ubuntu14.04.tar.xz
mv clang+llvm-${LLVM_VERSION}-linux-x86_64-ubuntu14.04 ${BUILD_DIR}/llvm
```

### Build CoreIR
```sh
cd ${BUILD_DIR}
git clone https://github.com/rdaly525/coreir.git -b ubuffer
cd coreir/build
cmake ..
make -j2
cd ${BUILD_DIR}
```

### Build Clockwork
```sh
cd ${BUILD_DIR}
git clone https://github.com/dillonhuff/clockwork.git -b aha
cd clockwork && ./misc/install_deps_linux.sh
make -j2 libcoreir-cgralib.so && make -j2 libclkwrk.so
```

### Build BufferMapping
```sh
cd ${BUILD_DIR}
git clone -b new_config https://github.com/joyliu37/BufferMapping
cd BufferMapping/cfunc && make lib -j2
```

# Clone RDAI
```sh
cd ${BUILD_DIR}
git clone https://github.com/thenextged/rdai.git
```


### Build Halide compiler
```sh
cd ${BUILD_DIR}
git clone https://github.com/StanfordAHA/Halide-to-Hardware.git
cd Halide-to-Hardware

# may need to specify full path to coreir
export COREIR_DIR="/home/<user>/coreir"
make -j2 distrib
```
