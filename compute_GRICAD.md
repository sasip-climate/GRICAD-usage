# Compute on GRICAD servers

Once you have set up your account (check [this tutorial](https://github.com/sasip-climate/catalog-shared-data-SASIP/blob/main/gricad.md)) and joined the pr-sasip group, you can now compute on CPU (ssh dahu.ciment) or GPU (ssh bigfoot.ciment)

### Workspaces

 * On each cluster (dahu, bigfoot) you have a personnal worspace : ```/home/yourlogin```
 * A personnal scratch workspace accessible from both dahu and bigfoot : ```/bettik/PROJECTS/yourproject/yourlogin``` with a quota of between 3 and 5 Tb per user.
 * same architecture for another scratch workspace called silenus instead of bettik
 * the summer data which is mirrored [online](https://ige-meom-opendap.univ-grenoble-alpes.fr/thredds/catalog/meomopendap/extract/SASIP/catalog.html) : ```/summer/sasip``` ;it is advised to copy data from it locally so that the computation is faster (not the same filesystem)

### Connection

 * Now that you are all set up, you just have to type ```ssh dahu``` 
 * If you need to transfer some data yo can do a simple ```scp mydata dahu:/path/to/data/.```
 * If the data you want to transfer is big, go through the cargo server : ```scp mydata yourlogin@cargo.univ-grenoble-alpes.fr:/path/to/data/.```

### Submitting jobs

  * You are actually sitting on login nodes (f-dahu or bigfoot), to do some computation you will need to request some computing nodes
  * You do that by either launching your script inside a job or ask for interactive access to a computing node :

<details>
<summary>An example for interactive computing</summary>
 
 ```oarsub -l /nodes=1/core=16,walltime=03:30:00 --project pr-data-ocean -I```
 
</details>

 * When your request is granted you will be connected to a specific dahu node and you will be able to compute there.
 * Maximum time limit is 12 hours
 * The memory allocated to your request is nb_cores_requested*node_memory/nb_cores_per_node, for instance on a classical dahu node there is a total of 192Gb per node, if you ask for 16 cores you will be granted 96Gb, on a fat node a total of 1.5Tb is available (check node properties with recap.py)


<details>
<summary>An example job</summary>
 
 ```
 #!/bin/bash

#OAR -n jobname
#OAR -l /nodes=2/core=1,walltime=00:01:30
#OAR --stdout jobname.out%jobid
#OAR --stderr jobname.err%jobid
#OAR --project data-ocean

yourscript
```
 
</details>

 * Make sure your job script is executable ```chmod +x job.ksh``` and then launch it with ```oarsub /path/to/the/job.ksh``` (you have to provide absolute path or be in the directory and type : ```oarsub -S ./job.ksh```

 * You can check the status of your job with ```oarstat -u yourlogin``` and kill your job if needed with ```oardel jobid``` with jobid being the first number in the result of oarsat

 * Maximum time limit on dahu is 2 days
 
 * If your code is not in the production phase yet, you can ask to test it first on a development queue by adding the option ```-t devel``` to your oarsub command or in your job with a maximum time limit of 30 minutes, you first need to connect to dahu-oar3 first : ```ssh dahu-oar3```
 
 * Another useful queue is the fat one (option ```-t fat```and provide access to nodes with a total of 1.5Tb of RAM per node)
 * A queue called visu is also available
 
 * For more informations about jobs read https://gricad-doc.univ-grenoble-alpes.fr/en/hpc/joblaunch/

### See availability of Dahu nodes : 

 * Command ```chandler``` in the terminal or go to the website : https://ciment-grid.univ-grenoble-alpes.fr/clusters/dahu/monika for instaneous availablity or https://ciment-grid.univ-grenoble-alpes.fr/clusters/dahu/drawgantt/drawgantt.php for availability over time (history and forecast)

 
---

## Using NCO tools (+ncdump & ncview)

   * put this line in your .bash_profile : ```source /applis/site/nix.sh```
   * download the desired package with nix :

```
nix-env -i -A netcdf
nix-env -i -A nco
nix-env -i -A ncview
```
   * the binaries are now in ```/home/yourlogin/.nix-profile/bin``` and accessible from anywhere (your PATH is updated when you source the nix application)

---
## Using Conda 

Conda is already installed. You just need to activate it using the command from the server.

```
source /applis/environments/conda.sh
```

### Create Personal Conda Environments


**Tip**: It's advisable to create conda environments in your `/bettik/username` directory. The main directory doesn't have a lot of space and you can easily fill up your home directory quota with conda packages (especially for machine learning). So use the `/bettik` directory.


1. Install Conda in Home Environment

1. Install an environment in `bettik`


Create conda from environment file:
```bash
conda env create -f environment.yml --prefix=/bettik/user/.conda/envs/env_name
```

1. Change the `.condarc` file to include all environment directories.

```bash
envs_dirs:
    - /bettik/username/.conda/envs
    - /home/username/.conda/envs
```

**Note**: You can add as many directories as you want. This just ensures that conda can talk to it.


---

## Run the jupyter notebook with conda environment on dahu (or bigfoot)

First, take a look at this tutorial to get familiar: https://gricad-doc.univ-grenoble-alpes.fr/notebook/hpcnb/

Below offers a simpler work structure.

You need to have a conda environment with jupyter installed in it, also make sure you have the lines :

```
c.NotebookApp.open_browser = False
c.NotebookApp.ip = '0.0.0.0'
```
in your .jupyter/jupyter_notebook_config.py
 
**In the First Terminal** - Start the JupyterLab session

1. Log into your server


```bash
ssh dahu
```

2. Start an interactive session

```bash
oarsub -I --project data-ocean -l /core=10,walltime=2:00:00 -> it will log automatically on a login node dahuX
```

3. Activate your conda environment with JupyterLab

```
conda activate jupyter
```

4. Start your jupyterlab session


```bash
jupyter notebook --no-browser --port 1234
```



**In the second terminal** - we will do the ssh tunneling procedure to view jupyterlab on your local machine.

1. Do the Tunneling

```bash
ssh -fNL 1234:dahuX:1234  dahu
```

2. Open `http://localhost:1234/?token=...(see the result of the jupyter notebook command)` on a browser in your laptop.


3. When you're done, make sure you close the tunnel you opened.



```bash
# get the process-ID number for ssh tunneling
lsof -i :1234
# kill that process
kill -9 PID
```

