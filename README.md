Steps to Run Coda Correlation Computations
----

Here we introduce how to configure programs/files, how to run coda correlation computations using the package [sacpy](https://github.com/sheng09/sacpy), and how to plot the generated correlograms, etc.

# Environment Configurations

## Settings for RSES compute2 users.
If you can access the server `compute2` at RSES, then configurations will be easy for you. Follow the steps here to configure your environment. Please note, you just need to configure for once, and then everything will be functional to you for always.

1. Configure your enviroment path. In your compute2 home folder, edit the file `~/.bash_profile` by appending the following settings to it:

    ```bash
    # SACPY by Sheng
    export PYTHONPATH=/home/seis/sheng/Program_Sheng/python3-package-sheng:${PYTHONPATH}
    export SACPYHOME=/home/seis/sheng/Program_Sheng/python3-package-sheng/sacpy
    export PATH=${SACPYHOME}/bin:${PATH}

    # sac
    export SACHOME=/home/seis/sheng/software/sac
    source ${SACHOME}/bin/sacinit.sh

    #HTOP
    export PATH="/home/seis/sheng/software/htop/bin:${PATH}"
    ```

2. Run the commands below to enable the settings:

    ```
    source ~/.bash_profile
    ```

## In your environment
You can download and install [sacpy](https://github.com/sheng09/sacpy) on your machines. 

Manuals coming soon....



# Computing coda correlations in parallel

We use an executetable files `cc_stack_v2.py` to do the computation. It is within the package [sacpy](https://github.com/sheng09/sacpy). After configuring your environment, especially setting your PATH, you can call it in anywhere without specifying the absolute path to it.

## Examples on compute2

If you can access the server `compute2` at RSES, then there are examples at `/home/seis/sheng/Lucie-Example/`. 

1. Inside the folder, `archive_beyond_21events_ALL_6.8+` is virtual link to our dataset. Inside the `archive_beyond_21events_ALL_6.8+`, you can find `201?/*.a/processed_aligned_ot/*BHZ`, and those are records in SAC format. Each single file is an individual time series at a single station for a single event, and the time series is for records in 0-10 hrs after the event's origin.

2. Also, inside the folder, `archive_beyond_21events_ALL_6.8+/h5vol/*.h5` store the same data but in different format. The `*.h5` files are in hdf5 format. Each of the h5 files corresponds to a single event, and a single h5 file contains multiple records at many stations. Therefore, we only have ~200 h5 files instead of millions of sac file, and that can accelerate I/O of our computations. Please note, both the sac format files and the h5 format files can be used at the input of our package [sacpy](https://github.com/sheng09/sacpy).

3. `run.sh` is an example for running the computation. Its use sac format as input. It generates outputs to the folder `out/`. You can view how we use the `cc_stack_v2.py` with many args. You can find all available args and related explanations simply by running `cc_stack_v2.py` alone without appending any args. To make it easy, a simple explanation is below. Please note, the `orterun -np 6` or `mpirun -np 6` proceding the command are for mpi parallel running, and obviously we use 6 procs in the examples. 

    ```bash
    orterun -np 6 \                  # use mpi parallel running with 6 procs
    cc_stack_sac.py \                
     -I "archive_beyond_21events_ALL_6.8+/201[01]/20*.a/processed_aligned_ot/*BHZ" \ # input wildcards, make sure use ""
     -T -5/10800/32400 -D 0.1 \                                                      # time window and sampling time interval
     -O out/cc --out_format hdf5,sac --log out/log \                                 # where to output, output format, and where to output logs
     --pre_detrend --pre_taper 0.005 \                                               # pre-processing parameters
     --stack_dist 0/180/1 --daz -0.1/20 --gcd_ev -0.1/30 \                           # stacking methods and selections of station pairs
     --w_temporal 128.0/0.02/0.06667 --w_spec 0.02 \                                 # whitening parameters for global coda correlation computations
     --post_fold --post_taper 0.005 --post_filter bandpass/0.01/0.1 --post_norm      # post-processing for the obtained correlatin stacks

    ```

4. `run_fast.sh` is similar to `run.sh` but take h5 format inputs, and hence it is much faster. It generates outputs to the folder `out_fast/`.





