Steps to Run Coda Correlation Computations
----

Here we introduce how to configure programs/files, how to run coda correlation computations using the package [sacpy](https://github.com/sheng09/sacpy), and how to plot the generated correlograms, etc.

# 1. Environment Configurations

## 1.1 Settings for RSES compute2 users.
If you can access the server `compute2` at RSES, then configurations will be easy for you. Follow the steps here to configure your environment. Please note, you just need to configure for once, and then everything will be functional to you for always.

a. Configure your enviroment path. In your compute2 home folder, edit the file `~/.bash_profile` by appending the following settings to it:

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

b. Run the commands below to enable the settings:

    ```
    source ~/.bash_profile
    ```

## 1.2 Settings for your own environments
You can download and install [sacpy](https://github.com/sheng09/sacpy) on your machines. 

Manuals coming soon....



# 2. Computing coda correlations in parallel

We use an executetable files `cc_stack_v2.py` to do the computation. It is within the package [sacpy](https://github.com/sheng09/sacpy). After configuring your environment, especially setting your PATH, you can call it in anywhere without specifying the absolute path to it.

## 2.1 Examples on compute2

If you can access the server `compute2` at RSES, then there are examples at `/home/seis/sheng/Lucie-Example/`. You can copy this folder or files within it to your directory, as you may dont have access to run files inplace.

a. Inside the folder, `archive_beyond_21events_ALL_6.8+` is virtual link to our dataset. Inside the `archive_beyond_21events_ALL_6.8+`, you can find `201?/*.a/processed_aligned_ot/*BHZ`, and those are records in sac format. Each single file is an individual time series at a single station for a single event, and all the time series are for records in 0-10 hrs after the event's origin.

b. Also, inside the folder, `archive_beyond_21events_ALL_6.8+/h5vol/*.h5` store the same data but in different format. The `*.h5` files are in hdf5 format. Each of the h5 files corresponds to a single event, and a single h5 file contains multiple records at many stations. Therefore, we only have ~200 h5 files instead of millions of sac files, and that can accelerate the I/O of our computations. Please note, both the sac format files and the h5 format files can be used as the input of our package [sacpy](https://github.com/sheng09/sacpy).

c. `run.sh` contains an example for running the computation. Its use sac format as input. It generates outputs to the folder `out/`. From it, you can view how we use the `cc_stack_v2.py` with many args. Also, you can find all available args and detailed explanations simply by running `cc_stack_v2.py` without appending any args. To make it easy, a simple explanation is below. Please note, the `orterun -np 6` or `mpirun -np 6` preceding the command are for mpi parallel running, and obviously we use 6 procs in the example. 

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

d. After running, you can check outputs inside the folder `out/`. As we specify `--out_format hdf5,sac`, the running output correlation stacks in both sac and h5 format. The former with filenames `cc_000.0_.sac`...`cc_180.0_.sac`, and the latter `cc.h5`, are for correlation stacks. The files `log_000.txt`...`log_005.txt` are log files in plain text.

e. `run_fast.sh` is similar to `run.sh` but take h5 format inputs, and hence it is much faster. It generates outputs to the folder `out_fast/`.


f. Plot the generated correlograms using the script `plot.sh`. It contains the commands using another executable file `plot_cc_wavefield.py` within the [sacpy](https://github.com/sheng09/sacpy). The same, it is self-documented. You can simply run `plot_cc_wavefield.py` without appending any args to view its manual. Below is a simplified example that takes the `out/cc.h5` as input and generate the figure `out/cc.png`. Besides, you can open the file `plot_cc_wavefield.py` and modify it by yourself or make it suit your purpose. Or, you can manipulate the sac files `cc_000.0_.sac`...`cc_180.0_.sac`.
    ````bash
    plot_cc_wavefield.py -T 50/4000 -D 0/180 -P out/cc.png --filter bandpass/0.02/0.0666666 \
        -I out/cc.h5 --plt figsize=4/10,title=Test,vmax=0.8,axhist=True,yticks=all,grid=True,interpolation=None,ylabel=True
    ````

g. Based on the examples, consider your exercise, such as (1) using different events, (2) trying just a single event, (3) try different stacking parameters, (4) plotting the generated correlation stacks by your own (e.g., invidual time series instead of a 2D greyscale image), etc.

