---
title: "ExaGO for Dummies"
date: 2024-03-23
draft: false
---

ExaGO is an high-performance power grid optimization program for stochastic, security-constrained, and multi-period AC optimal power flow problems. It was designed to be run on supercomputers to simulate thousands of different scenarios on the power grid, such as cyberattacks and natural disasters. 

This guide will show you how to run ExaGO on consumer hardware so you can simulate and visualize the powergrid for yourself, no supercomputer required! It can be easily installed in a virtual machine or Windows Subsystem for Linux. The following instructions assume you are using Ubuntu 22.04.

First, we'll install some pre-requisites.

```
sudo apt update && sudo apt install git vim build-essential gfortran
```

Now request an HSL License from https://licences.stfc.ac.uk/product/coin-hsl. Licenses are free for university students with a .edu email, but they take a couple days to grant access.

```
# Download CoinHSL, move it to the home directory, and rename the file
mv ~/Downloads/coinhsl-2022.11.09.tar.gz ~/coinhsl-archive-2022.11.09.tar.gz
```

Clone ExaGO and follow the instructions in ExaGO/docs/installing_with_spack.md.

```
git clone https://github.com/pnnl/ExaGO.git
git clone -c feature.manyFiles=true https://github.com/spack/spack.git
export PATH=$PWD/spack/bin:$PATH
source spack/share/spack/setup-env.sh
```

Edit CoinHSL's Spack config. You will have to use vim; beginners may want to look up a quick vim tutorial.

```
spack edit coinhsl
```

Add the following version and checksum. You can verify the checksum at https://licences.stfc.ac.uk/product/coin-hsl if desired.

```
version("2022.11.09", sha256="d6d9089bb9cf3eb0e4af195f1a2f10cd61da42eddf8da73a12b8c62902bceee3")
```

Remove the following from the config:

```
version(
        "2015.06.23",
        sha256="3e955a2072f669b8f357ae746531b37aea921552e415dc219a5dd13577575fb3",
        preferred=True,
    )
```

Now, install ExaGO. If the initial build fails, try running it again. Network errors often cause issues during install.

```
spack compiler find
spack install exago@develop%gcc \
  ^openmpi ^ipopt@3.12.10+coinhsl~mumps ^coinhsl+blas \
  ^petsc+mpi~hypre~superlu-dist~mumps+shared
```

Symlink libcoinhsl.so to the expected name libhsl.so. Keep in mind the directory named coinhsl-2022.11.09-... will be different for each build.

```
cd spack/opt/spack/linux-ubuntu22.04-skylake/gcc-11.4.0/coinhsl-2022.11.09-<INSERT_YOUR_PATH_HERE>/lib
sudo ln -s libcoinhsl.so libhsl.so 
```

Run opflow from the ExaGO directory.

```
spack load exago
cd ~/ExaGO
opflow datafiles/case_ACTIVSg2000.m
```

Congrats! Now we can run different scenarios on synthetic power grid data and stress test the fictional grid. In the next blog post, we'll explore visualizing the results of these simulations.
