.. _srw_workflow:
=================================
SRW Container Workflow
=================================

.. **Overview**::

As part of the Advance User Support Sprint 2 Planning deliverables, a minimalistic containerized version of the Unified Forecasting System (UFS) Short Range Weather (SRW) application was requested. 

There are a number of acceptance criteria set for this deliverable and as a result the containerized version of the SRW application would need to run on a high performance computer (HPC). As with the Medium Range Weather application a low resolution dataset with a shortened forecast period (12 hrs) was used.

Because the SRW weather application needs to run on a HPC, the end user would need to have access to a NOAA HPC. In our case, we had access to Orion. And in this example, only a Singularity container was proved which had the SRW application pre-installed and built using the HPC-Stack repository. 

This document will aid novice users in running the UFS SRW application with the intent of helping them become more familiar with the UFS and its applications. The document is broken into different sections so that it is easier for a user to understand what is required to run the SRW Container. The container itself is currently a proof of concept and significant work remains to ensure that the process of building containers that are offered to the community are fully in sync with the UFS software in use on the cloud and Tier-1 platforms.

 
.. **Prerequisites**:

There are a couple of things one must have before running the SRW application container. Below is a list of these items:
- Access to Orion (this workflow will only work on Orion as this is where the data is hosted)
- The SRW Singularity image
- Workflow instructions (found in this document under the Running the Workflow section)

.. **Running the Workflow**:

Once you have the proper access to the Orion HPC, you may run this workflow below. 
1. Log into Orion using the command below using your username:

a. .. code-block:: console

ssh -x username@Orion-login.hpc.msstate.edu

2. Run the following commands to download the SRW Singularity Image and convert it to a sandbox.

a. .. code-block:: console

cd /work/noaa/epic-ps/$USER

i. NOTE: if your $USER doesn’t exist, you may create it by replacing the ‘cd’ with ‘mkdir -p’ in the command above.

b. .. code-block:: console module load singularity

module load singularity 
singularity pull library://dcvelobrew/default/ubuntu-hpc-stack:latest
singularity build --sandbox ubuntu-hpc-stack-src ./ubuntu-hpc-stack_latest.sif

3. After the Singularity sandbox is complete, run the following commands to partition resources from Orion so that the SRW forecast application can run. NOTE: this process can take up to two hours to complete! 

.. code-block:: console

salloc -N 1 -n 40 -A epic-ps -q batch -t 2:30:00 --partition=orion

4. Run the command below to check if any resources have been assigned to your user.

.. code-block:: console

squeue -u $USER

If resources have been assigned to your user the output will look something like this:
JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
3500248     orion     bash  esnyder  R    1:22:27      1 Orion-23-03

5. Once resources have been allocated to your user, use the NODELIST to ssh into your node. Note, the node won’t have any connectivity to the internet.

.. code-block:: console

ssh Orion-##-##

In the example above, the command would be: ssh Orion-23-03
6. When you are on the Node run the following commands that will setup your environment so that you can run the Singularity container

.. code-block:: console

module load singularity
cd /work/noaa/epic-ps/$USER
cp ~/.bashrc .
singularity shell -H $PWD:$PWD -B /work:/work -w ./ubuntu-hpc-stack-src
bash
export PATH=$PATH:/home/builder/opt/bin:/home/builder/opt/miniconda/bin
conda init bash
exit

7. Now that your Singularity environment is setup, let’s configure the setup for the SRW now by running the following commands:

.. code-block:: console

Bash
export PATH=$PATH:/home/builder/opt/bin:/home/builder/opt/miniconda/bin
export PATH=$PATH:/home/builder/opt/bin
conda activate regional_workflow
cd /home/builder/ufs/ufs-srweather-app/regional_workflow/ush


8. In the ush directory, you can modify your EXPT_SUBDIR in the config.sh. This is the experiment directory, where the UFS Weather Model output files will be written to. To modify this directory run this command:

.. code-block:: console

vi config.sh

9. After the EXPT_SUBDIR field has been modified in the config.sh file, generate the workflow by doing the following:

.. code-block:: console

./generate_FV3LAM_wflow.sh
cd /home/builder/ufs/expt_dirs/EXPT_SUBDIR
NOTE: EXPT_SUBDIR is the field set in the config.sh from the previous step.
cp /home/builder/ufs/ufs-srweather-app/regional_workflow/ush/wrappers/* .
export EXPTDIR=$PWD
source ./var_defns.sh

10. Now you are ready to run the SRW forecast application workflow. The workflow has been broken down into individual scripts. Please run these scripts in order.

.. code-block:: console

./run_get_ics.sh
./run_get_lbcs.sh
./run_make_grid.sh
./run_make_orog.sh
./run_make_sfc_climo.sh
./run_make_ics.sh
./run_make_lbcs.sh
./run_fcst.sh
./run_post.sh

11. Resulting Output
The final output should look something like this. And the SRW weather model files can be found here:
