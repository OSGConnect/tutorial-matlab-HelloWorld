---
ospool:
    path: software_examples/matlab_runtime/tutorial-matlab-HelloWorld/README.md
---
 
# Basics of compiled MATLAB applications - Hello World example

[MATLAB®](http://www.mathworks.com/products/matlab/) is a licensed high level language and modeling toolkit. The [MATLAB Compiler™](http://www.mathworks.com/products/compiler/) lets you share MATLAB programs as standalone applications.  MATLAB Compiler is invoked with `mcc`. The compiler supports most toolboxes and user-developed 
interfaces. For more details, check the list of [supported toolboxes](http://www.mathworks.com/products/compiler/supported/compiler_support.html) 
and [ineligible programs](http://www.mathworks.com/products/ineligible_programs/).  


All applications created with MATLAB Compiler use [MATLAB Compiler Runtime™ (MCR)](http://www.mathworks.com/products/compiler/mcr/), which enables royalty-free deployment and use. We assume you have access to a server that has MATLAB compiler because the compiler is not available on OSG.  MATLAB Runtime is available 
on OSG. 

Although the compiled binaries are portable, they need to have a compatible, OS-specific matlab runtime to interpret the binary. We recommend the 
compilation of your matlab program against matlab versions that match the OSG [containers](https://portal.osg-htc.org/documentation/htc_workloads/using_software/available-containers-list/), with the compilation executed on a server with 
Scientific Linux so that the compiled binaries are portable on OSG machines.

In this tutorial, we learn the basics of compiling MATLAB programs on a licensed linux machine and running the 
compiled binaries using a matlab compiled runtime (MCR) in the OSG containers. 


### MATLAB script: `hello_world.m` 

Lets start with a simple MATLAB script `hello_world.m` that prints `Hello World!` to standard output. 
    
    function helloworld
        fprintf('\n=============')
        fprintf('\nHello, World!\n')
        fprintf('=============\n')
    end  

### Compilation 

*OSG does not have a license to use the MATLAB compiler*. On a Linux server with a MATLAB 
license, invoke the compiler `mcc`.  We turn off all graphical options (`-nodisplay`), disable Java (`-nojvm`), and instruct MATLAB to run this application as a single-threaded application (`-singleCompThread`):

    mcc -m -R -singleCompThread -R -nodisplay -R -nojvm hello_world.m

The flag `-m` means C language translation during compilation, and the flag `-R` indicates runtime options.  The compilation would produce the files: 

    `hello_world, run_hello_world.sh, mccExcludedFiles.log` and `readme.txt`

The file `hello_world` is the standalone executable. The file `run_hello_world.sh` is MATLAB generated shell script. `mccExcludedFiles.log` is the log file and `readme.txt` contains the information about the compilation process. We just need the standalone binary file `hello_world`. 

## Running standalone binary applications on OSG

To see which releases are available on OSG visit our available [containers](https://portal.osg-htc.org/documentation/htc_workloads/using_software/available-containers-list/) page :

### Tutorial files

Let us say you have created the standalone binary `hello_world`. Transfer the file `hello_world` to your Access Point. Alternatively, you may also use the readily available files by using the `git clone` command: 

    $ git clone https://github.com/OSGConnect/tutorial-matlab-HelloWorld # Copies input and script files to the directory tutorial-matlab-HelloWorld.
 
This will create a directory `tutorial-matlab-HelloWorld`. Inside the directory, you will see the following files
   
    hello_world             # compiled executable binary of hello_world.m
    hello_world.m           # matlab program
    hello_world.submit      # condor job description file
    hello_world.sh          # execution script

### Executing the MATLAB application binary

The compilation and execution environment need to the same. The file `hello_world` is a standalone binary of the matlab program `hello_world.m` which was compiled using MATLAB 2018b on a Linux platform. The Access Point and many of the worker nodes on OSG are based on Linux platform. In addition to the platform requirement, we also need to have the same MATLAB Runtime version. 

Load the MATLAB runtime for 2018b version via apptainer/singularity command.  On the terminal prompt, type

    $ apptainer shell /cvmfs/singularity.opensciencegrid.org/opensciencegrid/osgvo-matlab-runtime:R2018b

The above command sets up the environment to run the matlab/2018b runtime applications.  Now execute the binary

    $apptainer/singularity> ./hello_world
    (would produce the following output)

    =============
    Hello, World!
    =============

If you get the above output, the binary execution is successful. Now, exit from the apptainer/singularity environment typing `exit`. Next, we see how to submit the job on a remote execute point using HTCondor. 

### Job execution and submission files

Let us take a look at `hello_world.submit` file: 

    +SingularityImage = "/cvmfs/singularity.opensciencegrid.org/opensciencegrid/osgvo-matlab-runtime:R2018b"
    
    executable =  hello_world                

    Output = Log/job.$(Process).out⋅            # standard output 
    Error =  Log/job.$(Process).err             # standard error
    Log =    Log/job.$(Process).log             # log information about job execution
    
    queue 10                                    # Submit 10  jobs


Before we submit the job, make sure that the directory `Log` exists on the current working directory. Because HTCondor looks for `Log` directory to copy the standard output, error and log files as specified in the job description file. 

From your work directory, type

    $ mkdir -p Log

Absence of `Log` directory would send the jobs to held state. 

### Job submmision 

We submit the job using the `condor_submit` command as follows

	$ condor_submit hello_world.submit # Submit the condor job description file "hello_world.submit"

Now you have submitted an ensemble of 10 MATLAB jobs. Each job prints `hello world` on the standard 
output. Check the status of the submitted job,  

	$ condor_q username  # The status of the job is printed on the screen. Here, username is your login name.


### Job outputs 

The `hello_world.m` script sends the output to standard output. In the condor job description file, we expressed that the standard output is written on the `Log/job.$(ProcessID).out`. After job completion, ten output files are produced with the `hello world` message under the directory `Log`. 

## What's next? 
Sure, it is not very exciting to print the same message on 10 output files. In the subsequent MATLAB 
examples,  we see  how to scale up MATLAB computation on HTC environment. 
