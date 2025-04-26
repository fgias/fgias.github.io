---
title: "Starter Guide: Allen and the LHCb Stack — Installation, Development, Usage"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - CERN
  - LHCb
  - GPU
  - HPC
comments: true
---

This blog post combines various notes and technical instructions related to **Allen**, **the LHCb stack**, **CERN computing resources**, **authenticating with SSH and Kerberos**, **configuring Git**, and more.

- [The Allen Project](https://gitlab.cern.ch/lhcb/Allen)
- [Official Allen Documentation](https://allen-doc.docs.cern.ch/)
- [LHCb Stack Setup](https://gitlab.cern.ch/rmatev/lb-stack-setup/-/blob/master/README.md)
- [LHCb Starterkit Lessons](https://lhcb.github.io/starterkit-lessons/)

---

## Building Allen: Standalone

### LXPLUS

- Install ssh key for GitLab.
- Clone the repo:
  ```bash
  git clone ssh://git@gitlab.cern.ch:7999/lhcb/Allen.git
  ```
- Build with [CVMFS](https://cernvm.cern.ch/fs/) and CPU target:
  ```bash
  source /cvmfs/lhcb.cern.ch/lib/LbEnv
  mkdir build
  cd build
  cmake -DSTANDALONE=ON -DCMAKE_TOOLCHAIN_FILE=/cvmfs/lhcb.cern.ch/lib/lhcb/lcg-toolchains/LCG_101/x86_64-centos7-clang12-opt.cmake ..
  make
  ```
- Install Python dependencies:
  ```bash
  pip install wrapt pydotplus pydot sympy cachetools
  ```
- Put input MDF file into `Allen/input/` from `/scratch/allen_data/mdf_input/`.
- Run Allen:
  ```bash
  ./toolchain/wrapper ./Allen --sequence hlt1_pp_default --mdf ../input/upgrade_mc_minbias_scifi_v5_retinacluster_000_v1_newLHCbID.mdf -n 500 -m 500 -r 10 -t 16
  ```

---

### LXPLUS GPU

- Clone the repo:
  ```bash
  git clone ssh://git@gitlab.cern.ch:7999/lhcb/Allen.git
  ```
- Build with CVMFS and GPU target:
  ```bash
  source /cvmfs/lhcb.cern.ch/lib/LbEnv
  mkdir build
  cd build
  cmake -DSTANDALONE=ON -DCMAKE_TOOLCHAIN_FILE=/cvmfs/lhcb.cern.ch/lib/lhcb/lcg-toolchains/LCG_103/x86_64_v3-el9-gcc12+cuda12_1-opt.cmake ..
  make -j32
  ```

---

### MacOS M1

- Install Homebrew:
  ```bash
  /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
  ```
- Clone the repo:
  ```bash
  git clone ssh://git@gitlab.cern.ch:7999/lhcb/Allen.git
  ```
- Install dependencies:
  ```bash
  brew install llvm cpp-gsl catch2 zeromq nlohmann-json python3 boost
  pip3 install --user wrapt cachetools pydot sympy
  ```
- Clone UMESIMD:
  ```bash
  git clone https://github.com/edanor/umesimd.git /Users/<username>/umesimd
  export UMESIMD_ROOT_DIR=/Users/<username>/
  ```
- Symlink `libclang`:
  ```bash
  ln -s /Library/Developer/CommandLineTools/usr/lib/libclang.dylib /usr/local/lib/libclang.dylib
  ```
- Install CMake and ROOT:
  ```bash
  brew install cmake pkgconfig root
  ```
- Build Allen:
  ```bash
  mkdir build
  cd build
  cmake -DSTANDALONE=ON ..
  make
  ```
- Put input MDF file into `Allen/input/` from `/scratch/allen_data/mdf_input/`.
- Run Allen:
  ```bash
  ./Allen --sequence hlt1_pp_default --mdf ../input/upgrade_mc_minbias_scifi_v5_retinacluster_000_v1_newLHCbID.mdf -n 500 -m 500 -r 10 -t 16
  ```

---

## Building Allen: Gaudi / LHCb Stack

- Setup the environment:
  ```bash
  cd /afs/cern.ch/work/<username>/
  curl https://gitlab.cern.ch/rmatev/lb-stack-setup/raw/master/setup.py | python3 - stack
  cd stack
  $EDITOR utils/config.json
  BINARY_TAG=x86_64_v2-centos7-gcc11-opt
  make Allen
  ```

---

## Running Allen

- Event loop steered by Allen:
  ```bash
  ./Allen/build.x86_64_v2-centos7-gcc11-opt/run python Allen/Dumpers/BinaryDumpers/options/allen.py --mdf Allen/input/minbias/mdf/MiniBrunel_2018_MinBias_FTv4_DIGI_retinacluster_v1.mdf
  ```

- Event loop steered by Moore:
  ```bash
  ./Moore/run gaudirun.py Moore/Hlt/Moore/tests/options/default_input_and_conds_hlt1_retinacluster.py Moore/Hlt/Hlt1Conf/options/allen_hlt1_pp_default.py
  ```

---

## Installing in [CVMFS](https://cernvm.cern.ch/fs/)

- Packages are available at `/cvmfs/sft.cern.ch/lcg/releases`.
- If the package is missing, edit the appropriate `lcg-toolchains` fragment and add the entry:
  ```bash
  _add_lcg_entry("${LCG_releases_base}/gtest/1.11.0-21e8c/x86_64-centos9-gcc11-opt")
  ```

---

## Adding a New Device Algorithm in Allen

- Add a subdirectory in `device` called `new_algo`, with `src`, `include` and a `CMakeLists.txt`.


- Modify `device/CMakeLists.txt` to take the new folder into account.

  ```cpp
  add_subdirectory(new_algo)
  ```

- In `device/new_algo/CMakeLists.txt`, use `allen_add_device_library()` to add the new library, and give the include path with `target_include_directories()`.

  ```cpp
  file(GLOB new_algo_sources "src/*cu")

  allen_add_device_library(New_Algo STATIC
    ${new_algo_sources}
  )

  target_include_directories(New_Algo PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>)
  ```

- The include `AlgorithmTypes.cuh` is including other files, e.g. `BackendCommon.h`, so use `target_link_libraries` with the libraries below, to use all these other libraries in Allen. In the same file add:

  ```cpp
  target_link_libraries(New_Algo PRIVATE Backend HostEventModel EventModel Utils)
  ```

- To instantiate an algorithm, `INSTANTIATE_ALGORITHM()`, you habe to link the new library `New_Algo` to the `Stream` library: modify `stream/CMakeLists.txt`.

  ```cpp
  target_link_libraries(Stream
    PRIVATE
    ...
    New_Algo
    ...
  )
  ```
Why? Because otherwise you get the linker error: 

  ```bash
  libAllenLib.so: undefined reference to `Allen::TypeErasedAlgorithm 
  Allen::instantiate_algorithm<new_algo::new_algo_t>(std::__cxx11::basic_string<char, 
  std::char_traits<char>, std::allocator<char> > const&)'
  ```

- Define a _global kernel_, in order to be able to execute the algorithm on the device. In the `.cuh` file:

  ```cpp
  __global__ 
  void new_algo(Parameters);
  ```

- Add the implementation of the kernel in the `.cu` file:

  ```cpp
  __global__ 
  void new_algo::new_algo(new_algo::Parameters parameters)
  {
  ...
  }
```

- Define the methods of the algorithm, keeping in mind that an algorithm must define two methods: `set_arguments_size` and `operator()`. Also, in order to invoke host and global functions, wrapper methods `host_function` and `global_function` should be used.

- Implement the algorithm.

- Add the algorithm to the sequence.

- Run:

  ```bash
  ./toolchain/wrapper ./Allen --sequence new_algo --mdf /scratch/allen_data/mdf_input/upgrade-magdown-sim10-up08-30000000-digi_01_retinacluster_v1_newLHCbID.mdf -n 500 -m 500 -r 100 -t 16
  ```

---

## Allen Commands

### Measuring Physics Performance
```bash
./toolchain/wrapper ./Allen --sequence hlt1_pp_validation --mdf ../input/Upgrade_BsPhiPhi_MD_FTv4_DIGI_retinacluster_v1_newLHCbID.mdf -n 500 -m 500 -r 1000 -t 16
```

### Measuring Computational Performance
```bash
./toolchain/wrapper ./Allen --sequence hlt1_pp_default --mdf ../input/Upgrade_BsPhiPhi_MD_FTv4_DIGI_retinacluster_v1_newLHCbID.mdf -n 500 -m 500 -r 1000 -t 16
```

---

## LHCb Stack Instructions

### Setup on LHCb Online

- Clone and configure:
  ```bash
  curl https://gitlab.cern.ch/rmatev/lb-stack-setup/raw/master/setup.py | python3 - stack
  cd stack
  utils/config.py binaryTag x86_64_v3-el9-gcc13+detdesc-opt+g
  make Moore BUILDFLAGS='-j 10'
  ```

### Generate MDF Files

- Initialize the lhcb proxy:
  ```bash
  source /cvmfs/lhcb.cern.ch/lib/LbEnv-stable
  lhcb-proxy-init
  ```

- Dump an MDF file:
  ```bash
  ./Moore/run gaudirun.py Moore/Hlt/Moore/tests/options/default_input_and_conds_hlt1.py Moore/Hlt/RecoConf/options/mdf_for_standalone_Allen.py
  ```

---

## CERN Systems Access

### LXPLUS

- Connect:
  ```bash
  ssh -XY <username>@lxplus.cern.ch
  ```

- Connect with port forwarding:
  ```bash
  ssh -XY -L 1235:localhost:1235 <username>@lxplus.cern.ch
  ```

- Launch Jupyter notebook:
  ```bash
  jupyter notebook --port=1235 --no-browser
  ```

---

### LXPLUS GPU

- Direct connection:
  ```bash
  ssh <username>@lxplus-gpu.cern.ch
  ```

- If problems occur, connect first to lxplus and then to gpu:
  ```bash
  ssh <username>@lxplus.cern.ch
  ssh <username>@lxplus-gpu.cern.ch
  ```

---

### CERN [LHCb Online](https://lbtwiki.cern.ch/bin/view/Online/ConnectingFromOutside) GPU

- Connect through LXPLUS:
  ```bash
  ssh -tt lbgw ssh n4050101
  ```
- Chain ssh directly:
  ```bash
  ssh -tt <username>@lxplus.cern.ch ssh -tt lbgw ssh n4050101
  ```

---

## File Storage at CERN

File storage at CERN is normally organized as follows:

- User directory: `/afs/cern.ch/user/<username>/`
- Work directory: `/afs/cern.ch/work/<username>/`
- Private EOS: `/eos/user/<username>/`
- Public EOS: `/eos/lhcb/user/<username>/`

---

## VSCode Troubleshooting

- If you cannot connect with VSCode, log in manually via terminal first, then retry from VSCode.

- Connect via terminal and delete the VSCode server folder

  ```bash
  rm -rf ~/.vscode-server
  ```
  and then retry connecting.

- Sometimes can cause problems: Untick `Remote.SSH: Use Flock` in the remote VSCode server settings.

---

## Obtaining a CERN Grid Certificate

Finally, if you need help in obtaining a grid certificate see this [post](http://fotisgiasemis.com/blog/cern-grid-certificates/).

## SSH Configuration File

For help configuring your ssh configuration file see this [file](https://gitlab.cern.ch/rmatev/lb-stack-setup/-/blob/master/README.md).

## Authenticating with Kerberos

For full instructions see this [website](https://linux.web.cern.ch/docs/kerberos-access/). 

### Instructions on MacOS

- Install kerberos (MacOS normally has kerberos already installed).

- Copy the CERN krb5.conf file (found on the website above) to your `/etc/krb5.conf`.

- Modify your ssh config file (`~/.ssh/config`), under the lxplus host add:

  ```
  GSSAPIAuthentication yes
  GSSAPIDelegateCredentials yes
  ```

- Get kerberos tickets using `/usr/bin/kinit`:

  ```bash
  /usr/bin/kinit <username>@CERN.CH
  ```

- Check which kinit is being used: 

  ```bash
  which kinit
  ```

## Git/GitLab Configuration

SSH authentication to GitLab:

- Generate key:

  ```bash
  ssh-keygen -t ed25519 -C "<cern_email>@cern.ch"
  ```

- Copy its contents:

  ```bash
  tr -d '\n' < ~/.ssh/id_ed25519.pub | pbcopy
  ```

- Add to GitLab.

- Verify key:

  ```bash
  ssh git@gitlab.cern.ch -T -p 7999
  ```

Recommended Git configuration:

```bash
git config --global user.name "<name>"
git config --global user.email "<cern_email>@cern.ch"
git config --global color.ui "auto"

git config --global core.autocrlf input

git config --global credential.helper 'cache --timeout=28800'
```

## Done! ✅
 