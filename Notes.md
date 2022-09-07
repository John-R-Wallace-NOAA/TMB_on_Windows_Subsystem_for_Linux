
- Jeroen Ooms took over maintaining RTools for Windows R from Prof. Brian Ripley and Duncan Murdoch starting with RTools 4.0.  Neither RTools 4.0 nor RTools 4.2 have gdb.exe available: https://cran.r-project.org/bin/windows/Rtools/ 

- Mark Bravington tried to add gdb to RTOOLS40 on Windows 10 without success: https://stat.ethz.ch/pipermail/r-devel/2021-April/080623.html . 

   - Tomas Kalibera pointed out in response that gdb in the Rtools 3.5 installation can still be used, as I have done for R ver 4.2.1 (see Linux_vs_Windows_gdb_errors.md in this repo). 
      
    - As Mark points out, it does not work to just copy over gdb.exe from Rtools 3.5 (ver 7.9.1). (I also tried older RTools gdb.exe versions without success.)
    
- A Windows standalone gdb.exe ver 10.2 that does work with R ver 4.2.1 is here: http://www.equation.com/servlet/equation.cmd?fa=gdb . 
    However, I was unable to get the backtrace to look similar to gdb version 7.9.1's backtrace. The 10.2 version only stated that:
    "Backtrace stopped: previous frame identical to this frame (corrupt stack?)" 
    
- RTools 4.2 has major changes, with all executables now in the folders 'x86_64-w64-mingw32.static.posix' or 'C:\rtools42/usr/bin'. An empty skeleton
      of folders does still exist. Also, "All libraries are included, instead of relying on external sources for downloading them. 
      Rtools42 takes slightly over 3G when installed."
      
- RTools 4.0 and 4.2 use an environment variable: "RTOOLS4X_HOME" which gives the path of where Rtools 4.X is located (defaults to C:\rtools4X).
 
- On R ver 4.2.1 the environment path inside of R is then prepended with: C:\rtools42/x86_64-w64-mingw32.static.posix/bin and C:\rtools42/usr/bin 

      strsplit(Sys.getenv("PATH"), ';')
      [[1]]
       [1] "W:\\rtools42/x86_64-w64-mingw32.static.posix/bin"                     
       [2] "W:\\rtools42/usr/bin"
       ...
     
     
  and in R:
   
      Sys.which('g++')  # gives:
                                                                g++ 
      "W:\\rtools42\\x86_64-w64-mingw32.static.posix\\bin\\g++.exe" 
   
- On my devices, R ver 4.0.X and 4.1.X with RTools 4.0 do not prepend the path and 
   
      Sys.which('g++')  # gives:
                                       g++ 
      "W:\\Rtools\\mingw_64\\bin\\g++.exe"
   
  since old Rtools 3.5 'Rtools/mingw_64/Rtools/bin' is still in the path.
  
- However with a clean install of R ver 4.2.1 and Rtools 4.2, without Rtools 3.5 installed, TMB::gdbsource() under Windows 10
    fails due to the missing gdb.exe.
    
- StackOverFlow: https://stackoverflow.com/questions/18407563/gcc-doesnt-produce-line-number-information-even-with-g-option
   points out a dwarf4 versus dwarf2 options issue under Linux (which may be the cause of @iperedaagirre's TMB Issue #367) but didn't help me with line numbers in the gdb.exe error reporting under Windows.
   
- With the many options I have tried, I have not been able to get any gdb version under Windows 10 to give line numbers as Linux does and hence the reason for this WSL repo. See Linux_vs_Windows_gdb_errors.md in this repo.
   
- To edit the user system path and other variables on a Windows 10 machine without admin rights, search for 'env' and 
       click on 'Edit environmental variables for your account'. With admin rights, search for 'env', select 'Edit the system evironmental variables', select the 'Advanced' tab and click on 'Environmental Variables' at the bottom. 
     
