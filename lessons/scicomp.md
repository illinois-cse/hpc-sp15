[![](https://bytebucket.org/davis68/resources/raw/f7c98d2b95e961fae257707e22a58fa1a2c36bec/logos/baseline_cse_wdmk.png?token=be4cc41d4b2afe594f5b1570a3c5aad96a65f0d6)](http://cse.illinois.edu/)

# Introduction to Scientific Computing on a Cluster
#### Computational Science and Engineering Training • Spring 2015

This workshop is a bit of a challenge, and I hope it will gel together well.  This is intended to give you a whirlwind tour of the issues underlying scientific computing, particularly as that is executed today on distributed clusters.  What you should walk out of here with is at least a list of topics to dig deeper into, because semester-long courses could be written with the outline I am using to introduce scientific computing.



## What is Scientific Computing?
    -   Building computational models of physical phenomena for simulation
    -   Often requires massive linear-algebraic solution methods—thus HPC
    -   This workshop is designed to answer the question, "I've got an account on a system—now what do I do?"



## What is High-Performance Computing (HPC)?

[Slides](http://slides.com/uiuc-cse/hpc-fa14-2#/)

#### Strategies
<blockquote>
If you were plowing a field, which would you rather use: Two strong oxen or 1024 chickens?  (Seymour Cray)
</blockquote>

-   Concepts
    -   vectorization
    -   distributed computing
    -   threading
    -   ([Davis et al. 2012](http://link.springer.com/article/10.1007%2Fs11227-012-0789-3))

-   Hardware mapping
    -   multicore
    -   GPGPU
    -   cluster (nodes and head node)

-   Nomenclature
    -   Performance:  strong & weak scaling
        -   Speedup:
            $$S(N, P) = t_{N,P=1} / t_{N,P}$$
        -   Strong scaling:  how the solution time varies with the number
            of processors P for a fixed total problem size N.
            $$E(N) = t_1 / (N t_N)$$
        -   Weak scaling:  how the solution time varies with the number of
            processors P for a fixed problem size per processor N/P.
            $$E(N) = (t_1 / t_N)$$

-   Implementations (software libraries)
    -   Architecture:  MPI, OpenMP, CUDA/OpenCL
    -   Numerics:  BLAS, LAPACK, GNU SSL, Intel MKL, Trilinos



## Executing Scientific Code

-   Environment:  `$PATH`, `module`

-   Shells & architecture (graphic)
    -   We assume that you are comfortable with Linux and the command line, 
        since that's the operating system for every cluster in use today.

-   Queueing

-   Job Scripts (`mpiexec`, etc.)

-   Checkpoint often if your code supports it.



## Compiling Scientific Code

    wget https://github.com/uiuc-cse/hpc-sp15/raw/gh-pages/lessons/hpc-code-examples.tar.gz
    tar -xvzf hpc-code-examples
    module load gcc

-   Compiling & building (`gcc`, `make`)
    -   `$LD_LIBRARY_PATH$
    -   Examples:  OpenMP, MPI, GNU SSL, Intel MKL
        -   MPI:  why would you want to request fewer cores per node?
    -   `./configure ; make ; make install` as a common workflow
        -   `mpicc` compiler wrapper including MPI headers and linked libraries
        -   `-show`, `-std=c99`, `-o`
        -   OpenMP:  `$OMP_NUM_THREADS` environment variable
        -   local installations (`ln` and `~/bin`)
        -   troubleshooting with libraries:  try `ldd mpiexec`



## Developing Scientific Code
<blockquote>
Why do we never have time to do it right, but always have time to DO IT OVER?  (Anonymous)
</blockquote>

- Design then code.

- Daisie Huang [Scientific coding and software engineering: what's the difference?](http://www.software.ac.uk/blog/2015-02-06-scientific-coding-and-software-engineering-whats-difference)

- Carry out studies of the strong and weak scaling, as well as profiling so that you aren't wasting cluster cycles.


### Design

*Design* refers to the decisions you need to make about what the software will do and how it will work.  This includes deciding the language and libraries that you require, and the target platform.

- Modularize your code as much as practical.

- Don't reinvent the wheel—find existing libraries:  Boost, SciPy, Netlib, Scalapack, PETSc, etc.

-   Language support and libraries for HPC
    -   Fortran; C/C++; Python; R; MATLAB (why to choose each)
    -   MKL (BLAS/LAPACK/ScaLAPACK,BLACS), OpenBLAS; CUDA; MVAPICH2, OpenMPI;
        GSL; Boost; PETSc
    -   MPI:  MVAPICH2 v. OpenMPI:  broadly speaking, MVAPICH2 exhibits better
        scaling than OpenMPI for many problems at higher numbers of nodes
        and bandwidth [source](http://hpc.inspur.com/images/News/2012/11/23/11EB9AD954D64F86AF46AADB2ECF289E.pdf)
    -   Compiler versions and features (Intel, GCC (esp. 4.9+))
    -   [Language microbenchmarks across languages](http://julialang.org/benchmarks/)


### Construction

*Construction* refers to the process of actually coding the software.

You are not a serious software developer.  That is not to say that you are not serious, nor that you do not develop software; however, you think of yourself as an engineer first and as a coder only incidentally.

There are extremely sophisticated tool chains in use in software development today, but we are going to highlight only a few immediately useful selections.

#### External to coding
- Take the time to learn your development environment of choice well—there are language-aware code editors including code completion:  IDEs (Eclipse, XCode), `vi` (YouCompleteMe), `emacs`.

- Write self-documenting code:  good variable and function names that follow a convention.  If you don't have a convention yet, look at a lot of code on GitHub and see what you like, but be consistent.
    -   What does "using a convention" mean?
    -   [GNU coding standards](https://www.gnu.org/prep/standards/html_node/index.html)


- Use a compilation system:
    - [Make](https://www.gnu.org/software/make/)
    - [CMake](http://www.cmake.org/)
    - [GNU Autotools](https://en.wikipedia.org/wiki/GNU_build_system) (`./configure; make; make install`), etc.
    [Autotools tutorial](https://www.lrde.epita.fr/~adl/autotools.html)
    -    This also lets you query the hardware to optimize at compile time.

            >configure.ac
                AC_INIT(cache_test.c)
                AM_INIT_AUTOMAKE(cache_test, 1.0)
                : ${CFLAGS="-O2 -std=c99"}
                AC_PROG_CC
                AC_OUTPUT(Makefile)
            aclocal
            autoconf
            >Makefile.am
                bin_PROGRAMS = cache_test
                cache_test_SOURCES = cache_test.c
            automake --add-missing
            touch NEWS README AUTHORS ChangeLog

            ./configure
            make
        
        -   [Overview of Autotools toolchain](http://ptgmedia.pearsoncmg.com/images/1578701902/samplechapter/1578701902.pdf)

- Use version control:  Git, SVN, Mercurial, etc.
    -   `git clone https://github.com/losalamos/CLAMR`

- Use an issue-tracking system:  GitHub issues, Bugzilla, etc.

- Don't create new file formats:  find something existing that works (`HDF5`, `PDB`, `XYZ`, etc.).

#### Internal to coding
- Use configuration files and command-line flags rather than hard-coding values.

- Compile with all debug flags set (`-Wall -Werror -g`), and aim to compile with zero warnings.

- Avoid loops.  Unroll them if you must use them.

- Optimize aggressively (`gcc -O3`), but only once you've finished debugging the code.

Other potentially useful utilities include documentation generators and integrated development environments.


### Access

- Always include certain standard files when distributing your software:  `README`, `INSTALL`, `CITATION`, `LICENSE`.

- Have a long-term data management plan (now a requirement for NSF, other grants).


### Testing
<blockquote>
"Before software can be reusable it first has to be usable. (Ralph Johnson)"
</blockquote>

- Learn to use (and then actually use!) debugging and profiling tools.

    - Debuggers let you step through your code line-by-line and view the contents of variables and the results of evaluations _interactively_.  (This is invaluable for C/C++/Fortran programming.)
    [`gdb`](https://www.gnu.org/software/gdb/)
    
    - Profilers query the code frequently to build up a picture of how often your execution point is in any one part of the program.  This lets you characterize performance and identify bottlenecks.
    `gprof`
    [TAU](https://www.cs.uoregon.edu/research/tau/home.php)
    [Valgrind]() is an extremely versatile tool which lets you characterize memory performance, find memory leaks, and profile your code.  `memcheck` and `cachegrind` are two of the most commonly used utilities.
        $ module load valgrind
        $ valgrind --tool=memcheck --leak-check=yes --show-reachable=yes -v ./cache_test 10000
        ==12877== Memcheck, a memory error detector
        ==12877== Copyright (C) 2002-2013, and GNU GPL'd, by Julian Seward et al.
        ==12877== Using Valgrind-3.9.0 and LibVEX; rerun with -h for copyright info
        ==12877== Command: ./cache_test 10000
        ...
        --12877-- Valgrind library directory: /usr/local/apps/valgrind/3.9.0/lib/valgrind
        ...
        Matrix size:  10000x10000
        --12877-- REDIR: 0x4eaa520 (free) redirected to 0x4c273fd (free)
        --12877-- REDIR: 0x4ea9640 (malloc) redirected to 0x4c27a23 (malloc)
        ==12877== 
        ==12877== HEAP SUMMARY:
        ==12877==     in use at exit: 0 bytes in 0 blocks
        ==12877==   total heap usage: 10,001 allocs, 10,001 frees, 400,080,000 bytes allocated
        ==12877== 
        ==12877== All heap blocks were freed -- no leaks are possible
        ==12877== 
        ==12877== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 6 from 6)
        --12877-- 
        --12877-- used_suppression:      6 dl-hack3-cond-1 /usr/local/apps/valgrind/3.9.0/lib/valgrind/default.supp:1196
        ==12877== 
        ==12877== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 6 from 6)
    
        $ valgrind --tool=cachegrind ./cache_test 10000
        ==12881== Cachegrind, a cache and branch-prediction profiler
        ==12881== Copyright (C) 2002-2013, and GNU GPL'd, by Nicholas Nethercote et al.
        ==12881== Using Valgrind-3.9.0 and LibVEX; rerun with -h for copyright info
        ==12881== Command: ./cache_test 10000
        ==12881== 
        --12881-- warning: L3 cache found, using its data for the LL simulation.
        Matrix size:  10000x10000
        ==12881== 
        ==12881== I   refs:      504,082,163
        ==12881== I1  misses:            868
        ==12881== LLi misses:            860
        ==12881== I1  miss rate:        0.00%
        ==12881== LLi miss rate:        0.00%
        ==12881== 
        ==12881== D   refs:      201,473,856  (100,934,392 rd   + 100,539,464 wr)
        ==12881== D1  misses:    112,537,715  ( 12,524,791 rd   + 100,012,924 wr)
        ==12881== LLd misses:     97,883,166  (     54,665 rd   +  97,828,501 wr)
        ==12881== D1  miss rate:        55.8% (       12.4%     +        99.4%  )
        ==12881== LLd miss rate:        48.5% (        0.0%     +        97.3%  )
        ==12881== 
        ==12881== LL refs:       112,538,583  ( 12,525,659 rd   + 100,012,924 wr)
        ==12881== LL misses:      97,884,026  (     55,525 rd   +  97,828,501 wr)
        ==12881== LL miss rate:         13.8% (        0.0%     +        97.3%  )

First, learn to use debugging and profiling tools.  Two that are supported on Campus Cluster are `gdb` and `valgrind`.  `gdb`, the GNU Debugger, is designed to work with many programming languages besides C, but it can be painful to get it to work with a parallel program.  (Actually, any parallel debugging is uniquely painful.)  Anyway, if you intend to use `gdb`, you need to compile your code using the `-g` flag and `gcc`.

The other tool, `valgrind`, monitors memory behavior.  It can detect cache misses, memory leaks, and other problems which can lead to poor code performance and excessive memory demand.

- Develop and use a unit-testing scheme (even a simple one).
    Unit tests are ways of (1) confirming that the code you write runs as it should; and (2) verifying that changes you have made to a function do not break expected functionality.
    Now, formally, you're supposed to write the tests before you write the code, but most of us engineers don't map out our code ahead of time so specifically that that's feasible.  So we'll just look at an example of the type of tests you can do.
    -   `interval`
            git clone https://github.com/uiuc-cse/interval.git

    - [Stack Overflow:  Is it worthwhile to write unit tests for scientific research codes?](https://scicomp.stackexchange.com/questions/206/is-it-worthwhile-to-write-unit-tests-for-scientific-research-codes)

    - [Check](http://check.sourceforge.net/) is a unit-testing framework for C code.  


### Debugging
<blockquote>
It’s hard enough to find an error in your code when you’re looking for it; its even harder when you’ve _assumed_ your code is _error-free_.  (Steve McConnell)
</blockquote>

-   Your confusion is a clue that something is wrong—don't ignore it.
    -   Either your mental model of what the code is doing is wrong;
    -   or the model you have built isn't correct for the problem you've set it.

-   Premature optimization is the root of all evil.  (Donald Knuth)

-   `gdb` is a representative debugger.  Graphical interfaces for this and other tools exist, but behind the scenes they are just interacting with this program.

[![](http://imgs.xkcd.com/comics/future_self.png)](http://xkcd.com/)

-   Numerical error worksheet ([`numerical-error.ipynb`]())

---
<a id='credits'></a>
## Credits

Neal Davis developed these materials for [Computational Science and Engineering](http://cse.illinois.edu/) at the University of Illinois at Urbana–Champaign.  _Some tips were suggested by Jay Alameda and Mark van Moers of NCSA._

<img src="http://i.creativecommons.org/l/by/4.0/88x31.png" align="left">
This content is available under a [Creative Commons Attribution 4.0 Unported License](https://creativecommons.org/licenses/by/4.0/).

[![](https://bytebucket.org/davis68/resources/raw/f7c98d2b95e961fae257707e22a58fa1a2c36bec/logos/baseline_cse_wdmk.png?token=be4cc41d4b2afe594f5b1570a3c5aad96a65f0d6)](http://cse.illinois.edu/)