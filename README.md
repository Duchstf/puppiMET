# puppiMET

Puppi MET setup in correlator layer 2. 

## Prereqs

- Have a machine with `vivado` (license) installed
- Join egroups: https://e-groups.cern.ch/e-groups, otherwise the gitlab repo's won't be accessible
    ```
    hls-fml
    emp-fwk-users
    cms-tcds2-users
    cms-cactus 
    ```
## Install 

### Get Conda

- Prep a `path` which you want to use for your conda installation
- Download and install
    ```
    curl -L -O "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"
    bash Miniforge3-$(uname)-$(uname -m).sh
    ```
- In principle this should be enough, in practice, log out and back in. 
- Check if paths make sense
    ```
    which conda
    which python
    which pip
    ```

### Create an env
- Needs to be Python 2.7 (don't ask why)
- Get updated git also, system version might fail intransparently on krb gitlab auth

    ```
    mamba create -n py27 python=2.7
    mamba activate py27
    mamba install git
    which git # Should be in conda path now
    ```

<!-- ### Source Vivado
- At `correlator<#>.fnal.gov`
    ```
    source /data/Xilinx/Vivado/2022.2/settings64.sh
    ```
- At `mitfelix.mit.edu`
    ```
    source /tools/Xilinx/Vivado/2022.1/settings64.sh
    ```
  - Missing `2022.2` ?
- This is crucial -->

### Install emp-fwk

```
curl -L https://github.com/ipbus/ipbb/archive/dev/2022f.tar.gz | tar xvz
source ipbb-dev-2022f/env.sh
```

#### Setup Area
```
ipbb init p2fwk-work
cd p2fwk-work
```

#### Install depedencies

- Install via kerberos, run `kinit username@CERN.CH`

    ```
    ipbb add git https://:@gitlab.cern.ch:8443/p2-xware/firmware/emp-fwk.git -r v0.7.4
    ipbb add git https://gitlab.cern.ch/ttc/legacy_ttc.git -b v2.1
    ipbb add git https://:@gitlab.cern.ch:8443/cms-tcds/cms-tcds2-firmware.git -b v0_1_1
    ipbb add git https://gitlab.cern.ch/HPTD/tclink.git -r fda0bcf
    ipbb add git https://gitlab.cern.ch/dth_p1-v2/slinkrocket_ips.git -b v03.12
    ipbb add git https://:@gitlab.cern.ch:8443/dth_p1-v2/slinkrocket.git -b v03.12
    ipbb add git https://github.com/ipbus/ipbus-firmware -b v1.9

    #For the Jet setup
    ipbb add git https://:@gitlab.cern.ch:8443/rufl/RuflCore.git -r d3ddf86f
    ipbb add git https://:@gitlab.cern.ch:8443/cms-cactus/phase2/firmware/correlator-common.git -b master-133x
    ipbb add git https://:@gitlab.cern.ch:8443/cms-cactus/phase2/firmware/correlator-layer2.git
    ```

#### Build dependencies
- Setup CMSSW

    ```
    source /cvmfs/cms.cern.ch/cmsset_default.sh

    cd src/correlator-common/
    ./utils/setup_cmssw.sh -run CMSSW_13_3_0_pre3 cms-l1t-offline:phase2-l1t-integration-13_3_0_pre3 phase2-l1t-1330pre3_v14
    ```
- Build errors will happen, but can be ignored seemingly 
    ```
    Building edm plugin tmp/slc7_amd64_gcc10/src/L1Trigger/L1CaloTrigger/plugins/L1TriggerL1CaloTriggerAuto/libL1TriggerL1CaloTriggerAuto.so
    /cvmfs/cms.cern.ch/slc7_amd64_gcc10/external/gcc/10.3.0-84898dea653199466402e67d73657f10/bin/../lib/gcc/x86_64-unknown-linux-gnu/10.3.0/../../../../x86_64-unknown-linux-gnu/bin/ld: cannot find -lssl
    /cvmfs/cms.cern.ch/slc7_amd64_gcc10/external/gcc/10.3.0-84898dea653199466402e67d73657f10/bin/../lib/gcc/x86_64-unknown-linux-gnu/10.3.0/../../../../x86_64-unknown-linux-gnu/bin/ld: cannot find -lcrypto
    collect2: error: ld returned 1 exit status
    gmake: *** [config/SCRAM/GMake/Makefile.rules:1712: tmp/slc7_amd64_gcc10/src/L1Trigger/L1CaloTrigger/plugins/L1TriggerL1CaloTriggerAuto/libL1TriggerL1CaloTriggerAuto.so] Error 1
    gmake: *** [There are compilation/build errors. Please see the detail log above.] Error 2
    ```

- Export to env
    ```
    export CMSSW_VERSION=CMSSW_13_3_0_pre3
    ```

#### Compile dependencies
- These require older version of Vivado
    ```
    source /tools/Xilinx/Vivado/2019.1/settings64.sh
    # Correlator2
    source /opt/local/Xilinx/Vivado/2019.2/settings64.sh
    ```
- Compile
    ```
     cd ~/p2fwk-work/src/correlator-common/jetmet/htmht/
     bash synth_all.sh
     cd ../jec/
     vivado_hls run_Synth.tcl
     cd ../seededcone
     vivado_hls run_JetLoop.tcl
     vivado_hls run_JetFormat.tcl
     vivado_hls run_JetCompute.tcl
     cd ~/p2fwk-work/proj/jet-sim/ 
    ```

#### Link Cactus [Not needed for simulation]
- `uhal` is a required dependency, either link it:
    ```
    export LD_LIBRARY_PATH=/opt/cactus/lib:$LD_LIBRARY_PATH
    export PATH=/opt/cactus/bin:$PATH
    export PATH=/opt/cactus/bin/uhal/tools:$PATH
    ```
- or install it from [here](https://ipbus.web.cern.ch/doc/user/html/software/installation.html)

### Vivado License
- `synth` commands will fail without a license
- Obtain a license file eg `Xilinx.lic`
- Source `vivado` eg `source /tools/Xilinx/Vivado/2022.1/settings64.sh`
- Run the Vivado license managere `vlm` GUI (requires X11)
- Load license file

## Building bitfile and running simulation
- (Assume a fresh login) 
  - And a `(base)` conda env existing
  - `py27` env seems to not be necessary (FIXME)
- Source `ipbb` and `Vivado`
    ```
    source ipbb-dev-2022f/env.sh
    source /tools/Xilinx/Vivado/2022.1/settings64.sh
    #Correlator2
    source /opt/local/Xilinx/Vivado/2022.2/settings64.sh
    
    # Required for `uhal`
    export LD_LIBRARY_PATH=/opt/cactus/lib:$LD_LIBRARY_PATH 
    # Required for `genencoders`
    export PATH=/opt/cactus/bin/uhal/tools:$PATH
    
    cd p2fwk-work/
    ```
    
### A pass-through test with building the null algorithm

* In this part you will build a pass_through algorithm (it doesn't do anything for the inputs you put in and just outputs the same thing) for the VU9P boards. 

```
# Create env 
ipbb proj create vivado test_null emp-fwk:projects/examples/serenity/dc_vu9p so2/top.dep
proj/test_null/

# Run simulation
ipbb vivado generate-project --enable-ip-cache -1

# Run firmware
ipbb ipbus gendecoders
ipbb vivado generate-project
ipbb vivado synth -j4 impl -j4
ipbb vivado package

# ls package/test_null_mitfelix_server_231107_0520.tgz
```
- `null_algo` being tested [link](https://gitlab.cern.ch/p2-xware/firmware/emp-fwk/-/blob/v0.7.4/projects/examples/serenity/dc_vu9p/firmware/cfg/so2/top.dep?ref_type=tags)
- Is simulation the same as here? https://serenity.web.cern.ch/serenity/emp-fwk/firmware/instructions.html#step-3-build-firmware-or-run-simulation
- `package` step produces output files that can be tested on hardware, namely `.bit` and a zipped file

### Running the jet simulation


