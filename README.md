A Mixture of Tips
===
<!-- TOC -->

- [A Mixture of Tips](#a-mixture-of-tips)
- [1. Install and Configure Ubuntu](#1-install-and-configure-ubuntu)
  - [1.1 Install system](#11-install-system)
  - [1.2 Install Chinese sougou-pinyin input method](#12-install-chinese-sougou-pinyin-input-method)
  - [1.3 Configure gnome's appearance with `Tweak`](#13-configure-gnomes-appearance-with-tweak)
  - [1.4 Configure fonts](#14-configure-fonts)
- [2. Setup Different Servers](#2-setup-different-servers)
  - [2.1 OpenSSH Server](#21-openssh-server)
  - [2.2 Apache Server](#22-apache-server)
  - [2.3 Jupyter-lab Server](#23-jupyter-lab-server)
  - [2.4 Connect to a Remote Server via An Intermediate Server](#24-connect-to-a-remote-server-via-an-intermediate-server)
- [3. Package and Environment Management](#3-package-and-environment-management)
  - [3.1 Manage Python modules with `pip3`](#31-manage-python-modules-with-pip3)
  - [3.2 `conda` for Package and Environment Management](#32-conda-for-package-and-environment-management)
- [4. HPC, MPI, CUDA Things](#4-hpc-mpi-cuda-things)
  - [4.1 Enable `mpi4py` and `openmpi` for `cupy`](#41-enable-mpi4py-and-openmpi-for-cupy)
  - [4.2 Runtime Err `[LOG_CAT_ML] ml_discover_hierarchy exited with error`](#42-runtime-err-log_cat_ml-ml_discover_hierarchy-exited-with-error)
- [X. Some problems and answers](#x-some-problems-and-answers)
  - [X.1 Run `pyvista` from SSH Remote Server](#x1-run-pyvista-from-ssh-remote-server)
  - [X.2 `Rsync` cheatsheet](#x2-rsync-cheatsheet)
  - [X.3 What and why `Kill` a process](#x3-what-and-why-kill-a-process)

<!-- /TOC -->

# 1. Install and Configure Ubuntu

## 1.1 Install system
- Note `UEFI`, `Legacy` things in BIOS setting, as they are related to disk partitioning method. `UEFI` needs an `efi` partition.
- `/boot` should not be too small (e.g., <512Mb), or system updates sometimes fail due to insufficient space in `/boot`. 

## 1.2 Install Chinese sougou-pinyin input method
- Following the [offical site](https://shurufa.sogou.com/linux) to configure system, downloand and install sougou-pinyin.
- Note the required libary that is not listed in the offical site `qml-module-gsettings1.0`.
  Install it with:
  ```bash
  sudo apt install qml-module-gsettings1.0
  ```
- If it cannot work properly, check the lack of libraries by running `fcitx-autostart` in a terminal. Its output/logs will show which libraries are missing.


## 1.3 Configure gnome's appearance with `Tweak`

- **Instal Tweak**
  ```bash
  sudo apt install gnome-tweak-tool
  sudo apt install gnome-shell-extensions
  ```
- **Adjust theme**
  - Open `Tweaks`-->`Appearance`-->`Applications`, select one of the theme from dropdown list.
  - New themes can be downloaded from [GTK3/4 Themes](https://www.gnome-look.org/browse?cat=135).
  - New themes should be place in the folder `.themes` in users' home directory.
  - Here is [an adjusted theme](https://github.com/sheng09/install-ubuntu/blob/main/gnome-professional-solid-40.1-WANG.tgz) based on [Prof-Gnome-theme](https://www.gnome-look.org/p/1334194).
- **Transparent top bar**.
  - Install one more gnome extension: [Transparent Top Bar (Adjustable transparency)](https://extensions.gnome.org/extension/3960/transparent-top-bar-adjustable-transparency/).
  - Adjust `Extensions`-->`Transparent Top Bar (Adjustable transparency)`.
- **Transparent dock**
  - Install one more more gnome extension: [dash-to-dock](https://extensions.gnome.org/extension/307/dash-to-dock/).
  - Adjust `Extensions`-->`Dash to dock`. (`Intelligent autohide` can be good.)
  - Disable the ubuntu built-in dock in case of two overlapping docks.
     ```bash
     gnome-extensions disable ubuntu-dock@ubuntu.com
     ```
- **Other adjustments within Tweak**
  - `Extensions`-->`Horizontal workspaces`.
  - `Workspaces`-->`Number of Workspaces: 3`.

## 1.4 Configure fonts
- Manage fonts with **Font-manager** as a sudoer.
  - Instal `font-manager`:
     ```bash
     sudo apt install font-manager
     ```
  - Search for and get more font `.ttf` files, and add them with `font-manager`.
  - Many fancy fonts can be found [here](https://www.dafont.com/theme.php?cat=103) (e.g., [hacker](https://www.dafont.com/hack.font)).
- Add/manage fonts as an user.
  - Add
    ```bash
    mkdir ~/.local/share/fonts # create this directory if it does not exist
    mv font_name_.ttf ~/.local/share/fonts/
    fc-cache -f -v  #clear and regenerate your font cache
    ```
  - Check if a font is installed.
    ```bash
    fc-list | grep Arial # an example for checking for Arial
    ```
- Update **matplotlib** cache to enable using new fonts.
    ```bash
    rm ~/.cache/matplotlib -rf
    ```

# 2. Setup Different Servers 

## 2.1 OpenSSH Server

  Run the commands below to install and enable openssh server.
  ```
  sudo apt-get install openssh-server
  sudo systemctl status ssh  # check the status
  sudo systemctl enable ssh  # enable and start ssh
  sudo systemctl start ssh
  #
  sudo apt install net-tools
  ifconfig  # check the IP address of the machine
  ```

## 2.2 Apache Server
  Setup **Apache web server** for sharing files
  - Install `apache2`:
    ```bash
    sudo apt install apache2
    ```
  - Create a directory `foldername_whatever/` in home, and then create a symbolic link to it in `/var/www/html`. Finally, one can access `$ip$/toshare` from a web browser.
    ```bash
    cd ~
    mkdir foldername_whatever
    chmod -R o+x foldername_whatever # make sure the folder has permission of x for others
    cd /var/www/html
    sudo ln -s ~/foldername_whatever .
    # Then one can access $ip$/foldername_whatever, in which $ip$ is the IP/address of the machine.
    ```
  - Restart/stop apache2 server.
    ```bash
    sudo service apache2 stop
    sudo service apache2 restart
    sudo service apache2 reload   # etc
    ```
## 2.3 Jupyter-lab Server
   - Install `jupyterlab` via `pip3`.
     ```bash
     pip3 install jupyterlab
     ```
   - Enable and configure jupyterlab server.
     ```bash
     jupyter server --generate-config
     jupyter server password  # set the password
     nohup jupyter-lab --port $XXXX$ --no-browser > ~/.jupyter/nohuo.log & # replace XXXX to four digits, (e.g., 9832)
     ```
   - Access jupyterlab server on local machine. Open `http://localhost:$XXXX$` in a web browser, where $XXXX$ is the four digits.
   - Access jupyterlab server on a remote terminal.
     ```bash
     ssh -N -L $YYYY$:localhost:$XXXX$ username@server_address
     ```
     in which `$XXXX$` is the previous four digits set on the server, and `$YYYY$` is another four independent digits set on the remote terminal.
     Then, one can access on the remote terminal by opening `http://localhost:$YYYY$` in a web browser.

## 2.4 Connect to a Remote Server via An Intermediate Server

Sometimes, we need to use an intermediate server, a jump box, to connect to an server that cannot not be access publicly, for example, a internal computation node of within a cluster.

  - **Setting in VS-Code**

    Add the following settings into `somewhere/{yourusername}/.ssh/config` which can be accessed in VS-Code via searching for *Remote-SSH: Open Configuration File...* in command pannel (Ctrl+Shift+P).
    ```bash
    # Jump box with public IP address
    Host jump-box # name shown in vscode for the jump box
        HostName gadi.nci.org.au
        User your_user_name
        #IdentityFile ~/.ssh/jumpbox #not necessary if using password

    # Target machine with private IP address
    Host target-box # name shown in vscode for the target machine
        HostName gadi-cpu-bdw-0145.gadi.nci.org.au # IP of the machine can be seen by the jump-box
        User your_user_name
        #IdentityFile ~/.ssh/target
        ProxyCommand ssh -q -W %h:%p jump-box

    ```

  - **Manually ssh Connection**

    Coming when in demand.

# 3. Package and Environment Management

## 3.1 Manage Python modules with `pip3`
  - Several commands for managing/fixing pip3 modules and their dependency:
    ```bash
    # Specify version when install a module
    # The following example specify version 1.22
    pip3 install numpy==1.22

    # Ignore the existed module
    # The following example will ignore the installed matplotlib in /usr/local/...,
    # and install the matplotlib to ~/.local/....
    pip3 install --ignore-installed --user matplotlib

    #
    ```

## 3.2 `conda` for Package and Environment Management
  - Step1: install `miniconda3` following the [official manuals](https://conda.io/projects/conda/en/latest/user-guide/install/index.html).
  **Note: in the last step of installation, please DO NOT allow the installation to modify your `~/.bash_*` files. Such modification is dirty**

  - Step2: set the following alias into your `~/.bash_rc` or `~/.bash_profile`.
    ```bash
    alias start-conda='source /g/data/em78/SW/software/miniconda3/bin/activate && conda init'
    ```
    After running `source ~/.bash_rc` or `source ~/.bash_profile`, then you can use `start-conda` to enable the `base` conda environment, and `conda deactivate` to leave such `base` environment.
    **Note: please comment the generated conda related things in the `~/.bash_rc` or `~/.bash_profile`.**
    ```bash
    # Comment the below, even the comments, so that it would not be generated next time/
    ## >>> conda initialize >>>
    ## !! Contents within this block are managed by 'conda init' !!
    #__conda_setup="$('/g/data/em78/SW/software/miniconda3/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
    #if [ $? -eq 0 ]; then
    #    eval "$__conda_setup"
    #else
    #    if [ -f "/g/data/em78/SW/software/miniconda3/etc/profile.d/conda.sh" ]; then
    #        . "/g/data/em78/SW/software/miniconda3/etc/profile.d/conda.sh"
    #    else
    #        export PATH="/g/data/em78/SW/software/miniconda3/bin:$PATH"
    #    fi
    #fi
    #unset __conda_setup
    ## <<< conda initialize <<<
    ```

  - Step3-N: activate conda and setup an environment
    ```bash
    start-conda # enter the base
    # then an example from https://rapids.ai/start.html#get-rapids
    conda -p ~/conda_env/my_env1 -c rapidsai -c nvidia -c conda-forge cusignal==22.10 cuml==22.10 cudf==22.10 python==3.8.5 cudatoolkit=11.4
    conda activate ~/conda_env/my_env1
    ```


# 4. HPC, MPI, CUDA Things

## 4.1 Enable `mpi4py` and `openmpi` for `cupy`

  We need to install and configure something according to the offical document of [`mpi4py`](https://mpi4py.readthedocs.io/en/latest/install.html#using-conda), "*...The openmpi package on conda-forge has built-in `CUDA` support, but it is disabled by default. To enable it, follow the instruction outlined during conda install. Additionally, `UCX` support is also available once the ucx package is installed...*", and and offical document of [`cupy`](https://docs.cupy.dev/en/stable/user_guide/interoperability.html), "*...This new feature is added since `mpi4py` 3.1.0...*".

  - Step1: install `mpi4py>=3.1.0` with openmpi support:
    ```bash
    #make sure openmpi module is correctly loaded to match the one used by conda install
    # make sure if it is openmpi or mpich or something else that match your environment
    module load openmpi/4.1.4 cuda/11.5
    conda install -c conda-forge mpi4py==3.1.1 openmpi=4.1.4 
    ```
  - Step2: Enable CUDA and UCX supports following the instructions outputed by the `conda install ...` via either setting the environment before launching MPI programs:
    ```bash
    export OMPI_MCA_opal_cuda_support=true
    export OMPI_MCA_pml="ucx"
    export OMPI_MCA_osc="ucx"
    ```
    or launching MPI programs with arguements like:
    ```bash
    mpirun --mca opal_cuda_support 1  ...   # for cuda
    mpirun --mca pml ucx --mca osc ucx ...  # for ucx
    mpirun --mca opal_cuda_support 1  --mca pml ucx  --mca osc ucx ... # for both
    ```
  - Supplementary: an example to test if it works.
    ```py
    #/usr/bin/env python3
    #
    # exam.py
    #
    from mpi4py import MPI
    import cupy

    omm = MPI.COMM_WORLD
    ize = comm.Get_size()
    ank = comm.Get_rank()

    # Allreduce
    sendbuf = cupy.arange(10, dtype='i')
    recvbuf = cupy.empty_like(sendbuf)
    # always make sure the GPU buffer is ready before any MPI operation

    print("before comm", size, rank, recvbuf, sendbuf)

    cupy.cuda.get_current_stream().synchronize()
    comm.Allreduce(sendbuf, recvbuf)
    assert cupy.allclose(recvbuf, sendbuf*size)


    # Bcast
    if rank == 0:
        buf = cupy.arange(100, dtype=cupy.complex64)
    else:
        buf = cupy.empty(100, dtype=cupy.complex64)
    cupy.cuda.get_current_stream().synchronize()
    comm.Bcast(buf)
    assert cupy.allclose(buf, cupy.arange(100, dtype=cupy.complex64))

    # Send-Recv
    if rank == 0:
        buf = cupy.arange(20, dtype=cupy.float64)
        cupy.cuda.get_current_stream().synchronize()
        comm.Send(buf, dest=1, tag=88)
    else:
        buf = cupy.empty(20, dtype=cupy.float64)
        cupy.cuda.get_current_stream().synchronize()
        comm.Recv(buf, source=0, tag=88)
        assert cupy.allclose(buf, cupy.arange(20, dtype=cupy.float64))

    print("after comm", size, rank, recvbuf, sendbuf)
    ```
    Run it:
    ```
    mpirun --mca opal_cuda_support 1  --mca pml ucx  --mca osc ucx python3 exam.py
    ```

## 4.2 Runtime Err `[LOG_CAT_ML] ml_discover_hierarchy exited with error`

  When using `mpi4py` to call MPI on NCI nodes, warning/error happens while they
  do not affect the running and results:

  ```bash
  [LOG_CAT_ML] component basesmuma is not available but requested in hierarchy: basesmuma,basesmuma,ucx_p2p:basesmsocket,basesmuma,p2p
  [LOG_CAT_ML] ml_discover_hierarchy exited with error
  ```
  Ben Menadue at NCI explained to me this [issue](https://track.nci.org.au/servicedesk/customer/portal/5/HELP-185744):

  *I'm going to guess you have a Python script calling MPI? That error message comes from hcoll, one of the Mellanox accelerated collective components inside OpenMPI, and arises when it's not able to open some of its plugins. And the usual cause of this is trying to use MPI from a Python script.*

  *The issue arises because hcoll's plugins expect the main hcoll library to be loaded into the global symbol namespace. However, Python loads extension modules with the `RTLD_LOCAL` flag, which causes the extension module itself and any of its dependencies (including the MPI and hcoll libraries) to be loaded into a private symbol namespace. The idea is that different Python extension modules should be isolated from each other, but a side-effect is that hcoll's plugins can no longer resolve symbols from the main hcoll library and thus can't be loaded.*

  *It's a common issue that arises when trying to mix C libraries that use a plugin-like architecture with Python. If that plugin framework hasn't explicitly been designed to support such usage. OpenMPI itself used to the same problem, but we avoided it there by bundling all the plugins into the main library when building it. Unfortunately, since hcoll is only provided as a pre-built collection of libraries from Mellanox, we can't use the same trick for that.*

  *It's not a fatal error, it just means that some collective operations might not be as fast as they could be. However that's usually not a problem for Python scripts: using Python usually incurs other overheads that would dominate over any latency improvement that the accelerated transports provides.*

  *If you really did still want to use the accelerated transfers, the usual solution is to LD_PRELOAD the MPI library (libmpi.so), which forces it and its dependencies (including hcoll) to be loaded globally, i.e. how hcoll expects it. So something like this:*

  **Solution**
  ```bash
  mpirun -x LD_PRELOAD=libmpi.so ... python3 path/to/my/script.py ...
  ```
  

# X. Some problems and answers

## X.1 Run `pyvista` from SSH Remote Server
  - Some errors when running `pyvista` via ssh from a remote server:
    ```bash
    libGL error: No matching fbConfigs or visuals found
    libGL error: failed to load driver: swrast
    ```
    Solution
    ```bash
    # Set this on the remote server
    export DISPLAY=:0.0 glxinfo
    ```

## X.2 `Rsync` cheatsheet

  - **Only select specific files:**
    ```bash
    rsync -av --include="*/" --include "folder_pattern*/file_pattern*" \
          --exclude="*" remote_folder local_folder
    ```
  - Only select specific files and **exclude the sub-directories that don't contain any targeted files**
    ```bash
    rsync -av --prune-empty-dirs --include="*/" --include "folder_pattern*/file_pattern*" \
          --exclude="*" remote_folder local_folder
    ```

## X.3 What and why `Kill` a process

  Use the following command to find out details:
    ```bash
    dmesg -T| grep -E -i -B100 'killed process'
    ```
  
  Where `-B100` signifies the number of lines before the kill happened. Omit `-T` on Mac OS.
  
  The source is from [here](https://stackoverflow.com/questions/726690/what-killed-my-process-and-why)
