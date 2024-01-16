# How to run NextSIM-DG on GRICAD

Once you are all [set up on GRICAD servers](https://github.com/sasip-climate/catalog-shared-data-SASIP/blob/main/gricad.md) and familiar with how to [compute there](https://github.com/sasip-climate/GRICAD-usage/blob/main/compute_GRICAD.md), you may want to run the latest version of neXtSIM-DG. 
This tuto explains the different steps  to go through in order to compile and run the nextSIM-DG code on the GRICAD server:
1. _[Do it once for all]_ __Set up the environment, i.e. install the specific libraries required for nextSIM-DG__ 
2. __Activate your nextSIM-DG environment__ _[Do it each time you want to re-compile the code]_,
3. __Compile the code,__
4. __Run a simulation__.
   
<a name="part1"> ## 1. (Do it once for all) Set up the environment for nextSIM-DG </a> 
To compile the neXtSIM-DG code, you need some specific librairies, namely `cmake`,`boost`,`eigen`,`netcdf-cxx4`,`netCDF4`. 
One way to do so is to create a conda environment that contains all these libraries (do it once for all). And then you'll then have to activate this environment  _each time_ we want to compile neXtSIM-DG, as also explained below. 

The installation of the environment is a bit long to run on the frontal node so we put it in a script and launch it on a computing node. In the instructions below, the conda environment is created from a file where all the required libraries are specified (i.e. `2023-01-15-environment-nextsimdg.yml`). 

* Copy this script to your space on the GRICAD server. Let's name it `create_conda_env_nextsimdg.sh` :

```bash
#!/bin/bash

#OAR -n nextsimdg
#OAR -l /nodes=1/core=1,walltime=00:25:00
#OAR --stdout conda.%jobid%.stdout
#OAR --stderr conda.%jobid%.stderr
#OAR --project pr-sasip
#OAR -t devel

# link to the required environment file for conda:
ln -sf /summer/sasip/model-configurations/nextsim-DG/2023-01-15-environment-nextsimdg.yml .

# activate conda
source /applis/environments/conda.sh

# create your nextsimdg environment
conda env create -n nextsimdg -f 2023-01-15-environment-nextsimdg.yml
```

* Make the above script an executable :
```bash
chmod +x  create_conda_env_nextsimdg.sh
```

* Run this script on a computing node:
```
oarsub -S ./create_conda_env_nextsimdg.sh
```
You can then monitor if your run is running and when it is finished with the command : `oarstat -u mygricadlogin`.


## 2. Each time you start a new session and want to compile nextSIM-DG, activate its environment

* Activate:
```bash
source /applis/environments/conda.sh
conda activate nextsimdg
```

* As a check, you can print out the list of libraries installed in this environment:
```bash
source /applis/environments/conda.sh
conda list
```
It should contains the required packages: `cmake`,`boost`,`eigen`,`netcdf-cxx4`,`netCDF4`.

You"re now ready to compile the NeXtSIM-DG code.


## 3.  Compile neXtSIM-DG :

```bash
# get latest version of the code from the github repo
git clone -b develop https://github.com/nextsimhub/nextsimdg.git nextsimdg

# go to directory
cd nextsimdg

# create and go to a ./build/ directory where the code is going to be compiled
mkdir -p build
cd build

# run cmake to prepare the compilation (Amongs other things it checks that all the required packages are there).
cmake -DCMAKE_BUILD_TYPE=Release ..

# compile by running make
make
```

Be patient, as it might take several  minutes to compile. In the end, if successful,  it should produce a `nextsim` executable  located in the build repository.

## 4.  Run a simulation :
We give below the same example of simulation as the demo we ran in June 2023 at the SASIP General assembly meeting.

* Copy, adapt and put this `run_june23_gricad.sh` script in the `./run/` repository :
 
```bash
#!/bin/bash

#OAR -n nextsimdg
#OAR -l /nodes=1/core=4,walltime=00:30:00
#OAR --stdout nextsimdg.%jobid%.stdout
#OAR --stderr nextsimdg.%jobid%.stderr
#OAR --project pr-sasip
#OAR -t devel

# go to nextsim-dg directory
cd ~/your-path/nextsimdg/run

# link to the executable you've just compiled
ln -sf ../build/nextsim .

# link to the initial state and forcing files
ln -sf /summer/sasip/model-forcings/nextsim-dg/init_25km_NH.nc .
ln -sf /summer/sasip/model-forcings/nextsim-dg/25km_NH.ERA5_2010-01-01_2011-01-01.nc .
ln -sf /summer/sasip/model-forcings/nextsim-dg/25km_NH.TOPAZ4_2010-01-01_2011-01-01.nc .

# run the model
./nextsim --config-file config_june23.cfg --model.run 86400 --ConfigOutput.filename myoutput.nc > time.step
```

* Makes the script executable and run it as a job with:

```bash
chmod +x  run_june23_gricad.sh
oarsub -S ./run_june23_gricad.sh
```
NOTE: In the script above, you run the model using the config file `config_june23.cfg` where all the parameters are set. Any parameter set from the command line will overwrite the corresponding parameter value in the config file. In the example above, `--model.run 86400` resets the model run to 1 day (value given in seconds). The option `--ConfigOutput.filename  myoutput.nc`resets the name of the output file to  ` myoutput.nc`.

* Once it has run successfully, you can see that a new file  have been created in the current directory (i.e., `myoutput.nc`)   that contains the evolution of sea ice simulated by the model over the period we just set (1 day in the above example). 
