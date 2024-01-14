# Computer Architecture 2023: Term Project

<!-- This is a HackMD text block with slime green background-->

:::success

## Goal

- [ ] Make all examples in RISC-V Atom pass. This is the primary goal. Without this, we cannot measure the stability and correctness of the simulator.
- [ ] Verify RISC-V Atom with Verilator, and make sure it supports projects Dhrystone, etc. Verification ensures that the processors work faultlessly in real world.
- [ ] Tye to implement the RV32M instructions, and utilize `riscv-arch-test` or `riscv-tests` to verify RV32I/RV32M instructions exhaustedly to ensure the ISA meets the standard.
- [ ] Document all the progress in detail, including all encountered technical problems. You may want some discussion with the original developers with GitHub issues. Participate in the entire progress, and it is a more valuable outcome by eventually contributing the RV32M extension or some related implementations.

:::

[![hackmd-github-sync-badge](https://hackmd.io/HdMEALKjTnSFF_d7QE3ESw/badge)](https://hackmd.io/HdMEALKjTnSFF_d7QE3ESw)

For HackMD viewers, note that this documentation is synchronized between [GitHub](https://github.com/coding-ray/2023-ca-term-atom/blob/master/docs/readme.md) and HackMD.

All the code related to this project is in the following two repositories.

1. [coding-ray/2023-ca-term-atom](https://github.com/coding-ray/2023-ca-term-atom): All documentation related to this project goes here.
1. [coding-ray/riscv-atom](https://github.com/coding-ray/riscv-atom): The implementation of the M extension for [saursin/riscv-atom](https://github.com/saursin/riscv-atom) goes here.

This document consists of the following sections.

1. [Environment setup](#Environment-Setup): Building and installing RISC-V GNU toolchain, and building RISC-V Atom in a Docker container.
1. [To-do list](#To-do-List): Pending tasks.

## Environment Setup

In this section, we will guide you through the building and installation of RISC-V GNU toolchain and Verilator from source in a Docker container. After that, there will be the building procedure of RISC-V Atom. These steps are also verified to be applicable inside or outside a virtual machine.

### Install Docker Engine

Primary reference: [Install Docker Engine on Ubuntu | Docker Docs](https://docs.docker.com/engine/install/ubuntu/)

Docker will be used to wrap the entire project in a container. The following steps apply to Ubuntu Jammy 22.04.3, but it should be easy to customize them in the other Debian-based distributions with bash as the shell.

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

### Build and Install RISC-V GNU Toolchain from Source

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

## Collaboration on Both GitHub and HackMD

With some changes on HackMD, to synchronize changes from HackMD to GitHub, do the following steps.

1. Browse our [documentation on HackMD](https://hackmd.io/HdMEALKjTnSFF_d7QE3ESw). Desktop view of the HackMD page is easier to view than mobile view.
1. On the top right corner, click `...`, and the select `Versions and GitHub Sync`.
   ![Options in HackMD in which "Versions and GitHub Sync" is highlighted](https://i.imgur.com/e8q5ktr.png).
1. On the top right corner of the popup window, click `Push`.
   ![HackMD option to push to GitHub](https://i.imgur.com/pVaCYQa.png)
1. Select a branch to commit changes. `develop` is more recommended than `master`.
1. Enter proper commit title and message according to [docs/commit-convention.md](commit-convention.md) and the changes.

To synchronize changes from GitHub to HackMD, follow the steps above, but click `Pull` instead in the popup window.
![HackMD option to pull from GitHub](https://i.imgur.com/MKy9z7O.png)

## Implementation of RV32M Instructions

### Introduction

To implement the M extension, specifically RV32M, in RISC-V Atom, we have to modify the core components of RISC-V Atom under `${REPO_ROOT}/rtl/core/`, where `${REPO_ROOT}` is the root directory of RISC-V Atom.

In pp. 35—37 of [RISC-V Instruction Set Manual v2.2](https://riscv.org/wp-content/uploads/2017/05/riscv-spec-v2.2.pdf) (May 7, 2017), it defined the specification of the M extension, including the strategy to handle division by zero and division overflow.

![Division by zero and overflow](https://i.imgur.com/Si9vz6y.png)

### Required Modification in `/rtl/core/Defs.vh`

To include instructions for the `M extension`, different ALU operations are represented with four bits in the definition.

```diff
// ALU
-`define ALU_FUNC_ADD    3'd0
-`define ALU_FUNC_SUB    3'd1
-`define ALU_FUNC_XOR    3'd2
-`define ALU_FUNC_OR     3'd3
-`define ALU_FUNC_AND    3'd4
-`define ALU_FUNC_SLL    3'd5
-`define ALU_FUNC_SRL    3'd6
-`define ALU_FUNC_SRA    3'd7
+`define ALU_FUNC_ADD    4'd0
+`define ALU_FUNC_SUB    4'd1
+`define ALU_FUNC_XOR    4'd2
+`define ALU_FUNC_OR     4'd3
+`define ALU_FUNC_AND    4'd4
+`define ALU_FUNC_SLL    4'd5
+`define ALU_FUNC_SRL    4'd6
+`define ALU_FUNC_SRA    4'd7


// ALU M extension
+`define ALU_FUNC_MUL    4'd8
+`define ALU_FUNC_MULH   4'd9
+`define ALU_FUNC_MULHSU 4'd10
+`define ALU_FUNC_MULHU  4'd11
+`define ALU_FUNC_DIV    4'd12
+`define ALU_FUNC_DIVU   4'd13
+`define ALU_FUNC_REM    4'd14
+`define ALU_FUNC_REMU   4'd15

```

### Modify `Decode.v`

The instructions in the image below pertain to the `M extension`, and they all fall under the R-type category. To enable the decoder to understand these instructions, refer to the machine code definitions in the table for the corresponding RTL code
![image](https://i.imgur.com/LnaYGBg.png)

```diff
+            /* MUL   */
+            17'b0100001_000_0110011:
+            begin
+                instr_scope = "MUL";
+                rf_we_o = 1'b1;
+                rf_din_sel_o = 3'd2;
+                a_op_sel_o = 1'b0;
+                b_op_sel_o = 1'b0;
+                alu_op_sel_o = `ALU_FUNC_MUL;
+            end
+
+            /* MULH   */
+            17'b0100001_001_0110011:
+            begin
+                instr_scope = "MULH";
+                rf_we_o = 1'b1;
+                rf_din_sel_o = 3'd2;
+                a_op_sel_o = 1'b0;
+                b_op_sel_o = 1'b0;
+                alu_op_sel_o = `ALU_FUNC_MULH;
+            end
+
+            /* MULHSU   */
+            17'b0100001_010_0110011:
+            begin
+                instr_scope = "MULHSU";
+                rf_we_o = 1'b1;
+                rf_din_sel_o = 3'd2;
+                a_op_sel_o = 1'b0;
+                b_op_sel_o = 1'b0;
+                alu_op_sel_o = `ALU_FUNC_MULHSU;
+            end
+
+            /* MULHU   */
+            17'b0100001_011_0110011:
+            begin
+                instr_scope = "MULHU";
+                rf_we_o = 1'b1;
+                rf_din_sel_o = 3'd2;
+                a_op_sel_o = 1'b0;
+                b_op_sel_o = 1'b0;
+                alu_op_sel_o = `ALU_FUNC_MULHU;
+            end
+
+            /* DIV   */
+            17'b0100001_100_0110011:
+            begin
+                instr_scope = "DIV";
+                rf_we_o = 1'b1;
+                rf_din_sel_o = 3'd2;
+                a_op_sel_o = 1'b0;
+                b_op_sel_o = 1'b0;
+                alu_op_sel_o = `ALU_FUNC_DIV;
+            end
+
+            /* DIVU   */
+            17'b0100001_101_0110011:
+            begin
+                instr_scope = "DIVU";
+                rf_we_o = 1'b1;
+                rf_din_sel_o = 3'd2;
+                a_op_sel_o = 1'b0;
+                b_op_sel_o = 1'b0;
+                alu_op_sel_o = `ALU_FUNC_DIVU;
+            end
+
+            /* REM   */
+            17'b0100001_110_0110011:
+            begin
+                instr_scope = "REM";
+                rf_we_o = 1'b1;
+                rf_din_sel_o = 3'd2;
+                a_op_sel_o = 1'b0;
+                b_op_sel_o = 1'b0;
+                alu_op_sel_o = `ALU_FUNC_REM;
+            end
+
+            /* REMU   */
+            17'b0100001_111_0110011:
+            begin
+                instr_scope = "REMU";
+                rf_we_o = 1'b1;
+                rf_din_sel_o = 3'd2;
+                a_op_sel_o = 1'b0;
+                b_op_sel_o = 1'b0;
+                alu_op_sel_o = `ALU_FUNC_REMU;
+            end
```

Also, change the `alu_op_sel` due to the adding one bit in `Defs.vh`

```diff=
-output reg [2:0] alu_op_sel_o,
+output reg [3:0] alu_op_sel_o,
```

### Modify `AtomRV.v`

change the wire `d-alu_op_sel` to 4 bit.

```diff
        ////// Instruction Decode //////
        Instruction decode unit decodes instruction and sets various control
        signals throughout the pipeline. Is also extracts immediate values
        from instructions and sign extends them properly.
    */
    wire    [4:0]   d_rd_sel;
    wire    [4:0]   d_rs1_sel;
    wire    [4:0]   d_rs2_sel;
    wire    [31:0]  d_imm;

    wire            d_jump_en;
    wire    [2:0]   d_comparison_type;
    wire            d_rf_we;
    wire    [2:0]   d_rf_din_sel;
    wire            d_a_op_sel;
    wire            d_b_op_sel;
    wire            d_cmp_b_op_sel;
-   wire    [2:0]   d_alu_op_sel;
+   wire    [3:0]   d_alu_op_sel;
    wire    [2:0]   d_mem_access_width;
    wire            d_mem_load_store;
    wire            d_mem_we;


```

### Modify `alu.v`

Modify the input wire `sel_i` to be 4 bits and then integrate the circuitry to execute each instruction in the `M extension`

```diff
(
    input   wire    [31:0]  a_i,
    input   wire    [31:0]  b_i,
-   input   wire    [2:0]   sel_i,
+   input   wire    [3:0]   sel_i,

    output  reg     [31:0]  result_o
);

+/////// m extension
+    wire sel_mul 	= (sel_i == `ALU_FUNC_MUL);
+    wire sel_mulh	= (sel_i == `ALU_FUNC_MULH);
+    wire sel_mulhsu 	= (sel_i == `ALU_FUNC_MULHSU);
+    wire sel_mulhu 	= (sel_i == `ALU_FUNC_MULHU);
+    wire sel_div 	= (sel_i == `ALU_FUNC_DIV);
+    wire sel_divu 	= (sel_i == `ALU_FUNC_DIVU);
+    wire sel_rem 	= (sel_i == `ALU_FUNC_REM);
+    wire sel_remu 	= (sel_i == `ALU_FUNC_REMU);
+    /////// result of m extension
+    reg [63:0] mul_result;
+    reg [31:0] div_result;
+    reg [31:0] rem_result;
+     //mul,mulh,mulhsu,mulhu
+    always @(*) begin
+        if (sel_mul)
+            mul_result = $signed(a_i) * $signed(b_i);
+        else if (sel_mulhu)
+            mul_result = (a_i)*(b_i);
+        else if (sel_mulhsu)
+            mul_result = $signed(a_i)* (b_i);
+        else if (sel_mulh)
+            mul_result = $signed(a_i) * $signed(b_i);
+        else
+            mul_result= 64'h0  ;
+    end
+    //div divu
+    always @(*) begin
+        if (sel_div)
+            div_result = (b_i == 32'h0) ? 32'hffffffff:
+            	   	  (a_i == 32'h80000000 && b_i == 32'hffffffff) ? 32'h80000000:
+                	  $signed(a_i) / $signed(b_i);
+        else if (sel_divu)
+            div_result = (b_i == 32'h0) ? 32'hffffffff:
+           		  $unsigned($unsigned(a_i) / $unsigned(b_i));
+        else
+            div_result= 32'h0;
+    end
+    //rem remu
+    always @(*) begin
+        if (sel_rem)
+            rem_result = (b_i == 32'h0) ? a_i:
+            	   	  (a_i == 32'h80000000 && b_i == 32'hffffffff) ? 32'h0:
+                	  $signed(a_i) % $signed(b_i);
+        else if (sel_remu)
+            rem_result = (b_i == 32'h0) ? a_i:
+            		  $unsigned($unsigned(a_i) % $unsigned(b_i));
+        else
+            rem_result= 32'h0;
+    end


    // output of universal shifter
    reg [31:0] final_shift_output;
    always @(*) begin
        if (sel_sll)
            final_shift_output = reverse(shift_output[31:0]);
        else
            final_shift_output = shift_output[31:0];
    end

    // Final output mux
    always @(*) begin
        if (sel_add | sel_sub)
            result_o = arith_result;
        else if (sel_sll | sel_srl | sel_sra)
            result_o = final_shift_output;
        else if (sel_xor)
            result_o = a_i ^ b_i;
        else if (sel_or)
            result_o = a_i | b_i;
        else if (sel_and)
            result_o = a_i & b_i;
+        else if (sel_mul)
+            result_o = mul_result[31:0];
+        else if (sel_mulhu | sel_mulh | sel_mulhsu)
+            result_o = mul_result[63:32];
+        else if (sel_div | sel_divu)
+            result_o = div_result;
+        else if (sel_rem | sel_remu)
+            result_o = rem_result;
        else
            result_o = arith_result;
    end
```

## References

-
