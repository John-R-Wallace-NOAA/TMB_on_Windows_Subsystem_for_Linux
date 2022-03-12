blah blah blah 


     # You will need to be an administrator on your Windows machine to install WSL
     
     # Run PowerShell as Administrator (use search (the magnifier glass icon) and type < power > and click on 'Run as Administrator'
     # Right clicking in PowerShell will insert what previosly has been copied into the clipboard inside or outside or the PowerShell.
     
     # Change from C:\WINDOWS\system32 to c:\, if desired
     cd ../.. 
     
     # In the PowerShell first enable wsl to be installed in Windows:
     PS> Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
     
     # For help with the wsl command do:
     PS> wsl --help
     
     # Get a list of valid distributions
     PS> wsl  --list --online
     
     # Install Ubuntu-18.04 into wsl by entering  :
     PS> wsl --install -d Ubuntu-18.04 
     
     # A new window will open. Enter a user name and password. A (very) short password be accepted (a single space will do) and will be faster to type when needing elevated rights.
     # Exit out of the new window by typing < exit > and then start WSL in the PowerShell
     PS> wsl
     
     # If nothing happens when typing 'wsl' or the wrong distribution is opened, enter:
     PS> wsl -l
     PS> wsl -s Ubuntu-18.04
     
     # < wsl -s > sets the given distribution as the default
     # Now start WSL
     PS> wsl 
     
     # First commmands to run in Ubuntu -- explain commands and how c drive is in mnt folder as  < c >
     # Check Ubuntu version 
     lsb_release -a
     
     # Change to home folder
     cd ~
     
     # In Windows File Explorer create the folder "TMB_Debug" on the C: drive. Enter the folder and leave File Explorer open.
     
     # All the Windows' drives will be mounted in /mnt on Ubuntu, e.g. 'c' for 'C:' drive.
     ls ../../mnt
     
     # Create a change directory command in .bashrc to run at startup. Here the 'TMB_Debug' folder on the 'C:' drive is being used. This needs to be in the home folder (~).
     echo 'cd /mnt/c/TMB_Debug' > .bashrc
     
     # For now, change to this directory manually
     cd /mnt/c/TMB_Debug
     
     # Before installing apps, including R, here is some additional information
       
     # WSL files are in an \AppData\Local\Packages folder by user, e.g. mine are here:
     C:\Users\John\AppData\Local\Packages
     
     # The names look like (sort by most recently 'Date modified'):
     CanonicalGroupLimited.Ubuntu18.04onWindows_79rhkp1fndgsc
     
     # This folder can be renamed, backed up, and exported to a colleague.  (If you also use Docker, you may have exit out of Docker to work with the folder.)
     https://www.hanselman.com/blog/easily-move-wsl-distributions-between-windows-10-machines-with-import-and-export
     
     # WSL can also be opened by using Windows' search feature, typing  < wsl > and click on 'Run as Administrator'.  (There will be a different default color scheme.)
     
     
     # Make yourself root to do the following installs (the '$' prompt will change to '#', type  < exit > to close) (Mnemonic: 'The pound is stronger than the dollar.')
     sudo su
     
     # First do an update
     apt-get update
     
     #Install 'curl' (The '-y' option auto answers 'Yes' to prompts to continue.)
     apt-get -y install curl
     apt-get -y install libcurl4-openssl-dev
     
     
     #Install libssl-dev for the R package 'httr'
     apt-get -y install libssl-dev
     
     
     # Install gdb following: https://stackoverflow.com/questions/10255082/installing-r-from-cran-ubuntu-repository-no-public-key-error
     # The first command may take some time or perhaps will fail for a time - continue to retry - install R and return here if needed
     gpg --keyserver pgp.mit.edu --recv-key 51716619E084DAB9
     
     gpg -a --export 51716619E084DAB9 > jranke_cran.asc 
     apt-key add jranke_cran.asc 
     apt -y install gdb
     
     # Install BLAS and LAPACK packages and FORTRAN
     apt-get -y install libblas-dev liblapack-dev
     apt-get -y install gfortran
     
     
     # CRAN instructions for installing R in Ubuntu: https://cran.r-project.org/  Select 'Ubuntu' on < Download R for Linux (Debian, Fedora/Redhat, Ubuntu) >
     # < install build-essential> is not in the CRAN instuctions but was needed for me and others on the internet.
     apt update -qq
     apt -y install build-essential
     
     # install two helper packages we need
     apt -y install --no-install-recommends software-properties-common dirmngr
     
     # add the signing key (by Michael Rutter) for these repos
     # To verify key, run gpg --show-keys /etc/apt/trusted.gpg.d/cran_ubuntu_key.asc (This command doesn't work for me, and isn't needed to continue.)
     # Fingerprint: 298A3A825C0D65DFD57CBB651716619E084DAB9
     wget -qO- https://cloud.r-project.org/bin/linux/ubuntu/marutter_pubkey.asc | tee -a /etc/apt/trusted.gpg.d/cran_ubuntu_key.asc
     
     # add the R 4.0 repo from CRAN -- adjust 'focal' to 'groovy' or 'bionic' as needed
     add-apt-repository "deb https://cloud.r-project.org/bin/linux/ubuntu $(lsb_release -cs)-cran40/"
     
     # Then run
     apt -y install --no-install-recommends r-base
     
     
     # Startup R in Ubuntu. Note, to escape out of a running program on Linux use < Ctrl-c >.
     R
     
     # < Ctrl-z > jumps out into the bash shell; < ps > in the shell shows processes; < fg > in the shell goes back into R (One could start another R process and jump between them, Google for more info.)
     
     # Now in R
     options(width = 140) # Default command line width in Linux-alikes is too short
     install.packages(c('sys', 'askpass', 'jsonlite', 'mime', 'openssl', 'R6', 'curl', 'httr', 'remotes'))
     
     Sys.setenv(GITHUB_PAT='< Put your own GITHUB_PAT here >')  #  May work without this, at least for awhile.
     remotes::install_github("kaskr/adcomp/TMB", INSTALL_opts = "--no-staged-install")
     
     # Test TMB
     
     library(TMB)
     runExample('simple')
     opt
     
