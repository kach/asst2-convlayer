# CS348K (Mini-) Assignment 2: <br/> Optimizing a Convolutional Layer in Halide #

In this assignment you will optimize the performance of Halide code that evaluates a convolutional layer. 
Implementing a conv layer in Halide [is quite easy](http://cs348k.stanford.edu/spring20/lecture/dnneval/slide_024) (we give you the algorithm in Halide in the starter code). The challenge scheduling the layer efficiently using performance optimization ideas from class: SIMD vector processing, multi-core execution, and efficient blocking for cache locality. 

**You are allowed to use the reference Halide algorithm provided in the codebase verbatim** (see `StudentConvLayerGenerator` in `conv_layer_generators.cpp`). However, to improve the performance you will need to write an efficient Halide schedule. The starter code uses a naive/default Halide schedule, which corresponds to an evaluation order equivalent to code with loops that look like:

```
  // Initialization
  for n:
    for z:
      for y:
        for x:
          conv(...) = ...
  // Updates
  for n:
    for z:
      for y:
        for x:
          for r0:
            for r1:
              for r2:
                conv(...) = ...
```

Your job is to implement a custom Halide schedule that performs better than the default. (See [`Halide::Func::print_loop_nest()`](http://halide-lang.org/docs/class_halide_1_1_func.html#a365488c2eaf769c61635120773e541e1) to inspect and debug your schedule like this.) 

In general, this is a free-for-all assignment.  We want to you learn a bit about writing Halide schedulers, and try your hand at making performance go faster.  You *will* have to do some documentation learning and reading on your own.

# Resources and Documentation # 
* [Halide tutorials](http://halide-lang.org/tutorials/tutorial_introduction.html). In particular, see Tutorial 01 for a basic introduction, Tutorial 07 for a convolution example, and Tutorial 05 for an introduction to Halide schedules, and Tutorial 08 for more advanced scheduling topics.
* [Exhaustive Halide documentation](http://halide-lang.org/docs/). 

# Assignment mechanics #

__Step 1: Grab the assignment starter code:__

    git clone git@github.com:stanford-cs348k/asst2-convlayer.git
   
The codebase uses a simple `Makefile` as the build system. However, there is a dependency on Halide.  

__Step 2: Install Halide:__

To build the starter code, run `make` from the top level directory. The driver source code is in `src/`, and
the implementation of the convolution layer generator you will modify is in the top-level file `conv_layer_generators.cpp`.
Object files and binaries will be populated in `build/` and `bin/` respectively.

To install and use Halide follow the instructions at http://halide-lang.org/. In particular, you should [download a binary release of Halide](https://github.com/halide/Halide/releases). Once you've downloaded and untar'd the release, say into directory `halide_dir`, change the previous lines back, and also the following line in `Makefile`

    HALIDE_DIR=/MY/PATH/TO/HALIDE

to

    HALIDE_DIR=<halide_dir>

__Step 3: Build the code:__

    make

This creates a binary `bin/convlayer`.

__Running the starter code:__

To get commandline help, run the command:

    ./bin/convlayer -h

To run your scheduled conv layer try:

    ./bin/convlayer --schedule student

This code will run your version of the convolution layer using randomly generated inputs and weights. It will run for 3 trials, and report the minimum time of the 3 runs. To run correctly you must ensure that
Halide is in your library load path. On OSX this can be done like so:

    DYLD_LIBRARY_PATH=<halide_dir>/bin ./bin/convlayer <args>

and on Linux it will be:

    LD_LIBRARY_PATH=<halide_dir>/bin ./bin/convlayer <args>

__Modifying the code__

Programming in Halide is a form of [meta-programming]https://en.wikipedia.org/wiki/Metaprogramming().  Halide is embedded in C++, so you write C++ code that in turn generates a Halide program representation.  Then Halide compiles this representation into a library, which is linked by the binary `convlayer.`.  

In the code base, a generator is a C++ class that creates a Halide program.  Your modifications to the code should only go in the file `conv_layer_generators.cpp` inside the class `StudentConvLayerGenerator`. Inside that file you should not modify the implementation of the convolution layer algorithm. You should only add Halide scheduling directives to the program to make it run faster. Regions you should modify will be marked by comments that look like so:

    // BEGIN: CS348K STUDENTS MODIFY THIS CODE
    
    // END: CS348K STUDENTS MODIFY THIS CODE

We have provided a reference implementation in `conv_layer_generators.cpp` called `DefaultConvLayerGenerator`. This implementation generates a conv layer using the default Halide schedule. The code generated by `DefaultConvLayerGenerator` is called by `HalideConvolutionLayer`, which is located in the source file `./src/default_convolution_layer.cpp`.

# Handin and Grading #

Your handin should contain two components:
   * Your full source tree (as a zip file)
   * A writeup, in a file named `writeup.pdf`, that describes the iterative process you used to arrive at your solution.  At each step, we expect the writeup to say:
       * I tried XXX for the following reason.
       * Then I measured my performance and I saw an increase/decrease, etc.
       
This assignment is all about performance optimization, but we are not grading on the actual performance obtained.  We are grading on a three category basis: no-credit, credit, amazing.  Anyone would makes a legimate effort to optimized the code should get credit.  Students that obtain impressively high performance will get "amazing"!

