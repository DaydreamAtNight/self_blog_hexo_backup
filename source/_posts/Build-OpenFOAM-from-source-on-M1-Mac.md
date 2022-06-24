---
title: Compile OpenFOAM from source on M1 Mac
author: Ryan LI
toc: true
declare: true
index_img: /index/OpenFOAM.png
tags:
  - OpenFOAM
  - mac
date: 2022-06-09 17:11:13
---

{% note primary %}

I just spent 3 hours building **ESI OpenFOAM-v2112** from source code **locally**(no need docker) on my m1 MacBook air. This blog is to record the process.

{% endnote %}

<!-- more -->

Although here is a very convenient already-built source for mac: [gerlero](https://github.com/gerlero)/**[openfoam2112-app](https://github.com/gerlero/openfoam2112-app)**. I still chose to build it from the source on my own mac. 

{% note secondary %}

Helpful resources:

- [OpenFOAM v2112 source code](https://dl.openfoam.com/source/v2112/OpenFOAM-v2112.tgz)

- [BrushXue](https://github.com/BrushXue)/**[OpenFOAM-AppleM1](https://github.com/BrushXue/OpenFOAM-AppleM1)**

- [OpenFOAM installation on Mac.pdf](https://www.researchgate.net/publication/357395955_OpenFOAM_installation_on_Mac)

- [OpenFOAM wiki](https://develop.openfoam.com/Development/openfoam/-/wikis/building#darwin-mac-os)

- [OpenFOAM doc](https://develop.openfoam.com/Development/openfoam/-/blob/master/doc/Build.md)

{% endnote %}

### Preliminaries

[Command line tools:](https://mac.install.guide/commandlinetools/4.html)

```shell
xcode-select --install
```

[Homebrew](https://brew.sh/):

```shell
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### Procedures

1. Create a case-sensitive volume on mac. OpenFOAM requires a case-sensitive volume to build and run, yet mac does not support it by default. 

   Open `disk utility.app` and follow these settings:

   <img src="Case sensitive volume step one.png" style="zoom:50%;" />

   <img src="Case sensitive volume step two.png" style="zoom:50%;" />

   <img src="Case sensitive volume step three.png"  style="zoom:50%;" />

   <img src="Case sensitive volume finish.png"  style="zoom:50%;" />

2. Go to `/Volumes/OpenFOAMs/`, download and extract the [source code](https://dl.openfoam.com/source/v2112/OpenFOAM-v2112.tgz) and the [patches for mac](https://github.com/BrushXue/OpenFOAM-AppleM1) (thanks to [BrushXue](https://github.com/BrushXue)) to the same directory. 

3. Install dependencies with homebrew:

   ```shell
   brew install cmake open-mpi libomp adios2 boost fftw kahip metis 
   ```

4. Install modifiled `scotch` and `CGAL@4` (Thanks to [gerlero](https://github.com/gerlero) for creating this [tap](https://github.com/gerlero/homebrew-openfoam/tree/main/Formula))

   ```shell
   brew tap gerlero/openfoam
   brew install scotch-no-pthread cgal@4
   ```
   

5. And you probably need to add the following:

   ```shell
   export CPATH=/opt/homebrew/include
   export LIBRARY_PATH=/opt/homebrew/lib
   ```

6. Source OpenFOAM's environment bashrc:

   ```shell
   source etc/bashrc  
   ```

7. Check the system and build (about 1 hour on MacBook Air)

   ```shell
   foamSystemCheck
   ./Allwmake -j -s -l
   ```

8. Install `paraview` from Homebrew

   ```shell
   brew install --cask paraview
   ```

9. Add alias to `$home/.zshrc`

   ``` shell
   alias of="source /Volumes/OpenFOAMs/OpenFOAM-v2112/etc/bashrc"
   ```

   

