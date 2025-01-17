# SoundSynth

This repository contains sound synthesis code used by our papers:

* [Generative Modeling of Audible Shapes for Object Perception](http://sound.csail.mit.edu/papers/gensound_iccv.pdf), ICCV 2017

* [Shape and Material from Sound](http://sound.csail.mit.edu/papers/fastsound_nips.pdf), NIPS 2017

Project page:  [http://sound.csail.mit.edu/](http://sound.csail.mit.edu/)
 
# Auto-Install

Our code has been tested on 64-bit Ubuntu 14.04.5.
 
We provide auto install scripts for all required libraries except Matlab & [MKL](https://software.intel.com/en-us/mkl). You need to modify `auto_install.sh` to indicate the paths to MKL libraries, libiomp5 (often included in an MKL install), and Matlab.
 
Then, simply run `./auto_install.sh` (requires sudo privileges). This should install all the necessary components for you to synthesize sound, including
- [Eigen3](http://eigen.tuxfamily.org/index.php?title=Main_Page)
- [Qt5](http://doc.qt.io/qt-5/qt5-intro.html)
- [Protobuf 2.6.0](https://github.com/google/protobuf)
- [GSL](https://www.gnu.org/software/gsl/doc/html/index.html)

The script relies on [GCC](https://gcc.gnu.org/) and [CMake](https://cmake.org/). We have tested it with GCC 6.3.0 and CMake 3.7.1.
 
If you prefer to install the dependencies manually (e.g., you don't have sudo access), please refer to the [manual installation section at the end of the document](#manual-installation) for instructions on customized installation and usage.

# Demo

After auto installation, run (with Python 3 and [FFmpeg](https://www.ffmpeg.org/))

`python online_synth/gen_sound.py -r 1000 101 4 1 2`

You will find the result in `result/scene-1000/obj-101-1/mat-4-2`. The generated audio should sound like [this](http://sound.csail.mit.edu/data/merged.wav). Here, `-r` indicates rendering audio only (no video). The first number `1000` is the ID of the scene. Each of the following pair describes an object (its ID and material): `101` is for the floor, and `4` indicates it's made of wood; `1` is for the object (plate) and `2` is for its material (steel). You can also add in more objects by specifying different parameters. 

To render the corresponding video, you need to specify the path to [Blender](https://www.blender.org/) in `gen_sound.py`. Then, run the same command but this time without `-r`

`python online_synth/gen_sound.py 1000 101 4 1 2`

It should now render the video as well. The generated video should look like [this](http://sound.csail.mit.edu/data/result.mp4).

To play with more objects & materials, run `get_precompute_data.sh` to download our pre-computed data. All possible object/material combinations are listed in `data/obj_mat_list.txt`. 

# Data
   
[Object data after precomputation (6.3G)](http://sound.csail.mit.edu/data/pre_compute_data.zip): all possible object and material combinations are in `data/obj_mat_list.txt`. Please place the unzipped folder in `data`. Alternatively, `get_precompute_data.sh` will download the pre-compute data and place it in the right place for you.

[Sound-20k (Soundtrack only, 25G)](http://sound.csail.mit.edu/data/sound-20k.tar.gz): the dataset used in our ICCV 2017 paper. Metadata can be found in `data/data_entry.txt`.
 
# Usage
 
The code is structured as two separate parts: **offline pre-computation** and **online synthesis**.
 
The definitions of scenes and materials are located in the `config` folder.
 
### Offline pre-computation
  
The input to offline pre-computation is a tetrahedron mesh. One can convert an ordinary triangular mesh to a tetrahedron mesh using [TetGen](http://wias-berlin.de/software/tetgen/) or [IsoStuffer](https://github.com/cxzheng/ModalSound), as described by [Labelle and Shewchuk 2007](http://www.cs.berkeley.edu/~jrs/papers/stuffing.pdf).
  
#### Running pre-computation
  
Our pre-computation step uses two different solvers for calculating sound propagation: a direct BEM solver and an accelerated BEM FMM solver. In short, the direct method is faster when your mesh has a small number of faces, and the FMM solver is faster and more memory-efficient when the mesh has more faces. We suggest using the direct solver for meshes with less than 1000 faces. 

One can run pre-computation using the direct method via `pre_compute/run_precalc_bem.sh` or using the FMM solver via `pre_compute/run_precalc.sh`. Note that you need to pass an object id followed by a material id to the script. The structure of the data folder should be `data/DATASET_NAME/OBJECT_ID`; all generated files would be in `data/DATASET_NAME/OBJECT_ID/MATERIAL_ID`. For example

`./run_precalc.sh 0 1`

will pre-compute object 0 with material 1 defined in `config/material/material-1.cfg` using the FMM solver.

### Online synthesis
 
The online synthesis part uses [Bullet](https://github.com/bulletphysics/bullet3) for physical simulation and a modified version of [Modal Sound Synthesis](https://github.com/cxzheng/ModalSound) for sound synthesis.

#### Running online simulation
  
The main file is `online_synth/gen_sound.py`.
  
As described above, for video rendering, you need to install [Blender](https://www.blender.org/) and specify its path in `gen_sound.py`. Also [FFmpeg](https://www.ffmpeg.org) is used to combine separate sound tracks or rendered images into a single soundtrack / video.
  
`gen_sound.py` has some options. For example, one can use `-r` to skip rendering and `-v` to skip combining rendered images and sound tracks into a single video. Please refer to the help function for a detailed explanation. 

To use the script, one needs to specify the scene id, and a number of paired object id and material id, as

`gen_sound.py scene-id object1-id object1-material-id object2-id object2-material-id`

# Manual Installation

**Note**: This section is only for users who choose not to use our auto-installation script.

#### Building Modal Sound
  
One can run `modelsound/auto_install.sh` to install all required libraries and build all the necessary binaries.
  
If you wish to manually install the libraries or to customize their locations, please see [this document](https://github.com/ztzhang/SoundSynth/blob/master/documents/building_modalsound.md) for a detailed description. 
  
#### Building file generators
  
1. enter `file_generators`
2. run `./compile_cpps.sh`
  
#### Installing BEM/FMM Solver
  
We solve the Helmholtz equation related to sound propagation using a direct BEM method and an accelerated version using FMM. 
  
1. For the direct BEM method, one needs to install the [Nihu Matlab library](http://last.hit.bme.hu/nihu/index.html) and specify its path in `pre_compute/run_precalc_bem.sh`.
2. For the FMM BEM method, one needs to install the [FMMlib3d libraries](https://cims.nyu.edu/cmcl/fmm3dlib/fmm3dlib.html) and specify its path in `pre_compute/run_precalc.sh`.
  
An auto-install script is provided in `pre_compute/external/auto_install.sh`. Note that you need to specify your Matlab installation path in the first line of `auto_install.sh`. 

 
#### Building Bullet
  
1. enter `Bullet3/build3`
2. run `premake4_linux64 gmake`
3. enter gmake, run `make App_RigidBodyFromObjExampleGui`
4. The built binary is located in `bullet3/bin`.
  
If you need to modify our simulation code, they are located in `bullet3/modified_scripts`.
