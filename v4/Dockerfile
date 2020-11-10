FROM centos:7.6.1810

# set up base
RUN yum install -y epel-release \
    && yum groupinstall -y "Development tools" \
    && yum install -y \
        libusbx pciutils-libs pciutils lsof ethtool fuse-libs \
        ca-certificates wget openssh-server openssh-clients net-tools \
        numactl-devel gtk2 atk cairo tcsh libnl3 tcl libmnl tk

# set up workdir
ENV INSTALL_PREFIX=/opt
WORKDIR /tmp/mpi

# download and install mlnx
RUN wget -q -O - http://content.mellanox.com/ofed/MLNX_OFED-5.1-0.6.6.0/MLNX_OFED_LINUX-5.1-0.6.6.0-rhel7.6-x86_64.tgz | tar -xzf - \
    && ./MLNX_OFED_LINUX-5.1-0.6.6.0-rhel7.6-x86_64/mlnxofedinstall --user-space-only --without-fw-update --all --force \
    && rm -rf MLNX_OFED_LINUX-5.1-0.6.6.0-rhel7.6-x86_64

# download and install HPC-X
ENV HPCX_VERSION="v2.7.0"
RUN cd ${INSTALL_PREFIX} && \
    wget -q -O - https://azhpcstor.blob.core.windows.net/azhpc-images-store/hpcx-v2.7.0-gcc9.2.0-MLNX_OFED_LINUX-5.1-0.6.6.0-redhat7.6-x86_64.tbz | tar -xjf - \
    && HPCX_PATH=${INSTALL_PREFIX}/hpcx-${HPCX_VERSION}-gcc-MLNX_OFED_LINUX-5.1-0.6.6.0-redhat7.6-x86_64 \
    && HCOLL_PATH=${HPCX_PATH}/hcoll \
    && UCX_PATH=${HPCX_PATH}/ucx

# download and install MVAPICH2
ENV MV2_VERSION="2.3.4"
RUN wget -q -O - http://mvapich.cse.ohio-state.edu/download/mvapich/mv2/mvapich2-${MV2_VERSION}.tar.gz | tar -xzf - \
    && cd mvapich2-${MV2_VERSION} \
    && ./configure --prefix=${INSTALL_PREFIX}/mvapich2-${MV2_VERSION} --enable-g=none --enable-fast=yes \
    && make -j$(nproc) \
    && make install \
    && cd .. \
    && rm -rf mvapich2-${MV2_VERSION}

ENV PATH="${INSTALL_PREFIX}/mvapich2-${MV2_VERSION}/bin:${PATH}"

# install and setup benchmarks
WORKDIR /opt/benchmarks

# download and install mpiBench
RUN wget -q -O - https://codeload.github.com/LLNL/mpiBench/tar.gz/master | tar -xzf - \
    && mv ./mpiBench-master ./mpiBench \
    && cd mpiBench/ \
    && make

# download and install osu micro benchmarks
RUN wget -q -O - http://mvapich.cse.ohio-state.edu/download/mvapich/osu-micro-benchmarks-5.6.3.tar.gz | tar -xzf - \
    && mv ./osu-micro-benchmarks-5.6.3 ./osu-micro-benchmarks \
    && cd osu-micro-benchmarks/ \
    && ./configure CC=mpicc CXX=mpicxx \
    && make \
    && make install