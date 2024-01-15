# How to run nextsimdg on GRICAD

Once you are all [set up on GRICAD servers](https://github.com/sasip-climate/catalog-shared-data-SASIP/blob/main/gricad.md) and familiar with how to [compute there](https://github.com/sasip-climate/GRICAD-usage/blob/main/compute_GRICAD.md), you may want to run the latest version of neXtSIM-DG. Below we provide a tutorial how to comple nextsim-dg and run an example simulation.

## Create your computing environment :
To compile the neXtSIM-DG code, you need some specific librairies, namely `cmake`,`boost`,`eigen`,`netcdf-cxx4`,`netCDF4`. 
So you first need (once for all) to create a conda environment that contains all these libraries, and you'll then have to activate it _each time_ we want to compile neXtSIM-DG, as explained below. The installation of the environment is a bit long to run on the frontal node so we put it in a script and launch it on a computing node. One way to proceed is to use the environment file that we provide here (2023-01-15-environment-nextsimdg.yml, tested on GRICAD). 

The create_conda_env_nextsimdg.sh script :

```bash
#!/bin/bash

#OAR -n nextsimdg
#OAR -l /nodes=1/core=1,walltime=00:30:30
#OAR --stdout conda.%jobid%.stdout
#OAR --stderr conda.%jobid%.stderr
#OAR --project pr-sasip
#OAR -t devel

source /applis/environments/conda.sh
conda env create -n nextsimdg -f 2023-01-15-environment-nextsimdg.yml

```

And then we run it :
```bash
chmod +x  create_conda_env_nextsimdg.sh
oarsub -S ./create_conda_env_nextsimdg.sh
```

We can monitor if our run is running and when it is finished with the command : ```oarstat -u mygricadlogin```
Then we can activate the conda environment with :

```bash
source /applis/environments/conda.sh
conda activate nextsimdg
```

The creation is done only once but the activation must be done each time we want to compile nextsimdg.



- Compile neXtSIM-DG :

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
