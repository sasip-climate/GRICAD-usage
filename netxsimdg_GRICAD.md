# How to run nextsimdg on GRICAD

Once you are all [set up on GRICAD servers](https://github.com/sasip-climate/catalog-shared-data-SASIP/blob/main/gricad.md) and familiar with how to [compute there](https://github.com/sasip-climate/GRICAD-usage/blob/main/compute_GRICAD.md), you may want to run the latest version of neXtSIM-DG.

- Create your computing environment :

```bash
source /applis/environments/conda.sh
conda create --name nextsimdg
conda activate nextsimdg
conda install -c conda-forge boost
conda install -c anaconda cmake
conda install -c conda-forge eigen
conda install -c conda-forge netcdf-cxx4
```

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
  - copy, adapt and put this run_june23_gricad.sh script in the run repository of netxsimdg :
 
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
