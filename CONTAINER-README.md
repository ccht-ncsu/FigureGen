# Using the FigureGen Containers

Contact: Georgia Stuart, georgia.stuart@austin.utexas.edu
7 February 2021

## Input File Setup

For both Singularity and Docker container use, the path option for `ghostscript` and `gmt` in the FigureGen input
parameters file (ending in `.inp`) **must be left blank**. See `autotest/docker-test/BathyFilledCPT.inp` for example.

## Singularity Containers

[Singularity](https://sylabs.io/docs/) is a container platform designed for use with HPC systems.
We provide three Singularity definition files:
- `figuregen.def` provides MPI-enabled FigureGen for use on local Linux/Unix machines.
- `figuregen-serial.def` provides serial (not MPI-enabled) FigureGen for use on local Linux/Unix machines.
- `figuregen-tacc.def` is compatible with TACC or TACC-like HPC systems.

In addition, we maintain a [FigureGen folder](https://cloud.sylabs.io/library/_collection/602041171e573cd09be5c019) at the [Singularity Container Library](https://cloud.sylabs.io/library).

### Pre-Built Singularity Images

Running singularity containers assumes **your current working directory is where your ADCIRC datafiles are**.
By default, singularity mounts your current working directory and `$HOME` so we have direct file access.

To download the image from the cloud library (e.g., to the root `FigureGen` folder of this repository, or to some other
convenient location for singularity images), run the command:

```bash
singularity pull figuregen.sif library://georgiastuart/figuregen/figuregen
```

This command will pull down the latest `figuregen` image located at the cloud library and save it locally to `figuregen.sif`. Similarly, replace `figuregen` with `figuregen-serial` or `figuregen-tacc` to run the serial or TACC versions of `figuregen`, respectively.

```bash
singularity pull figuregen-serial.sif library://georgiastuart/figuregen/figuregen-serial
singularity pull figuregen-tacc.sif library://georgiastuart/figuregen/figuregen-tacc
```

### Building Singularity Images

To build a Singularity image yourself, from the root `FigureGen` directory of this repository, use this command *on a machine where you have sudo privileges*:

```
sudo singularity build figuregen.sif container-files/singularity-def-files/figuregen.def
```

Replace `figuregen.def` with `figuregen-serial.def` or `figuregen-tacc.def` as appropriate.

### Running Singularity Containers

-------------------------
**NOTE**

To run on TACC machines (excluding Lonestar 5), you must load the module:

```
module load tacc-singularity
```
**Do not run Singularity on a TACC login node.
Request an idev node or include in a slurm script**.

To use an `idev` node on `Stampede2`, run

```
idev -A <account number> -N 1 -n <processes (at least 2)>
```

**At least two processes must be specified to run the TACC-compatible singularity container.** It *cannot* run serially.

------------------------

To run the tests in this repository, `cd` to `autotest/TestFiles` and run the following
command (for example):

#### MPI Example
```bash
singularity exec <path to figuregen.sif> mpirun -np <num processes> figuregen -I ../Tests/BathyFilledCPT.inp
```

#### Serial Example

```bash
singularity exec <path to figuregen-serial.sif> figuregen -I ../Tests/BathyFilledCPT.inp
```

#### TACC/HPC Example

For example, on Stampede2. **DO NOT RUN ON LOGIN NODE.**

```bash
ibrun singularity exec <path to figuregen-tacc.sif> figuregen -I ../Tests/BathyFilledCPT.inp
```

**Note:** The TACC container gives some harmless errors that can be safely ignored.


## Docker Containers

For use on MacOS and Windows, we also provide Docker images. We maintain two Docker images for use on local machines (not HPC clusters. Use Singularity for HPC).

```
georgiastuart/figuregen
georgiastuart/figuregen-serial
```

### Install Docker

Mac instructions: https://docs.docker.com/docker-for-mac/install/

Windows instructions: https://docs.docker.com/docker-for-windows/install/

### Run FigureGen Containers

Running `FigureGen` in a Docker container requires four steps:
1. Pull the desired Docker image (listed above)
2. Launch the container and bind the host directory of interest to the `/data` directory in the container
3. Execute `figuregen`, setting the working directory inside the container on the command line
4. Stop the docker container and remove it

Steps 2 and 3 can be set two different ways depending on whether FigureGen will be executed multiple times (more details in the description below). Steps 1 and 4 are the same for any launch and execution scenario.

#### Pull the Docker Image

First, pull the Docker image with the following command:
```
docker pull georgiastuart/figuregen
```
Similarly, `georgiastuart/figuregen-serial`.

This step normally only needs to be done once, the first time FigureGen is used on that platform. It would also have to be repeated if the image is updated on DockerHub.

#### Launch, Execute, Stop, Remove

There are two ways to use the FigureGen container. The first way, described in this section, is to launch the docker container in the same directory that holds the FigureGen controls parameter file (`*.inp`) and all the ADCIRC data files that it will need. In this case, you will be binding your desired data directory on the host machine to the `/data` directory in the container. Once execution is complete in this scenario, you will need to stop and remove the container before repeating this sequence in a different directory.

To start, use the following command (**assuming your current working directory is where your data and input files are**):

##### Mac/Linux Launch

```
docker run -d -it --name figuregen --mount type=bind,source="$(pwd)",target=/data georgiastuart/figuregen
```

##### Windows Launch (Command Prompt)

```
docker run -d -it --name figuregen --mount type=bind,source="%cd%",target=/data georgiastuart/figuregen
```

##### Windows Launch (Power Shell)

```
docker run -d -it --name figuregen --mount type=bind,source="${PWD}",target=/data georgiastuart/figuregen
```

Similarly for `georgiastuart/figuregen-serial`.

##### Execute FigureGen

Next, we will execute the `FigureGen` program and set the working directory to `/data`:

```
docker exec -it -w /data figuregen mpirun -np <num processes> figuregen -I <input file name>
```
To run FigureGen in serial, the executable is named `figuregen` although the container is named `figuregen-serial`:
```
docker exec -it -w /data figuregen-serial figuregen -I <input file name>
```

##### Stop and Remove the Container

Finally, after your plots are satisfactory, we shut down and remove the Docker container. **A new container must be launched with 'docker run' each time you want to plot in a new directory**.

```
docker stop figuregen
docker container rm figuregen
```
#### Launch, Multiple Executions

A second way to use the FigureGen container is to specify a "data root" directory on the host machine when running it (e.g., `/home/myname/alldata`). The idea is that this directory contains multiple subdirectories with ADCIRC data and associated FigureGen control parameter files (`*.inp`). Then FigureGen can be executed more than once, with data in different subdirectories, without shutting down, removing, and relaunching the container in betweeen executions in different directories. This second method is described in this section.

To start, use the following command (**assuming the /home/myname/alldata (or c:\myname\alldata) directory contains subdirectories where your data and input files are**):

##### Mac/Linux Launch

```
docker run -d -it --name figuregen --mount type=bind,source="/home/myname/alldata",target=/data georgiastuart/figuregen
```

##### Windows Launch

```
docker run -d -it --name figuregen --mount type=bind,source="c:\myname\alldata",target=/data georgiastuart/figuregen
```

Similarly for `georgiastuart/figuregen-serial`.

##### Execute FigureGen

Next, we will execute the `FigureGen` program multiple times and specify the subdirectory containing the data each time. If we assume that there are three subdirectories and that they are named according to the pattern `run[n]`, for example `/home/myname/alldata/run1` (or `c:\myname\alldata\run1`):

```
docker exec -it --workdir=/data/run1 figuregen mpirun -np <num processes> figuregen -I <input file name>
docker exec -it --workdir=/data/run2 figuregen mpirun -np <num processes> figuregen -I <input file name>
docker exec -it --workdir=/data/run3 figuregen mpirun -np <num processes> figuregen -I <input file name>
```
To run FigureGen in serial, the executable is named `figuregen` although the container is named `figuregen-serial`:
```
docker exec -it --workdir=/data/run1 figuregen-serial figuregen -I <input file name>
docker exec -it --workdir=/data/run2 figuregen-serial figuregen -I <input file name>
docker exec -it --workdir=/data/run3 figuregen-serial figuregen -I <input file name>
```

##### Stop and Remove the Container

Finally, after your plots are satisfactory, and you don't have any more plots to make, you can shut down and remove the Docker container. **A new container must be launched with 'docker run' each time you want to plot in a new 'data root' directory**.

```
docker stop figuregen
docker container rm figuregen
```

## Tests Completed:

1. Tested `figuregen.sif / figuregen-serial.sif` on Ubuntu 20.10 Host Machine (7 Feb 2021)
2. Tested `figuregen-tacc.sif` on TACC's Stampede2 on idev node (7 Feb 2021)
3. Tested `georgiastuart/figuregen` / `georgiastuart/figuregen-serial` Docker container on MacOS 10.15.7 (7 Feb 2021)
4. Tested `georgiastuart/figuregen` Docker container on Windows 10 (7 Feb 2021)
