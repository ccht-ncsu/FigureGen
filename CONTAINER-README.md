# Using the FigureGen Containers

Contact: Georgia Stuart, georgia.stuart@austin.utexas.edu
7 February 2021

## Input File Setup

For both Singularity and Docker container use, the path option for `ghostscript` and `gmt`
**must be left blank**. See `autotest/docker-test/BathyFilledCPT.inp` for example.

## Singularity Containers

[Singularity](https://sylabs.io/docs/) is a container platform designed for use with HPC systems.
We provide three Singularity definition files:
- `figuregen.def` provides MPI-enabled FigureGen for use on local Linux/Unix machines.
- `figuregen-serial.def` provides serial (not MPI-enabled) FigureGen for use on local Linux/Unix machines.
- `figuregen-tacc.def` is compatible with TACC or TACC-like HPC systems.

In addition, we maintain a [FigureGen folder](https://cloud.sylabs.io/library/_collection/602041171e573cd09be5c019) at the [Singularity Container Library](https://cloud.sylabs.io/library).

### Using pre-built Singularity containers from the library

Running singularity containers assumes **your current working directory is where your ADCIRC datafiles are**.
By default, singularity mounts your current working directory and `$HOME` so we have direct file access.

To download the image from the cloud library (e.g., to the root `FigureGen` folder of this repository, or to some other
convenient location for singularity images), run the command:

```bash
singularity pull figuregen.sif library://georgiastuart/figuregen/figuregen
```

This command will pull down the latest `figuregen` image located at the cloud library and save it locally
to `figuregen.sif`. Similarly, replace `figuregen` with `figuregen-serial` or `figuregen-tacc` to run the
serial or TACC versions of `figuregen`, respectively.

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

**At least two processes must be specified to run the TACC-compatible singularity container.** It *cannot*
run serially.

------------------------

```bash
singularity pull figuregen-serial.sif library://georgiastuart/figuregen/figuregen-serial
singularity pull figuregen-tacc.sif library://georgiastuart/figuregen/figuregen-tacc
```

To run the tests in this repository, `cd` to `autotest/TestFiles` and run the following
command (for example):

##### MPI Example
```bash
singularity exec <path to figuregen.sif> mpirun -np <num processes> figuregen -I ../Tests/BathyFilledCPT.inp
```

##### Serial Example

```bash
singularity exec <path to figuregen-serial.sif> figuregen -I ../Tests/BathyFilledCPT.inp
```

##### TACC/HPC Example (e.g. on Stampede2. DO NOT RUN ON LOGIN NODE.)

```bash
ibrun singularity exec <path to figuregen-tacc.sif> figuregen -I ../Tests/BathyFilledCPT.inp
```

**Note:** The TACC container gives some harmless errors that can be safely ignored.

### Building Singularity Containers

To build a Singularity container yourself, from the root `FigureGen` directory of this repository,
use this command *on a machine where you have sudo privileges*:

```
sudo singularity build figuregen.sif container-files/singularity-def-files/figuregen.def
```

Replace `figuregen.def` with `figuregen-serial.def` or `figuregen-tacc.def` as appropriate.

## Docker Containers

For use on MacOS and Windows, we also provide Docker containers. **All docker container instructions
rely on your data files and input files being in the same directory** (e.g., the `autotest/docker-test` directory in this
repository).

We maintain two Docker images for use on local machines (not HPC clusters. Use Singularity for HPC).

```
georgiastuart/figuregen
georgiastuart/figuregen-serial
```

### Install Docker

Mac instructions: https://docs.docker.com/docker-for-mac/install/
Windows instructions: https://docs.docker.com/docker-for-windows/install/

### Run FigureGen Containers

Running `FigureGen` in a Docker container requires four steps:
1. Pull the desired Docker container (listed above)
1. Launch the container and bind the directory with your data and input files to `/data`
2. Execute the `figuregen` executable and set working directory to `/data`
4. Shut the docker container down

#### Pull the Docker Image

First, pull the Docker image with the following command:

```
docker pull georgiastuart/figuregen
```

Similarly, `georgiastuart/figuregen-serial`.

#### Launch the Docker Container

We need to launch the docker container while binding your desired data directory to `/data`
in the container. Use the following command (**assuming your current working directory is
where your data and input files are**):

##### Mac/Linux

```
docker run -d -it --name figuregen --mount type=bind,source="$(pwd)",target=/data georgiastuart/figuregen
```

##### Windows (Command Prompt)

```
docker run -d -it --name figuregen --mount type=bind,source="%cd%",target=/data georgiastuart/figuregen
```

##### Windows (Power Shell)

```
docker run -d -it --name figuregen --mount type=bind,source="${PWD}",target=/data georgiastuart/figuregen
```

Similarly for `georgiastuart/figuregen-serial`.

#### Run the FigureGen Program

Next, we will run the `FigureGen` program and set the working directory to `/data`:

```
docker exec -it -w /data figuregen mpirun -np <num processes> figuregen -I <input file name>
```
To run FigureGen in serial, the executable is named `figuregen` although the container is named `figuregen-serial`:
```
docker exec -it -w /data figuregen-serial figuregen -I <input file name>
```

#### Shut Down the Container

Finally, after your plots are satisfactory, we shut down and remove the Docker container. **A new container must
be launched each time you want to plot in a new directory**.

```
docker stop figuregen
docker container rm figuregen
```

## Tests Completed:

1. Tested `figuregen.sif / figuregen-serial.sif` on Ubuntu 20.10 Host Machine (7 Feb 2021)
2. Tested `figuregen-tacc.sif` on TACC's Stampede2 on idev node (7 Feb 2021)
3. Tested `georgiastuart/figuregen` / `georgiastuart/figuregen-serial` Docker container on MacOS 10.15.7 (7 Feb 2021)
4. Tested `georgiastuart/figuregen` Docker container on Windows 10 (7 Feb 2021)
