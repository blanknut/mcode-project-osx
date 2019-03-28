# HP-41 MCODE Example Project on OSX

This is my setup for HP-41 MCODE (Machine Language) projects. It uses
the HP-41 Software Software Development Toolkit (SDK41) available from
the [HP-41 Archive Web Site](http://hp41.org/) to compile, link and
debug the project. As the SDK41 binaries are DOS programs, I use the
amazing [DOSBox emulator](https://www.dosbox.com/) to run them.

As my personal computer is a MacBook Pro running OSX, you probably have
to adjust a few things if you're on a Linux system. Windows users may
give [Cygwin](https://www.cygwin.com/) a try.

To follow the explanations below, you should be familiar with the basic
usage of SDK41.

## Prerequisites
First, you  have to install DOSBox and make sure it runs fine. Afterwards,
get SDK41 and copy it to a folder of your choice. Set the environment
variable `SDK41` to point to this directory.

If you also want to deploy the final ROM image to your
[HP-41CL](http://www.systemyde.com/hp41/) calculator you have to set
environment variable CLUPDATE to the pathname of the clupdate.jar tool
and environment variable USBSERIAL to the device name of your USB serial
connector (on my system the device name is `/dev/tty.usbserial-A1068IY7`).

## Project Organization
The root of the project folder contains the build script `build.sh`.
It's a simple bash script offering the following build targets:
* _clean_: remove all build output files
* _build_: build the MCODE project ROM image
* _test_ (depends on build): run the HP-41 emulator for this project
* _deploy_ (depends on build): deploy the ROM image to a HP-41CL calculator

The MCODE source files as well as the linker file (.lnk) and the loader
file (.lod) are located below the `src` subfolder. Please note that these
files _must use CR LF line terminators_. Usually, editors allow to choose
which line terminators to use when saving a file. If you want to check
if the source files have the proper line endings, you can execute the
following command:
```
> file src/*
```

The `build` subfolder will contain all the output files of the build
process and the standard ROM images from SDK41.

## Usage
* Clone this project.
* Change to the project folder.
* Run `./build.sh` to build the example ROM image.
* run `./build.sh test` to build and run the example ROM image in the
  HP-41 emulator.

### Deploy to HP-41CL Calculator
The _deploy_ build target assumes that the clupdate tool is available
on your computer and that your HP-41CL calculator is connected.
In parallel to the build command you have to run a small program on your
HP-41CL calculator to receive the ROM image. I use something like:
```
LBL 'RCVROM'
TURBO50
UPLUG2L
SERINI
BAUD48
"80D000-0FFF"
YIMP
"80D-RAM"
PLUG2L
END
```
Adjust it to your personal needs and preferences.

## Implementation Notes
If you know bash a little bit, the build script should be mostly self-explanatory.
Anyway, here are some details about it.

* Function `guessProjectName()` is a bit hacky and ad-hoc because it derives
  the project name from the file title of the linker file.
* You may notice that the A41 compiler is never called directly. Instead, the
  function `createDosBuildScript()` runs the L41 linker with option /a. This
  implicitly runs the assembler if the source file is newer than the object
  file. This is not only convenient but has the advantage that redirection
  of standard output works well for L41 but (for some reason) not for A41.
  The build scripts redirects stdout to `build/build.log` and scans it for
  errors.
* Function `runDosScript()` illustrates how to run DOSBox. The interesting
  thing here are the options:
  * `-Wn` creates a new DOSBox instance and causes open to wait until DOSBox
    has exited.
  * `-j` launches DOSBox hidden which is convenient for compilation but for
    obvious reasons not for debugging.
* Functions `createDosBuildScript()` and `createDosTestScript()` create the
  DOS batch files for running the compiler/linker and the emulator inside
  DOSBox. Note that I run the emulator with option `/j` so it uses Jacobs/De
  Arras instructions. That's just a matter of personal taste.
* Function `lowerCaseFileNames()` converts the uppercase DOS file names to
  lower case. Again, a matter of personal taste (and perhaps of easier and
  faster file name completion).

## References
I guess you already know the following web sites:
* [The HP-41 Archive](http://hp41.org/) web site contains _a lot_ of interesting
  information about the HP-41 calculator.
* If you have any questions about HP calculators have a look at the forum on
  [The Museum of HP Calculators](http://hpmuseum.org/). There are many friendly
  HP calculator enthusiasts that are willing to share their knowledge and
  experience.
* [Systemyde 41CL](http://www.systemyde.com/hp41/), the ultimate upgrade of your HP-41 calculator.
