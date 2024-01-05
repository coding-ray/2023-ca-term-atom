# config part 1/2
ARG UBUNTU_VERSION=jammy-20231211.1

FROM ubuntu:${UBUNTU_VERSION} as base

# config part 2/2
ARG TIMEZONE="Asia/Taipei"
ARG USERNAME=ray
ARG PASSWORD=0
# recommended: stable (officially recommended branch), v5.020 (successfully built)
ARG VERILATOR_CHECKOUT="v5.020"
# recommended: master (officially recommended branch), 2023.12.20 (successfully built)
ARG TOOLCHAIN_CHECKOUT="2023.12.20"
# recommended: main (officially recommended branch), f0d15fc (successfully built)
ARG ATOM_CHECKOUT="f0d15fc"

ARG HOME_DIR="/home/${USERNAME}"
ARG VERILATOR_INSTALL_PATH="/opt/verilator"
ARG TOOLCHAIN_INSTALL_DIR="/opt/riscv-gnu-toolchain"
ARG ATOM_DIR="${HOME_DIR}/riscv-atom"

# update apt index
RUN apt -qq update

# build Verilator
RUN echo "Verilator" && \
    apt -qq install -y autoconf bison ccache flex g++ git help2man libfl2 libfl-dev \
        libgoogle-perftools-dev make numactl perl perl-doc python3 && \
    git clone --recursive https://github.com/verilator/verilator /tmp/verilator && \
    cd /tmp/verilator && \
    git checkout ${VERILATOR_CHECKOUT} && \
    unset VERILATOR_ROOT && \
    autoconf && \
    ./configure --prefix ${VERILATOR_INSTALL_PATH}&& \
    make -j `nproc` && \
    make test && \
    make install && \
    rm -rf /tmp/verilator

# build RISC-V GNU toolchain
RUN echo "RISC-V GNU toolchain" && \
    apt -qq install -y autoconf automake autotools-dev bc bison build-essential cmake \
        curl flex gawk git gperf gtkwave libexpat-dev libglib2.0-dev libgmp-dev \
        libmpc-dev libmpfr-dev libreadline-dev libtool ninja-build patchutils python3 \
        python3-pip screen texinfo zlib1g-dev && \
    git clone https://github.com/riscv/riscv-gnu-toolchain /tmp/toolchain && \
    cd /tmp/toolchain && \
    git checkout ${TOOLCHAIN_CHECKOUT} && \
    mkdir -p "${TOOLCHAIN_INSTALL_DIR}" && \
    ./configure --prefix="${TOOLCHAIN_INSTALL_DIR}" --enable-multilib && \
    echo "Building RISC-V GNU toolchain and installing it to:\n  ${TOOLCHAIN_INSTALL_DIR}" && \
    make -j `nproc` && \
    echo "Installed RISC-V GNU toolchain to:\n  ${TOOLCHAIN_INSTALL_DIR}" && \
    echo "export PATH=\"${TOOLCHAIN_INSTALL_DIR}/bin:\$PATH\"" >> "/bashrc_addition.txt" && \
    rm -rf /tmp/toolchain



FROM ubuntu:${UBUNTU_VERSION}

# unminimize the image, install `man`, and upgrade the entire system
RUN yes | unminimize && \
    apt -qq update && \
    apt -qq install -y man-db && \
    apt upgrade -y

# create a normal user in `sudo` group
RUN apt -qq install -y sudo && \
    useradd -m ${USERNAME} && \
    chsh -s /bin/bash ${USERNAME} && \
    echo "${USERNAME}:0" | chpasswd && \
    echo "root:0" | chpasswd && \
    usermod -aG sudo "${USERNAME}" && \
    touch "/root/.hushlogin" "${HOME_DIR}/.hushlogin" "${HOME_DIR}/.sudo_as_admin_successful" && \
    chown ${USERNAME}:${USERNAME} "${HOME_DIR}/.hushlogin" "${HOME_DIR}/.sudo_as_admin_successful"

COPY --from=base --chown=${USERNAME}:${USERNAME} "/bashrc_addition.txt" "/bashrc_addition.txt"
COPY --from=base --chown=${USERNAME}:${USERNAME} "/opt/*" "/opt/"

RUN cat "/bashrc_addition.txt" >> ${HOME_DIR}/.bashrc && \
    rm "/bashrc_addition.txt"

# set up the time zone
RUN ln -fs /usr/share/zoneinfo/${TIMEZONE} /etc/localtime && \
    echo ${TIMEZONE} > /etc/timezone && \
    apt -qq install -y tzdata

# build RISC-V Atom
RUN echo "RISC-V Atom" && \
    apt -qq install -y git python3 build-essential gtkwave screen libreadline-dev && \
    git clone https://github.com/saursin/riscv-atom "${ATOM_DIR}" && \
    cd "${ATOM_DIR}" && \
    git checkout "${ATOM_CHECKOUT}" && \
    cat requirements.txt | tr '\n' ' ' | xargs -I{} su - ${USERNAME} -c "pip install --user {}" && \
    echo "# pip, python and misc binaries\nexport PATH=\"${HOME_DIR}/.local/bin:\$PATH\"" >> "${HOME_DIR}/.bashrc" && \
    source sourceme && \
    echo "# set environment variables for RISC-V Atom\nsource \"${ATOM_DIR}/sourceme\"" >> ${HOME_DIR}/.bashrc && \
    echo "Building RISC-V Atom" && \
    make soctarget=atombones && \
    echo "Finish building RISC-V Atom" && \
    chown -R ${USERNAME}:${USERNAME} "${ATOM_DIR}"

# install a text editor
RUN apt -qq install -y vim


USER ${USERNAME}
WORKDIR ${HOME_DIR}

# run forever unless stopped
CMD ["sleep", "infinity"]