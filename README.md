Steps to Run Coda Correlation Computations
----

Here we introduce how to configure programs/files, how to run coda correlation computations using the package [sacpy](https://github.com/sheng09/sacpy), and how to plot the generated correlograms, etc.

# Configurations

## Settings for RSES compute2 users.
If you can access the server compute2 at RSES, then configurations will be easy for you. Follow the steps here to configure your environment. Please note, you just need to configure for once, and then everything will be functional to you for always.

- Edit the enviroment `PATH`. In your compute2 home folder, edit the file `~/.bash_profile` by appending the following settings to it:

```
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

- Run the commands below to enable the settings:

```
source ~/.bash_profile
```

## In your environment
You can download and install [sacpy](https://github.com/sheng09/sacpy) on your machines. 

Manuals coming soon....


