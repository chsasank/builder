FROM nvidia/cuda:8.0-devel-centos6

RUN yum install -y git wget curl

ENV CUDNN_VERSION 6.0.21

RUN CUDNN_DOWNLOAD_SUM=9b09110af48c9a4d7b6344eb4b3e344daa84987ed6177d5c44319732f3bb7f9c && \
    curl -fsSL http://developer.download.nvidia.com/compute/redist/cudnn/v6.0/cudnn-8.0-linux-x64-v6.0.tgz -O && \
    echo "$CUDNN_DOWNLOAD_SUM  cudnn-8.0-linux-x64-v6.0.tgz" | sha256sum -c - && \
    tar -xzf cudnn-8.0-linux-x64-v6.0.tgz -C /usr/local && \
    rm cudnn-8.0-linux-x64-v6.0.tgz && \
    ldconfig

RUN wget https://developer.nvidia.com/compute/cuda/8.0/Prod2/patches/2/cuda_8.0.61.2_linux-run && \
    chmod +x cuda_8.0.61.2_linux-run && \
    cp /usr/local/cuda/version.txt /tmp/ && \
    ./cuda_8.0.61.2_linux-run --silent --accept-eula --installdir=/tmp && \
    yes | cp -P /tmp/lib64/* /usr/local/cuda/lib64/

ENV LC_ALL en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US.UTF-8

# MKL
RUN mkdir -p /opt/intel/lib
COPY mkl_libs/libmkl_core.a /opt/intel/lib/libmkl_core.a
COPY mkl_libs/libmkl_gnu_thread.a /opt/intel/lib/libmkl_gnu_thread.a
COPY mkl_libs/libmkl_intel_lp64.a /opt/intel/lib/libmkl_intel_lp64.a

# get devtoolset-3
RUN yum install -y yum-utils
RUN yum install -y centos-release-scl
RUN yum-config-manager --enable rhel-server-rhscl-7-rpms
RUN yum install -y devtoolset-3-gcc
RUN yum install -y devtoolset-3-gcc-c++
RUN yum install -y devtoolset-3-gcc-gfortran
RUN yum install -y devtoolset-3-binutils
ENV PATH=/opt/rh/devtoolset-3/root/usr/bin:$PATH
ENV LD_LIBRARY_PATH=/opt/rh/devtoolset-3/root/usr/lib64:/opt/rh/devtoolset-3/root/usr/lib:$LD_LIBRARY_PATH


# cmake
RUN yum install -y cmake
COPY build_scripts /build_scripts
RUN bash build_scripts/build.sh && rm -r build_scripts

ENV SSL_CERT_FILE=/opt/_internal/certs.pem

RUN rm -rf /opt/python/cp26-cp26m /opt/_internal/cpython-2.6.9-ucs2
RUN rm -rf /opt/python/cp26-cp26mu /opt/_internal/cpython-2.6.9-ucs4
RUN rm -rf /opt/python/cp33-cp33m /opt/_internal/cpython-3.3.6
RUN rm -rf /opt/python/cp34-cp34m /opt/_internal/cpython-3.4.6

# magma
RUN wget http://icl.cs.utk.edu/projectsfiles/magma/downloads/magma-2.2.0.tar.gz && \
    tar -xvf magma-2.2.0.tar.gz && \
    cd magma-2.2.0 && \
    wget https://raw.githubusercontent.com/pytorch/builder/master/conda/magma-cuda80-2.2.0/cmakelists.patch && \
    wget https://raw.githubusercontent.com/pytorch/builder/master/conda/magma-cuda80-2.2.0/thread_queue.patch && \
    patch <cmakelists.patch && \
    patch -p1 <thread_queue.patch && \
    mkdir build && \
    cd build && \
    cmake .. -DUSE_FORTRAN=OFF -DGPU_TARGET="All" -DCMAKE_INSTALL_PREFIX=$PREFIX && \
    make -j$(getconf _NPROCESSORS_CONF) && \
    make install && \
    cd ..
