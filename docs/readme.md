# Computer Architecture 2023: Term Project

[![hackmd-github-sync-badge](https://hackmd.io/HdMEALKjTnSFF_d7QE3ESw/badge)](https://hackmd.io/HdMEALKjTnSFF_d7QE3ESw)

This document consists of the following sections.

1. [Environment setup](#Environment-Setup): Building and installing RISC-V GNU toolchain and RISC-V Atom in a Docker container.
1. [To-do list](#To-do-List): Pending tasks.

## Environment Setup

In this section, we built and installed RISC-V GNU toolchain and RISC-V Atom in a Docker container.

To easily set up everything with Docker Compose and `make` installed, execute `make start` in `<project-root>/docker/`.


### Install Docker Engine

Primary reference: [Install Docker Engine on Ubuntu | Docker Docs](https://docs.docker.com/engine/install/ubuntu/)

Docker will be used to wrap the entire project in a container. The following steps apply to Ubuntu Jammy 22.04.3, but it should be easy to customize them in the other Debian-based distros with bash as the shell.
1. Uninstall all conflicting packages.
    ```shell
    for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do
      if [ ! -z "$(apt list --installed $pkg 2>&1 | grep installed)" ]; then
        sudo apt purge -y $pkg;
      else
        echo "Not installed: $pkg"
      fi
    done
    ```
    One-line and simplified version: `echo docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc | xargs sudo apt purge y $p`
1. Allow `apt` to fetch a repository over HTTPS.
    ```shell
    sudo apt update && sudo apt install -y ca-certificates curl gnupg
    ```
1. Add Docker’s official GPG key.
    ```shell
    sudo install -m 0755 -d /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
      sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    sudo chmod a+r /etc/apt/keyrings/docker.gpg
    ```
1. Set up the repository.
    ```shell
    echo \
    "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```
1. Install the latest Docker engine and Docker Compose.
    ```shell
    sudo apt update
    sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    ```
1. Add the current user to the `docker` group to access Docker commands without `sudo`.
    ```shell
    sudo usermod -aG docker $USER
    ```
1. Re-login to take the effect, or enter `newgrp docker` to apply the changes in the current terminal.


### Set up a Docker Container

1. Launch an Ubuntu Jammy 22.04.3 container, with hostname `ca-term-ubuntu` (arbitrary) and container name `ca-term-ubuntu` (arbitrary) in the background (`-d` to detach from it, and `sleep infinity` to keep it running).
    ```shell
    docker run --rm -d \
      --name ca-term-ubuntu \
      --hostname ca-term-ubuntu \
      ubuntu:jammy-20231211.1 \
      sleep infinity
    ```
1. Attach to the container.
    ```shell
    docker exec -it ca-term-ubuntu /bin/bash
    ```
1. Un-minimize the environment, and install the `man` command and its docs.
    ```shell
    unminimize
    apt update
    apt install man-db
    ```
1. Set the time zone to Asia/Taipei. ([Reference: bash - apt-get install tzdata noninteractive - Stack Overflow](https://stackoverflow.com/a/44333806))
    ```shell
    ln -fs /usr/share/zoneinfo/Asia/Taipei /etc/localtime
    echo Asia/Taipei > /etc/timezone
    apt install -y tzdata
    ```
1. In the container, create a normal user `ray` (arbitrary) with password `0` (arbitrary) in group `sudo` (to use `sudo` as user `ray`). Note that the following commands are executed in the container as `root`.
    ```shell
    apt update
    apt install -y sudo
    useradd -m ray
    chsh -s /bin/bash ray
    echo "ray:0" | chpasswd
    echo "root:0" | chpasswd
    usermod -aG sudo ray
    ```
1. Leave the container, and login as `ray` (the same as previously created user).
    ```shell
    exit
    docker exec -u ray -w /home/ray -it ca-term-ubuntu /bin/bash
    ```
1. Remove the login message, and remove the warning message from `sudo`. (Reference: [Giving yourself a quieter SSH login](https://web.archive.org/web/20200924150633/https://debian-administration.org/article/546/Giving_yourself_a_quieter_SSH_login))
    ```shell
    touch ~/.hushlogin ~/.sudo_as_admin_successful
    ```
1. In the future, to stop the container, run the following command. Docker will automatically remove the stopped container since we added `--rm` in `docker run` previously. To keep the content but stopping the container, remove `--rm` in `docker run`.
    ```shell
    docker stop ca-term-ubuntu
    ```


### Build and Install Verilator from Source

1. Install the dependencies to build or run [Verilator](https://github.com/verilator/verilator) from source. Verilator will be used by Atomsim to Verilate Verilog RTL into C++. (Primary reference: [Installation — Verilator Devel 5.021 documentation](https://veripool.org/guide/latest/install.html#package-manager-quick-install))
    ```shell
    sudo apt install -y git help2man perl python3 make autoconf g++ flex bison ccache
    sudo apt install -y libgoogle-perftools-dev numactl perl-doc
    
    # Ubuntu only
    sudo apt install libfl2 libfl-dev

    # Ubuntu only (ignore since it gives error)
    # sudo apt install zlibc zlib1g zlib1g-dev
    ```
1. Clone Verilator into `~/verilator`.
    ```shell
    git clone https://github.com/verilator/verilator ~/verilator
    cd ~/verilator
    ```
1. Build Verilator from the `stable` branch.
    ```shell
    unset VERILATOR_ROOT
    git pull        # get the latest content
    git checkout stable

    autoconf        # create ./configure script
    ./configure     # configure and create Makefile
    make -j `nproc`
    make test       # make sure all tests passes before continuing
    ```
1. Install Verilator, and check its version.
    ```shell
    sudo make install
    verilator --version
    ```


### Build and Install RISC-V GNU Toolchain

1. Compile and install RISC-V GNU toolchain from source, for we found the script `install-toolchain.sh` provided by RISC-V Atom is buggy. The `configure` command targets RV64GC with glibc by default, so `--with-arch=rv64gc` is redundant. The entire repo (with its submodules and compiled objects) takes around 14 GiB of disk space, so be aware of the free spaces on the disk. After you issue `time make`, take a break. It took me 45 minutes.
    ```shell
    git clone https://github.com/riscv/riscv-gnu-toolchain ~/toolchain
    cd ~/toolchain

    sudo apt install -y autoconf automake autotools-dev curl python3 python3-pip libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev libexpat-dev ninja-build git cmake libglib2.0-dev

    ./configure --prefix="$HOME/.local/share/riscv-gnu-toolchain" --enable-multilib
    time make -j `nproc`
    ```
1. Add the binaries of the toolchain to PATH.
    ```shell
    echo "export PATH=\"\$HOME/.local/share/riscv-gnu-toolchain/bin:\$PATH\"" >> ~/.bashrc
    source ~/.bashrc
    ```
1. Check the version of `gcc` in the toolchain.
    ```shell
    riscv64-unknown-elf-gcc --version
    ```


### Install the Other Dependencies of RISC-V Atom
Primary reference: [riscv-collab/riscv-gnu-toolchain: GNU toolchain for RISC-V, including GCC](https://github.com/riscv-collab/riscv-gnu-toolchain)

1. Install the dependencies to build or run RISC-V Atom. For more info about these packages, please check the official documentation provided above.
    ```shell
    sudo apt install -y git python3 build-essential gtkwave screen libreadline-dev
    ```
1. Install the required Python packages. This step is not documented, but without the packages in `requirements.txt`, we cannot build the AtomSim simulator.
    ```shell
    wget -O - https://raw.githubusercontent.com/saursin/riscv-atom/main/requirements.txt | xargs pip install
    ```
1. Add `~/.local/bin/` to path as the warning from `pip` instructs.
    ```shell
    echo "export PATH=\"\$HOME/.local/bin:\$PATH\"" >> ~/.bashrc
    source ~/.bashrc
    ```


### Build RISC-V Atom

Primary reference: [Building RISC-V Atom — RISC-V Atom v1.2 documentation](https://riscv-atom.readthedocs.io/en/latest/pages/getting_started/building.html)

1. Clone RISC-V Atom into ~/riscv-atom.
    ```shell
    git clone https://github.com/saursin/riscv-atom.git ~/riscv-atom
    cd ~/riscv-atom
    ```
1. Set the environment variables, and allow them to be automatically set on login.
    ```shell
    source sourceme
    echo -e "# set environment variables for RISC-V Atom\nsource \"$(pwd)/sourceme\"" >> ~/.bashrc
    ```
1. Build the AtomSim simulator.
    ```shell
    make soctarget=atombones
    ```
1. Verify the build.
    ```shell
    which atomsim
    atomsim --help
    ```
1. Run through all examples. (Fixme: failed on lots of tests)
    ```shell
    make soctarget=atombones run-all
    ```