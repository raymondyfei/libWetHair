libWetHair
================
libWetHair is an open source project for the physical simulation of liquid and wet hairs. It has been compiled and tested on Mac OS X (with both Intel and Apple M1 chips), Ubuntu Linux, Windows, and licensed under the Mozilla Public License v. 2.0.

We would like to hear from you if you appreciate this work.

It is the original implementation of paper A Multi-Scale Model for Simulating Liquid-Hair Interactions (refer to our [project page](http://www.cs.columbia.edu/cg/liquidhair/) for more details). This code base contains the following parts:

 - A liquid simulator implementing the affine-particle-in-cell method.
 - A hair simulator implementing the elastic rods model.
 - A reduced-liquid simulator for the simulation of flow on hairs.
 - Cohesion effects between the hairs
 - Coupling between the hairs and liquid, including dragging, capturing and dripping effect.

Paper Errata
--------------------
The following corrections apply to Section 3.1 of *A Multi-Scale Model for Simulating Liquid-Hair Interactions*. We thank [Sofia Iannicelli](https://www.sofiaiannicelli.org/) for identifying these issues and independently confirming the corrected energy derivative.

**Interface lengths and surface energy (Eqs. (1)--(4)).** The solid interface lengths $l_S$, $l_{S1}$, and $l_{S2}$ should refer to the **hair-liquid (solid-liquid)** boundary, rather than the hair-air boundary. With this convention, Young's equation gives the cross-sectional surface energy per unit hair length as

$$
E = \sigma \left(l_{\mathrm{liquid-air}} - \cos(\theta) l_{\mathrm{solid-liquid}}\right) + \mathrm{const}.
$$

The additive constant is independent of the hair separation. Accordingly, the solid-interface terms in Eqs. (1) and (2) must have **minus** signs. With $l_A$ denoting the total liquid-air interface length, the corrected equations are

$$
dE_s = \sigma \left[l_A(s) - \cos(\theta) l_S(s)\right] ds,
$$

$$
dE_s = \sigma \left[l_A(s) - \cos(\theta_1) l_{S1}(s) - \cos(\theta_2) l_{S2}(s)\right] ds.
$$

Equation (3), as printed, gives the length of only one meniscus. To include both the upper and lower menisci, the total liquid-air interface length is

$$
l_A(s) = 2R\left[\pi - (\theta_1 + \alpha_1 + \theta_2 + \alpha_2)\right].
$$

Equivalently, if $l_A$ retains the single-meniscus definition printed in Eq. (3), its contribution to the energy must be $2l_A$. The solid-liquid interface length replacing Eq. (4) is

$$
l_{Si}(s) = 2r_i\alpha_i,\qquad i=1,2,
$$

without the printed $\pi-\alpha_i$. For identical hairs, with $r_1=r_2=r$, $\alpha_1=\alpha_2=\alpha$, and $\theta_1=\theta_2=\theta$, these corrections give

$$
E = \sigma\left[2R(\pi-2\theta-2\alpha) - 4r\alpha\cos(\theta)\right] + \mathrm{const}.
$$

Differentiating this energy with respect to separation at fixed liquid cross-sectional area $A_L$, as in Eqs. (7)--(8), agrees with `CohesionTable::computedEdd` in [libWetHair/CohesionTableGen.cpp](libWetHair/CohesionTableGen.cpp). The implemented hair-hair energy derivative already incorporates these corrections.

**Distance convention (Eq. (5) and Fig. 5).** Equation (5) as printed gives the surface-to-surface gap, whereas Fig. 5 and the remainder of the paper use the center-to-center distance. The equation for the center-to-center distance is

$$
d = R\sum_{i=1,2}\cos(\theta_i+\alpha_i) + \sum_{i=1,2}r_i\cos\alpha_i.
$$

This is the printed Eq. (5) plus $r_1+r_2$. For fixed hair radii, the two distance conventions differ only by a constant, so the energy derivative is unchanged. The implementation uses the center-to-center convention: `CohesionTable::computeR` rearranges $d=2R\cos(\theta+\alpha)+2r\cos\alpha$ to solve for $R$.

Dependencies
--------------------
libWetHair has several dependencies and by default it will fetch them through CMake's `FetchContent`. Please make sure your internet is smooth before the first time of configuration.

Alternatively, if preferring a package manager, one can turn on the switch `LIBWETHAIR_FIND_DEPENDENCIES`, which will have cmake to find those dependencies through a package manager.

Compilation
-----------------
libWetHair has been tested with AppleClang (under Mac OS X), GCC 4.8+ (under Linux), and Microsoft Visual Studio (under Windows 10).

To compile libWetHair, you'll need CMake or CMake-GUI (https://cmake.org).

For CMake:
1. make a directory, say, *build*, with *mkdir build*, enter the *build* directory, type *cmake ..*
2. Optionally, you can adjust the options with *ccmake .*
3. Optionally, you may use cmake to generate project files corresponding to different IDEs. For example, on Windows you may use `cmake -G "Visual Studio 17 2022" -A x64` to generate a project for Visual Studio 2022 using x64 architecture.
4. type *make* to compile the code. For speeding up the compilation process you may use *make -j*.

Similar (or more convenient) configuration can be done in CMake-GUI.

In addition, to configure for the demo on Windows, you'll need to have `fxc.exe` contained in one of the directories specified through your system's `PATH` environment variable. This executable is usually under your installation directory for Windows SDK (e.g., for Windows 10, it's in a directory named with a specific SDK version under `C:\Program Files (x86)\Windows Kits\10\bin`).

Run the Demo
--------------------
To run the demo of libWetHair, you may simply use the command line argument *-s [scene_file]* to specify the scene to be loaded. For example, you may type

./libWetHair -s assets/unit_tests/cylinder_2.xml

to run the simulation of the scene containing two hairs being cohesive to each other. 

All the parameters can be modified offline in the scene description XML files. Some can be changed online in the user interface provided by the demo program.

USAGE: 

   ./libWetHair  [-j <string>] [-o <integer>] [-d <boolean>] [-g <integer>] [-p <boolean>] [-s <string>] [--] [--version] [-h]

Where: 

   -j <string>,  --initfile <string>
     Binary file to load simulation state from for initialization

   -o <integer>,  --readableoutput <integer>
     readable file to save simulation state to; after getting the output here, you may do surface reconstruction and rendering with the [Houdini .HIP file](http://www.cs.columbia.edu/cg/liquidhair/pseudo_dog.hipnc) by renaming the HIP file into the name of the folder storing all output files (please check the python scripts in the HIP file for details).

   -d <boolean>,  --display <boolean>
     Run the simulation with display enabled if 1, without if 0

   -g <integer>,  --generate <integer>
     Generate PNG if 1, not if 0

   -p <boolean>,  --paused <boolean>
     Begin the simulation paused if 1, running if 0

   -s <string>,  --scene <string>
     Simulation to run; an xml scene file

   --,  --ignore_rest
     Ignores the rest of the labeled arguments following this flag.

   --version
     Displays version information and exits.

   -h,  --help
     Displays usage information and exits.

Contact
-----------
Please contact the author (yf2320@columbia.edu) for questions and bug report, or consulting for the usage of this code base.

BibTeX Citation
----------------------

```bibtex
@article{Fei:2017:liquidhair,
  title={A Multi-Scale Model for Simulating Liquid-Hair Interactions},
  author={Fei, Yun (Raymond) and Maia, Henrique Teles and Batty, Christopher and Zheng, Changxi and Grinspun, Eitan},
  journal={ACM Trans. Graph.},
  volume={36},
  number={4},
  year={2017},
  doi={10.1145/3072959.3073630},
}
```
