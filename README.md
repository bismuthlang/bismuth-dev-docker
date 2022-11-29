## About
This is a docker image that might be useful if you're in CS 4533 (Techniques of Programming Language Translation) or CS 544 (Compiler Construction). Right now, it just you a ready-to-use antlr shared library, as well as cmake, llvm, lcov (code coverage), and other basic tools. However, the toolchain can be extended to incorportate other tools as needed.

Thank you to [Jack Sullivan](https://github.com/jhsul) for making this initial [repository](https://github.com/jhsul/cs4533-docker) and [docker image](https://hub.docker.com/r/jackhsullivan/cs4533-docker) from which I have extended and build my own image. 

## Downloading the Docker Image

This takes a long time but you only need to do it once:

```sh
docker pull ahfriedman/cs544-cs4533-docker
```

## Running the Docker Image 

Run with:

```sh
docker run -it -v ~/<your_class_folder>:/home/shared ahfriedman/cs544-cs4533-docker
```

### Calculator Example

On your *host* machine, clone the course repository 

```sh
git clone https://bitbucket.org/gfp-public-course-materials/compiler-projects-all/src/master/ ~/cs4533
```

Run the docker image with the shared folder

```sh
docker run -it -v ~/cs4533/1-calculator-starter:/home/shared jackhsullivan/cs4533-docker
```

## Using the Docker Image

Once you have run the docker image with the shared folder, you should be prompted with a terminal. The terminal will start in the working directory of /home/. The project files (the shared folder) is located at /home/shared. 

The first time you open the project in the docker image, you'll want to do is copy the antlr runtime into the project's files as shown below *(Note: this should only need to be done once. Afterwards, the files should stay updated across runs of the docker image)*.

```sh
cd /home
cp libantlr4-runtime.a shared/antlr/lib/
cd shared
```

Once in the `/home/shared` directory (assuming that it is also the top-level directory of your project), you should be abel to build and run the project as per usual: 

```sh
cmake -S . -B build
cmake --build build
cmake --install build
install/calculator
```

You should get:
```
OUTPUT
(<EOF><EOF> <EOF> <EOF>)
```

## Common Issues

This section describes a few common issues you may encounter when using this container
to run the sample code. 

**NOTE: Whenever making changes to the files related to the project's build process (basically anything aside from your code) it is a good idea to remove the existing build folder and rebuild the entire project as this is sometimes required for these kinds of changes to take effect.**

### LLVM Not Linking/LLVM Not Found

In the starter code, LLVM defaults to the path `/usr/local/opt/llvm`; however, in this container, 
LLVM usually ends up in `/usr/lib/llvm-14` (the last number, in this case 14, may change depending on llvm version).
To fix this, update the `cmake/LLVM.cmake` file to correctly reference your LLVM installation. 

### ANTLR Not Linking/ANTLR Not Found

This issue can sometimes be resolved by editing the 
`target_link_libraries` command in *both* 
`src\CMakeLists.txt` and `src\test\CMakeLists.txt` to 
include `parser_lib`. For example, the corrected command 
may appear as: 

```
target_link_libraries(tests 
  PRIVATE
  ${ANTLR_RUNTIME_LIB}
  parser_lib
  lexparse_lib
  sym_lib
  semantic_lib
  utility_lib
  codegen_lib
  ${LLVM_LIBS}
  Catch2::Catch2WithMain
)
```

### ANTLR/LLVM Linked, but functions undefined

This project can be sensitive to which compiler you use--in particular, clang seems to work better than gcc (which would make sense given that we are working with llvm); however, on Linux, the default compiler tends to be `gcc`. While this docker contaner attempts to set `clang` and `clang++` to be used as your default compiler, if you are still running into issues, it may be worthwhile to rebuild your project (and your ANTLR library) with `CC=clang CXX=clang++` prefixed to all cmake commands. For example, 
`cmake -S . -B build` becomes `CC=clang CXX=clang++ cmake -S . -B build` and `cmake --build build` becomes `CC=clang CXX=clang++ cmake --build build`. 
