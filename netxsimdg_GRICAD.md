# How to run NextSIM-DG on GRICAD

*Note : this tutorial is based on the 2023, June version of nextsim_dg, it will updated as soon as a new release is available !*

Once you are all [set up on GRICAD servers](https://github.com/sasip-climate/catalog-shared-data-SASIP/blob/main/gricad.md) and familiar with how to [compute there](https://github.com/sasip-climate/GRICAD-usage/blob/main/compute_GRICAD.md), you may want to run the latest version of neXtSIM-DG. 
This tuto explains the different steps  to go through in order to compile and run the nextSIM-DG code on the GRICAD server:
1. _[Do it once for all]_ <a href="#part1"> __Set up the environment, i.e. install the specific libraries required for nextSIM-DG__ </a> 
2. <a href="#part2">__Activate your nextSIM-DG environment__</a>  _[Do it each time you want to re-compile the code]_
3. <a href="#part3"> __Compile the code__</a> 
4. <a href="#part4">__Run a simulation__</a> 
   
## 1. <a name="part1"> (Do it once for all) Set up the environment for nextSIM-DG </a> 
To compile the neXtSIM-DG code, you need some specific librairies, namely `cmake`,`boost`,`eigen`,`netcdf-cxx4`,`netCDF4`. 
One way to do so is to create a conda environment that contains all these libraries (do it once for all). And then you'll then have to activate this environment  _each time_ we want to compile neXtSIM-DG, as also explained below. 

The installation of the environment is a bit long to run on the frontal node so do it interactively on a computation node, request one with :

```bash
oarsub -l /nodes=1/core=1,walltime=01:30:00 --project pr-sasip -I
```

Once you are connected to a dahu or bigfoot node (you can see that the start of the command line switched from yourlogin@f-dahu starts with yourlogin@dahu112 for instance), build the conda environment with the list of libraries specified ina file sitting on summer :

```bash
conda create --name nextsimdg --file /summer/sasip/model-configurations/neXtSIM-DG/demo-june2023/spec-file-SLX.txt
```

## 2. <a name="part2"> Each time you start a new session and want to compile nextSIM-DG, activate its environment </a> 

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
It should contains the required packages: `cmake`,`boost`,`eigen`,`netcdf-cxx4`,`netcdf4`.

You are now ready to compile the NeXtSIM-DG code.


## 3.  <a name="part3"> Compile neXtSIM-DG : </a> 

```bash
# download the develop branch version of the code
git clone -b develop https://github.com/nextsimhub/nextsimdg.git nextsimdg

# go to directory and get the version of the code that is compatible with June 2023 hands-on :
cd nextsimdg
git checkout 493ec8fb

# create and go to a ./build/ directory where the code is going to be compiled
mkdir -p build
cd build

# run cmake to prepare the compilation (Amongs other things it checks that all the required packages are there).
cmake -DCMAKE_BUILD_TYPE=Release ..

# compile by running make
make
```

Be patient, as it might take several  minutes to compile. In the end, if successful,  it should produce a `nextsim` executable  located in the build repository.

## 4.  <a name="part4"> Run a simulation : </a> 
We give below the same example of simulation as the demo we ran in June 2023 at the SASIP General assembly meeting.

As it will complete under 30mn, we will run it on the devel class of jobs that is faster than the regular one, but for that we first need to connect to dahu-oar3 : ```ssh dahu-oar3```

If your job needs more than 30mn, remove the ```#OAR -t devel``` line in the script below and no need to connect to dahu-oar3

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
