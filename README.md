Below are instructions to install the TMB R package (https://github.com/kaskr/adcomp) running under command-line Ubuntu using WSL (Windows Subsystem for Linux).
This installation allows TMB::gdbsource() to properly debug CPP files while using a PC running Windows 10 as the main OS.
It also provides R on an Ubuntu installation with GDB debugging software for any other purpose. 

See Linux_vs_Windows_gdb_errors.md in this repo for a comparison of the Linux and Windows gdb error reports, and see the Notes.md for additional information.

For those with a different Ubuntu installation such as a dual-boot PC or VirtualBox, jumping past the WSL instuctions will give a guide to installing the folowing apps on Ubuntu release 18.04: curl, libssl-dev (for the httr R package), GDB, and the most recent version of R. 

WSL saves all the software in a single folder, can be used for computational reproducibility, exporting to another machine, or sharing with a colleague. The path to the WSL folder is given below. See also:

https://www.hanselman.com/blog/easily-move-wsl-distributions-between-windows-10-machines-with-import-and-export

     # You will need to be an administrator on your Windows machine to install WSL.
     
     # Run PowerShell as Administrator (use search (the magnifier glass icon) and type < power > and click on 'Run as Administrator'
     # Right clicking in PowerShell will insert what previously has been copied into the clipboard inside or outside or the PowerShell.
     
     # If the PowerShell starts in C:\WINDOWS\system32 the change to c:\, if desired
     cd ../.. 
     
     # In the PowerShell first enable wsl to be installed in Windows:
     PS> Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
     
     # For help with the wsl command do:
     PS> wsl --help
     
     # Get a list of valid distributions. Ubuntu 20.04 is available but may have issues.
     PS> wsl  --list --online
     
     # Install Ubuntu-18.04 into wsl by entering  :
     PS> wsl --install -d Ubuntu-18.04 
     
     # A new window will open. Enter a user name and password. A (very) short password be accepted (a single space will do) 
     #   and will be faster to type when needing elevated rights.
     # Exit out of the new window by typing < exit > and then start WSL in the PowerShell. < exit > also exits out of WSL.
     PS> wsl
     
     # If nothing happens when typing 'wsl' or the wrong distribution is opened, enter:
     PS> wsl -l
     PS> wsl -s Ubuntu-18.04
     
     # < wsl -s > sets the given distribution as the default. Now try WSL again:
     PS> wsl 
     
     # Check Ubuntu version (note that this flavor of Ubuntu is 'Bionic')
     lsb_release -a
     
     # In Windows File Explorer create the folder "TMB_Debug" on the C: drive. Enter the folder and leave File Explorer open.
     # Copy the 'simpleError.cpp' and simpleError.R files from the R_and_Cpp folder in this repo to C:\TMB_Debug
     
     # All the Windows' drives will be mounted in /mnt on Ubuntu, e.g. 'c' for 'C:' drive:
     ls ../../mnt
     
     # Create a change directory command in .bashrc to run at startup. '.bashrc' needs to be in the home folder (~).
     cd ~
     echo 'cd /mnt/c/TMB_Debug' > .bashrc
     ls -al
     
     # For now, change to this directory manually
     cd /mnt/c/TMB_Debug
     
     # Before installing the apps, including R, here is some additional information.
       
     # WSL files are in an \AppData\Local\Packages folder by user, e.g. mine are here:
     C:\Users\John\AppData\Local\Packages
     
     # The name (sort by most recently 'Date modified') will appear similar to:
     CanonicalGroupLimited.Ubuntu18.04onWindows_79rhkp1fndgsc
     
     # This folder can be renamed, backed up, saved for computational reproducibility, and exported to a colleague.  
     # (If you also use Docker, you may have exit out of Docker before working with the folder.)
     https://www.hanselman.com/blog/easily-move-wsl-distributions-between-windows-10-machines-with-import-and-export
     
     # WSL can also be opened by using the Windows' search feature: type < wsl > and click on 'Run as Administrator'.  
     # (There will be a different default color scheme.)
     
     
     # Make yourself root to do the following installs using the password set above.
     # The '$' prompt will change to '#', type < exit > to close. (Mnemonic: 'The pound is stronger than the dollar.')
     sudo su
     
     # First do an update
     apt-get update
     
     # Install 'curl' (The handy '-y' option auto answers 'Yes' to prompts to continue or stop.)
     apt-get -y install curl
     apt-get -y install libcurl4-openssl-dev
          
     # Install libssl-dev for the R package 'httr'
     apt-get -y install libssl-dev
          
     # Install gdb following: https://stackoverflow.com/questions/10255082/installing-r-from-cran-ubuntu-repository-no-public-key-error
     # The first command may take some time or perhaps will fail for a time - continue to retry - install R and return here if needed
     gpg --keyserver pgp.mit.edu --recv-key 51716619E084DAB9
     
     gpg -a --export 51716619E084DAB9 > jranke_cran.asc 
     apt-key add jranke_cran.asc 
     apt -y install gdb
     
     # Install BLAS, LAPACK, and FORTRAN packages
     apt-get -y install libblas-dev liblapack-dev
     apt-get -y install gfortran
     
     
     # CRAN instructions for installing R in Ubuntu: https://cran.r-project.org/  
     # On the CRAN website select 'Ubuntu' on the line: < Download R for Linux (Debian, Fedora/Redhat, Ubuntu) >
     # < install build-essential> is not in the CRAN instuctions but was needed by me and others on the internet.
     apt update -qq
     apt -y install build-essential
     
     # install two helper packages we need
     apt -y install --no-install-recommends software-properties-common dirmngr
     
     # add the signing key (by Michael Rutter) for these repos
     # To verify key, run gpg --show-keys /etc/apt/trusted.gpg.d/cran_ubuntu_key.asc 
     # (The verify key command doesn't work for me, but isn't needed to continue.)
     # Fingerprint: 298A3A825C0D65DFD57CBB651716619E084DAB9
     wget -qO- https://cloud.r-project.org/bin/linux/ubuntu/marutter_pubkey.asc | tee -a /etc/apt/trusted.gpg.d/cran_ubuntu_key.asc
     
     # add the R 4.0 repo from CRAN
     # Here we use lsb_release -cs to access which Ubuntu flavor you run: one of “impish”, “hirsute”, “focal”, “bionic”, …
     add-apt-repository "deb https://cloud.r-project.org/bin/linux/ubuntu $(lsb_release -cs)-cran40/"
          
     # Then run
     apt -y install --no-install-recommends r-base
     
     
     # Startup R in Ubuntu. Note, to escape out of a running program on Linux use < Ctrl-c >.
     R
     
     # < Ctrl-z > jumps out into the bash shell; < ps > in the shell shows processes; < fg > in the shell goes back into R 
     # (One could start another R process and jump between them, Google for more info.)
     
     # Now in R (see below for GitHub install)
     options(width = 140) # Default command line width in Linux-alikes is too short - adjust 140 to fit your current window size
     install.packages(c('sys', 'askpass', 'jsonlite', 'mime', 'openssl', 'R6', 'curl', 'httr', 'remotes', 'TMB'))
     
     # Installing TMB from GitHub also works (or for any other package that is on GitHUb)
     # There is less scrolling of C++ code on the screen with the GitHub install.
     # install.packages(c('sys', 'askpass', 'jsonlite', 'mime', 'openssl', 'R6', 'curl', 'httr', 'remotes'))
     # Sys.setenv(GITHUB_PAT='< Put your own GITHUB_PAT here >')  #  May work without this, at least for awhile.
     # remotes::install_github("kaskr/adcomp/TMB", INSTALL_opts = "--no-staged-install")
     
     # Test TMB
     library(TMB)
     runExample('simple')
     opt
     
     # Create an out-of-bounds error and verify the gdbsource() gives the correct line where the error occurs
     # Compiling 'simpleError.cpp' works:
     if(file.exists('simpleError.o')) file.remove(c('simpleError.o')); if(file.exists('simpleError.so')) file.remove(c('simpleError.so'))
     compile('simpleError.cpp', "-O0 -g")
     
     # But sourcing 'simpleError.R' will crash R
     source('simpleError.R')
     
     # Therefore, get back into R and use gdbsource() in Linux to find the line of code (30) that has the error
     library(TMB)
     gdbsource('simpleError.R')
     
     # Try using GDB interactively (see https://betterexplained.com/articles/debugging-with-gdb/ 
     #    and http://manpages.ubuntu.com/manpages/trusty/man1/gdb.1.html )
     gdbsource('simpleError.R', interactive = TRUE)
     (gdb) help
     (gdb) help stack
     (gdb) bt
     (gdb) quit
     
     # When doing further exploration, unload simpleError.so, if needed
     getLoadedDLLs()
     dyn.unload(dynlib("simpleError"))
     
 ---     
 
     # -- Create a .Rprofile file to autoload TMB into the search() path (and the MASS package, as another example) on R startup. --
     
     #     Provide your GITHUB_PAT from GitHub and uncomment the line to auto add to the 
     #         R system environment (use Sys.getenv('GITHUB_PAT') to view).
     #     Do not share your WSL file folder with your GITHUB_PAT intact.
     #     The CRAN repository and line width (= 140) are also set.
     #     cf. /etc/R/Rprofile.site which is more global.
     echo 'r <- getOption("repos")' > /mnt/c/TMB_Debug/.Rprofile
     echo 'r["CRAN"] <- "https://cloud.r-project.org"' >> /mnt/c/TMB_Debug/.Rprofile
     echo 'old <- getOption("defaultPackages")' >> /mnt/c/TMB_Debug/.Rprofile
     echo 'options(repos = r, width = 140, defaultPackages = c(old, "MASS", "TMB"))' >> /mnt/c/TMB_Debug/.Rprofile
     echo 'rm(r, old)' >> /mnt/c/TMB_Debug/.Rprofile
     # echo 'Sys.setenv(GITHUB_PAT="Your GITHUB_PAT here")' >> /mnt/c/TMB_Debug/.Rprofile
     
     R
     search()
     Sys.getenv('GITHUB_PAT')
     
---
     
     # -- pdf() is the defalut plotting device when the X11cairo graphics device is not installed (see the next section) --
     
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
     browseURL('Rplots.pdf') # Alternatively, jump back to C:\TMB_Debug in Window's File Explorer and view the 2 figures in 'Rplots.pdf'
     
---

     
     # -- Enable the X11cairo graphics device under R --
     
     # Loosely following the early information here:
     #     https://stackoverflow.com/questions/61110603/how-to-set-up-working-x11-forwarding-on-wsl2
     
     # But note that a comment points out that the extra inbound rule is only needed if you want to avoid allowing access for all
     #   public networks. So don't deal with the Windows firewall (if at all) until this installation is finished since the 
     #   install of VcXsrv adds inbound rules for you. Also see the comment about my firewall issues below.  
     
     # Install the VcXsrv Windows X Server:
     #     https://sourceforge.net/projects/vcxsrv/   
     
     # Setup VcXsrv following the video below stopping around the 5:17 mark since the rest will be covered below using
     #   the shorter export DISPLAY command given by Comment 190 on the Stack Overflow website given above.
     
     # Also, the XLaunch file talked about in the video does not need to be edited since < DisableAC = "True" > is sufficient 
     #   and < ExtraParams'"-ac" > is not needed.
     
     #     https://www.youtube.com/watch?v=6_mbd1hvUnE
     
     
     # Back in Ubnutu under WSL:
       
     # # The goal for the export commands is to extract the IP address of the X Terminal, e.g.
     ip route list default | awk '{print $3}' 
     # 172.27.224.1
     
     # and add the following two lines to the export environment:
     #   declare -x WSL_HOST_IP="172.27.224.1"
     #   declare -x LIBGL_ALWAYS_INDIRECT="1"
     export
     
     # This is done at login by adding the following lines to ~/.bashrc:
     #   export DISPLAY=$(ip route list default | awk '{print $3}'):0
     #   export LIBGL_ALWAYS_INDIRECT=1
     
     # The following echo commands will easily add the exports to ~/.bashrc:
     cd ~
     echo 'export DISPLAY=$(ip route list default | awk '"'"'{print $3}'"'"'):0' >> .bashrc
     echo 'export LIBGL_ALWAYS_INDIRECT=1' >> .bashrc
     
     # Install the x11-apps
     sudo su
     apt update
     apt -y install x11-apps
     exit
     
     # With the VcXsrv X Server running and the icon in the Taskbar, restart the PowerShell or Ubuntu via Wsltty (see below)
     # Check for the added export commands
     export | grep DISPLAY
     export | grep LIBGL_ALWAYS
     
     # Try xeyes and xclock ('&' puts the app directly into the background)
     xeyes &
     xclock &
     
     # If that works start R and run the following code to obtain the pie chart seen in the picture below:
     R
     require(grDevices)
     pie(rep(1, 24), col = rainbow(24), radius = 0.9)
     dev.cur()
     
     # After some failed attempts at this installation, I ended up with extra 'VcXsrv windows xserver' entries in 
     #    the Windows Defender Firewall > Advanced Settings > Inbound Rules section.
     # I deleted the extra entries and left only one entry turned on (with white check mark within a green circle). 
     #    After this change the X server worked fine (perhaps with reboot).
     # While troubleshooting, I temporarily turned off the Windows Firewall Defender and the X server also worked then.
     # Note also, that right-clicking on the Inbound rules allows settings to be changed.
     
     
     # For direct launching of Ubuntu from Windows install Mintty as a terminal for WSL:
     https://github.com/mintty/wsltty
     
     # Go up one level for a look at the entire repo:
     https://github.com/mintty
     
     # Additional information can be found on the website given:
     http://mintty.github.io/
     
     # Right-clicking within the Mintty terminal gives a menu with standard commands such as 'Copy' and 'Paste' (unlike PowerShell).
     
     
     # The background color, text size and color (called the foreground color in Mintty), and other options for both PowerShell 
     #   and Mintty are user configurable by right-clicking the top bar and going to 'Properties' or 'Options', respectively.
     
     

![R X11 on Ubuntu under WSL](https://github.com/John-R-Wallace-NOAA/TMB_on_Windows_Subsystem_for_Linux/blob/main/R%20X11%20on%20Ubuntu%20under%20WSL.png)
