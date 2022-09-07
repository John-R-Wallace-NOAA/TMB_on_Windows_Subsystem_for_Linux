   
     # ----  Comparing gdb output in Linux versus Windows ----
     
     # -- Under Linux --
     
     # After learning the bascis of the gdb language from here:  https://betterexplained.com/articles/debugging-with-gdb/ 
     #       and http://manpages.ubuntu.com/manpages/trusty/man1/gdb.1.html
     #   the commands given to gdb under Linux can be seen in the first line of the non-interactive section 
     #   at the bottom of TMB::gdbsource():
     
     TMB::gdbsource
     function (file, interactive = FALSE) 
     {
         if (!file.exists(file)) 
             stop("File '", file, "' not found")
         if (.Platform$OS.type == "windows") {
             return(.gdbsource.win(file, interactive))
         }
         gdbscript <- tempfile()
         if (interactive) {
             gdbcmd <- c(paste("run --vanilla <", file), "bt")
             gdbcmd <- paste(gdbcmd, "\n", collapse = "")
             cat(gdbcmd, file = gdbscript)
             cmd <- paste("R -d gdb --debugger-args=\"-x", gdbscript, 
                 "\"")
             system(cmd, intern = FALSE, ignore.stdout = FALSE, ignore.stderr = TRUE)
             return(NULL)
         }
         else {
             cat("run\nbt\nquit\n", file = gdbscript)
             cmd <- paste("R --vanilla < ", file, " -d gdb --debugger-args=\"-x", 
                 gdbscript, "\"")
             txt <- system(cmd, intern = TRUE, ignore.stdout = FALSE, 
                 ignore.stderr = TRUE)
             attr(txt, "file") <- file
             class(txt) <- "backtrace"
             return(txt)
         }
     }
     
     #  < cat("run\nbt\nquit\n", file = gdbscript) > gives the gdb commands that can be used in the gdbsource() interactive mode:
     (gdb) run
     (gdb) bt  # backtrace
     (gdb) ...
     (gdb) quit 
     
     # In WSL Linux R run:
     setwd("/mnt/c/TMB_Debug")
     library(TMB)
     
     # In straight Linux
     #   - create a TMB_Debug folder and copy the 'simpleError.cpp' and simpleError.R files from the R_and_Cpp folder 
     #          in this repo to that folder. 
     #   - setwd() to the TMB_Debug folder
     
     if(file.exists('simpleError.o')) file.remove(c('simpleError.o'))
     if(file.exists('simpleError.so')) file.remove(c('simpleError.so')) 
     compile('simpleError.cpp', "-O0 -g")
     
     # Sourcing 'simpleError.R' will crash R
     # source('simpleError.R')
     
     # Running non-interactive gdbsource() gives the error at line 30
     gdbsource('simpleError.R')
     Errors:
      #3  ... in objective_function<double>::operator() ... at simpleError.cpp:30
          
     # But lets look at the Linux gdb output to compare to what we get under Windows (see below)
     gdbsource('simpleError.R', interactive = TRUE)
     (gdb) run
     (gdb) bt
     
     #0  __GI_raise (sig=sig@entry=6) at ../sysdeps/unix/sysv/linux/raise.c:51
     #1  0x00007ffff73687f1 in __GI_abort () at abort.c:79
     #2  0x00007fffeee21e2a in Eigen::DenseCoeffsBase<Eigen::Array<double, -1, 1, 0, -1, 1>, 1>::operator() (this=0x7fffffff9680, index=5)
         at /usr/local/lib/R/site-library/RcppEigen/include/Eigen/src/Core/DenseCoeffsBase.h:425
     #3  0x00007fffeee0af14 in objective_function<double>::operator() (this=0x7fffffff9770) at simpleError.cpp:30
     #4  0x00007fffeedfa290 in getParameterOrder (data=0x55555d083008, parameters=0x55555d082fb8, report=0x55555d0b1dc0, control=0x55555561afe0)
         at /home/wallacej/R/x86_64-pc-linux-gnu-library/4.1/TMB/include/tmb_core.hpp:2021
     #5  0x00007ffff78131a4 in ?? () from /usr/lib/R/lib/libR.so
     #6  0x00007ffff78136f6 in ?? () from /usr/lib/R/lib/libR.so
     #7  0x00007ffff7851cf6 in ?? () from /usr/lib/R/lib/libR.so
     #8  0x00007ffff785e700 in Rf_eval () from /usr/lib/R/lib/libR.so
     #9  0x00007ffff786051f in ?? () from /usr/lib/R/lib/libR.so
     #10 0x00007ffff78612e7 in Rf_applyClosure () from /usr/lib/R/lib/libR.so
     #11 0x00007ffff785538c in ?? () from /usr/lib/R/lib/libR.so
     #12 0x00007ffff785e700 in Rf_eval () from /usr/lib/R/lib/libR.so
     #13 0x00007ffff786051f in ?? () from /usr/lib/R/lib/libR.so
     #14 0x00007ffff78612e7 in Rf_applyClosure () from /usr/lib/R/lib/libR.so
     #15 0x00007ffff785e8d3 in Rf_eval () from /usr/lib/R/lib/libR.so
     #16 0x00007ffff7863803 in ?? () from /usr/lib/R/lib/libR.so
     #17 0x00007ffff785eaf2 in Rf_eval () from /usr/lib/R/lib/libR.so
     #18 0x00007ffff789293a in Rf_ReplIteration () from /usr/lib/R/lib/libR.so
     #19 0x00007ffff7892d11 in ?? () from /usr/lib/R/lib/libR.so
     #20 0x00007ffff7892dc8 in run_Rmainloop () from /usr/lib/R/lib/libR.so
     #21 0x000055555540083b in main ()
     #22 0x00007ffff7349c87 in __libc_start_main (main=0x555555400820 <main>, argc=2, argv=0x7fffffffde78, init=<optimized out>, fini=<optimized out>,
         rtld_fini=<optimized out>, stack_end=0x7fffffffde68) at ../csu/libc-start.c:310
     #23 0x000055555540087a in _start ()
     
     # Note the line #3 gives the file name and the line that the error occurs:  "at simpleError.cpp:30"
     
     # TMB::print.backtrace() greps that pattern and returns the line number the error occurs on
     function (x, ...) 
     {
         pattern <- "\\ at\\ .*\\.cpp\\:[0-9]+"
         x <- grep(pattern, x, value = TRUE)
         if (length(x) == 0) 
             x <- "Program returned without errors"
         else x <- c("Errors:", x)
         cat(paste(x, "\n"))
     }
     
          
     # -- Under Windows --
     
     # If not already done, create the TMB_Debug folder and copy the 'simpleError.cpp' and simpleError.R files from 
         the R_and_Cpp folder in this repo to that folder.
     
     # Running 64-bit R ver 4.2.1 with <C:\Rtools\mingw_64\bin> (where 64-bit gdb ver 7.9.1 is found) in the system path.  
     #      Rtools is version 3.5.0.4   (FYI, the system path also has <C:\Rtools\bin> before <C:\Rtools\mingw_64\bin>)
     setwd('C:/TMB_Debug')
     library(TMB)
     
     if(file.exists('simpleError.o')) file.remove(c('simpleError.o'))
     if(file.exists('simpleError.dll')) file.remove(c('simpleError.dll')) # Windows dll 
     compile('simpleError.cpp', "-O0 -g")
     
     # Sourcing 'simpleError.R' will crash R
     # source('simpleError.R')
     
     
     # Note in WIndows,  TMB:::.gdbsource.win()   is used:
     
     function (file, interactive = FALSE) 
     {
         gdbscript <- tempfile()
         txt <- paste("set breakpoint pending on\nb abort\nrun --vanilla -f", 
             file, "\nbt\n")
         cat(txt, file = gdbscript)
         cmd <- paste("gdb Rterm -x", gdbscript)
         if (interactive) {
             cmd <- paste("start", cmd)
             shell(cmd)
             return(NULL)
         }
         else {
             txt <- system(cmd, intern = TRUE, ignore.stdout = FALSE, 
                 ignore.stderr = TRUE)
             attr(txt, "file") <- file
             class(txt) <- "backtrace"
             return(txt)
         }
     }
     
     # Running non-interactive gdbsource() gives
     gdbsource('simpleError.R')
     Program returned without errors 
     
     # "Program returned without errors" doesn't mean there are not errors, just that <at simpleError.cpp:30> is not present.
     
     # Try using interactive command
     gdbsource('simpleError.R', interactive = TRUE)
     
     # If the system path doesn't contain the Rterm(), you will see:
     # ...
     # Rterm: No such file or directory.
     # ...
     
     # Having already dealt with the path for GDB, I used Sys.which() in the upated .gdbsource.win() code snippet below:
     
     file <- 'simpleError.R'
     gdbscript <- tempfile()
     txt <- paste("set breakpoint pending on\nb abort\nrun --vanilla -f", file, "\nbt\n")
     cat(txt, file = gdbscript)
     # file.show(gdbscript)  # Look at the commands in gdbscript temp file, if desired.
     
     cmd <- paste0("gdb ", Sys.which('Rterm'), " -x ", gdbscript)
     (cmd <- paste("start", cmd))
     shell(cmd)
     
     # which immediately (no gdb commands given) gives:
     
     TMB has received an error from Eigen. The following condition was not met:
     index >= 0 && index < size()
     Please check your matrix-vector bounds etc., or run your program through a debugger.
     
     Breakpoint 1, 0x00007ffcb219f1e7 in msvcrt!abort () from C:\WINDOWS\System32\msvcrt.dll
     #0  0x00007ffcb219f1e7 in msvcrt!abort () from C:\WINDOWS\System32\msvcrt.dll
     #1  0x0000000065ba0957 in simpleError!_ZN5Eigen15DenseCoeffsBaseINS_5ArrayIdLin1ELi1ELi0ELin1ELi1EEELi1EEclEx ()
        from C:\TMB_Debug\simpleError.dll
     #2  0x0000000065b2ecff in simpleError!_ZN18objective_functionIdEclEv () from C:\TMB_Debug\simpleError.dll
     #3  0x0000000065b044f7 in simpleError!getParameterOrder () from C:\TMB_Debug\simpleError.dll
     #4  0x000000006c7a7b96 in Rf_NewFrameConfirm () from C:\MRO\MRO\bin\x64\R.dll
     #5  0x000000006c7a886d in Rf_NewFrameConfirm () from C:\MRO\MRO\bin\x64\R.dll
     #6  0x000000006c7ed189 in R_initAssignSymbols () from C:\MRO\MRO\bin\x64\R.dll
     #7  0x000000006c7fcbf1 in Rf_eval () from C:\MRO\MRO\bin\x64\R.dll
     #8  0x000000006c7fe907 in R_cmpfun1 () from C:\MRO\MRO\bin\x64\R.dll
     #9  0x000000006c7ffb6a in Rf_applyClosure () from C:\MRO\MRO\bin\x64\R.dll
     #10 0x000000006c7f4f54 in R_initAssignSymbols () from C:\MRO\MRO\bin\x64\R.dll
     #11 0x000000006c7fcbf1 in Rf_eval () from C:\MRO\MRO\bin\x64\R.dll
     #12 0x000000006c7fe907 in R_cmpfun1 () from C:\MRO\MRO\bin\x64\R.dll
     #13 0x000000006c7ffb6a in Rf_applyClosure () from C:\MRO\MRO\bin\x64\R.dll
     #14 0x000000006c7fcd9c in Rf_eval () from C:\MRO\MRO\bin\x64\R.dll
     #15 0x000000006c801e4a in R_execMethod () from C:\MRO\MRO\bin\x64\R.dll
     #16 0x000000006c7fcfe5 in Rf_eval () from C:\MRO\MRO\bin\x64\R.dll
     #17 0x000000006c827291 in Rf_ReplIteration () from C:\MRO\MRO\bin\x64\R.dll
     #18 0x000000006c827601 in Rf_ReplIteration () from C:\MRO\MRO\bin\x64\R.dll
     #19 0x000000006c82769d in run_Rmainloop () from C:\MRO\MRO\bin\x64\R.dll
     #20 0x000000006c82773e in Rf_mainloop () from C:\MRO\MRO\bin\x64\R.dll
     #21 0x000000000040171a in ?? ()
     #22 0x000000000040158a in ?? ()
     #23 0x00000000004013c5 in ?? ()
     #24 0x000000000040152b in ?? ()
     #25 0x00007ffcb39f7034 in KERNEL32!BaseThreadInitThunk () from C:\WINDOWS\System32\kernel32.dll
     #26 0x00007ffcb3cc2651 in ntdll!RtlUserThreadStart () from C:\WINDOWS\SYSTEM32\ntdll.dll
     #27 0x0000000000000000 in ?? ()
     Backtrace stopped: previous frame inner to this frame (corrupt stack?)
     (gdb)
     
     
     # Note that there is no "\\ at\\ .*\\.cpp\\:[0-9]+" pattern that gives the line number of the error for print.backtrace() to find.
     
    
