# VoxelMap - The CUDA Voxel Mapping Library

VoxelMap is a high-performance occupancy mapping library for processing large-scale depth measurements in real-time, using CUDA-enabled GPUs.

# Table of Contents  
* [Features](#Features)
    * [Occupancy grid mapping](#Occupancy-grid-mapping)  
    * [Serialization](#Serialization)
    * [Visualization](#Visualization)
* [Requirements](#Requirements)
    * [Build dependencies](#Build-dependencies)
    * [Run-time dependencies](#Run-time-dependencies)
* [Benchmarks](#Benchmarks)
* [Q&A](#Q&A)

# Features

## Occupancy grid mapping
VoxelMap maintains an efficient occupancy grid, where each voxel is occupied with a certain probability. The map is stored as a concurrent tree-based data structure for efficient insertion of scans in parallell. The input interface is flexible and simply accepts a batch of rays with a hit at some distance or a not-hit.

There is no limit on the map extent in terms of coordinate size/range of travel, as unneccesary regions will be purged to bound the memory usage. The maximum amount of memory usage is a fixed and pre-allocated user-defined limit. This makes “out-of-memory” problems impossible. The available map extent for a given memory allocation is determined by several factors, like voxel size. The map is internally stored in GPU-memory, but the interface expects data in CPU-memory.

## Serialization
Copying a large-scale map like this is prohibitively expensive. VoxelMap avoids this by facilitating loading/storing of the changed parts of the map only, so there is no redundant transfer of information. The changed parts of the map can also be compressed, converting the probability in each voxel to a binary flag, to further save bandwith. This happens entirely on the GPU, which avoids large transfers from GPU to CPU-memory. The compressed changes can then be efficiently serialized to an output stream to be safely shared/stored across platforms, e.g for visualization purposes.

## Visualization
VoxelMap includes a small OpenGL-based rendering library using custom GLSL sharders for efficient visualisation of a large amount of voxels. The input to this visualization is typically the compressed output from the serialization. This can be used as a standalone visualization program and/or integrated into custom visualization tools.

# Requirements

## Build dependencies

Libraries required at compile-time:
* Eigen3
* CUDA

## Run-time dependencies

Libraries required at run-time:
* libcudart.so

# Benchmarks

## Synthetic data

![simple_benchmark](gifs/simple_benchmark.gif)

To get a rough estimate of the performance of VoxelMap, the average time to incorporate a new scan is measured. The scan is created by setting each distance measurement equal some assumed average trace length. Each scan is identical, but the sensor origin is shifted 1m along the x-axis for each insert operation. This is repeated 1000 times, or until 10 minutes has passed, and the average update time is computed. This procedure will by no means yield realitic maps, but it will give an indication of runtime cost for a given sensor setup. Update times for Octomap, with the exact same input, is included for comparison. Colored voxels represent occupied space, and is here shown as a dense trail of occupied space as the LIDAR moves along.

The benchmark is run on NVIDIA's Jetson AGX Xavier, with a voxel size of 0.5m. The tables below show the average update time, given an assumed average trace length. To give a worst-case estimate, each hit/miss is traced individually and registers a potential hit even if multiple rays hit the same voxel. It is however possible to merge hits corresponding to the same voxel, to reduce trace times and/or reduce double counting.


### Sensor A
* 64 channels, 45&deg; VFOV
* 1024 sectors, 360&deg; HFOV

Library | 40m average trace | 60m average trace | 80m average trace
--- | --- | --- | ---
VoxelMap | 7.8 ms | 13.6 ms | 19.8 ms
OctoMap | 2.0 s | 6.7 s | 13.8 s

### Sensor B
* 128 channels, 45&deg; VFOV
* 2048 sectors, 360&deg; HFOV

Library | 40m average trace | 60m average trace | 80m average trace
--- | --- | --- | ---
VoxelMap | 27.2 ms | 44.6 ms | 65.0 ms
OctoMap | 3.2 s | 9.0 s | 21.5 s



# Q&A
`- Is it possible to take into account pose evolution during the scan, provided that every scan column is timestamped?`

Yes, it is possible. The input to the update operation is simply a batch of rays, so it would be possible in a number of ways without affecting the performance of the update operation.

`- Do you discriminate unknown space from free/occupied?`

No. The default assumption is free space, so unknown equals completely free.

`- Can the map be cropped for a given radius? Say remove all voxels that are > 200m from the drone? Or voxels that have not been updated for 5 min?`

Yes, it's possible. This would come at virtually no runtime cost, and could be done after each insert operation.

`- How difficult it is to incorporate data from additional range sensors (with lower range and FoV)?`

This is supported by design, without affecting performance any more than additional rays from the original sensor would.
