---
title: "Get familiar with cesm"
teaching: 0
exercises: 0
questions:
- "How to setup CESM on abel?"
- "How to run a cesm case?"
- "How to monitor my cesm case?"
objectives:
- "Learn to setup cesm on abel"
- "Learn to run and monitor a simple cesm case on abel"
keypoints:
- "cesm"
- "High-Performance Computing"
- "abel"
- "slurm"
---


# First practical: get familiar with cesm

<img src="../fig/practicals.jpg">

We do all the practicals on <font color="red">Abel</font>.  

*   [Notur Initialization](#notur-initialization)
*   [Create a New case](#create-a-new-case)
*   [Running a case](#running-a-case)
*   [Monitor your test run](#monitor-your-test-run)
*   [Check the 1 month test run](#check-the-1-month-test-run)
*   [Move your files on NIRD](#move-your-files-on-nird).


### Notur Initialization

Make sure you have set-up your SSH keys properly and you can transfer files with scp without entering your password. If not go [here](http://www.mn.uio.no/geo/english/services/it/help/using-linux/ssh-tips-and-tricks.html).  

To run CAM-5.3 on abel, we will use:

*   <font color="green">Subversion client (version 1.6.11) to get CESM source code</font>
*   Fortran and C compilers (intel 2015.0 compilers)
*   NetCDF library (netcdf4.3.3.1)
*   MPI (intel openmpi 1.8.3)

To be able to compile and run CESM on abel, no changes to the source code were necessary; we just had to adapt a few scripts for setting the compilers and libraries used by CESM.  
To simplify and allow you to run CESM as quickly as possible, we have prepared a set-up script geo4962_notur.bash.  

<font color="red">On Abel:</font>  

<pre>cd $HOME

git clone https://github.uio.no/annefou/GEO4962.git
cd $HOME/GEO4962/setup

./geo4962_notur.bash
</pre>

The script above copies the source code in $HOME/cesm/cesm_1_2_2 and creates symbolic links for the input data necessary to run our model configuration in /work/users/$USER/inputdata. Input data can be large this is why we create symbolic links instead of making several copies (one per user). The main copy is located in /work/users/annefou/public/inputdata.  

### Create a New case

Now that you have the CESM source code in $HOME/cesm/cesm_1_2_2, you can have a first look at the code.  
![](../fig/tree_source.png)  
We will build and run CAM in its standalone configuration i.e. without all the other components.  
The basic workflow to run the CESM code is the following:

*   Create a New Case
*   Invoke cesm_setup
*   Build the Executable
*   Run the Model and Output Data Flow

To create a new case, we will be using create_newcase script. It is located in $HOME/cesm/cesm1_2_2/scripts.  
There are many options and we won't discuss all of them. To get the full usage of create_newcase (<font color="red">On Abel</font>):  


<pre>./create_newcase --help
</pre>

The 4 main arguments of create_newcase are explained on the figure below: ![](../fig/newcase.png)  

<font color="red">On Abel:</font>


<pre>cd $HOME/cesm/cesm1_2_2/scripts

#
# Simulation 1: short simulation
#
module load cesm/1.2.2

./create_newcase -case ~/cesm_case/f2000.T31T31.test -res T31_T31 -compset F_2000_CAM5 -mach abel
</pre>



*   **case**: specifies the name and location of the case being created. It creates a new case in $HOME/cesm_case and its name is f2000.T32T31.test
*   **res**: specifies the model resolution (resolution of the grid). Each model resolution can be specified by its alias, short name or long name:
    *   alias: T31_T31 (atm/lnd_ocn/ice)
    *   short name: T31_T31
    *   long name: a%T31_l%T31_oi%T31_r%r05_m%gx3v7_g%null_w%null (atm,lnd,ocn/ice,river,lnd mask, lnd-ice,wave)  
    The full list of supported grid is given [here](http://www.cesm.ucar.edu/models/cesm1.2/cesm/doc/modelnl/grid.html).
*   **compset**: specifies the component set, i.e., component models, forcing scenarios and physics options for those models.  
    As for the resolution, the component set can be specified by its alias, short name or long name:
    *   alias: FC5
    *   short name: F_2000_CAM5
    *   long name: 2000_CAM5_CLM40%SP_CICE%PRES_DOCN%DOM_RTM_SGLC_SWAV  
    The notation for the compset longname is:  
    `

    <pre>   TIME_ATM[%phys]_LND[%phys]_ICE[%phys]_OCN[%phys]_ROF[%phys]_GLC[%phys]_WAV[%phys][_BGC%phys]
    </pre>

    The compset longname has the specified order:  
    **atm, lnd, ice, ocn, river, glc wave cesm-options**  
    Where:

    <pre>   TIME = Time period (e.g. 2000, 20TR, RCP8...)
       ATM  = [CAM4, CAM5, DATM, SATM, XATM]
       LND  = [CLM40, CLM45, DLND, SLND, XLND]
       ICE  = [CICE, DICE, SICE, SICE]
       OCN  = [POP2, DOCN, SOCN, XOCN,AQUAP,MPAS]
       ROF  = [RTM, DROF, SROF, XROF]
       GLC  = [CISM1, SGLC, XGLC]
       WAV  = [WW3, DWAV, SWAV, XWAV]
       BGC  = optional BGC scenario
    </pre>

    The OPTIONAL %phys attributes specify submodes of the given system  
    The list of available component set is given [here](http://www.cesm.ucar.edu/models/cesm1.2/cesm/doc/modelnl/compsets.html).  
    In our case we have:
    *   TIME = 2000: we are running our model for present days
    *   ATM = CAM5: we will be using CAM5 for the atmospheric component
    *   LND = CLM40%SP: Community Land Model (CLM) 4 with prescribed satellite phenology
    *   ICE = CICE%PRES: we will be running CESM with prescribed cice (Community Ice CodE)
    *   OCN = DOCN%DOM: Climatological Data Ocean Model (DOCN) with Data Ocean mode (see more [here](http://www.cesm.ucar.edu/models/ocn-docn/docn4.0/userguide.html))
    *   ROF = RTM: we will be using the default [River Transport Model](http://www.cesm.ucar.edu/models/cesm1.2/rtm/) (RTM) model
    *   GLC = SGLC: stub land-ice Model
    *   WAV = SWAV: stub ocean-wave model  

*   **mach**: specifies the machine where CESM will be compiled and run. We will be running CESM on abel (a set of scripts for abel can be found in $HOME/cesm/cesm1_2_2/scripts/ccsm_utils/Machines)

Now you should have a new directory in $HOME/cesm_case/f2000.T31T31.test corresponding to our new case (<font color="red">On Abel:</font>)


<pre>cd ~/cesm_case/f2000.T31T31.test
</pre>

Check the content of the directory and browse the sub-directories:  
![](../fig/casedir_test.png)  
For this tests (and all our simulations), we do not wish to have a "cold" start and we will therefore restart and continue an existing simulation we have previously run.  

<font color="red">On Abel:</font>


<pre>./xmlchange RUN_TYPE=hybrid
./xmlchange RUN_REFCASE=f2000.T31T31.control
./xmlchange RUN_REFDATE=0009-01-01
</pre>

We use xmlchange, a small script to update variables (such as RUN_TYPE, RUN_REFCASE, etc.) defined in xml files. All the xml files contained in your test case directory will be used by cesm_setup to generate your configuration setup (Fortran namelist, etc.):  

<font color="red">On Abel</font>:  

<pre>ls *.xml

</pre>

To change the duration of our test simulation and set it to 1 month:`

<pre>./xmlchange -file env_run.xml -id STOP_N -val 1
./xmlchange -file env_run.xml -id STOP_OPTION -val nmonths
</pre>

Now we are ready to set-up our model configuration and build the cesm executable.  

<font color="red">On Abel:</font>

<pre>./cesm_setup

./f2000.T31T31.test.build

</pre>
 
After building CESM for your configuration, a new directory (and a set of sub-directories) are created in /work/users/$USERS/f2000.T31T31.test:

*   **bld**: contains the object and CESM executable for your configuration
*   **run**: this directory will be used during your simulation run to generate output files, etc.

### Running a case

Namelists can be changed before configuring and building CESM but it can also be done before running your test case. Then, you cannot use xmlchange and update the xml files, you need to directly change the namelist files.  
The default history file from CAM is a monthly average. We can change the output frequency with the namelist variable **nhtfrq**

*   If nhtfrq=0, the file will be a monthly average
*   If nhtfrq>0, frequency is input as number of timesteps.
*   If nhtfrq<0, frequency is input as number of hours.

For instance to change the history file from monthly average to daily average, we set the namelist variable nhtfrq = -24\. We also need to do the following changes (to copy restart files in your running directory, etc.):  

<font color="red">On Abel:</font>

We need to add two lines to the CAM5 namelist (called **user_nl_cice**):

<font color="red">On Abel:</font> 

    cat >> user_nl_cice << EOF
    grid_file = '/work/users/$USER/inputdata/share/domains/domain.ocn.48x96_gx3v7_100114.nc'
    kmt_file = '/work/users/$USER/inputdata/share/domains/domain.ocn.48x96_gx3v7_100114.nc'
    EOF

**[cat](http://www.linfo.org/cat.html)** is a unix shell command to display the content of files or combine and create files. Using >> followed by a filename (here user_nl_cam) means we wish to concatenate information to a file. If it does not exist, it is automatically created. Using << followed by a string (here EOF) means that the content we wish to concatenate is not in a file but written after EOF until another EOF is found.  

Finally, we copy the control restart files (contains the state of the model at a given time so we can restart it). The files are stored on norStore; they were generated from a previous simulations where we have run the model for several years):  

<font color="red">On Abel:</font>

<pre>scp login.nird.sigma2.no:/projects/NS1000K/GEO4962/outputs/runs/f2000.T31T31.control/run/f2000.T31T31.control.*.0009-01-01-00000.nc  /work/users/$USER/f2000.T31T31.test/run/.
scp login.nird.sigma2.no:/projects/NS1000K/GEO4962/outputs/runs/f2000.T31T31.control/run/rpointer.* /work/users/$USER/f2000.T31T31.test/run/.
</pre>


Now we wish to run our model and as it may run for several days, we need to use the batch scheduler (SLURM) from abel. Its role is to dispatch jobs to be run on the cluster. It reads information given in your job command file (named here f2000.T31T31.test.run). This file contains information on the number of processors to use (ntasks), the amount of memory per processor (mem-per-cpu) and the maximum amount of time you wish to allow for your job (time).  

Check what is in your current job command file (f2000.T31T31.test.run):

<pre>#SBATCH --job-name=f2000.T31T31.test
#SBATCH --time=08:59:00
#SBATCH --ntasks=32
#SBATCH --account=nn1000k
#SBATCH --mem-per-cpu=4G
#SBATCH --cpus-per-task=1
#SBATCH --output=slurm.out
</pre>

The lines starting with **#SBATCH** are not comments but SLURM directives.  
You can submit your test case to <font color="red">abel</font>:

<pre>    
./f2000.T31T31.test.submit

</pre>


### Monitor your test run

The script "f2000.T31T31.test.submit" submits a job to the job scheduler on abel. More information can be found [here](http://www.uio.no/english/services/it/research/hpc/abel/help/user-guide/).

*   To monitor your job on <font color="red">abel:</font>

    <pre>squeue -u $USER
    </pre>

Full list of available commands and their usage can be found [here](http://www.uio.no/english/services/it/research/hpc/abel/help/user-guide/queue-system.html).

### Check the 1 month test run

<font color="red">On Abel</font>: During your test case run, CAM-5.3 generates outputs in the "run" directory:  

![](../fig/rundir_test.png)  
At the end of your run, the run directory will only contain files that are needed to continue an existing simulation but all the model outputs are moved to another directory (archive directory). On abel this directory is semi-temporary which means data will be automatically deleted after a short period of time.  
![](../fig/archivedir_test.png)  
Check your run was successful and generated all the necessary files you need for your analysis. <font color="red">On Abel:</font>

<pre>cd /work/users/$USER/f2000.T31T31.test/run
ls -lrt

cd /work/users/$USER/archive/f2000.T31T31.test/atm/hist
ls -lrt
</pre>

You should see a number of netCDF files (each of them ends with ".nc").  
You can quickly visualize your data on abel (to make sure your simulation ran OK): <font color="red">On Abel:</font>

<pre>cd /work/users/$USER/archive/f2000.T31T31.test/atm/hist
module load ncview
ncview f2000.T31T31.test.cam.h0.0001-01-31-00000.nc
</pre>

If you click on 3D or 4D to select a variable, your data should appear:  
![](../fig/ncview_D2.png)  
Here, PS (2D variable) is plotted.  

### Move your files on NIRD

First make sure your run was successful and check all the necessary output files were generated.  

To post-process and visualize your model outputs, it is VERY IMPORTANT you move them from Abel to norStore. Remember that all model outputs are generated in a semi-temporary directory and all your files will be removed after a few weeks!  
If you haven't set-up your SSH keys, the next commands (ssh and [rsync](http://www.tecmint.com/rsync-local-remote-file-synchronization-commands/)) will require you to enter your Unix password.  

<font color="red">On Abel:</font>

<pre>ssh login.nird.sigma2.no 'mkdir -p /projects/NS1000K/GEO4962/outputs/$USER/runs'
ssh login.nird.sigma2.no 'mkdir -p /projects/NS1000K/GEO4962/outputs/$USER/archive'

rsync -avz /work/users/$USER/f2000.T31T31.test $USER@login.nird.sigma2.no:/projects/NS1000K/GEO4962/outputs/$USER/runs/.

rsync -avz /work/users/$USER/archive/f2000.T31T31.test $USER@login.nird.sigma2.no:/projects/NS1000K/GEO4962/outputs/$USER/archive/.
</pre>

{% include links.md %}
