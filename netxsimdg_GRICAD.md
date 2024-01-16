# How to run nextsimdg on GRICAD

Once you are all [set up on GRICAD servers](https://github.com/sasip-climate/catalog-shared-data-SASIP/blob/main/gricad.md) and familiar with how to [compute there](https://github.com/sasip-climate/GRICAD-usage/blob/main/compute_GRICAD.md), you may want to run the latest version of neXtSIM-DG. 

his tuto explains the different steps  to go through in order to compile and run the nextSIM-DG code on the GRICAD server:
1. (Do it once for all) Set up the environment (specific libraries required)  for nextSIM-DG 
2. Activate your nextSIM-DG environment each time you want to re-compile the code
3. Compile the code
4. Run a simulation.
   
## 1. (Do it once for all) Set up the environment for nextSIM-DG 
To compile the neXtSIM-DG code, you need some specific librairies, namely `cmake`,`boost`,`eigen`,`netcdf-cxx4`,`netCDF4`. 
You can do so by creating a conda environment that contains all these libraries (do it once for all), and then you'll then have to activate it _each time_ we want to compile neXtSIM-DG, as explained below. The installation of the environment is a bit long to run on the frontal node so we put it in a script and launch it on a computing node. In the instruction below the conda environment is created from a file where all the required libraries are specified (2023-01-15-environment-nextsimdg.yml). 

* Copy this script to your space on the GRICAD server. Let's name it `create_conda_env_nextsimdg.sh` :

```bash
#!/bin/bash

#OAR -n nextsimdg
#OAR -l /nodes=1/core=1,walltime=00:30:30
#OAR --stdout conda.%jobid%.stdout
#OAR --stderr conda.%jobid%.stderr
#OAR --project pr-sasip
#OAR -t devel

# link to the suggested environment file for conda:
ln -sf 2/summer/sasip/023-01-15-environment-nextsimdg.yml

# activate conda
source /applis/environments/conda.sh

# create your nextsimdg environment
conda env create -n nextsimdg -f 2023-01-15-environment-nextsimdg.yml
```

* Make this script an executable :
```bash
chmod +x  create_conda_env_nextsimdg.sh
```

* Run this script on a computing node
oarsub -S ./create_conda_env_nextsimdg.sh
```

You can monitor if your run is running and when it is finished with the command : `oarstat -u mygricadlogin`.


## 2. Each time you start a new session and want to compile nextSIM-DG, activate its environment

* Activate:
```bash
source /applis/environments/conda.sh
conda activate nextsimdg
```

* As a check, print out the list of libraries installed in this environment:
```bash
source /applis/environments/conda.sh
conda list
```



## 3.  Compile neXtSIM-DG :

```bash
git clone -b develop https://github.com/nextsimhub/nextsimdg.git nextsimdg
cd nextsimdg
mkdir -p build
cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
make
```

It should produce a nextsim executable in the build repository.

- Run on GRICAD :
  - copy, adapt and put this run_june23_gricad.sh script in the `/run/` repository of netxsimdg :
 
```bash
#!/bin/bash

#OAR -n nextsimdg
#OAR -l /nodes=1/core=4,walltime=00:11:30
#OAR --stdout nextsimdg.%jobid%.stdout
#OAR --stderr nextsimdg.%jobid%.stderr
#OAR --project pr-sasip
#OAR -t devel

cd ~/git/nextsimdg/run

ln -sf ../build/nextsim

ln -sf /summer/sasip/model-forcings/nextsim-dg/init_25km_NH.nc .
ln -sf /summer/sasip/model-forcings/nextsim-dg/25km_NH.ERA5_2010-01-01_2011-01-01.nc .
ln -sf /summer/sasip/model-forcings/nextsim-dg/25km_NH.TOPAZ4_2010-01-01_2011-01-01.nc .

./nextsim --config-file config_june23.cfg --model.run 86400 > time.step
```
   - launch the job :

```bash
chmod +x  run_june23_gricad.sh
oarsub -S ./run_june23_gricad.sh
```