add in error files

     # Create an out of bounds error and verify the gdbsource() gives the correct line where the error is
     # Compiling 'simpleError.cpp' will work:
     compile('simpleError.cpp', "-O0 -g")
     
     # Sourcing 'simpleError.R' will crash R
     source('simpleError.R')
     
     # Therefore, get back into R and use gdbsource() in Linux and find the line of code (30) that has the error
     library(TMB)
     gdbsource('simpleError.R')
     
     
     # When doing further explopration, unload simpleError.so, if needed
     getLoadedDLLs()
     dyn.unload(dynlib("simpleError"))
     
---
     
     # -- Extra --
     # The defalut plotting device is 'pdf'
     
     butterfly <- function (alpha = 4, beta = 12, plot = TRUE, ...) {
     
         theta <- seq(0, 24 * pi, len = 2000)
         radius <- exp(cos(theta)) - 2 * cos(round(alpha) * theta) + 
             sin(theta/beta)^5
         x <- -radius * sin(theta)
         y <-  radius * cos(theta)
         if(plot)
            plot(range(x) + c(-0.1, 0.1), range(y) + c(-0.1, 0.1), xlab = "",  
              ylab = "", type = "n", xaxt = "n", yaxt = "n", bty = "n")   
         lines(x, y, ...)
     }
     
     butterfly(col = 'Dodger blue')
     dev.cur()
     butterfly(2.5, 12, col = 'green')
     dev.off()
     
     # Jump back to C:\TMB_Debug in the open Window's File Explorer and view the 2 figures in 'Rplots.pdf'.
     


