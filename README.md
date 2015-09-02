[title]: - "Basics of compiled MATLAB applications - Hello World example"  
[TOC]
 
 
## Overview

[MATLAB®](http://www.mathworks.com/products/matlab/) is a licensed high level language and interactive modeling and development toolkit. The [MATLAB Compiler™](http://www.mathworks.com/products/compiler/) lets you share MATLAB programs as standalone applications.  All applications created with MATLAB Compiler use [MATLAB Runtime™ (MCR)](http://www.mathworks.com/products/compiler/mcr/), which enables royalty-free deployment and use.  

MATLAB Compiler is invoked with `mcc`.  Most toolboxes and user-developed interfaces are supported. For more details, check the list of [supported toolboxes](http://www.mathworks.com/products/compiler/supported/compiler_support.html) and 
[ineligible programs](http://www.mathworks.com/products/ineligible_programs/). 

OSG Connect has several MATLAB releases installed, and MATLAB Runtime is available on all OSG sites using the OASIS software service using `module` commands.

In this tutorial we learn the basics of compiling MATLAB programs on a licensed machine and running compiled binaries using MCR on the the OSG.

### MATLAB script: `hello_world.m` 

Lets start with a simple MATLAB script `hello_world.m` that prints `Hello World!` to standard output. 
    
    function helloworld
        fprintf('\n=============')
        fprintf('\nHello, World!\n')
        fprintf('=============\n')
    end  

### Compilation 

On a machine with a MATLAB license, invoke the compiler `mcc`. We turn off all graphical options (`-nodisplay`), disable Java (`-nojvm`), and instruct MATLAB to run this application as a single-threaded application (`-singleCompThread`):

    mcc -m -R -singleCompThread -R -nodisplay -R -nojvm hello_world.m

The flag `-m` means C language translation during compilation, and the flag `-R` indicates runtime options.  The compilation would produce the files: 

   `hello_world, run_hello_world.sh, mccExcludedFiles.log` and `readme.txt`

The file `hello_world` is the compiled binary file and `run_hello_world.sh` is the script file that executes the binary. `mccExcludedFiles.log` is the log file and `readme.txt` contains the information about the compilation process. 

## Running standalone binary applications on OSG

To see which releases are available on OSG:

    $ ssh username@login.osgconnect.net   # login on OSG connect login node
    $ module avail matlab
    
    --------------------------------- /cvmfs/oasis.opensciencegrid.org/osg/modules/modulefiles/Core -----------------------------------------
    matlab/2013b    matlab/2014a    matlab/2014b    matlab/2015a (D)

    Where:
     (D):  Default Module

    Use "module spider" to find all possible modules.
    Use "module keyword key1 key2 ..." to search for all possible modules matching any of the "keys".


### Tutorial files

Let us say you have created the standalone binary `hello_world`. Transfer the files `hello_world` and `run_hello_world.sh` to login.osgconnect.net. You may also use the readily available files on login.osgconnect.net via `tutorial` command: 

$ tutorial matlab-HelloWorld # Copies input and script files to the directory tutorial-matlab-HelloWorld.
 
This will create a directory `tutorial-matlab-HelloWorld`. Inside the directory, you will see the following files
   
    hello_world             # compiled executable binary of hello_world.m
    hello_world.submit      # condor job description file
    run_hello_world.sh      # execution script

### Executing the MATLAB application binary

The shell script `run_hello_world.sh` executes the binary `hello_world`. It takes the path of the MATLAB runtime as an 
input argument. The supplied `hello_world` binary is compiled on a Linux machine with MATLAB 2015a. This means we need to have the same MATLAB Runtime to execute the binary. The MATLAB runtime for 2014b version is located in the path  `/cvmfs/oasis.opensciencegrid.org/osg/modules/matlab/2014b/v84/`

On the terminal prompt, type

    $ ./run_hello_world.sh /cvmfs/oasis.opensciencegrid.org/osg/modules/matlab/2014b/v84/
    (would produce the following output)

    ------------------------------------------
    Setting up environment variables
    ---
    LD_LIBRARY_PATH is .:/cvmfs/oasis.opensciencegrid.org/osg/modules/matlab/2014b/v84//runtime/glnxa64:/cvmfs/oasis.opensciencegrid.org/osg/modules/matlab/2014b/v84//bin/glnxa64:/cvmfs/oasis.opensciencegrid.org/osg/modules/matlab/2014b/v84//sys/os/glnxa64:/cvmfs/oasis.opensciencegrid.org/osg/modules/matlab/2014b/v84//sys/opengl/lib/glnxa64
    
    =============
    Hello, World!
    =============

If you get the above output, the binary execution is successful. Next, we see how we submit the job to a remote worker machine 
using HTcondor. 

### Job execution and submission files

Let us take a look at `hello_world.submit` file: 

    Universe = vanilla                          # One OSG Connect vanilla, the preffered job universe is "vanilla"

    Executable =  run_hello_world.sh    # Job execution file which is transffered to worker machine
    Arguments = /cvmfs/oasis.opensciencegrid.org/osg/modules/matlab/2014b/v84/ # path of matlab runtime libraries 
    transfer_input_files = hello_world  # list of file(s) need be transffered to the remote worker machine 

    Output = Log/job.$(Process).out⋅            # standard output 
    Error =  Log/job.$(Process).err             # standard error
    Log =    Log/job.$(Process).log             # log information about job execution

    requirements = HAS_CVMFS_oasis_opensciencegrid_org =?= True   # Check if the worker machine has CVMFS 

    queue 10                                     # Submit 10  jobs


Before job submission, we need to make sure that the directory `Log` exists on the current working directory. Because HTcondor looks for `Log` directory to copy the standard output, error and log files as specified in the job description file. 

From your work directory, type

    $ mkdir -p Log

Absence of `Log` directory would send the jobs to held state. 

### Job submmision 

We submit the job using `condor_submit` command as follows

	$ condor_submit hello_world.submit //Submit the condor job description file "wigner_distribution.submit"

Now you have submitted the an ensemble of 10 MATLAB jobs printing `hello world` on the standard output. You can check the status of the submitted job by using the `condor_q` command as follows

	$ condor_q username  # The status of the job is printed on the screen. Here, username is your login name.


### Job outputs 

The `hello_world.m` script sends the output to standard output. In the condor job description file, we expressed that the standard output is written on the `Log/job.$(ProcessID).out`. After job completion, ten output files are produced with the `hello world` message under the directory `Log`. 

## What's next? 
Sure, it is not very exciting to print same message on 10 output files. In the subsequent MATLAB examples,  we see  how to scale up MATLAB computation on HTC environment. 

## Getting help
For assistance or questions, please email the OSG User Support team  at [user-support@opensciencegrid.org](mailto:user-support@opensciencegrid.org) or visit the [help desk and community forums](http://support.opensciencegrid.org).
