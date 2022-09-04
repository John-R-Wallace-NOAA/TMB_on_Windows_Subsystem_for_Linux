
- Since Jeroen Ooms took over maintaining Rtools from Prof. Brian Ripley and Duncan Murdoch gdb.exe has not been available in 
    RTools 4.0 nor in Rtools 4.2 ( https://cran.r-project.org/bin/windows/Rtools/ ). 
    
- Mark Bravington tried to add gdb to RTOOLS40 on Windows 10 without success: https://stat.ethz.ch/pipermail/r-devel/2021-April/080623.html .
       Tomas Kalibera pointed out in response that gdb in the Rtools 3.5 installation can still be used, as I have done for R ver 4.2.1 (see Linux_vs_Windows_gdb_errors.md). 
       As Mark points out, it does not work to just copy over gdb.exe from Rtools 3.5 (ver 7.9.1). (I also tried older RTools gdb.exe versions without success.)
    
- A standalone gdb.exe ver 10.2 that does work with R ver 4.2.1 is here: < http://www.equation.com/servlet/equation.cmd?fa=gdb >.
    However, I was unable to get the backtrace to work like gdb version 7.9.1 (from Rtools 3.5). The 10.2 version only stated that:
    "Backtrace stopped: previous frame identical to this frame (corrupt stack?)"
    
- RTools 4.2 has major changes, with all executables now in the folders 'x86_64-w64-mingw32.static.posix' and 'C:\rtools42/usr/bin'. An empty skeleton
      of folders still exists. Also, "All libraries are included, instead of relying on external sources for downloading them. 
      Rtools42 takes slightly over 3G when installed."
      
- RTools 4.0 and 4.2 use an environment variable: "RTOOLS4X_HOME" which gives the path of where Rtools 4.X is located (defaults to C:\rtools4X).
 
- On R ver 4.2.1 the environment path is then prepended with: C:\rtools42/x86_64-w64-mingw32.static.posix/bin and C:\rtools42/usr/bin 
   and in R:
   
      Sys.which('make') #  gives:
                                   make 
      "W:\\rtools42\\usr\\bin\\make.exe" 
   
- On my devices 4.0.X and 4.1.X do not prepend the path and 
-    
      Sys.which('make') # gives:
                            make 
      "c:\\Rtools\\bin\\make.exe" 
   
  since Rtools\bin is further down the path.
  
- However with a clean install of R ver 4.2.1 and Rtools 4.2, without Rtools 3.5 installed, TMB::gdbsource() under Windows
    will fail due to a missing gdb.exe.
    
- StackOverFlow: https://stackoverflow.com/questions/18407563/gcc-doesnt-produce-line-number-information-even-with-g-option
   points out a dwarf4 versus dwarf2 issue under Linux (which may be @iperedaagirre's issue) but didn't help me with line numbers under Windows.
   
- With the many options I have tried, I have not been able to get any gdb version under Windows 10 to give line numbers and hence the reason for this WSL repo.     
   
- To edit the user system path and other variables on a Windows 10 machine without Admin rights, search for 'env' and 
       click on 'Edit environmental variables for your account'.
     
