---
id: eYw
title: rio-tiler install in windows
pubDate: 2020-12-09T06:19:26.000Z
isDraft: false
tags:
  - python
  - cog
categories:
  - map
---

Previously, using rio-tiler and referring to rio-viz, I made a small application that can provide WTMS services for multiple images simultaneously. Some time ago I needed to deploy it on a Windows server, ran into a few problems, and wrote them down here.

rio-tiler depends on rasterio, rasterio depends on gdal, and installing these on Windows is a bit tricky.

Directly installing rio-tiler with pip will get stuck and throw errors when installing gdal and rasterio.

At first, I found the corresponding versions of GDAL and rasterio on [Unofficial Windows Binaries for Python Extension Packages](https://www.lfd.uci.edu/~gohlke/pythonlibs/#rasterio). Even after installing them, there were still some issues, such as rasterio throwing an error that pyproj’s CRS definitions could not be found when imported.

Initially I used conda’s default channel. In rio-tiler=2.0.0rc3, the dependency rasterio ≥ 1.1.7 is required, but that version wasn’t available in the default channel. After switching to the conda-forge channel, I found the versions were much more complete.

First, I used miniconda3, which is smaller in size and straightforward to install. Then I created and activated a new environment.

```bash
conda create -n rasterio python=3.7
conda activate rasterio
```

```bash
conda search rasterio -c conda-forge
Loading channels: done
# Name                       Version           Build  Channel
rasterio                      0.36.0  py27h43d01bd_1  pkgs/main
rasterio                      0.36.0  py35h49e1f75_1  pkgs/main
rasterio                      0.36.0  py36hb8ea33a_1  pkgs/main
rasterio                      1.0.13  py36h6bd7d87_0  pkgs/main
rasterio                      1.0.13  py37h6bd7d87_0  pkgs/main
rasterio                      1.0.18  py36h6bd7d87_0  pkgs/main
rasterio                      1.0.18  py37h6bd7d87_0  pkgs/main
rasterio                      1.0.21  py36h6bd7d87_0  pkgs/main
rasterio                      1.0.21  py37h6bd7d87_0  pkgs/main
rasterio                      1.0.23  py36hbf02ebe_0  conda-forge
rasterio                      1.0.24  py36h163c445_0  conda-forge
rasterio                      1.0.24  py37h163c445_0  conda-forge
rasterio                      1.0.25  py36h163c445_0  conda-forge
rasterio                      1.0.25  py37h163c445_0  conda-forge
rasterio                      1.0.25  py37he350917_1  conda-forge
rasterio                      1.0.26  py36h163c445_1  conda-forge
rasterio                      1.0.26  py37h163c445_1  conda-forge
rasterio                      1.0.27  py36h163c445_0  conda-forge
rasterio                      1.0.27  py37h163c445_0  conda-forge
rasterio                      1.0.28  py36h163c445_0  conda-forge
rasterio                      1.0.28  py36h163c445_1  conda-forge
rasterio                      1.0.28  py36h2617b1b_2  conda-forge
rasterio                      1.0.28  py37h163c445_0  conda-forge
rasterio                      1.0.28  py37h163c445_1  conda-forge
rasterio                      1.0.28  py37h2617b1b_2  conda-forge
rasterio                       1.1.0  py36h039b02d_0  pkgs/main
rasterio                       1.1.0  py36h2617b1b_0  conda-forge
rasterio                       1.1.0  py37h039b02d_0  pkgs/main
rasterio                       1.1.0  py37h2617b1b_0  conda-forge
rasterio                       1.1.1  py36h163c445_0  conda-forge
rasterio                       1.1.1  py36h2617b1b_0  conda-forge
rasterio                       1.1.1  py37h163c445_0  conda-forge
rasterio                       1.1.1  py37h2617b1b_0  conda-forge
rasterio                       1.1.1  py38h163c445_0  conda-forge
rasterio                       1.1.1  py38h2617b1b_0  conda-forge
rasterio                       1.1.2  py36h163c445_0  conda-forge
rasterio                       1.1.2  py37h163c445_0  conda-forge
rasterio                       1.1.2  py37h2617b1b_0  conda-forge
rasterio                       1.1.2  py38h163c445_0  conda-forge
rasterio                       1.1.3  py36h163c445_0  conda-forge
rasterio                       1.1.3  py36h2617b1b_0  conda-forge
rasterio                       1.1.3  py37h163c445_0  conda-forge
rasterio                       1.1.3  py37h2617b1b_0  conda-forge
rasterio                       1.1.3  py38h163c445_0  conda-forge
rasterio                       1.1.3  py38h2617b1b_0  conda-forge
rasterio                       1.1.4  py36h2409764_0  conda-forge
rasterio                       1.1.4  py36ha22ed69_0  conda-forge
rasterio                       1.1.4  py37h02db82b_0  conda-forge
rasterio                       1.1.4  py37h91b820b_0  conda-forge
rasterio                       1.1.4  py38h151dc71_0  conda-forge
rasterio                       1.1.4  py38hef609b1_0  conda-forge
rasterio                       1.1.5  py36h2409764_0  conda-forge
rasterio                       1.1.5  py36ha22ed69_0  conda-forge
rasterio                       1.1.5  py36ha22ed69_1  conda-forge
rasterio                       1.1.5  py37h02db82b_0  conda-forge
rasterio                       1.1.5  py37h02db82b_1  conda-forge
rasterio                       1.1.5  py37h91b820b_0  conda-forge
rasterio                       1.1.5  py38h151dc71_0  conda-forge
rasterio                       1.1.5  py38h151dc71_1  conda-forge
rasterio                       1.1.5  py38hef609b1_0  conda-forge
rasterio                       1.1.6  py36hc1acebe_0  conda-forge
rasterio                       1.1.6  py36hc1acebe_1  conda-forge
rasterio                       1.1.6  py37hce843d0_0  conda-forge
rasterio                       1.1.6  py37hce843d0_1  conda-forge
rasterio                       1.1.6  py38hf2e4ed7_0  conda-forge
rasterio                       1.1.6  py38hf2e4ed7_1  conda-forge
rasterio                       1.1.7  py36hc1acebe_0  conda-forge
rasterio                       1.1.7  py37hce843d0_0  conda-forge
rasterio                       1.1.7  py37hce843d0_1  conda-forge
rasterio                       1.1.7  py38hf2e4ed7_0  conda-forge
rasterio                       1.1.7  py38hf2e4ed7_1  conda-forge
rasterio                       1.1.7  py39h11aa1b2_1  conda-forge
rasterio                       1.1.8  py37hc4b0cd6_0  conda-forge
rasterio                       1.1.8  py38h5653988_0  conda-forge
rasterio                       1.1.8  py39hfec4536_0  conda-forge
```

Then I directly specified the version to install:

```bash
conda install rasterio=1.1.8
```

Using conda to install rasterio conveniently also solved the gdal problem, and I didn’t encounter any other issues during installation.

At this point, in most cases you might run into an issue where rio-color fails to install, because it has a C++ dependency and requires:

> error: Microsoft Visual C++ 14.0 is required. Get it with "Microsoft Visual C++ Build Tools": [http://landinghub.visualstudio.com/visual-cpp-build-tools](http://landinghub.visualstudio.com/visual-cpp-build-tools)

In this case, just follow the prompt and use visual-cpp-build-tools to install the relevant dependencies (check the following two items):

- MSVC v142 – VS 2019 C++ x64/x86 build tools  
- Windows 10 SDK (10.0.18362.0)

After installing these dependencies, I ran into another problem: roughly that the executable "rc.exe" could not be found. After some searching, I found that “rc.exe” existed in the following three directories. I chose to add the x64 folder (since the system is x64) to the PATH environment variable, and that fixed the issue.

```bash
C:\Program Files (x86)\Windows Kits\10\bin\10.0.18362.0\arm64\
C:\Program Files (x86)\Windows Kits\10\bin\10.0.18362.0\x64\
C:\Program Files (x86)\Windows Kits\10\bin\10.0.18362.0\x86\
```

At this point, barring any surprises, you should be able to install rio-tiler with the following command:

```bash
pip install rio-tiler --pre
```